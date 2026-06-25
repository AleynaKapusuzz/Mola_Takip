# 🎯 Mola Takip Sistemi

📋 Proje Özeti

Mola Takip Sistemi, çalışanların giriş-çıkış kartını okuyarak çalışma süresi, mola süresi ve mesai saatlerini otomatik olarak hesaplayan bir web uygulamasıdır. SQL Server veritabanından terminal kayıtlarını çekerek, vardiya bilgilerine göre iş gücü yönetimini sağlar.

---

## 🛠️ Teknoloji Stack 

| Teknoloji | Amaç |
|-----------|------|
| **FastAPI** | REST API oluşturma, endpoint yönetimi |
| **SQLAlchemy + Pandas** | SQL Server ile veri iletişimi ve veri işleme |
| **Uvicorn** | ASGI sunucusu (API hosting) |
| **Python 3.9+** | Backend geliştirme |
| **HTML/CSS/JavaScript** | Web arayüzü ve kullanıcı etkileşimi |
| **NSSM** | Windows hizmet olarak uygulama çalıştırma |

---

## 🌐 Erişim Linki

```
http://yapayzeka:8501
```

## 🔍 Sistem Hakkında

### Ne Yapılıyor?

```
Meyer Turnike Sistemi
        ↓
    [Pool Tablosu]
    (giriş/çıkış kayıtları)
        ↓
[Mola Takip - Backend]
  - Veri işleme
  - Çift okutma tespiti
  - Mola hesaplaması
  - Anomali analizi
        ↓
[Web Arayüzü + Excel]
  - Dashboard
  - Detaylı raporlar
  - Admin paneli
```

---

## 🔄 Proje Mantığı

### Veri Akışı

```
SQL Server (Terminal Kayıtları)
         ↓
   database.py (Sorgu ve Çekme)
         ↓
   models.py (Vardiya Tespiti & Mola Hesaplama)
         ↓
   hesaplama.py (Anomali Tespiti & Çift Okutma Kontrolü)
         ↓
   FastAPI (API Endpoint'leri)
         ↓
   HTML/JavaScript (Web Arayüzü)
```

### Temel İş Akışı

1. **Veri Çekme**: Belirtilen tarih aralığında terminal 1013, 1014, 1015 vb. numaralandırılmış kapılardan giriş-çıkış kayıtlarını çeker
2. **Vardiya Tespiti**: Giriş saatine göre çalışanın hangi vardiyada (Gündüz/Ara/Gece) olduğunu belirler
3. **Mola Hesabı**: Vardiya ve göreve (bakım/elektrik/normal) göre molaya hak olan süreyi hesaplar
4. **Anomali Tespiti**: Çift okutma (4 saniye içinde aynı tip iki kayıt) ve ek çalışmalar tespit eder
5. **Sonuç**: Personelin günlük çalışma, mola ve mesai sürelerini sunara gönderir

---



---


## 📁 Proje Yapısı

```
mola-takip-sistemi/
│
├── app.py                      ← FastAPI uygulaması (endpoints)
├── hesaplama.py                ← Mola hesaplama algoritması
├── models.py                   ← Veri modelleri ve vardiya tanımları
├── config.py                   ← Ayarlar ve DB bağlantısı
├── database.py                 ← Veritabanı sorgulama
├── service_mola_takip.py       ← NSSM servisi başlatıcı
│
├── static/
│   ├── index.html              ← Web arayüzü (HTML)
│   └── app.js                  ← Frontend JavaScript
│
├── data/
│   ├── users.json              ← Kullanıcı verileri
│   ├── db.json                 ← Oturum ve istatistik verileri
│   └── log_mola.txt            ← Uygulama logları
│
├── .env                        ← Ortam değişkenleri (GİTİGNORE'da)
├── .gitignore                  ← Git ignore kuralları
│
└── requirements.txt            ← Python bağımlılıkları
```

---

## 🔄 Mimarı ve İş Akışı

### Veri Akış Diyagramı

