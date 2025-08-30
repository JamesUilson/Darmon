# Darmon

> **Darmon** — dorixonalar va xizmat ko‘rsatish (delivery) uchun ishlab chiqilgan to‘liq ekotizim: mijoz mobil ilovasi, sotuvchi (dorixona) paneli, kuryer ilovasi, veb admin va REST API backend.

---

## Muqaddima

Ushbu README loyihaning maqsadi, arxitekturasi va loyihani mahalliy yoki server muhitida ishga tushirish bo‘yicha to‘liq va professional hujjatdir. README loyihani yangi ishlab chiquvchilarga (contributor), deploy bo‘lishni rejalashtirayotgan DevOps muhandislariga va product egalariga mo‘ljallangan.

> **Eslatma:** repozitoriya tarkibi: `dorihona_app`, `curyers_app` (kuryerlar uchun), `dorihona_web`, `dorihona_backend` va bir nechta APK fayllar mavjud (repo rootida `darmon.apk`, `app-debug.apk`). citeturn0view0

---

## Jadval (Table of contents)

1. Xulosa (Overview)
2. Asosiy funksiyalar (Features)
3. Arxitektura (Architecture)
4. Repo tuzilmasi (Repository structure)
5. Tez boshlash (Quickstart)

   * Talablar (Prerequisites)
   * Backend ishga tushirish (local)
   * Frontend (Flutter) ilovalarni ishga tushirish
   * APK o‘rnatish (local testing)
6. Ma’lumotlar bazasi va model tuzilmasi (Data models)
7. API hujjatlari & misollar (Endpoints & Examples)
8. To‘lovlar va komissiya logikasi (Payments)
9. Rollar va ruxsatlar (Roles & Permissions)
10. Test qilish va CI/CD
11. Deploy (production) tavsiyalari
12. Hissa qo‘shish (Contributing)
13. Litsenziya va aloqa
14. Qo‘shimcha: roadmap, known issues

---

## 1) Xulosa (Overview)

Darmon — dorixonalar uchun mahsulotlarni ro‘yxatlash, mijozlardan buyurtma qabul qilish, kuryerlar orqali yetkazib berish va to‘lovlarni boshqarish uchun mo‘ljallangan kompleks platformadir. Loyihaning maqsadi — dorixonalar va ularning mijozlari o‘rtasidagi buyurtma jarayonini maksimal darajada avtomatlashtirish: buyurtma qabul qilish, to‘lovni onlayn qabul qilish (Click yoki Payme), 2% platforma komissiyasini avtomatik ajratish va kuryerlarni real vaqtda kuzatish (foydalanuvchi tomonidan faqat kuzatiladi, nazorat huquqi yo‘q).

---

## 2) Asosiy funksiyalar (Features)

* 4 ta asosiy rol:

  * **user** (mijoz)
  * **seller** (dorixona admini / sotuvchi)
  * **courier** (kuryer)
  * **owner** (platforma asosiy admini)
* Telefon orqali SMS kod bilan ro‘yxatdan o‘tish (OTP)
* JWT asosidagi autentifikatsiya (tokenlar)
* Kartaga online to‘lov: Click yoki Payme integratsiyasi
* To‘lovlarni avtomatik taqsimlash: **2% platforma komissiyasi**, qolgan **98%** sotuvchining hisobiga o‘tadi
* Savat (cart) → buyurtma (order) → to‘lov (payment) → yetkazib berish (delivery) jarayoni
* Kuryer real vaqtda GPS lokatsiyasi (faqat monitoring, foydalanuvchi uni nazorat qila olmaydi)
* Manzil maydonlari: lokatsiya (coordinate), uy raqami, pod’ekz (pod’ezd), qo‘shimcha eslatma
* Sharhlar va reytinglar (mijoz → kuryer, mijoz → sotuvchi)
* Seller panel: mahsulotlar (CRUD), statistika, balans, yechish tarixi
* Order va OrderPaymentInfo modellari alohida: to‘lov va buyurtma logikasi ajratilgan

---

## 3) Arxitektura (high-level)

