
## فایل 1: install-comfyui.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       نصب خودکار ComfyUI
echo ========================================
echo.

:: بررسی VRAM
echo بررسی VRAM کارت گرافیک...
wmic path win32_VideoController get AdapterRAM,Name /format:list | findstr /r /v "^$" | findstr "AdapterRAM\|Name"
echo.
echo توصیه: حداقل 4GB VRAM و بهتر 8GB
echo.
pause

:: بررسی وجود Git
echo بررسی نصب Git...
git --version >nul 2>&1
if %errorlevel% neq 0 (
    echo خطا: Git نصب نشده است!
    echo لطفاً ابتدا Git را از git-scm.com دانلود و نصب کنید.
    echo آدرس: https://git-scm.com/download/win
    pause
    exit /b 1
)

:: ایجاد مسیر پروژه
echo ایجاد مسیر پروژه...
if not exist "D:\Ai" mkdir "D:\Ai"
if not exist "D:\Ai\ComfyUI" mkdir "D:\Ai\ComfyUI"
cd /d "D:\Ai\ComfyUI"

:: دانلود نسخه portable ComfyUI
echo دانلود ComfyUI Portable...
echo این فرآیند ممکن است چند دقیقه طول بکشد (فایل حجیم است)...
echo.

:: بررسی نوع GPU
echo بررسی نوع کارت گرافیک...
nvidia-smi >nul 2>&1
if %errorlevel% equ 0 (
    echo ✓ GPU NVIDIA شناسایی شد
    set GPU_TYPE=nvidia
) else (
    echo AMD/Intel GPU یا CPU mode
    set GPU_TYPE=cpu
)

:: دانلود ComfyUI بر اساس نوع GPU
if "%GPU_TYPE%"=="nvidia" (
    echo دانلود نسخه NVIDIA...
    curl -L -o comfyui_portable.7z "https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_nvidia_cu118_or_cpu.7z"
) else (
    echo دانلود نسخه CPU...
    curl -L -o comfyui_portable.7z "https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_cpu.7z"
)

:: بررسی موفقیت دانلود
if not exist "comfyui_portable.7z" (
    echo خطا در دانلود! لطفاً اتصال اینترنت را بررسی کنید.
    pause
    exit /b 1
)

:: استخراج فایل (نیاز به 7zip)
echo استخراج فایل...
echo بررسی نصب 7-Zip...
"C:\Program Files\7-Zip\7z.exe" >nul 2>&1
if %errorlevel% equ 0 (
    "C:\Program Files\7-Zip\7z.exe" x comfyui_portable.7z -y
) else (
    "C:\Program Files (x86)\7-Zip\7z.exe" >nul 2>&1
    if %errorlevel% equ 0 (
        "C:\Program Files (x86)\7-Zip\7z.exe" x comfyui_portable.7z -y
    ) else (
        echo خطا: 7-Zip نصب نشده است!
        echo لطفاً 7-Zip را از https://www.7-zip.org/ دانلود و نصب کنید.
        echo سپس این اسکریپت را مجدداً اجرا کنید.
        pause
        exit /b 1
    )
)

:: حذف فایل فشرده
del comfyui_portable.7z

:: نصب ComfyUI Manager
echo نصب ComfyUI Manager...
cd ComfyUI\custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
cd ..\..\

:: ایجاد پوشه‌های مدل
echo ایجاد پوشه‌های مورد نیاز...
if not exist "ComfyUI\models\checkpoints" mkdir "ComfyUI\models\checkpoints"
if not exist "ComfyUI\models\upscale_models" mkdir "ComfyUI\models\upscale_models"
if not exist "ComfyUI\models\vae" mkdir "ComfyUI\models\vae"
if not exist "ComfyUI\models\loras" mkdir "ComfyUI\models\loras"
if not exist "ComfyUI\models\controlnet" mkdir "ComfyUI\models\controlnet"
if not exist "ComfyUI\user\workflows" mkdir "ComfyUI\user\workflows"

:: ایجاد اسکریپت تنظیم CUDA برای GPUهای قدیمی
echo ایجاد اسکریپت تنظیمات...
(
echo @echo off
echo echo تنظیم ComfyUI برای GPUهای قدیمی/کم قدرت...
echo .\python_embeded\python.exe -s ComfyUI\main.py --disable-cuda-malloc --windows-standalone-build
echo pause
) > run_comfyui_legacy_gpu.bat

:: ایجاد اسکریپت معمولی
(
echo @echo off
echo echo راه‌اندازی ComfyUI...
echo .\python_embeded\python.exe -s ComfyUI\main.py --windows-standalone-build
echo pause
) > run_comfyui_normal.bat

