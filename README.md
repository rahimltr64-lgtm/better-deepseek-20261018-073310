# 🎓 تعلّم معي — تطبيق تعليمي للأطفال

تطبيق موبايل بـ **Flutter** يعلّم الأطفال **الحروف العربية (٢٨ حرفاً)** و**الأرقام (٠ – ١٠)** بطريقة تفاعلية، مع واجهة عربية كاملة (RTL)، وضع ليلي/فاتح، نُطق صوتي، لعبة اختبار، وتذكير يومي.

---

## ⚠️ إصلاح خطأ البناء على Klencod IDE

الخطأ الذي ظهر:

Flutter failed to read file at ".../android/app/build.gradle"
PathNotFoundException: Cannot open file ...

**السبب:** أرشيف المشروع يحتوي على مجلد `lib/` و`pubspec.yaml` فقط، بينما مجلد `android/` (ومعه `app/build.gradle` والأيقونات والموارد) يُنشأ بأمر `flutter create` — ولم يكن موجوداً في المستودع، ففشل البناء.

**الحل:** أُضيف الآن:

1. `.github/workflows/build-apk.yml` — يقوم تلقائياً بإنشاء مجلد `android/` قبل البناء.
2. `platform_overrides/` — تعديلاتنا على `AndroidManifest.xml` و`build.gradle` (صلاحيات الإشعارات + core library desugaring).

### طريقة التطبيق

1. انسخ هذه الملفات إلى **جذر مستودعك** بنفس المسارات:

.github/workflows/build-apk.yml
platform_overrides/android/app/src/main/AndroidManifest.xml
platform_overrides/android-kts/app/build.gradle.kts
platform_overrides/android-groovy/app/build.gradle
2. ارفعها (`git add . && git commit -m "fix android build" && git push`).
3. إن كان Klencod IDE يولّد ملف workflow خاصاً به، **احذفه أو عطّله** حتى لا يعمل مسارَان للبناء بالتوازي.
4. أعد البناء من جديد.

### لماذا يعمل هذا؟

- `flutter create . --platforms=android` يُنشئ المجلد كاملاً: الأيقونات، `MainActivity.kt`، `res/`، وملفات Gradle.
- `--project-name kids_learning` يضمن أن اسم الحزمة صحيح حتى لو كان اسم المجلد غير صالح.
- ننسخ كود المشروع من نسخة احتياطية بعد `flutter create` حتى لا يُستبدل `lib/` أو `pubspec.yaml`.
- نكتشف تلقائياً إن كان القالب يستخدم **Groovy** أو **Kotlin DSL** وننسخ الملف المناسب.
- `coreLibraryDesugaring` مطلوب من `flutter_local_notifications`، وبدونه يفشل `:app:desugarDebugFileDependencies`.

---

## ✨ الميزات

- 🔤 **الحروف العربية**: ٢٨ حرفاً مع اسم الحرف وكلمة تبدأ به وصورة تعبيرية.
- 🔢 **الأرقام**: من ٠ إلى ١٠ مع الرقم العربي والنطق اللفظي وعدد الأشياء.
- 🔊 **النطق الصوتي** بالعربية عبر محرك TTS في الجهاز.
- 🎯 **لعبة الاختبار**: ١٠ أسئلة عشوائية مع نتيجة ونجوم.
- ⭐ **نظام النجوم والتقدّم** محفوظ محلياً.
- 🌙 **الوضع الليلي / الفاتح / حسب النظام**.
- 🔔 **تذكير يومي** قابل للتفعيل وتحديد وقته.
- 🧭 **دعم كامل للعربية واتجاه RTL**.

---

## 🚀 التشغيل محلياً

flutter pub get
flutter create . --project-name kids_learning --org com.example --platforms=android
flutter run

> بعد أول `flutter create` طبّق ملفات `platform_overrides/` يدوياً إن أردت الإشعارات.

### بناء APK محلياً

flutter build apk --release

الملف الناتج:

build/app/outputs/flutter-apk/app-release.apk

---

## 📁 هيكل المشروع

lib/
├── main.dart
├── app.dart
├── core/
│   ├── data/          # بيانات الحروف والأرقام
│   ├── services/      # التخزين، TTS، الإشعارات
│   ├── theme/         # الألوان والثيمات
│   └── widgets/       # بطاقات مشتركة
└── features/
├── splash/
├── home/
├── letters/
├── numbers/
├── quiz/
├── progress/
└── settings/

.github/workflows/build-apk.yml   # بناء APK تلقائياً
platform_overrides/               # تعديلات ملفات Android

---

## 🎨 خط عربي (اختياري)

حمّل خط **Cairo** من Google Fonts، ضعه في `assets/fonts/`، ثم أضف في `pubspec.yaml`:

flutter:
fonts:

- family: Cairo
fonts:

- asset: assets/fonts/Cairo-Regular.ttf
- asset: assets/fonts/Cairo-Bold.ttf
weight: 700

وفي `lib/core/theme/app_theme.dart` أضف داخل `ThemeData`:

fontFamily: 'Cairo',

---

## 🔊 ملاحظة عن الصوت

- **Android**: الإعدادات ← اللغة والإدخال ← تحويل النص إلى كلام ← تثبيت حزمة العربية.
- **iOS**: الإعدادات ← إمكانية الوصول ← المحتوى المنطوق ← الأصوات ← العربية.

---

## 🔔 ملاحظة عن الإشعارات

- تُطلب صلاحية الإشعارات عند تفعيل التذكير من شاشة الإعدادات (Android 13+).
- التذكير مجدول بـ `inexactAllowWhileIdle` ليعمل دون الحاجة إلى صلاحية المنبّه الدقيق، وقد يتأخر دقائق في وضع توفير الطاقة.

---

صُنع بـ ❤️ لتعلّم الأطفال.