```
+------------------+       +--------------------+      +----------------+
| dorihona_app     | <---> | dorihona_backend   | <--> | Postgres/DB    |
| (Flutter - user) |       | (Django + DRF)     |      +----------------+
+------------------+       | - JWT              |      | Redis (optional)|
                           | - Payments (Click) |      +----------------+
+------------------+       | - Orders, Payment  |
| curyers_app      | <---> +--------------------+
| (Flutter)        |
+------------------+

+------------------+
| dorihona_web     |
| (admin/marketing)
+------------------+
```

**Texnologiyalar (repo asosida):** Dart / Flutter (mobil ilovalar), HTML (veb), C++/CMake (yoki boshqa native qo‘shimchalar), Ruby/Swift qism (repo tili ko‘rsatkichlari). Repo rootida APK fayllar mavjud — `darmon.apk`, `app-debug.apk`. citeturn0view0

> Backend kodida `serializers.py`, `views.py`, `permissions.py` kabi fayllar mavjudligi Django REST Framework ishlatilganligini ko‘rsatadi — bu loyihaning arxitektura qarori bo‘lib, READMEda backend uchun Django+DRF asosida ko‘rsatmalar berilgan.

---

## 4) Repo tuzilmasi

(Asosiy papkalar — repoda mavjudligini tekshiring.)

```
Darmon/
├─ dorihona_app/      # Flutter - mijoz ilovasi
├─ dorihona_web/      # Veb frontend / admin
├─ dorihona_backend/  # API server (Django + DRF)
├─ curyers_app/       # Kuryerlar uchun Flutter ilova
├─ app_test/          # Test materiallar / debug
├─ darmon.apk         # Release/debug APK fayli (repo rootida)
├─ app-debug.apk      # Debug APK
└─ README.md          # (Siz hozir ko‘rayotgan hujjat)
```

Fayl va papkalar haqidagi asosiy dalillar repo sahifasida ko‘rsatilgan. citeturn0view0

---

## 5) Tez boshlash (Quickstart)

### Talablar (Prerequisites)

* Git
* Python 3.10+ (backend uchun)
* pip / virtualenv
* PostgreSQL (recommended) yoki SQLite (dev)
* Redis (realtime yoki Celery uchun, ixtiyoriy)
* Flutter SDK (2.10+ yoki oxirgi LTS, mobil ilovalar uchun)
* Android SDK (agar APK build qilmoqchi bo‘lsangiz)
* Node.js + npm yoki bundler — agar `dorihona_web` frontendida ishlatilgan bo‘lsa (lokal o‘rnatishni tekshiring)

> **Eslatma:** agar backend PHP yoki Node bo‘lsa, `dorihona_backend` papkasidagi `requirements.txt` yoki `package.json` ni tekshirib oling — quyidagi buyruqlar umumiy va ko‘pchilik Django+DRF bo‘yicha berilgan.

### 5.1 Backend (Django + DRF) — Lokal ishga tushirish

**1. Reponi klon qilish**

```bash
git clone https://github.com/JamesUilson/Darmon.git
cd Darmon/dorihona_backend
```

**2. Virtual muhiti va dependensiyalar**

```bash
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
# .venv\Scripts\activate  # Windows
pip install -r requirements.txt
```

**3. Environment (misol `.env`)**
`.env` faylida quyidagilar bo‘lishi kerak (namuna):

```ini
DEBUG=True
SECRET_KEY=replace_this_with_secure_key
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
DATABASE_URL=postgres://user:pass@localhost:5432/darmon_db
REDIS_URL=redis://localhost:6379/0
PAYME_MERCHANT_ID=your_payme_id
PAYME_SECRET=your_payme_secret
CLICK_ID=your_click_id
CLICK_SECRET=your_click_secret
STRIPE_SECRET=optional_if_used
```

> **Maxfiy ma’lumotlar** (API kalitlari, DB parollari) hech qachon public repoda saqlanmasin. Ishda GitHub Secrets yoki server muhit o‘zgaruvchilaridan foydalaning.

**4. Migratsiyalar va superuser**