:: ایجاد workflow پایه برای upscale
echo ایجاد workflow نمونه...
(
echo {
echo   "1": {
echo     "inputs": {
echo       "ckpt_name": "model.safetensors"
echo     },
echo     "class_type": "CheckpointLoaderSimple"
echo   },
echo   "2": {
echo     "inputs": {
echo       "text": "beautiful girl, high quality, detailed",
echo       "clip": ["1", 1]
echo     },
echo     "class_type": "CLIPTextEncode"
echo   },
echo   "3": {
echo     "inputs": {
echo       "text": "bad quality, blurry, worst quality",
echo       "clip": ["1", 1]
echo     },
echo     "class_type": "CLIPTextEncode"
echo   },
echo   "4": {
echo     "inputs": {
echo       "width": 512,
echo       "height": 512,
echo       "batch_size": 1
echo     },
echo     "class_type": "EmptyLatentImage"
echo   },
echo   "5": {
echo     "inputs": {
echo       "seed": 123456,
echo       "steps": 8,
echo       "cfg": 2.0,
echo       "sampler_name": "dpmpp_2m",
echo       "scheduler": "karras",
echo       "denoise": 1.0,
echo       "model": ["1", 0],
echo       "positive": ["2", 0],
echo       "negative": ["3", 0],
echo       "latent_image": ["4", 0]
echo     },
echo     "class_type": "KSampler"
echo   },
echo   "6": {
echo     "inputs": {
echo       "samples": ["5", 0],
echo       "vae": ["1", 2]
echo     },
echo     "class_type": "VAEDecode"
echo   },
echo   "7": {
echo     "inputs": {
echo       "filename_prefix": "ComfyUI",
echo       "images": ["6", 0]
echo     },
echo     "class_type": "SaveImage"
echo   }
echo }
) > ComfyUI\user\workflows\basic_workflow.json

:: دانلود مدل پایه (اختیاری)
echo.
echo آیا می‌خواهید یک مدل پایه دانلود شود؟ (y/n)
set /p download_model="مدل SD 1.5 (حدود 4GB): "
if /i "%download_model%"=="y" (
    echo دانلود مدل SD 1.5...
    curl -L -o "ComfyUI\models\checkpoints\sd_v15.safetensors" "https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors"
)

:: نمایش اطلاعات نهایی
echo.
echo ========================================
echo         نصب با موفقیت انجام شد!
echo ========================================
echo.
echo مسیر نصب: D:\Ai\ComfyUI
echo.
echo نحوه اجرا:
if "%GPU_TYPE%"=="nvidia" (
    echo   • برای GPU های جدید: run_nvidia_gpu.bat
    echo   • برای GPU های قدیمی: run_comfyui_legacy_gpu.bat
) else (
    echo   • run_cpu.bat (کند اما کار می‌کند)
)
echo   • آدرس وب: http://127.0.0.1:8188
echo.
echo پوشه‌های مهم:
echo   • مدل‌ها: ComfyUI\models\checkpoints\
echo   • آپ‌اسکیلرها: ComfyUI\models\upscale_models\
echo   • خروجی تصاویر: ComfyUI\output\
echo   • گردش کارها: ComfyUI\user\workflows\
echo.
echo برای مدل‌ها به civitai.com مراجعه کنید
echo برای آپ‌اسکیلرها به openmodeldb.info مراجعه کنید
echo.
pause
```

## فایل 2: start-comfyui.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo      راه‌اندازی ComfyUI
echo ========================================

cd /d "D:\Ai\ComfyUI"

if not exist "ComfyUI\main.py" (
    echo خطا: ComfyUI نصب نشده!
    echo لطفاً ابتدا اسکریپت نصب را اجرا کنید.
    pause
    exit /b 1
)

echo انتخاب روش اجرا:
echo 1. GPU NVIDIA (عادی)
echo 2. GPU NVIDIA (قدیمی/مشکل دار)
echo 3. CPU (کند)
echo.
set /p choice="انتخاب شما (1-3): "

if "%choice%"=="1" (
    if exist "run_nvidia_gpu.bat" (
        echo راه‌اندازی با GPU معمولی...
        start run_nvidia_gpu.bat
    ) else (
        echo راه‌اندازی با تنظیمات عادی...
        start run_comfyui_normal.bat
    )
) else if "%choice%"=="2" (
    echo راه‌اندازی با تنظیمات GPU قدیمی...
    start run_comfyui_legacy_gpu.bat
) else if "%choice%"=="3" (
    if exist "run_cpu.bat" (
        echo راه‌اندازی با CPU...
        start run_cpu.bat
    ) else (
        echo راه‌اندازی با CPU...
        .\python_embeded\python.exe -s ComfyUI\main.py --cpu --windows-standalone-build
    )
) else (
    echo انتخاب نامعتبر!
    pause
    exit /b 1
)

timeout /t 3 /nobreak >nul
echo.
echo ComfyUI در حال راه‌اندازی است...
echo چند ثانیه صبر کنید، سپس مرورگر باز خواهد شد
echo آدرس: http://127.0.0.1:8188
echo.
pause
```