```
┌─────────────────────────────────────────────────────────────────┐
│                      SQL SERVER (Meyer)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Pool Tablosu│  │Sicil Tablosu │  │cbo_Gorev Tab.│          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    ODBC Driver 17
                           │
        ┌──────────────────▼──────────────────┐
        │    database.py (fetch_raw_records)   │
        │  - 10 dakika caching                │
        │  - Paralel bağlantı havuzu           │
        └──────────────────┬──────────────────┘
                           │
                    Ham giriş/çıkış
                     kayıtları (DataFrame)
                           │
        ┌──────────────────▼──────────────────┐
        │  hesaplama.py (ham_veri_isle)        │
        │  - Çift okutma tespiti               │
        │  - Geri okutma temizliği             │
        │  - Çalışma/mola periyodları          │
        │  - Gece vardiyası düzeltmesi         │
        └──────────────────┬──────────────────┘
                           │
                    İşlenmişveri (DataFrame)
                    (çalışma/mola sure'leri)
                           │
        ┌──────────────────▼──────────────────┐
        │ hesaplama.py (mola_hesapla)          │
        │  - Vardiya tespiti                   │
        │  - Mola limiti hesaplama             │
        │  - Anomali tespiti                   │
        │  - İstatistik & Model               │
        └──────────────────┬──────────────────┘
                           │
                    MolaSonucu nesneleri
                           │
        ┌──────────────────▼──────────────────┐
        │     app.py (API endpoints)           │
        │  - /analiz (JSON)                    │
        │  - /analiz/excel (XLSX özet)         │
        │  - /analiz/excel-detay (XLSX detay)  │
        └──────────────────┬──────────────────┘
                           │
        ┌──────────────────▼──────────────────┐
        │     Frontend (app.js + HTML)         │
        │  - Dashboard & tablolar              │
        │  - Filtreler ve arama                │
        │  - İndir düğmeleri                   │
        └──────────────────────────────────────┘
```


---
## 💻 Kurulum

### Ön Koşullar

- **Python 3.9+** (3.10+ önerilir)
- **SQL Server** 2016 veya üzeri
- **ODBC Driver 17 for SQL Server**
- **Windows** işletim sistemi (NSSM için)

### 1. Depo İndir

```bash
git clone https://github.com/yourusername/mola-takip-sistemi.git
cd mola-takip-sistemi
```

### 2. Python Ortamını Kur

```bash
# Virtual environment oluştur
python -m venv venv

# Ortamı aktifleştir (Windows)
venv\Scripts\activate

# Ortamı aktifleştir (Linux/Mac)
source venv/bin/activate
```

### 3. Bağımlılıkları Yükle

```bash
pip install -r requirements.txt
```

### 4. .env Dosyasını Oluştur

`.env` dosyasını proje root'unda oluştur ve SQL Server bilgilerini gir:

```env
DB_SERVER=GEDIKDB\GDK
DB_PORT=1433
DB_NAME=GEDIKPILIC14930_Meyer
DB_USER=your_username
DB_PASSWORD=your_password
API_HOST=0.0.0.0
API_PORT=8001
APP_ENV=production
```

### 5. Uygulamayı Çalıştır

**Geliştirme ortamında:**
```bash
python app.py
```

Ardından tarayıcıda açın: **http://localhost:8501**

---

## ⚙️ Konfigürasyon

### Vardiya Tanımları (`models.py`)

```python
VARDIYALAR = [
    {
        "ad": "Gündüz",
        "baslangic": time(8, 0),
        "bitis": time(16, 10),
        "mesai_esigi": time(16, 40)
    },
    {
        "ad": "Ara",
        "baslangic": time(16, 0),
        "bitis": time(0, 30),
        "mesai_esigi": time(1, 0)
    },
    {
        "ad": "Gece",
        "baslangic": time(0, 0),
        "bitis": time(8, 0),
        "mesai_esigi": time(8, 30)
    }
]
```

### Mola Limitleri (`models.py`)

| Görev Tipi | Normal Çıkış | Mesai Yapan |
|-----------|---------|-----------|
| Normal personel | 45 dakika | 60 dakika |
| Bakım/Elektrik | 60 dakika | 90 dakika |

### Bakım/Elektrik Görevleri

`models.py` içinde `BAKIM_ELEKTRIK_GOREVLER` listesine eklenen görevler farklı mola limitine tabidir.

### Çift Okutma Eşiği

```python
CIFT_OKUTMA_ESIGI_DK: float = 4 / 60  # 4 saniye
```

Aynı tipte (giriş veya çıkış) kayıtlar 4 saniye içinde gelirse çift okutma olarak işaretlenir.

