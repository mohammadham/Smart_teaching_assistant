# Smart_teaching_assistant_-_-_-
سند فنی و اجرایی «پروژه‌ی آموزش‌یار هوشمند»  
نسخه‌ی 0.1 – MVP کاملاً لوکال  
(قابل ارتقا به سیستم ماژولار با پکیج‌های قابل دانلود)

──────────────────────────  
۱) معماری کلان  
──────────────────────────  
```
┌─────────────────────────────┐
│  Flutter (Dart) – UI/UX     │
├─────────────────────────────┤
│  Service Layer (Dart)       │
│  ├── AudioRecorder          │
│  ├── SpeechToTextService    │
│  ├── QaEngine               │
│  └── TextToSpeechService    │
├─────────────────────────────┤
│  Native Bridge (Kotlin/Swift)│
│  └── TensorFlow Lite / ONNX │
├─────────────────────────────┤
│  Assets / Models / Logs     │
└─────────────────────────────┘
```

اصول طراحی  
• اجرای کامل آفلاین در نسخه‌ی 0.1  
• قرارداد ماژولی برای نسخه‌های بعد (interface + factory)  
• لاگ‌گیری realtime (StreamController)  
• هر مدل باید در قالب فایل‌های جدا (`qa.tflite`, `stt.tflite`, `tts.tflite`) قرار گیرد تا بعداً از سرور Laravel جایگزین شود.  

──────────────────────────  
۲) ساختار پوشه‌ای پروژه  
──────────────────────────  
```
ai_tutor/
├─ android/app/src/main/jniLibs/  ← libtflite.so
├─ assets/
│  ├─ models/
│  │  ├─ qa.tflite
│  │  ├─ stt.tflite
│  │  └─ tts/
│  │      ├─ fastspeech2.tflite
│  │      └─ vocoder.tflite
│  └─ data/
│      └─ qa_map.json            ← دانش دوره‌ی ابتدایی
├─ lib/
│  ├─ main.dart
│  ├─ ui/
│  │  └─ home_page.dart
│  ├─ services/
│  │  ├─ log_service.dart
│  │  ├─ stt_service.dart
│  │  ├─ qa_service.dart
│  │  └─ tts_service.dart
│  ├─ native/
│  │  └─ tflite_helper.dart     ← invokeMethod
│  └─ models/
│      └─ qa_model.dart
```

──────────────────────────  
۳) فهرست کارها (Backlog)  
──────────────────────────  
| گام | عنوان | خروجی قابل تست | چگونه تست کنیم |
|-----|-------|----------------|----------------|
| 0   | آماده‌سازی محیط | Flutter SDK, NDK, TFLite | `flutter doctor` |
| 1   | لاگ realtime | صفحه‌ی لاگ Stream | نوشتن در لاگ → نمایش سریع در UI |
| 2   | ضبط صدا | کلید ضبط + فایل WAV | پخش فایل ضبط‌شده |
| 3   | STT | متن فارسی از صدا | unit-test با فایل‌های تست |
| 4   | QA | پاسخ متن از متن سوال | تست با qa_map.json |
| 5   | TTS | پخش صدا از متن | شنیدن صدا |
| 6   | یکپارچه‌سازی | پرسش صوتی → پاسخ صوتی | سناریوی End-to-End |
| 7   | آماده‌سازی ماژول دانلود | قرارداد interface | تست دانلود فایل جعلی |
| 8   | سرور Laravel | endpoint /packages | curl → دریافت فایل |
| 9   | جایگزینی پویا | دانلود و جایگزینی model | تست عدم کرش |

──────────────────────────  
۴) پیاده‌سازی جزئی  
──────────────────────────  

### 4-1) لاگ‌سرویس (log_service.dart)
```dart
import 'dart:async';

class LogService {
  static final _controller = StreamController<String>.broadcast();
  static Stream<String> get stream => _controller.stream;

  static void i(String tag, String msg) {
    final line = "[${DateTime.now().toIso8601String()}] $tag: $msg";
    _controller.add(line);
    print(line);
  }

  static void close() => _controller.close();
}
```

### 4-2) صفحه‌ی لاگ (home_page.dart)
```dart
StreamBuilder<String>(
  stream: LogService.stream,
  builder: (c, snap) {
    final txt = snap.data ?? "";
    return SingleChildScrollView(
      child: Text(txt, style: const TextStyle(fontSize: 12)),
    );
  },
)
```