```bash
python manage.py migrate
python manage.py createsuperuser
```

**5. Lokal serverni ishga tushirish**

```bash
python manage.py runserver 0.0.0.0:8000
# yoki gunicorn uchun (production-like):
# gunicorn projectname.wsgi:application --bind 0.0.0.0:8000
```

**6. Qo‘shimcha xizmatlar**

* Agar real-time (WebSocket) yoki kuryer lokatsiyasi uchun Channels ishlatilsa, Redis kerak bo‘ladi.
* Asenkron ishlar (to‘lov webhook handling, push notification queue) uchun Celery + Redis tavsiya etiladi.

### 5.2 Frontend — Flutter ilovalarni ishga tushirish

**Mijoz ilovasi (`dorihona_app`)**

```bash
cd ../../dorihona_app
flutter pub get
flutter run                # yoki --device-id bo‘lib telefon yoki emulatorni ko‘rsating
```

**Kuryer ilovasi (`curyers_app`)**

```bash
cd ../curyers_app
flutter pub get
flutter run
```

**Release build (APK)**

```bash
flutter build apk --release
# chiqadi: build/app/outputs/flutter-apk/app-release.apk
```

> API server URL (misol `API_BASE_URL`) ilova konfiguratsiyasida yoki `--dart-define` orqali berilishi mumkin, masalan:

```bash
flutter run --dart-define=API_BASE_URL=https://api.example.com
```

### 5.3 APK o‘rnatish (tez test uchun)

Repo rootida mavjud APK fayllarni qurilmangizga o‘rnatish mumkin:

* `darmon.apk` — release/debug build (repo root). citeturn0view0
* **Android sozlamalarida**: `Settings → Security → Unknown sources` ni yoqing (faqat test uchun)

---

## 6) Ma’lumotlar bazasi va model tuzilmasi (Data models)

Quyida loyihaning asosiy modellarini Django syntaxida namunaviy ko‘rinishda keltirdim — repo ichidagi `models.py` fayllariga moslashtiring.

**CustomUser (qisqacha)**

```python
from django.contrib.auth.models import AbstractUser
class CustomUser(AbstractUser):
    ROLE_CHOICES = (
        ('user','User'),
        ('seller','Seller'),
        ('courier','Courier'),
        ('owner','Owner'),
    )
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='user')
    phone = models.CharField(max_length=20, unique=True)
    # qo'shimcha maydonlar: wallet_balance, rating, etc.
```

**Product** (sotuvchi tomonidan kiritiladi)

```python
class Product(models.Model):
    seller = models.ForeignKey(CustomUser, related_name='products', on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    image = models.ImageField(upload_to='products/')
```

**Order va OrderItem**

```python
class Order(models.Model):
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    seller = models.ForeignKey(CustomUser, related_name='orders_as_seller', on_delete=models.CASCADE)
    status = models.CharField(choices=STATUS_CHOICES, default='pending')
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)

class OrderItem(models.Model):
    order = models.ForeignKey(Order, related_name='items', on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.SET_NULL, null=True)
    quantity = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)  # price at time of order
```

**OrderPaymentInfo** (alohida model — to‘lov logikasi uchun)

```python
class OrderPaymentInfo(models.Model):
    order = models.OneToOneField(Order, on_delete=models.CASCADE, related_name='payment_info')
    method = models.CharField(max_length=50)  # click, payme, cash
    transaction_id = models.CharField(max_length=255, null=True, blank=True)
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    commission = models.DecimalField(max_digits=10, decimal_places=2)  # 2% etc.
    seller_amount = models.DecimalField(max_digits=10, decimal_places=2)
    status = models.CharField(choices=PAYMENT_STATUS_CHOICES, default='pending')
```

**CourierLocation** (kuryerlar uchun real-time lokatsiya)

```python
class CourierLocation(models.Model):
    courier = models.ForeignKey(CustomUser, limit_choices_to={'role':'courier'}, on_delete=models.CASCADE)
    lat = models.DecimalField(max_digits=9, decimal_places=6)
    lng = models.DecimalField(max_digits=9, decimal_places=6)
    timestamp = models.DateTimeField(auto_now=True)
```