## فایل 3: stop-comfyui.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       توقف ComfyUI
echo ========================================

echo توقف فرآیندهای ComfyUI...
taskkill /F /IM python.exe >nul 2>&1
taskkill /F /IM pythonw.exe >nul 2>&1

echo.
echo ComfyUI متوقف شد.
echo.
pause
```

## فایل 4: download-models.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo     دانلود مدل‌های ComfyUI
echo ========================================

cd /d "D:\Ai\ComfyUI"

if not exist "ComfyUI\models" (
    echo خطا: ComfyUI نصب نشده!
    pause
    exit /b 1
)

echo مدل‌های موجود برای دانلود:
echo.
echo === CHECKPOINT MODELS ===
echo 1. Stable Diffusion 1.5 (4GB) - پایه و سریع
echo 2. Stable Diffusion XL Base (6.6GB) - کیفیت بالا
echo 3. DreamShaper XL (6.6GB) - فانتزی و هنری
echo.
echo === UPSCALER MODELS ===
echo 11. Real-ESRGAN x4 (17MB) - عمومی
echo 12. LDSR (2GB) - کیفیت بالا
echo 13. SwinIR x4 (47MB) - کیفیت متوسط
echo.
echo === VAE MODELS ===
echo 21. SD 1.5 VAE (319MB)
echo 22. SDXL VAE (319MB)
echo.
echo 0. خروج
echo.

:menu
set /p choice="شماره مدل مورد نظر: "

if "%choice%"=="1" (
    echo دانلود Stable Diffusion 1.5...
    curl -L -o "ComfyUI\models\checkpoints\sd15_v1-5-pruned-emaonly.safetensors" "https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="2" (
    echo دانلود Stable Diffusion XL Base...
    curl -L -o "ComfyUI\models\checkpoints\sdxl_base_1.0.safetensors" "https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="3" (
    echo دانلود DreamShaper XL...
    echo برای این مدل به civitai.com مراجعه کنید
    echo آدرس: https://civitai.com/models/112902/dreamshaper-xl
) else if "%choice%"=="11" (
    echo دانلود Real-ESRGAN x4...
    curl -L -o "ComfyUI\models\upscale_models\RealESRGAN_x4plus.pth" "https://github.com/xinntao/Real-ESRGAN/releases/download/v0.1.0/RealESRGAN_x4plus.pth"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="12" (
    echo دانلود LDSR...
    curl -L -o "ComfyUI\models\upscale_models\LDSR.ckpt" "https://heibox.uni-heidelberg.de/f/578df07c8fc04ffbadf3/?dl=1"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="13" (
    echo دانلود SwinIR x4...
    curl -L -o "ComfyUI\models\upscale_models\SwinIR_4x.pth" "https://github.com/JingyunLiang/SwinIR/releases/download/v0.0/003_realSR_BSRGAN_DFOWMFC_s64w8_SwinIR-L_x4_GAN.pth"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="21" (
    echo دانلود SD 1.5 VAE...
    curl -L -o "ComfyUI\models\vae\vae-ft-mse-840000-ema-pruned.safetensors" "https://huggingface.co/stabilityai/sd-vae-ft-mse-original/resolve/main/vae-ft-mse-840000-ema-pruned.safetensors"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="22" (
    echo دانلود SDXL VAE...
    curl -L -o "ComfyUI\models\vae\sdxl_vae.safetensors" "https://huggingface.co/stabilityai/sdxl-vae/resolve/main/sdxl_vae.safetensors"
    echo ✓ دانلود کامل شد
) else if "%choice%"=="0" (
    echo خروج...
    goto end
) else (
    echo انتخاب نامعتبر!
)

echo.
echo دانلود مدل دیگر؟ (y/n)
set /p continue="ادامه: "
if /i "%continue%"=="y" goto menu

:end
echo.
echo تمام مدل‌های دانلود شده در پوشه ComfyUI\models قرار دارند
echo.
pause
```

## فایل 5: update-comfyui.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo     به‌روزرسانی ComfyUI
echo ========================================