### 4-3) STT سرویس (stt_service.dart)
```dart
import 'package:tflite_flutter/tflite_flutter.dart';
import 'package:flutter/services.dart';

class SttService {
  late Interpreter _interpreter;

  Future<void> load() async {
    final modelData = await rootBundle.load('assets/models/stt.tflite');
    _interpreter = Interpreter.fromBuffer(modelData.buffer);
    LogService.i("STT", "Model loaded.");
  }

  Future<String> recognize(List<int> wavBytes) async {
    // پیش‌پردازش wavBytes به input tensor
    final input = _preprocess(wavBytes);
    final output = List.filled(1 * 128, 0).reshape([1, 128]);
    _interpreter.run(input, output);
    return _postProcess(output);
  }

  List<double> _preprocess(List<int> wav) => []; // TODO
  String _postProcess(List<List<int>> raw) => "سلام"; // TODO
}
```

### 4-4) QA سرویس (qa_service.dart)
```dart
class QaService {
  late final Map<String, String> _qaMap;

  Future<void> load() async {
    final jsonStr = await rootBundle.loadString('assets/data/qa_map.json');
    _qaMap = Map<String, String>.from(json.decode(jsonStr));
    LogService.i("QA", "Knowledge loaded (${_qaMap.length} pairs).");
  }

  String answer(String question) =>
      _qaMap[question.trim()] ?? "متأسفم، پاسخی ندارم.";
}
```

### 4-5) TTS سرویس (tts_service.dart)
```dart
import 'package:flutter_tts/flutter_tts.dart';

class TtsService {
  final FlutterTts _tts = FlutterTts();

  Future<void> init() async {
    await _tts.setLanguage("fa-IR");
    await _tts.setSpeechRate(0.5);
    LogService.i("TTS", "Ready.");
  }

  Future<void> speak(String text) => _tts.speak(text);
}
```

### 4-6) یکپارچه‌سازی (main.dart)
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await LogService.i("APP", "Starting…");
  await SttService().load();
  await QaService().load();
  await TtsService().init();
  runApp(const MyApp());
}
```

### 4-7) UI ساده
```dart
Row(
  children: [
    ElevatedButton(
      onPressed: () async {
        final wav = await AudioRecorder.record(3);
        final q = await SttService().recognize(wav);
        LogService.i("UI", "Question: $q");
        final a = QaService().answer(q);
        LogService.i("UI", "Answer: $a");
        await TtsService().speak(a);
      },
      child: const Text("بپرس"),
    ),
    const SizedBox(width: 8),
    ElevatedButton(
      onPressed: () => TtsService().speak("پاسخ نمونه"),
      child: const Text("بازپخش"),
    ),
  ],
)
```

──────────────────────────  
۵) تست هر گام  
──────────────────────────  
1. **لاگ** → اجرا کنید، باید خطوط زمان‌دار دیده شود.  
2. **ضبط صدا** → فایل در `/data/user/0/.../cache/test.wav` ساخته شود.  
3. **STT** → یک فایل تست WAV (مثلاً «سلام») بدهید؛ خروجی متن باید در لاگ ظاهر شود.  
4. **QA** → در qa_map.json یک جفت «سلام»→«علیک» بگذارید و بپرسید «سلام».  
5. **TTS** → دکمه‌ی «بازپخش» صدا را پخش کند.  
6. **End-to-End** → دکمه‌ی «بپرس» را بزنید، صحبت کنید، پاسخ را بشنوید.  

──────────────────────────  
۶) آماده‌سازی برای ماژول دانلود (گام 7)  
──────────────────────────  
1. قرارداد interface:
```dart
abstract class ModelProvider {
  Future<void> ensureModel(String packId);
}
```
2. پیاده‌سازی اولیه `AssetModelProvider` فقط از assets بار می‌کند؛ بعداً `HttpModelProvider` جایگزین می‌شود.  
3. سرور Laravel:  
   - Route: `GET /packages/{packId}` → دانلود zip حاوی `.tflite` و `qa_map.json`.  
   - ذخیره در `getApplicationDocumentsDirectory()`.

──────────────────────────  
۷) نکته‌ی پایانی  
──────────────────────────  
• برای تبدیل مدل‌های HuggingFace به TFLite از دستور زیر استفاده کنید:
```bash
python -m transformers.onnx --model=HooshvareLab/distil-fa-minilm qa.onnx
python -m onnxruntime.tools.convert_onnx_models_to_ort qa.onnx
```
سپس با `onnx2tf` یا `onnx-tflite` به `.tflite` تبدیل کنید.  

• فایل `qa_map.json` را می‌توان با یک اسکریپت ساده از PDF کتاب‌ها تولید کرد (استخراج سؤال/جواب).

اکنون می‌توانید با اجرای دستور زیر پروژه را روی یک گوشی واقعی تست کنید:
```bash
flutter run --release --target-platform android-arm64
```
