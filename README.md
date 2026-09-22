# دفتر تسجيل الخطوط — مشروع Android (Capacitor)

تحويل كامل للتطبيق الحالي إلى مشروع Android حقيقي عبر Capacitor، مع **مشاركة PDF أصلية
حقيقية عبر Android Share Sheet** (واتساب، تيليجرام، Gmail، درايف، بلوتوث... إلخ)، وطباعة
وتنزيل مستقلين تماماً عن المشاركة.

## ⚠️ ملاحظة صريحة ومهمة

بنيت هذا المشروع بكل عنايته — ثبّتت حزم Capacitor الرسمية فعلياً، ولّدت مشروع Android
حقيقي بمكوناته الكاملة، ودمجت جسر Capacitor JS الحقيقي (مو محاكاة) واختبرته، وكتبت Plugin
أندرويد أصلي (Java) للطباعة، واختبرت **منطق JavaScript لكل مسار (مشاركة/طباعة/تنزيل)
بمحاكاة كاملة لبيئة Android الأصلية** وتأكدت أنه يستدعي واجهات Capacitor الصحيحة
بالمعاملات الصحيحة بالضبط.

**لكن بيئة العمل الحالية لا تملك Android SDK ولا Gradle الحقيقي** (محجوبان شبكياً هنا —
نفس القيد بالضبط الذي واجه محاولة Flutter سابقاً)، لذلك **لم أستطع فعلياً بناء APK ولا
تشغيله على جهاز حقيقي للتأكد النهائي 100%**. الكود مكتوب ومراجَع بدقة مطابقة لتوثيق
Capacitor الرسمي وAndroid PrintManager API، لكن التأكد النهائي — خصوصاً نافذة المشاركة
الفعلية ونافذة الطباعة على جهاز حقيقي — يحتاج تشغيله عندك.

## ما الذي تم اختباره فعلياً (ولّد نتائج حقيقية وليست افتراضات)

- ✅ تثبيت حزم Capacitor الرسمية عبر npm (نسخ حقيقية: core/cli/android@8.5.2،
  share@8.0.2، filesystem@8.1.3)
- ✅ توليد مشروع Android حقيقي عبر `npx cap add android` — كل ملفات Gradle، الـ
  AndroidManifest، الـ FileProvider (ضروري لمشاركة الملفات) موجودة وصحيحة
- ✅ جسر Capacitor JS الحقيقي يعمل بدون أي خطأ (`window.Capacitor.Plugins.Share`
  و`.Filesystem` مسجّلان وجاهزان)
- ✅ بمحاكاة بيئة Android حقيقية (native mode)، تأكدت أن:
  - زر **مشاركة** يستدعي `Filesystem.writeFile()` بملف PDF حقيقي (167 كيلوبايت) ثم
    `Share.share()` بالمسار والعنوان الصحيحين
  - زر **تنزيل PDF** يستدعي `Filesystem.writeFile()` لمجلد Documents، **منفصل تماماً**
    عن زر المشاركة
  - زر **طباعة** يستدعي الـ Plugin الأصلي الجديد `PrintDoc.printHtml()`، **منفصل تماماً**
    عن الزرين الآخرين
- ✅ في المتصفح العادي (بدون Capacitor)، كل شيء يعمل بالضبط كما كان — لم يتأثر أي منطق
  لآسياسيل أو زين أو التقارير أو السجلات
- ✅ صفر أخطاء JavaScript في كل الاختبارات

## ما لم يُختبر (يحتاج جهازك)

- بناء APK فعلي (يحتاج Android SDK + Gradle الحقيقيين)
- فتح نافذة Android Share Sheet الفعلية والتأكد أنها تعرض واتساب/تيليجرام/Gmail
- ظهور نافذة الطباعة الأصلية لأندرويد (Print Dialog) فعلياً
- صلاحيات وقت التشغيل (لا يُفترض أنها مطلوبة هنا لأن كل المسارات تستخدم مجلدات التطبيق
  الخاصة (Cache/Documents النطاق الخاص) التي لا تحتاج إذن `WRITE_EXTERNAL_STORAGE`، لكن
  الجهاز الحقيقي هو الحكم الأخير)

## بنية المشروع

