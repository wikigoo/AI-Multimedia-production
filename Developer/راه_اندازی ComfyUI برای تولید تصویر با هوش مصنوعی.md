**راهنمای عملی: راه‌اندازی ComfyUI برای تولید تصویر با هوش مصنوعی به صورت محلی (روی کامپیوتر شخصی)**

**مقدمه:** این راهنما شما را در مراحل نصب ComfyUI روی کامپیوتر ویندوزی، به همراه ComfyUI Manager، یک مدل Checkpoint برای تولید تصویر و یک مدل Upscaler برای افزایش وضوح تصویر، راهنمایی می‌کند. این راه‌اندازی به شما امکان می‌دهد تصاویر را به صورت محلی و بدون محدودیت تولید کنید و پایه‌ای برای تکنیک‌های پیشرفته‌تر مانند ایجاد شخصیت‌های با چهره ثابت است (که در آموزش‌های آینده توسط سازنده ویدیو پوشش داده خواهد شد).

---

**فاز ۱: پیش‌نیازها و بررسی سیستم**

1. **بررسی حافظه VRAM کارت گرافیک (GPU) شما:**  
     
   * کلید **Windows \+ R** را فشار دهید تا پنجره Run باز شود.  
   * `dxdiag` را تایپ کرده و Enter را بزنید.  
   * در پنجره "DirectX Diagnostic Tool"، به تب **Display** بروید. (اگر لپ‌تاپ با چند GPU دارید، اطلاعات مربوطه ممکن است زیر تب **Render** باشد).  
   * به دنبال "Display Memory (VRAM)" بگردید. سازنده ویدیو اشاره می‌کند که 8 گیگابایت توصیه می‌شود، اما او نشان می‌دهد که با 4 گیگابایت نیز کار می‌کند (هرچند کندتر خواهد بود). حداقل به 4 گیگابایت نیاز دارید.

   

