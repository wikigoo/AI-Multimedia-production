
در ادامه اسکریپت‌های خودکار برای نصب AnythingLLM در مسیر `D:\Ai\AnythingLLM` ارائه می‌دهم:

## فایل 1: install-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       نصب خودکار AnythingLLM
echo ========================================
echo.

:: بررسی وجود Docker
echo بررسی نصب Docker...
docker --version >nul 2>&1
if %errorlevel% neq 0 (
    echo خطا: Docker نصب نشده است!
    echo لطفاً ابتدا Docker Desktop را نصب کنید.
    pause
    exit /b 1
)

:: بررسی وجود Docker Compose
echo بررسی نصب Docker Compose...
docker-compose --version >nul 2>&1
if %errorlevel% neq 0 (
    echo خطا: Docker Compose نصب نشده است!
    pause
    exit /b 1
)

:: ایجاد مسیر پروژه
echo ایجاد مسیر پروژه...
if not exist "D:\Ai" mkdir "D:\Ai"
if not exist "D:\Ai\AnythingLLM" mkdir "D:\Ai\AnythingLLM"
cd /d "D:\Ai\AnythingLLM"

:: ایجاد پوشه داده‌ها
echo ایجاد پوشه‌های مورد نیاز...
if not exist "anythingllm-data" mkdir "anythingllm-data"
if not exist "anythingllm-data\documents" mkdir "anythingllm-data\documents"
if not exist "anythingllm-data\vector-cache" mkdir "anythingllm-data\vector-cache"
if not exist "anythingllm-data\logs" mkdir "anythingllm-data\logs"

:: ایجاد فایل docker-compose.yml
echo ایجاد فایل docker-compose.yml...
(
echo version: '3.8'
echo.
echo services:
echo   anythingllm:
echo     image: mintplexlabs/anythingllm:latest
echo     container_name: anythingllm
echo     ports:
echo       - "3001:3001"
echo     volumes:
echo       - ./anythingllm-data:/app/server/storage
echo       - ./anythingllm-data/.env:/app/server/.env
echo     environment:
echo       - STORAGE_DIR=/app/server/storage
echo       - NODE_ENV=production
echo       - HOST=0.0.0.0
echo       - SERVER_PORT=3001
echo     restart: unless-stopped
echo     networks:
echo       - anythingllm-network
echo.
echo networks:
echo   anythingllm-network:
echo     driver: bridge
) > docker-compose.yml

:: ایجاد فایل .env
echo ایجاد فایل تنظیمات...
(
echo # تنظیمات پایه AnythingLLM
echo NODE_ENV=production
echo SERVER_PORT=3001
echo HOST=0.0.0.0
echo.
echo # تنظیمات ذخیره‌سازی
echo STORAGE_DIR=/app/server/storage
echo.
echo # تنظیمات امنیتی
echo JWT_SECRET=%RANDOM%%RANDOM%%RANDOM%
echo.
echo # تنظیمات شبکه
echo CORS_ORIGINS=*
echo.
echo # تنظیمات لاگ
echo LOG_LEVEL=info
) > anythingllm-data\.env

:: دانلود و اجرای کانتینر
echo دانلود و راه‌اندازی AnythingLLM...
docker-compose pull
docker-compose up -d

:: بررسی وضعیت
timeout /t 10 /nobreak >nul
echo.
echo بررسی وضعیت سرویس...
docker-compose ps

:: نمایش اطلاعات دسترسی
echo.
echo ========================================
echo         نصب با موفقیت انجام شد!
echo ========================================
echo.
echo آدرس دسترسی:
echo   محلی: http://localhost:3001
echo   شبکه محلی: http://%COMPUTERNAME%:3001
echo.
for /f "tokens=2 delims=:" %%a in ('ipconfig ^| findstr /c:"IPv4"') do (
    for /f "tokens=1" %%b in ("%%a") do echo   IP: http://%%b:3001
)
echo.
echo مسیر نصب: D:\Ai\AnythingLLM
echo.
echo دستورات مفید:
echo   مشاهده لاگ: docker-compose logs -f
echo   توقف سرویس: docker-compose down
echo   راه‌اندازی مجدد: docker-compose restart
echo.
pause
```

## فایل 2: start-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo      راه‌اندازی AnythingLLM
echo ========================================

cd /d "D:\Ai\AnythingLLM"

if not exist "docker-compose.yml" (
    echo خطا: فایل docker-compose.yml یافت نشد!
    echo لطفاً ابتدا اسکریپت نصب را اجرا کنید.
    pause
    exit /b 1
)

echo راه‌اندازی سرویس...
docker-compose up -d

timeout /t 5 /nobreak >nul
docker-compose ps

echo.
echo AnythingLLM در حال اجرا است:
echo http://localhost:3001
echo.
pause
```

## فایل 3: stop-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       توقف AnythingLLM
echo ========================================

cd /d "D:\Ai\AnythingLLM"

if not exist "docker-compose.yml" (
    echo خطا: فایل docker-compose.yml یافت نشد!
    pause
    exit /b 1
)

echo توقف سرویس...
docker-compose down

echo.
echo AnythingLLM متوقف شد.
echo.
pause
```

## فایل 4: update-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo     به‌روزرسانی AnythingLLM
echo ========================================

cd /d "D:\Ai\AnythingLLM"

if not exist "docker-compose.yml" (
    echo خطا: فایل docker-compose.yml یافت نشد!
    pause
    exit /b 1
)

echo توقف سرویس فعلی...
docker-compose down

echo دانلود آخرین نسخه...
docker-compose pull

echo راه‌اندازی مجدد...
docker-compose up -d

timeout /t 5 /nobreak >nul
docker-compose ps

echo.
echo به‌روزرسانی انجام شد!
echo http://localhost:3001
echo.
pause
```

