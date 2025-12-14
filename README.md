import requests
import os
import threading
from concurrent.futures import ThreadPoolExecutor

# --- КОНФИГУРАЦИЯ ---
THREAD_COUNT = 4  # Количество потоков (сегментов). Обычно 4-8 достаточно.
CHUNK_SIZE = 1024 * 1024 * 5 # Размер куска для записи (5 MB)
# ---------------------

class SegmentedDownloader:
    def __init__(self, url, filename, num_threads=THREAD_COUNT):
        self.url = url
        self.filename = filename
        self.num_threads = num_threads
        self.file_size = 0
        self.segments = []
        self.temp_files = []

    def _get_file_size(self):
        """Получает размер файла и проверяет поддержку Range-запросов."""
        try:
            with requests.head(self.url, allow_redirects=True) as r:
                r.raise_for_status()
                
                # Проверка поддержки частичного контента
                if 'bytes' not in r.headers.get('Accept-Ranges', ''):
                    print("⚠️ Сервер не поддерживает Range-запросы. Скачивание будет однопоточным.")
                    return False

                self.file_size = int(r.headers.get('Content-Length', 0))
                if self.file_size == 0:
                    raise Exception("Не удалось получить размер файла (Content-Length: 0).")
                
                print(f"Общий размер файла: {self.file_size / (1024*1024):.2f} MB")
                return True
                
        except requests.exceptions.RequestException as e:
            print(f"❌ Ошибка при проверке URL: {e}")
            return False

    def _download_segment(self, start_byte, end_byte, segment_index):
        """Скачивает один сегмент файла в отдельном потоке."""
        
        headers = {'Range': f'bytes={start_byte}-{end_byte}'}
        temp_filename = f"{self.filename}.part{segment_index}"
        self.temp_files.append(temp_filename)
        
        try:
            print(f"-> Поток {segment_index}: Скачивание диапазона {start_byte}-{end_byte}")
            
            with requests.get(self.url, headers=headers, stream=True) as r:
                r.raise_for_status()
                
                with open(temp_filename, 'wb') as f:
                    for chunk in r.iter_content(chunk_size=CHUNK_SIZE):
                        if chunk:
                            f.write(chunk)
            
            print(f"<- Поток {segment_index}: Завершено.")
            return True
            
        except requests.exceptions.RequestException as e:
            print(f"❌ Ошибка в потоке {segment_index}: {e}")
            return False

    def _combine_files(self):