2. **نصب Git:**  
     
   * به آدرس [https://git-scm.com/download/win](https://git-scm.com/download/win) بروید.  
   * "Standalone Installer" را دانلود کنید (مثلاً 64-bit Git for Windows Setup).  
   * نصب‌کننده را اجرا کنید. در تمام مراحل روی "Next" کلیک کنید تا نصب کامل شود. Git برای ComfyUI Manager مورد نیاز است.

---

**فاز ۲: نصب ComfyUI**

1. **دانلود ComfyUI (نسخه پرتابل/قابل حمل):**  
     
   * به صفحه GitHub ComfyUI بروید: [https://github.com/comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)  
   * به بخش "Installing" بروید.  
   * زیر "Windows Portable"، لینک "Direct link to download" را پیدا کرده و روی آن کلیک کنید. این کار یک فایل فشرده `.7z` را دانلود می‌کند (مثلاً `ComfyUI_windows_portable_nvidia_cu118_or_cpu.7z`). این فایل حجیم است (بیش از ۱ گیگابایت).

   

2. **استخراج ComfyUI:**  
     
   * پس از دانلود، فایل فشرده `.7z` را در مکان دلخواه خود استخراج کنید (مثلاً `D:\ComfyUI_portable`). برای این کار به برنامه‌ای مانند 7-Zip یا WinRAR نیاز دارید.  
   * پوشه استخراج شده چندین گیگابایت حجم خواهد داشت (در ویدیو حدود ۵-۶ گیگابایت).

   

3. **نصب ComfyUI Manager:**  
     
   * به صفحه GitHub ComfyUI Manager بروید: [https://github.com/LtdrData/ComfyUI-Manager](https://github.com/LtdrData/ComfyUI-Manager)  
   * به بخش "Installation" و به طور خاص **"Installation (method2) (Installation for portable ComfyUI version: ComfyUI-Manager only)"** بروید.  
   * لینک فایل `install-manager-for-portable-version.bat` را پیدا کنید.  
   * روی این لینک **راست کلیک** کرده و "Save link as..." (ذخیره پیوند به عنوان...) را انتخاب کنید.  
   * به **پوشه اصلی** (root directory) ComfyUI که استخراج کرده‌اید بروید (مثلاً `D:\ComfyUI_portable`) و فایل `.bat` را در آنجا ذخیره کنید.  
   * به پوشه ComfyUI خود رفته و روی فایل `install-manager-for-portable-version.bat` که ذخیره کرده‌اید **دوبار کلیک** کنید.  
     * اگر Windows Defender SmartScreen آن را مسدود کرد، روی "More info" (اطلاعات بیشتر) و سپس "Run anyway" (به هر حال اجرا کن) کلیک کنید.  
   * یک پنجره خط فرمان (command prompt) ظاهر می‌شود، دستوراتی را اجرا می‌کند و سپس به طور خودکار بسته می‌شود. ComfyUI Manager اکنون نصب شده است.

---

**فاز ۳: افزودن مدل‌ها**

1. **دانلود یک مدل Checkpoint (برای تولید تصویر):**  
     
   * سازنده ویدیو از "WildCardX-XL TURBO" استفاده می‌کند. شما می‌توانید این مدل و بسیاری از مدل‌های دیگر را در [https://civitai.com/](https://civitai.com/) پیدا کنید.  
   * مدل مورد نظر خود را جستجو کنید (مثلاً "WildCardX-XL TURBO").  
   * فایل مدل را دانلود کنید (معمولاً یک فایل `.safetensors` با حجم حدود ۶-۷ گیگابایت است).  
   * فایل مدل دانلود شده را در این زیرپوشه ComfyUI قرار دهید: `[پوشه ComfyUI شما]\ComfyUI\models\checkpoints\` (مثلاً `D:\ComfyUI_portable\ComfyUI\models\checkpoints\`)

   

2. **دانلود یک مدل Upscaler (برای افزایش وضوح):**  
     
   * سازنده ویدیو از "4x-ClearRealityV1" استفاده می‌کند. این مدل را می‌توانید در [https://openmodeldb.info/](https://openmodeldb.info/) یا سایر مخازن مدل پیدا کنید.  
   * "ClearRealityV1" یا یک آپ‌اسکیلر 4x مشابه را جستجو کنید.  
   * فایل مدل را دانلود کنید (معمولاً یک فایل `.pth` با حجم نسبتاً کم، مثلاً ۷ مگابایت است).  
   * فایل مدل آپ‌اسکیلر دانلود شده را در این زیرپوشه ComfyUI قرار دهید: `[پوشه ComfyUI شما]\ComfyUI\models\upscale_models\` (مثلاً `D:\ComfyUI_portable\ComfyUI\models\upscale_models\`)

---

**فاز ۴: اجرای ComfyUI و تولید اولین تصویر**

1. **اجرای ComfyUI:**  
     
   * به پوشه اصلی ComfyUI portable خود بازگردید (مثلاً `D:\ComfyUI_portable`).  
   * روی `run_nvidia_gpu.bat` (اگر GPU انویدیا دارید) یا `run_cpu.bat` (اگر فقط از CPU استفاده می‌کنید که بسیار کند خواهد بود) دوبار کلیک کنید.  
   * یک پنجره خط فرمان ظاهر شده و شروع به بارگذاری ComfyUI می‌کند.  
   * پس از مدت کوتاهی، مرورگر وب شما باید به طور خودکار به آدرس `http://127.0.0.1:8188/` باز شود و رابط کاربری ComfyUI را نشان دهد.

   

2. **عیب‌یابی خطای CUDA (در صورت بروز):**  
     
   * اگر با خطای "CUDA error" مواجه شدید (به خصوص با GPUهای انویدیا قدیمی‌تر یا کم‌قدرت‌تر مانند 840M در ویدیو)، باید اسکریپت اجرا را تغییر دهید:  
     * تب مرورگر ComfyUI و پنجره خط فرمان را ببندید.  
     * در پوشه اصلی ComfyUI، روی `run_nvidia_gpu.bat` راست کلیک کرده و "Edit" (ویرایش) را انتخاب کنید (این کار ممکن است آن را در Notepad باز کند).  
     * خطی مشابه این خواهید دید: `.\python_embeded\python.exe -s ComfyUI\main.py --windows-standalone-build`  
     * این خط را با افزودن `--disable-cuda-malloc` *قبل از* `--windows-standalone-build` تغییر دهید. خط جدید باید اینگونه باشد: `.\python_embeded\python.exe -s ComfyUI\main.py --disable-cuda-malloc --windows-standalone-build`  
     * فایل را ذخیره کرده و Notepad را ببندید.  
     * دوباره `run_nvidia_gpu.bat` را اجرا کنید. پنجره خط فرمان اکنون باید برای GPU شما عبارت "native" را نشان دهد و خطا باید برطرف شده باشد.

   

3. **تولید یک تصویر پایه (گردش کار پیش‌فرض):**  
     
   * پس از اجرای ComfyUI در مرورگر، یک گردش کار مبتنی بر گره (node) پیش‌فرض خواهید دید.  
   * گره "Load Checkpoint" باید به طور خودکار مدل checkpoint دانلود شده شما را انتخاب کرده باشد (مثلاً "wildcardxXL TURBO").  
   * گره "CLIP Text Encode (Prompt)" یک پرامپت مثبت پیش‌فرض خواهد داشت (مثلاً "beautiful scenery nature glass bottle..."). می‌توانید بعداً آن را تغییر دهید.  
   * گره "Empty Latent Image" اندازه تصویر را تنظیم می‌کند (پیش‌فرض 512x512).  
   * روی دکمه **"Queue Prompt"** کلیک کنید (یا Ctrl+Enter را فشار دهید).  
   * همزمان با پردازش گره‌ها، خطوط سبز رنگی دور آن‌ها ظاهر می‌شود.  
   * تصویر نهایی در گره "Save Image" ظاهر می‌شود.  
   * همچنین می‌توانید روی آیکون "Queue (q)" در نوار کناری سمت چپ کلیک کنید تا پیشرفت را مشاهده نمایید.

   

4. **بارگذاری گردش کار بزرگ‌نمایی سفارشی و تولید تصویر بزرگ‌نمایی شده:**  
     
   * سازنده ویدیو یک فایل گردش کار سفارشی (`upsclare-image.json`) را در کانال تلگرام خود (ProshTech) ارائه می‌دهد. این فایل را دانلود کنید.  
   * فایل `upsclare-image.json` را در این مسیر قرار دهید: `[پوشه ComfyUI شما]\ComfyUI\user\workflows\`  
   * در رابط کاربری وب ComfyUI:  
     * روی آیکون پوشه در نوار کناری سمت چپ کلیک کنید (راهنمای شناور: "Workflows (w)").  
     * گردش کار `upsclare-image.json` شما باید در لیست باشد. برای بارگذاری روی آن کلیک کنید.  
     * چیدمان گره‌ها تغییر می‌کند و اکنون شامل گره‌های "Load Upscale Model" و "Upscale Image (using Model)" می‌شود.  
   * در گره "Load Upscale Model"، آپ‌اسکیلری که دانلود کرده‌اید را انتخاب کنید (مثلاً `4x-ClearRealityV1.pth`).  
   * **پرامپت مثبت** (گره "CLIP Text Encode" بالایی) را تغییر دهید. سازنده ویدیو از این پرامپت استفاده می‌کند: `dark hair, girl, slim, Denim Button-down shirt dress with patch pockets --tilted_upscale`  
   * **پرامپت منفی** (گره "CLIP Text Encode" پایینی) را تغییر دهید. سازنده ویدیو از یک پرامپت جامع استفاده می‌کند (در کانال تلگرام او موجود است). یک پرامپت پایه می‌تواند این باشد: `(worst quality, low quality, normal quality, lowres, low details, oversaturated, undersaturated, overexposed, underexposed, grayscale, bw, bad photo, bad photography, bad art:1.4), (watermark, signature, text, logo, font, username, error, typo, words, letters, digits, autograph, trademark, name:1.2)`  
   * روی **"Queue Prompt"** کلیک کنید.  
   * تصویر تولید شده و سپس بزرگ‌نمایی خواهد شد.

---

**فاز ۵: محل ذخیره تصاویر تولید شده شما**

* تمام تصاویر تولید شده در این زیرپوشه ComfyUI ذخیره می‌شوند: `[پوشه ComfyUI شما]\ComfyUI\output\` (مثلاً `D:\ComfyUI_portable\ComfyUI\output\`)  
* فایل‌ها معمولاً با نام‌هایی مانند `ComfyUI_00001_.png`، `ComfyUI_00002_.png` و غیره نامگذاری می‌شوند.

---

**نتیجه‌گیری:** شما اکنون با موفقیت ComfyUI را روی کامپیوتر محلی خود راه‌اندازی کرده‌اید، اجزای ضروری را نصب کرده‌اید و اولین تصاویر پایه و بزرگ‌نمایی شده خود را تولید کرده‌اید. این مراحل، اساس کاوش در تکنیک‌های پیشرفته‌تر تولید تصویر را تشکیل می‌دهد. به یاد داشته باشید که برای دریافت فایل‌های گردش کار و نمونه‌های پرامپت، به کانال تلگرام سازنده ویدیو (ProshTech) مراجعه کنید.  