## فایل 5: logs-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       مشاهده لاگ‌های AnythingLLM
echo ========================================

cd /d "D:\Ai\AnythingLLM"

if not exist "docker-compose.yml" (
    echo خطا: فایل docker-compose.yml یافت نشد!
    pause
    exit /b 1
)

echo مشاهده لاگ‌ها... (برای خروج Ctrl+C بزنید)
docker-compose logs -f anythingllm
```

## فایل 6: backup-anythingllm.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       پشتیبان‌گیری AnythingLLM
echo ========================================

cd /d "D:\Ai\AnythingLLM"

set backup_date=%date:~6,4%-%date:~3,2%-%date:~0,2%
set backup_time=%time:~0,2%-%time:~3,2%-%time:~6,2%
set backup_name=anythingllm-backup-%backup_date%_%backup_time%

echo ایجاد پشتیبان: %backup_name%

if not exist "backups" mkdir "backups"

echo فشرده‌سازی داده‌ها...
powershell -Command "Compress-Archive -Path 'anythingllm-data\*' -DestinationPath 'backups\%backup_name%.zip'"

echo.
echo پشتیبان‌گیری انجام شد: backups\%backup_name%.zip
echo.
pause
```

## فایل 7: PowerShell Script (install-anythingllm.ps1)

```powershell
# نصب خودکار AnythingLLM با PowerShell
Write-Host "========================================" -ForegroundColor Green
Write-Host "       نصب خودکار AnythingLLM" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green

# بررسی Docker
try {
    docker --version | Out-Null
    Write-Host "✓ Docker یافت شد" -ForegroundColor Green
} catch {
    Write-Host "✗ Docker نصب نشده است!" -ForegroundColor Red
    Write-Host "لطفاً ابتدا Docker Desktop را نصب کنید." -ForegroundColor Yellow
    Read-Host "برای ادامه Enter بزنید"
    exit
}

# ایجاد مسیر
$projectPath = "D:\Ai\AnythingLLM"
if (!(Test-Path $projectPath)) {
    New-Item -ItemType Directory -Path $projectPath -Force
    Write-Host "✓ مسیر پروژه ایجاد شد" -ForegroundColor Green
}

Set-Location $projectPath

# ایجاد پوشه داده‌ها
$dataPath = "$projectPath\anythingllm-data"
if (!(Test-Path $dataPath)) {
    New-Item -ItemType Directory -Path $dataPath -Force
    Write-Host "✓ پوشه داده‌ها ایجاد شد" -ForegroundColor Green
}

# ایجاد docker-compose.yml
$dockerComposeContent = @"
version: '3.8'

services:
  anythingllm:
    image: mintplexlabs/anythingllm:latest
    container_name: anythingllm
    ports:
      - "3001:3001"
    volumes:
      - ./anythingllm-data:/app/server/storage
      - ./anythingllm-data/.env:/app/server/.env
    environment:
      - STORAGE_DIR=/app/server/storage
      - NODE_ENV=production
      - HOST=0.0.0.0
      - SERVER_PORT=3001
    restart: unless-stopped
    networks:
      - anythingllm-network

networks:
  anythingllm-network:
    driver: bridge
"@

$dockerComposeContent | Out-File -FilePath "docker-compose.yml" -Encoding UTF8
Write-Host "✓ فایل docker-compose.yml ایجاد شد" -ForegroundColor Green

# ایجاد فایل .env
$envContent = @"
# تنظیمات پایه AnythingLLM
NODE_ENV=production
SERVER_PORT=3001
HOST=0.0.0.0

# تنظیمات ذخیره‌سازی
STORAGE_DIR=/app/server/storage

# تنظیمات امنیتی
JWT_SECRET=$(Get-Random)$(Get-Random)$(Get-Random)

# تنظیمات شبکه
CORS_ORIGINS=*

# تنظیمات لاگ
LOG_LEVEL=info
"@

$envContent | Out-File -FilePath "$dataPath\.env" -Encoding UTF8
Write-Host "✓ فایل تنظیمات ایجاد شد" -ForegroundColor Green

# دانلود و اجرا
Write-Host "دانلود و راه‌اندازی AnythingLLM..." -ForegroundColor Yellow
docker-compose pull
docker-compose up -d

Start-Sleep -Seconds 10

# نمایش وضعیت
Write-Host "`n========================================" -ForegroundColor Green
Write-Host "         نصب با موفقیت انجام شد!" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green

Write-Host "`nآدرس دسترسی:" -ForegroundColor Cyan
Write-Host "  محلی: http://localhost:3001" -ForegroundColor White
Write-Host "  شبکه محلی: http://$env:COMPUTERNAME:3001" -ForegroundColor White

$ip = (Get-NetIPAddress -AddressFamily IPv4 | Where-Object {$_.IPAddress -like "192.168.*"}).IPAddress
if ($ip) {
    Write-Host "  IP: http://$ip:3001" -ForegroundColor White
}

Write-Host "`nمسیر نصب: $projectPath" -ForegroundColor Yellow

Read-Host "`nبرای ادامه Enter بزنید"
```

## نحوه استفاده:

1. **فایل‌ها را در مسیر `D:\Ai\AnythingLLM` ذخیره کنید**
2. **برای نصب**: `install-anythingllm.bat` را به عنوان Administrator اجرا کنید
3. **برای راه‌اندازی**: `start-anythingllm.bat`
4. **برای توقف**: `stop-anythingllm.bat`
5. **برای به‌روزرسانی**: `update-anythingllm.bat`

این اسکریپت‌ها تمام فرآیند نصب و مدیریت AnythingLLM را خودکار می‌کنند و امکان دسترسی از شبکه محلی را فراهم می‌آورند.