> Bu kod faqat namunadir — haqiqiy `models.py` fayllarida barcha kerakli fieldlar, indeklar va signallar qo‘shilishi kerak.

---

## 7) API hujjatlari & misollar (Endpoints)

Quyidagi API manzillar va javoblar uchun misollar keltirildi — `dorihona_backend` ichidagi `urls.py` va `views.py` asosida moslashtiring.

### Avtorizatsiya (SMS OTP)

**1) OTP yaratish**

```
POST /api/auth/send-otp/
Body: { "phone": "+998901234567" }
```

**2) OTP tekshirish va token olish**

```
POST /api/auth/verify-otp/
Body: { "phone": "+998901234567", "code": "1234" }
Response: { "access": "<jwt>", "refresh": "<jwt>" }
```

### Mahsulot va Savat

```
GET  /api/products/
POST /api/cart/        # savatga qo'shish
GET  /api/cart/
POST /api/checkout/    # buyurtma yaratish va to'lovni boshlash
```

### Buyurtma va to‘lov

```
GET  /api/orders/              # userga tegishli
GET  /api/orders/{id}/
POST /api/payments/create/     # to'lovni yaratish (Click/Payme)
POST /api/payments/webhook/    # to'lov provayderidan webhook
```

**Authorization**: `Authorization: Bearer <access_token>`

**Curl misol — buyurtma yaratish**

```bash
curl -X POST https://api.example.com/api/checkout/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"cart_id": 12, "address_id": 5, "payment_method": "click"}'
```

> Har bir endpoint uchun `swagger` yoki `redoc` qo‘shishni tavsiya qilamiz — bu developerlar uchun APIni tushunishni soddalashtiradi.

---

## 8) To‘lovlar va komissiya logikasi (Payments)

Platforma siyosatiga ko‘ra: to‘lov kelganda **2%** platforma komissiyasi olinadi va qolgan **98%** sotuvchining hisobiga o‘tkaziladi. Qo‘shimcha bank/seri provayder to‘lovlari bo‘lsa, ular alohida hisobga olinadi.

**Oddiy hisoblash**

```python
TOTAL = Decimal('100000')  # so'mda yoki mahalliy valyutada
PLATFORM_COMMISSION_RATE = Decimal('0.02')
commission = (TOTAL * PLATFORM_COMMISSION_RATE).quantize(Decimal('0.01'))
seller_amount = (TOTAL - commission).quantize(Decimal('0.01'))
```

**Jarayon**

1. Foydalanuvchi to‘lovni amalga oshiradi (Click/Payme).
2. To‘lov tasdiqlanganda `OrderPaymentInfo` yangilanadi (`status=paid`, `transaction_id` saqlanadi).
3. Orqaga chiqma (refund) yoki chargeback bo‘lsa, tegishli rollback logikasi ishlab chiqilishi kerak.
4. Pulni sotuvchiga avtomatik o‘tkazish uchun platforma bankoti yoki payout scripts ishlatiladi (bank API/Payme/Click saqlashlari bilan integratsiya qilinadi).

---

## 9) Rollar va ruxsatlar (Roles & Permissions)

Quyidagi ro‘yxat rol va ularning asosiy imkoniyatlari (permission matrix)ni ko‘rsatadi:

* **user**:

  * Mahsulotlarni ko‘rish, savatga qo‘shish, buyurtma berish, kuryerni kuzatish, sharh qoldirish
* **seller**:

  * Mahsulot CRUD, buyurtmalarni ko‘rish va status yangilash (tayyor, topshirildi), yechish so‘rovlari
* **courier**:

  * O‘ziga yuklangan buyurtmalarni qabul qilish, statuslarni yangilash (olindi, yetkazildi), GPS lokatsiyasini yuborish
* **owner**:

  * Barcha tizimni boshqarish: barcha foydalanuvchilar, buyurtmalar, moliyaviy hisobotlar, jarimalar va maoshlarni ko‘rish va tahrirlash