```
www/index.html          نفس التطبيق الحالي بالضبط + جسر Capacitor مدمج + منطق المشاركة/
                         الطباعة/التنزيل الجديد المدرك للبيئة (Native أم متصفح)
capacitor.config.json   إعدادات Capacitor (appId: com.simtracker.records)
package.json            حزم Capacitor المطلوبة
android/                مشروع Android الكامل (Gradle + AndroidManifest + الكود الأصلي)
  app/src/main/java/com/simtracker/records/
    MainActivity.java   يسجّل الـ PrintPlugin الجديد
    PrintPlugin.java    Plugin أصلي بلغة Java يستخدم Android PrintManager الحقيقي
```

## خطوات البناء (على جهازك، بعد تثبيت Android Studio)

1. فك ضغط المجلد، وافتح طرفية بداخله.
2. ثبّت الحزم:
   ```
   npm install
   ```
3. زامن المشروع (ينسخ آخر نسخة من www/ إلى المشروع الأصلي، ويحدّث الـ Plugins):
   ```
   npx cap sync android
   ```
4. افتح المشروع في Android Studio:
   ```
   npx cap open android
   ```
   (أو افتح مجلد `android/` يدوياً من Android Studio مباشرة)
5. دع Android Studio يزامن Gradle تلقائياً (أول مرة تحمّل توزيعة Gradle 8.14.3 وAndroid
   SDK المطلوبة — يحتاج إنترنت طبيعي، هذا ما كان محجوباً عني هنا فقط).
6. وصّل جهاز أندرويد حقيقي (مع تفعيل USB Debugging) أو استخدم محاكي، ثم اضغط **Run** ▶️
   من Android Studio مباشرة — هذا أسهل طريقة للتجربة الفورية.
7. لبناء APK قابل للتوزيع:
   - من Android Studio: **Build → Build Bundle(s) / APK(s) → Build APK(s)**
   - أو من الطرفية: `cd android && ./gradlew assembleDebug`
   - الملف يطلع بمسار: `android/app/build/outputs/apk/debug/app-debug.apk`

## إذا عدّلت التطبيق لاحقاً

أي تعديل تسويه على الملف الأصلي (نسخة الويب) يُنسخ لهذا المشروع بنفس الخطوة:
```
cp /path/to/updated/sim-tracker.html www/index.html
npx cap sync android
```
**مهم:** إذا حدّثت `index.html` من مصدر خارجي، تأكد إنه يحتفظ بأسطر جسر Capacitor
الثلاثة القريبة من نهاية الملف (قبل `<script>(function(){` الخاص بمنطق التطبيق) — هذي
الأسطر ضرورية لعمل المشاركة/الطباعة/التنزيل الأصلي.

## كيف يعمل الفصل بين المتصفح والتطبيق الأصلي

كل الكود الجديد يتحقق أولاً:
```js
function isCapacitorNative(){
  return !!(window.Capacitor && window.Capacitor.isNativePlatform && window.Capacitor.isNativePlatform());
}
```
- **داخل تطبيق Android المبني**: `true` → يُستخدم Capacitor Share/Filesystem/PrintPlugin
  الحقيقيين.
- **في أي متصفح عادي** (بما فيها فتح `www/index.html` مباشرة، أو نشره كموقع ويب): `false`
  → يعمل بالضبط بنفس منطق Web Share API/الطباعة عبر iframe الذي كان يعمل سابقاً، بدون أي
  تغيير.

هذا يعني نفس ملف `index.html` صالح للاستخدام كملف ويب مستقل تماماً كما كان، وأيضاً كتطبيق
Android حقيقي — بدون تفريع نسختين منفصلتين.

## إذا واجهت خطأ عند البناء

- **خطأ Gradle sync**: تأكد إن جهازك متصل بإنترنت طبيعي (غير مقيّد) أول مرة — يحتاج تحميل
  توزيعة Gradle وAndroid SDK.
- **خطأ compile في PrintPlugin.java**: راجع إن `compileSdkVersion`/`minSdkVersion` في
  `android/variables.gradle` متوافقين مع نسخة Android Studio عندك؛ عادة لا حاجة لتغيير
  شيء.
- **المشاركة/الطباعة لا تعمل فعلياً على الجهاز**: الصق لي رسالة الخطأ بالضبط (من
  Logcat في Android Studio) وأصلحها مباشرة.
