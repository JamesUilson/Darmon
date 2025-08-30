# Darmon

**Dorixonalar uchun buyurtma va yetkazib berish platformasi**

---

## 1. Umumiy ma'lumot

Darmon — dorixonalar va mijozlar o'rtasida buyurtma, yetkazib berish va to'lovlarni boshqarish uchun mo'ljallangan tizim. Loyihaning tarkibiga mijoz ilovasi, sotuvchi (seller) paneli, kuryer ilovasi va REST API (backend) kiradi.

---

## 2. Komponentlar

* **Mobile (customer)** — mijozlar uchun Flutter ilovasi
* **Mobile (courier)** — kuryerlar uchun Flutter ilovasi
* **Seller panel** — sotuvchilar/panel admini uchun interfeys
* **Backend API** — autentifikatsiya, buyurtmalar, to'lovlar va ma'lumotlarni boshqarish
* **Admin / Web** — tizim ma'muriyati va monitoringi

---

## 3. Texnologiya (qisqacha)

* Flutter / Dart — mobil ilovalar
* Python + Django (+DRF) — REST API
* RDBMS (Postgres yoki SQLite) — ma'lumotlar ombori
* Redis — ixtiyoriy (kesh/queue uchun)

---

## 4. Tez boshlash (Quickstart)

### Talablar

* Git
* Python 3.10+
* Flutter SDK
* Android SDK (emulator yoki qurilma)
* PostgreSQL (yoki SQLite)

### Reponi klon qilish

```bash
git clone https://github.com/JamesUilson/Darmon.git
cd Darmon
```

### Backend (lokal)

```bash
cd dorihona_backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv/Scripts/activate
pip install -r requirements.txt
```

**Konfiguratsiya**

* Loyihada .env yoki muqobil muhit o'zgaruvchilari orqali sozlash ishlatiladi.
* Kerakli o'zgaruvchilar: server URL, ma'lumotlar bazasi ulanishi, maxfiy kalitlar, to'lov provayder credentiallari (real qiymatlarni README ichida ko'rsatilmang).

**Migratsiyalar va superuser**

```bash
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:8000
```

### Mobile ilovalar (Flutter)

Mijoz ilovasi va kuryer ilovasi dorihona\_app va curyers\_app papkalarida joylashgan bo'lishi mumkin.

```bash
cd ../../dorihona_app
flutter pub get
flutter run --dart-define=API_BASE_URL=http://127.0.0.1:8000
```

Yoki kuryer ilovasini ishga tushirish:

```bash
cd ../curyers_app
flutter pub get
flutter run --dart-define=API_BASE_URL=http://127.0.0.1:8000
```

**Release APK**

```bash
flutter build apk --release
# natija: build/app/outputs/flutter-apk/app-release.apk
```

---

## 5. Konfiguratsiya va maxfiy ma'lumotlar

* Maxfiy ma'lumotlar (API kalitlari, DB parollari, to'lov provayder credentiallari) faqat muhit o'zgaruvchilari yoki server secrets orqali saqlansin.
* REPO ichida haqiqiy credential yozilmasin.

---

## 6. API (qisqacha)

* Backend REST API orqali xizmatlar taqdim etiladi.
* Autentifikatsiya tokenlar orqali amalga oshiriladi.
* To'lov provayderi credentiallari muhit o'zgaruvchilari orqali sozlanadi.

---

## 7. Test va CI

* Backend uchun unit testlar va migratsiyalar ishlatiladi.
* Flutter ilovalar uchun unit va widget testlar mavjud bo'lishi mumkin.

---

## 8. Contributing (qisqacha)

1. Fork qiling.
2. Yangi branch yarating: git checkout -b feature/your-feature.
3. O'zgartirish kiriting, commit qiling va PR yuboring.

---

## 9. Litsenziya

Loyiha litsenziyasi LICENSE faylida ko'rsatilgan.

---

## 10. Aloqa

Repo muallifi va texnik aloqa: GitHub repozitoriyasi orqali.

---

*Ushbu README loyihaning umumiy, texnik hujjati sifatida yozildi — ichida tijorat sirlariga oid batafsil ish mantig‘i yoki maxfiy credentiallar keltirilmagan.*