---

### Mola Hesaplama Algoritması

```python
# Giriş: PersonelID, Görev, Vardiya, Çalışma Süresi, Mola Süresi

1. Vardiya Bul
   • Giriş saatine göre (Gündüz, Ara, Gece)
   
2. Mola Limitini Hesapla
   • Vardiya + Mesai Durumu + Görev Tipi
   • Normal çıkış → 45 dk (Bakım: 60 dk)
   • Mesai → 60 dk (Bakım: 90 dk)
   
3. Asımı Hesapla
   • Asım = max(0, MolaSüresi - Limit)
   
4. Anomali Tespit Et
   • Çift okutma: aynı tip iki kayıt 4 saniye içinde
   • Mola > Çalışma ve Mola > Limit × 3
   • Çalışma yok ama mola var
   
5. Sonuç
   • MolaSonucu object
     - ad_soyad, gorev, tarih, vardiya
     - calisma_dk, mola_dk, asim_dk
     - anomali, cift_okutma, hareketler
```

### Çift Okutma Tespiti

Aynı kişinin aynı günde, **aynı tipte** (giriş veya çıkış) iki kayıdı 4 saniye içinde gelirse:

```
09:00:00 [Giriş]  ✓ Normal
09:00:02 [Giriş]  ← Çift Okutma! (2 saniye fark)
         ↓ Sistem: 2. kaydı atar ve işaretler
         
09:30:00 [Çıkış]  ✓ Normal
15:00:00 [Giriş]  ✓ Normal
15:00:05 [Giriş]  ← Çift Okutma! (5 saniye > 4 sn, ama göz at?)
```

---


---

## 📊 Kullanım Örnekleri

### Örnek 1: 1 Aylık Analiz

1. Uygulamaya admin/admin123 ile giriş yap
2. Başlangıç: 2025-02-01
3. Bitiş: 2025-02-28
4. Görevler: (boş bırak = tümü)
5. **Analiz Yap** butonuna tıkla
6. Sonuçları tablo'da görüntüle

**Beklenen Sonuçlar:**
- Yeşil: Normal (asım yok)
- Sarı: Asım var
- Mor: Çift okutma var
- Kırmızı: Anomali var

### Örnek 2: Bakım Görevleri Filtrele

1. Analiz formunda görevler seç: "BAKIM", "ELEKTRİKÇİ"
2. Analiz Yap
3. Sadece bakım personelinin mola özeti gösterilir
4. Not: Bakım görevlileri normal personelden daha fazla mola (60/90 dk) alırlar

### Örnek 3: Özet Excel İndir

1. Analiz Yap
2. **Özet Excel İndir** butonuna tıkla
3. `mola_ozet_2025-02-01_2025-02-28.xlsx` indirilir

**İçerik:**
- Her kişi x 1 satır
- Özet istatistikler (toplam çalışma, mola, asım)
- Renk kodlama (Asım=Sarı, Çift=Mor, Anomali=Kırmızı)

### Örnek 4: Detaylı Excel İndir

1. Analiz Yap
2. **Detaylı Excel İndir** butonuna tıkla
3. `mola_detay_2025-02-01_2025-02-28.xlsx` indirilir

**İçerik:**
- Her kişi x Tüm çalışma/mola periyodları
- Sütunlar: Ad | Soyad | Tarih | Giriş | Çıkış | Çalışma | Ara | Görev
- İK tarafından kullanıma uygun format

### Örnek 5: Admin Paneli - Yeni Kullanıcı Ekle

1. Admin giriş yap
2. Admin Panel'i aç
3. **Yeni Kullanıcı** formunu doldur:
   - Username: ali
   - Şifre: 1234
   - Görüntü Adı: Ali Kaya
   - Role: user
4. **Ekle** butonuna tıkla
5. Yeni kullanıcı tabloya eklenir

### Örnek 6: Admin Paneli - Oturumu Sonlandır

1. Admin Panel'de **Oturum Yönetimi** sekmesine git
2. Çıkarmak istediğin kullanıcıyı bul
3. **Çıkış Yap** butonuna tıkla
4. Kullanıcı anlık 401 hatası alır ve login ekranına döner

---
## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

**Son Güncelleme:** Şubat 2025  
**Sürüm:** 1.0.0  
**Durum:** Production Ready ✅
