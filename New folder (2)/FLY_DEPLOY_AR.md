# 🚀 خطوات نشر البوت على Fly.io

## ⚠️ تنبيه مهم قبل البدء

البوت تبعك حالياً شغّال على Replit. لما ترفعه على Fly.io بنفس BOT_TOKEN، **تيليغرام بيرفض تشغيل نسختين بنفس الوقت** (long polling conflict).

→ بعد ما يشتغل على Fly.io، **لازم توقف Replit Deployment** عشان يبقى Fly.io هو النسخة الوحيدة.

---

## الخطوة 1: تثبيت أداة Fly CLI

افتح **Shell** هنا في Replit (تحت أيقونة الـ Tools)، والصق هذا الأمر:

```bash
curl -L https://fly.io/install.sh | sh
export FLYCTL_INSTALL="/home/runner/.fly"
export PATH="$FLYCTL_INSTALL/bin:$PATH"
```

تأكّد من نجاح التثبيت:
```bash
flyctl version
```

---

## الخطوة 2: تسجيل حساب على Fly.io

```bash
flyctl auth signup
```

- رح يفتح صفحة في المتصفح، سجّل بـ Gmail أو GitHub.
- **رح يطلب منك بطاقة دفع** — أدخلها (آمنة، الباقة المجانية ما تخصم منك أي مبلغ).

---

## الخطوة 3: إنشاء التطبيق

```bash
flyctl apps create game-top-up-bot
```

> لو الاسم مأخوذ، جرّب اسم ثاني مثل `hamza-game-bot` وعدّله في `fly.toml` (السطر الأول).

---

## الخطوة 4: إنشاء قرص دائم لقاعدة البيانات

```bash
flyctl volumes create bot_data --region fra --size 1 --yes
```

> هذا قرص بحجم 1 جيجا في فرانكفورت لتخزين قاعدة البيانات (مجاني ضمن الحد).

---

## الخطوة 5: ضبط المتغيرات السرية

انسخ هذه الأوامر **واحد واحد** والصق المفاتيح من Replit Secrets مكان `XXX`:

```bash
flyctl secrets set BOT_TOKEN="XXX"
flyctl secrets set FASTCARD_TOKEN="XXX"
flyctl secrets set SHAMCASH_TOKEN="XXX"
flyctl secrets set SYRIATEL_CASH_TOKEN="XXX"
flyctl secrets set SESSION_SECRET="XXX"
flyctl secrets set ADMIN_ID="5570859353"
```

---

## الخطوة 6: نقل قاعدة البيانات الحالية (مهم!)

قاعدة البيانات الحالية فيها: الزبائن، الأرصدة، الطلبات، الأسعار اليدوية. **لازم ننقلها**.

أولاً انشر البوت بدون نسخة (عشان ينشأ القرص):

```bash
flyctl deploy --no-cache
```

**انتظر حتى يخلص النشر ويصير `running`**، ثم أوقف الماكينة مؤقتاً عشان نقدر ننسخ الملف:

```bash
flyctl machine list
flyctl machine stop <MACHINE_ID>
```

ارفع الملف للقرص الدائم:

```bash
flyctl ssh sftp shell
# داخل الـ shell:
put bot/database.db /data/database.db
exit
```

شغّل البوت:
```bash
flyctl machine start <MACHINE_ID>
```

---

## الخطوة 7: راقب اللوغ وتأكد البوت شغّال

```bash
flyctl logs
```

تقدر تشوف الرسائل بتطلع لما تحكي مع البوت من تيليغرام. اضغط `Ctrl+C` للخروج.

---

## الخطوة 8: ⛔ أوقف نسخة Replit (مهم جداً!)

افتح Replit ← **Deployments** ← اضغط على البوت ← **Stop / Pause Deployment**.

> بدون هاي الخطوة، البوت رح يطلع 409 Conflict ويوقف على Fly.io!

---

## ✅ خلصنا! تأكد:

- ابعث `/start` للبوت من تيليغرام → لازم يرد فوراً.
- جرّب `/admin` من حسابك → لازم تطلع لوحة الأدمن.
- جرّب طلب صغير → لازم يخلص بنجاح.

---

## 🔧 أوامر مفيدة بعد النشر

| الأمر | الوظيفة |
|------|---------|
| `flyctl logs` | شوف اللوغ المباشر |
| `flyctl status` | حالة البوت |
| `flyctl deploy` | نشر تحديثات جديدة |
| `flyctl ssh console` | دخول لداخل الماكينة |
| `flyctl secrets list` | عرض أسماء الأسرار |

---

## 💰 الكلفة

ضمن الباقة المجانية (Free Allowance):
- **3 ماكينات** shared-cpu-1x بـ 256MB ✅
- **3 جيجا** قرص دائم ✅
- **160 جيجا** ترافيك شهرياً ✅

البوت تبعك يستخدم 1 ماكينة + 1 جيجا قرص = **مجاني تماماً**.
