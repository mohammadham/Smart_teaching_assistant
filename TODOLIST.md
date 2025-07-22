To-Do List رسمی – پروژه «آموزش‌یار هوشمند» (نسخهٔ ۰.۱ → ۱.۰)  
فرمت: [x] انجام شده | [ ] باقی‌مانده | اولویت (۱=بالا) | مسئول

---

### ✅ فاز ۰ – آماده‌سازی محیط
- [x] نصب Flutter 3.22 + Dart 3  
- [x] نصب Android SDK + NDK 25.2  
- [x] نصب Python 3.11 برای تبدیل مدل‌ها  
- [x] ریپو Git + برنچ develop  
- [x] تنظیم GitHub Actions برای CI ساده (build apk)  

---

### 🔧 فاز ۱ – ساختار اولیه
- [x] ساخت پروژه Flutter (ai_tutor)  
- [x] ایجاد پوشه‌بندی استاندارد lib/, assets/, native/  
- [x] اضافه کردن پلاگین‌های pubspec:  
  - tflite_flutter: ^2.12.0  
  - flutter_tts: ^3.8.5  
  - path_provider: ^2.1.2  
  - dio: ^5.4.0 (برای دانلود بعدی)  
- [x] تنظیم minSdkVersion 24 و ndk abiFilters arm64-v8a  
- [ ] نوشتن README کوتاه (نحوهٔ build) – اولویت ۳  

---

### 📄 فاز ۲ – لاگ Real-Time
- [x] پیاده‌سازی LogService با StreamController  
- [x] ساخت ویجت LogPanel (StreamBuilder)  
- [ ] تست لاگ در حالت release روی دو دیوایس – اولویت ۲  

---

### 🎙️ فاز ۳ – ضبط صدا
- [x] پیاده‌سازی AudioRecorder (native Kotlin)  
- [x] تبدیل PCM → WAV 16 kHz 16-bit  
- [ ] یونیت‌تست با فایل ساکن در assets – اولویت ۲  

---

### 🗣️ فاز ۴ – STT (Speech-to-Text)
- [x] انتخاب Whisper Tiny 39 M → کمی‌سازی 8-bit → 75 MB  
- [x] تبدیل به stt.tflite (Python script: convert_whisper_tflite.py)  
- [x] پیاده‌سازی SttService.load() / recognize()  
- [ ] بررسی accuracy روی ۱۰ جملهٔ فارسی – اولویت ۲  
- [ ] بهینه‌سازی حذف سکوت با VAD سبک – اولویت ۴  

---

### 🧠 فاز ۵ – QA Core
- [x] ساخت qa_map.json اولیه (۱۰۰ سؤال/جواب ابتدایی)  
- [x] پیاده‌سازی QaService.answer()  
- [ ] اسکریپت Python استخراج خودکار از PDF کتاب‌ها – اولویت ۳  

---

### 🔊 فاز ۶ – TTS (Text-to-Speech)
- [x] انتخاب FastSpeech2 + vocoder Griffin-Lim → 40 MB  
- [x] تبدیل به tts.tflite & vocoder.tflite  
- [x] پیاده‌سازی TtsService.init() / speak()  
- [ ] تنظیم pitch & speed با اسلایدر در UI – اولویت ۳  

---

### 🧩 فاز ۷ – یکپارچه‌سازی MVP
- [x] اتصال تمام سرویس‌ها در HomePage  
- [x] دکمه «بپرس» (ضبط → STT → QA → TTS)  
- [x] دکمه «بازپخش»  
- [ ] سناریوی تست End-to-End روی ۵ کاربر – اولویت ۱  

---

### 📦 فاز ۸ – ماژول دانلود
- [ ] تعریف interface ModelProvider + دو پیاده‌سازی Asset / Http  
- [ ] طراحی قرارداد پکیج (pack.json + .zip)  
- [ ] پیاده‌سازی PackDownloader با Dio + ProgressBar – اولویت ۲  
- [ ] ذخیره در getApplicationDocumentsDirectory()  
- [ ] تست دانلود و جایگزینی مدل – اولویت ۲  

---

### 🌐 فاز ۹ – سرور Laravel
- [ ] ریپو جدا laravel-ai-packages  
- [ ] ساخت جدول packages (id, name, version, size, url)  
- [ ] API GET /packages  
- [ ] API GET /packages/{id}/download  
- [ ] تست با Postman – اولویت ۲  

---

### 🔍 فاز ۱۰ – تست و کیفیت
- [ ] نوشتن ۲۰ تست واحد (flutter test) – اولویت ۳  
- [ ] تست عملکرد FPS و RAM با Flutter DevTools – اولویت ۳  
- [ ] تست آفلاین کامل در حالت airplane mode – اولویت ۱  

---

### 🚀 فاز ۱۱ – انتشار آزمایشی
- [ ] ساخت apk release signed  
- [ ] آپلود در Firebase App Distribution – اولویت ۳  
- [ ] جمع‌آوری گزارش‌های Crashlytics – اولویت ۳  

---

### 📅 جدول زمانی پیشنهادی (۴ هفته)
| هفته | کارها |
|------|-------|
| ۱ | فاز ۰ تا ۳ |
| ۲ | فاز ۴ و ۵ و ۶ |
| ۳ | فاز ۷ و تست‌های اولیه |
| ۴ | فاز ۸ و ۹ و انتشار آزمایشی |

---

### 🛠️ اسکریپت‌های کمکی
- `tools/convert_whisper_tflite.py`  
- `tools/extract_qa.py` (از PDF کتاب درسی)  
- `scripts/build_apk.sh`

---

### 🗂 برچسب‌های Git
- `v0.1-mvp` – پس از اتمام فاز ۷  
- `v0.2-download` – پس از فاز ۸ و ۹  

همین لیست را در GitHub Projects به صورت Kanban هم می‌توانید import کنید.
