# خطوات نشر الكود والنتائج على GitHub

## المتطلبات قبل البدء
- حساب GitHub: ✅ موجود (MarwanMAlfalah)
- Git مثبت على MacBook: تحقق بالأمر `git --version`
- إذا غير مثبت: نزّله من https://git-scm.com/download/mac

---

## الخطوة 1: تجهيز الملفات المحلية

افتح Terminal وانتقل إلى مجلد عمل جديد:

```bash
cd ~/Desktop
mkdir phishing-detection-paper
cd phishing-detection-paper
```

انسخ الملفات التالية إلى هذا المجلد:

| الملف | المصدر |
|-------|--------|
| `Content_Based_Phishing_Detection_v6_local.ipynb` | الـ notebook النهائي بعد التشغيل (مع المخرجات) |
| `requirements.txt` | الذي حضرته لك سابقاً |
| `README.md` | الذي حضرته لك سابقاً |
| `LICENSE` | ملف ترخيص MIT |
| `.gitignore` | لتجاهل الملفات المؤقتة |
| مجلد `results/` | كل المخرجات (CSV + PNG + PDF + JSON) |

---

## الخطوة 2: تنظيف الملفات قبل الرفع

داخل مجلد `results/`، احذف الملفات الحساسة الكبيرة:

```bash
cd results
rm _checkpoint.csv          # 6 MB - ملف مؤقت
rm errors.csv               # 785 KB - سجل الأخطاء
cd ..
```

احتفظ فقط بملفات النتائج الأساسية (CSV, PNG, PDF, JSON, XLSX).

---

## الخطوة 3: إنشاء المستودع على GitHub

افتح https://github.com/new واضبط:

- **Repository name:** `content-phishing-detection`
- **Description:** `Reproducible pipeline for "Beyond URL Analysis: Evaluating Content-Based Phishing Detection Using HTML Structural Features"`
- **Public** (لتسهيل المراجعة من قبل المراجعين)
- ❌ لا تختر "Add a README" — لأنك ستضيفه من جهازك
- ❌ لا تختر "Add .gitignore" 
- اختر License: MIT

اضغط **Create repository**

---

## الخطوة 4: رفع الملفات

في Terminal داخل مجلد المشروع:

```bash
# تهيئة Git
git init
git branch -M main

# إضافة كل الملفات
git add .

# commit أول
git commit -m "Initial release: complete reproducible pipeline for AITA paper"

# ربط بمستودع GitHub
git remote add origin https://github.com/MarwanMAlfalah/content-phishing-detection.git

# رفع
git push -u origin main
```

عند طلب username وكلمة المرور:
- **Username:** MarwanMAlfalah
- **Password:** استخدم Personal Access Token (ليس كلمة المرور العادية)
  - أنشئه من: https://github.com/settings/tokens → Generate new token (classic) → اختر `repo` scope

---

## الخطوة 5: التحقق من النجاح

افتح المتصفح على:
```
https://github.com/MarwanMAlfalah/content-phishing-detection
```

تأكد من ظهور:
- ✅ ملف README.md يعرض بشكل صحيح
- ✅ الـ notebook (.ipynb) يفتح ويعرض النتائج
- ✅ مجلد results/ يحتوي الجداول والأشكال
- ✅ Badge MIT License ظاهر

---

## الخطوة 6: تحديث رابط GitHub في الورقة

افتح ملف الورقة `Research_Paper_Final_v3.docx` وحدّث المرجع [19] إلى:

```
[19] Al-Falah, M. (2026). Content-Based Phishing Detection — Reproducible Pipeline. 
GitHub repository: https://github.com/MarwanMAlfalah/content-phishing-detection
```

---

## (اختياري - يُنصح به) الخطوة 7: إنشاء DOI من Zenodo

Zenodo يعطي رقم DOI دائم يربط بمستودع GitHub — مهم للنشر الأكاديمي:

1. اذهب إلى https://zenodo.org/account/settings/github/
2. سجّل دخول بحساب GitHub
3. فعّل (toggle ON) المستودع `content-phishing-detection`
4. عُد إلى GitHub المستودع → **Releases** → **Create a new release**
5. Tag version: `v1.0.0`
6. Release title: `Initial release - AITA submission`
7. اضغط **Publish release**

Zenodo سيعطيك تلقائياً DOI خلال دقيقة. أضفه في الورقة بعد رابط GitHub.

---

## ملاحظة مهمة لأمان البيانات

⚠️ **لا ترفع** ملف `phish_tranco_dataset.csv` (6 MB) إذا يحتوي روابط phishing فعلية حية، لأن GitHub قد يصنفه كمحتوى ضار. بدلاً من ذلك:

- ارفع فقط: features matrix (بدون URLs) أو
- ضع رابط لـ PhishTank وTranco في الـ README مع كود إعادة التحميل

أو يمكنك رفعه إلى Zenodo بدلاً من GitHub.

---

## أوامر مفيدة لاحقاً

عند تحديث الكود:
```bash
git add .
git commit -m "Description of changes"
git push
```

عند نشر إصدار جديد:
```bash
git tag v1.1.0
git push --tags
```

---

## الملخص النهائي

بعد إكمال هذه الخطوات سيكون لديك:
- ✅ مستودع GitHub عام مع الكود الكامل
- ✅ README احترافي
- ✅ requirements.txt دقيق
- ✅ ترخيص MIT
- ✅ DOI من Zenodo (اختياري لكن يُنصح به)
- ✅ رابط جاهز للإضافة في الورقة العلمية

**الوقت المتوقع:** 30-45 دقيقة لإتمام كل الخطوات.