Backendda permissionlar `permissions.py` orqali aniq belgilanadi va `views.py`da qo‘llanadi.

---

## 10) Testlar va CI/CD

**Test yozish**

* Backend: `pytest` yoki Django `manage.py test`
* Frontend: Flutter unit va widget testlar

**GitHub Actions — minimal CI (misol)**

* Har bir PR uchun:

  * `flake8` (Python lint)
  * `pytest` (backend testlar)
  * `flutter analyze` va `flutter test`

**Namuna workflow** (yaxshi boshlang‘ich)

```yaml
name: CI
on: [push, pull_request]
jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - run: pip install -r dorihona_backend/requirements.txt
      - run: cd dorihona_backend && pytest
  flutter:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: 'stable'
      - run: cd dorihona_app && flutter pub get && flutter test
```

---

## 11) Deploy (production) tavsiyalari

**Backend**

* Dockerize qiling va `gunicorn` + `nginx` orqali reverse proxy qiling.
* HTTPS: Let's Encrypt / Certbot yordamida SSL o‘rnating.
* Ma’lumotlar bazasi: managed Postgres yoki RDS.
* Munosib monitoring: Sentry (error monitoring), Prometheus/Grafana (metrics).

**Ilovalar/Push notifications**

* Push notifications uchun Firebase Cloud Messaging (FCM) yoki APNs (iOS)
* To‘lov webhooklarini xavfsiz saqlang (secret token bilan tekshiring)

**Keystore**

* Android release uchun keystore fayllarini xavfsiz saqlang (GitHub Secrets yoki Vault)

---

## 12) Hissa qo‘shish (Contributing)

1. Reponi fork qiling.
2. Yangi branch yarating: `git checkout -b feature/feature-name`.
3. O‘zgartirish kiriting, test yozing.
4. PR yuboring, muhim o‘zgartirishlarda description va screenshot kiritishni unutmang.

**Kod stili**

* Backend: PEP8 (flake8)
* Flutter: `flutter analyze`, dartfmt

**Issue yaratish**

* Yangi feature yoki bug uchun issue oching va step-by-step reproducible example kiriting.

---

## 13) Litsenziya va aloqa

MIT License (default). Agar boshqa litsenziya kerak bo‘lsa (`Apache-2.0`, `GPL-3.0`) shu faylni yangilang.

Muallif: James Uilson (repo egasi). Aloqa: GitHub profil orqali yoki README ichida ko‘rsatilgan email.

---

## 14) Qo‘shimcha: roadmap & known issues

**Roadmap (tavsiyalar)**

* Push notificationlar (FCM integratsiyasi) — yuqori ustuvorlik
* Avtomatik payoutlar va moliyaviy hisobotlar (seller dashboard)
* Testlar va end-to-end testlar (Cypress yoki Appium)
* Analytics: ko‘rsatkichlar va funnel tracking

**Known issues**

* Mobil va web uchun versiyalar sinxronizatsiyasi (API versiyalarni boshqarish kerak)
* Ma’lumotlar migratsiyasi uchun alohida scriptlar yo‘q bo‘lishi mumkin (migrations tekshiring)

---

### Yakuniy so‘z

Men loyihaning papkalar tuzilmasini va mavjud APK fayllarni repodan tekshirdim (rootda `darmon.apk`, `app-debug.apk`, va `dorihona_*` papkalari mavjud). Har bir qismni batafsilroq hujjatlashtirishni xohlasangiz (masalan: `dorihona_backend` ichidagi `settings.py`, `Dockerfile`, `requirements.txt` yoki `dorihona_app` ichidagi `pubspec.yaml`ni asosida aniq sozlamalar yozish), shu papkalardagi fayllarni yo‘llab bering — men ularni o‘qib, README ni qonuniy va aniq parametrlar bilan yangilayman. citeturn0view0

---

> **Fayl:** `README.md` — to‘liq. Agar xohlasangiz men bu READMEni repo rootiga GitHub PR orqali ham joylab bera olaman (siz menga remote access yoki PR yaratish ruxsatini bersangiz).