cd /d "D:\Ai\ComfyUI"

if not exist "ComfyUI" (
    echo خطا: ComfyUI نصب نشده!
    pause
    exit /b 1
)

echo توقف ComfyUI...
taskkill /F /IM python.exe >nul 2>&1

echo به‌روزرسانی ComfyUI اصلی...
cd ComfyUI
git pull origin master
cd ..

echo به‌روزرسانی ComfyUI Manager...
cd ComfyUI\custom_nodes\ComfyUI-Manager
git pull origin main
cd ..\..\..

echo.
echo به‌روزرسانی انجام شد!
echo.
pause
```

## فایل 6: backup-comfyui.bat

```batch
@echo off
chcp 65001 >nul
echo ========================================
echo       پشتیبان‌گیری ComfyUI
echo ========================================

cd /d "D:\Ai\ComfyUI"

set backup_date=%date:~6,4%-%date:~3,2%-%date:~0,2%
set backup_time=%time:~0,2%-%time:~3,2%-%time:~6,2%
set backup_name=comfyui-backup-%backup_date%_%backup_time%

echo ایجاد پشتیبان: %backup_name%

if not exist "backups" mkdir "backups"

echo پشتیبان‌گیری از تنظیمات و workflows...
powershell -Command "Compress-Archive -Path 'ComfyUI\user\*' -DestinationPath 'backups\%backup_name%-settings.zip'"

echo آیا می‌خواهید از مدل‌ها نیز پشتیبان بگیرید؟ (y/n)
echo توجه: این کار زمان زیادی خواهد برد
set /p backup_models="پشتیبان مدل‌ها: "

if /i "%backup_models%"=="y" (
    echo پشتیبان‌گیری از مدل‌ها...
    powershell -Command "Compress-Archive -Path 'ComfyUI\models\*' -DestinationPath 'backups\%backup_name%-models.zip'"
)

echo.
echo پشتیبان‌گیری انجام شد!
echo فایل‌های پشتیبان در پوشه backups ذخیره شدند
echo.
pause
```

## فایل 7: PowerShell Script (install-comfyui.ps1)

```powershell
# نصب خودکار ComfyUI با PowerShell
Write-Host "========================================" -ForegroundColor Green
Write-Host "       نصب خودکار ComfyUI" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green

# بررسی VRAM
Write-Host "بررسی VRAM کارت گرافیک..." -ForegroundColor Yellow
$gpu = Get-WmiObject Win32_VideoController | Where-Object {$_.AdapterRAM -ne $null} | Select-Object Name, @{Name="VRAM_GB";Expression={[math]::Round($_.AdapterRAM/1GB,1)}}
$gpu | Format-Table -AutoSize
Write-Host "توصیه: حداقل 4GB VRAM و بهتر 8GB" -ForegroundColor Cyan

# بررسی Git
try {
    git --version | Out-Null
    Write-Host "✓ Git یافت شد" -ForegroundColor Green
} catch {
    Write-Host "✗ Git نصب نشده است!" -ForegroundColor Red
    Write-Host "لطفاً Git را از https://git-scm.com/download/win دانلود کنید" -ForegroundColor Yellow
    Read-Host "برای ادامه Enter بزنید"
    exit
}

# ایجاد مسیر
$projectPath = "D:\Ai\ComfyUI"
if (!(Test-Path $projectPath)) {
    New-Item -ItemType Directory -Path $projectPath -Force
    Write-Host "✓ مسیر پروژه ایجاد شد" -ForegroundColor Green
}

Set-Location $projectPath

# تشخیص نوع GPU
$isNvidia = $false
try {
    nvidia-smi | Out-Null
    $isNvidia = $true
    Write-Host "✓ GPU NVIDIA شناسایی شد" -ForegroundColor Green
} catch {
    Write-Host "AMD/Intel GPU یا CPU mode" -ForegroundColor Yellow
}

# دانلود ComfyUI
Write-Host "دانلود ComfyUI..." -ForegroundColor Yellow
$downloadUrl = if ($isNvidia) {
    "https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_nvidia_cu118_or_cpu.7z"
} else {
    "https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_cpu.7z"
}

try {
    Invoke-WebRequest -Uri $downloadUrl -OutFile "comfyui_portable.7z" -UseBasicParsing
    Write-Host "✓ دانلود کامل شد" -ForegroundColor Green
} catch {
    Write-Host "✗ خطا در دانلود!" -ForegroundColor Red
    exit
}

