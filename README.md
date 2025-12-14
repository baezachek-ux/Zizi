<#
.SYNOPSIS
Скрипт для автоматической установки популярных приложений на Windows с помощью Winget.
#>

$RequiredApps = @(
    "Microsoft.PowerShell"  # Если требуется последняя версия PowerShell Core
    "Google.Chrome"         # Веб-браузер
    "Mozilla.Firefox"       # Веб-браузер
    "Microsoft.Teams"       # Корпоративный мессенджер
    "Discord.Discord"       # Мессенджер
    "VSCode"                # Редактор кода (Visual Studio Code)
    "VLC"                   # Медиаплеер
    "7zip.7zip"             # Архиватор
    "Spotify.Spotify"       # Музыкальный стриминг
)

Write-Host "--- Запуск пакетной установки приложений через Winget ---" -ForegroundColor Cyan

foreach ($AppId in $RequiredApps) {
    Write-Host "Установка приложения: $AppId..." -ForegroundColor Yellow
    
    # Команда Winget для установки. -h означает "silent" (тихая установка)
    try {
        winget install --id "$AppId" --accept-package-agreements --accept-source-agreements -h
        
        # Проверка кода выхода
        if ($LASTEXITCODE -eq 0) {
            Write-Host "✅ Успешно установлено: $AppId" -ForegroundColor Green
        } else {
            Write-Host "❌ Ошибка установки $AppId. Код выхода: $LASTEXITCODE" -ForegroundColor Red
        }
    }
    catch {
        Write-Host "❌ Критическая ошибка при выполнении Winget для $AppId: $($_.Exception.Message)" -ForegroundColor Red
    }
    
    Write-Host ""
}

Write-Host "--- Пакетная установка завершена ---" -ForegroundColor Cyan
