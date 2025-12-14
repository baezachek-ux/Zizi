import pandas as pd
from datetime import datetime
import os

LOG_FILE = 'sales_call_log.csv'

def setup_log_file():
    """Создает файл журнала звонков, если он не существует."""
    if not os.path.exists(LOG_FILE):
        df = pd.DataFrame(columns=[
            'timestamp', 'lead_name', 'phone', 'duration_seconds', 
            'outcome', 'next_step', 'operator_notes'
        ])
        df.to_csv(LOG_FILE, index=False)
        print(f"Создан новый файл журнала продаж: {LOG_FILE}")
    else:
        print(f"Файл журнала {LOG_FILE} уже существует.")

def log_call_result(lead_data, duration, outcome, next_step, notes):
    """
    Записывает результат одного звонка в CSV-файл.
    
    :param lead_data: Словарь с данными лида (name, phone)
    :param duration: Длительность звонка в секундах
    :param outcome: Результат (Назначена встреча, Недоступен, Отказ, Продажа)
    :param next_step: Следующее действие (Перезвонить завтра, Отправить email)
    :param notes: Заметки оператора
    """
    
    try:
        new_entry = {
            'timestamp': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
            'lead_name': lead_data['name'],
            'phone': lead_data['phone'],
            'duration_seconds': duration,
            'outcome': outcome,
            'next_step': next_step,
            'operator_notes': notes
        }
        
        # Загрузка существующего журнала и добавление новой записи
        df = pd.read_csv(LOG_FILE)
        df_new_row = pd.DataFrame([new_entry])
        df = pd.concat([df, df_new_row], ignore_index=True)
        
        df.to_csv(LOG_FILE, index=False)
        print(f"✅ Результат звонка для {lead_data['name']} успешно залогирован.")

    except Exception as e:
        print(f"❌ Ошибка при логировании данных: {e}")

# --- Использование ---
if __name__ == "__main__":
    setup_log_file()
    
    # Имитация работы оператора с первым лидом
    first_lead = LEAD_DATABASE[0]
    log_call_result(
        lead_data=first_lead,
        duration=125,
        outcome="Назначена встреча",
        next_step="Отправить презентацию по почте",
        notes="Заинтересован в продукте Premium. Перезвонить через 3 дня."
    )
    
    # Имитация работы оператора со вторым лидом
    second_lead = LEAD_DATABASE[1]
    log_call_result(
        lead_data=second_lead,
        duration=5,
        outcome="Недоступен",
        next_step="Повторный набор через 2 часа",
        notes="Автоответчик."
    )