# استخراج با 7-Zip
Write-Host "استخراج فایل..." -ForegroundColor Yellow
$sevenZipPath = @(
    "C:\Program Files\7-Zip\7z.exe",
    "C:\Program Files (x86)\7-Zip\7z.exe"
) | Where-Object { Test-Path $_ } | Select-Object -First 1

if ($sevenZipPath) {
    & $sevenZipPath x comfyui_portable.7z -y
    Remove-Item comfyui_portable.7z
    Write-Host "✓ استخراج کامل شد" -ForegroundColor Green
} else {
    Write-Host "✗ 7-Zip یافت نشد! لطفاً از https://www.7-zip.org/ دانلود کنید" -ForegroundColor Red
    exit
}

# نصب ComfyUI Manager
Write-Host "نصب ComfyUI Manager..." -ForegroundColor Yellow
Set-Location "ComfyUI\custom_nodes"
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
Set-Location $projectPath

# ایجاد پوشه‌ها
$folders = @(
    "ComfyUI\models\checkpoints",
    "ComfyUI\models\upscale_models", 
    "ComfyUI\models\vae",
    "ComfyUI\models\loras",
    "ComfyUI\user\workflows"
)

foreach ($folder in $folders) {
    if (!(Test-Path $folder)) {
        New-Item -ItemType Directory -Path $folder -Force
    }
}

Write-Host "✓ پوشه‌ها ایجاد شدند" -ForegroundColor Green

# ایجاد workflow نمونه
$workflowContent = @'
{
  "1": {
    "inputs": {
      "ckpt_name": "model.safetensors"
    },
    "class_type": "CheckpointLoaderSimple"
  },
  "2": {
    "inputs": {
      "text": "beautiful girl, high quality, detailed",
      "clip": ["1", 1]
    },
    "class_type": "CLIPTextEncode"
  },
  "3": {
    "inputs": {
      "text": "bad quality, blurry, worst quality",
      "clip": ["1", 1]
    },
    "class_type": "CLIPTextEncode"
  },
  "4": {
    "inputs": {
      "width": 512,
      "height": 512,
      "batch_size": 1
    },
    "class_type": "EmptyLatentImage"
  },
  "5": {
    "inputs": {
      "seed": 123456,
      "steps": 20,
      "cfg": 7.0,
      "sampler_name": "euler",
      "scheduler": "normal",
      "denoise": 1.0,
      "model": ["1", 0],
      "positive": ["2", 0],
      "negative": ["3", 0],
      "latent_image": ["4", 0]
    },
    "class_type": "KSampler"
  },
  "6": {
    "inputs": {
      "samples": ["5", 0],
      "vae": ["1", 2]
    },
    "class_type": "VAEDecode"
  },
  "7": {
    "inputs": {
      "filename_prefix": "ComfyUI",
      "images": ["6", 0]
    },
    "class_type": "SaveImage"
  }
}
'@

$workflowContent | Out-File -FilePath "ComfyUI\user\workflows\basic_workflow.json" -Encoding UTF8

# نمایش نتیجه
Write-Host "`n========================================" -ForegroundColor Green
Write-Host "         نصب با موفقیت انجام شد!" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green

Write-Host "`nمسیر نصب: $projectPath" -ForegroundColor Yellow

Write-Host "`nنحوه اجرا:" -ForegroundColor Cyan
if ($isNvidia) {
    Write-Host "  • GPU عادی: run_nvidia_gpu.bat" -ForegroundColor White
    Write-Host "  • GPU قدیمی: اضافه کردن --disable-cuda-malloc" -ForegroundColor White
} else {
    Write-Host "  • CPU: run_cpu.bat (کند)" -ForegroundColor White
}

Write-Host "  • آدرس وب: http://127.0.0.1:8188" -ForegroundColor White

Write-Host "`nپوشه‌های مهم:" -ForegroundColor Cyan
Write-Host "  • مدل‌ها: ComfyUI\models\checkpoints\" -ForegroundColor White
Write-Host "  • آپ‌اسکیلرها: ComfyUI\models\upscale_models\" -ForegroundColor White
Write-Host "  • خروجی: ComfyUI\output\" -ForegroundColor White

Write-Host "`nمنابع مدل:" -ForegroundColor Cyan
Write-Host "  • https://civitai.com/ (مدل‌های اصلی)" -ForegroundColor White
Write-Host "  • https://openmodeldb.info/ (آپ‌اسکیلرها)" -ForegroundColor White

Read-Host "`nبرای ادامه Enter بزنید"
```

## نحوه استفاده:

1. **فایل‌ها را در مسیر `D:\Ai\ComfyUI` ذخیره کنید**
2. **برای نصب**: `install-comfyui.bat` را