# Mola_Takip

# 🎯 Mola Takip Sistemi

Üretim tesisinde çalışan personelin **mola ve ara sürelerini** otomatik olarak takip eden, analiz eden ve raporlayan akıllı bir sistemdir.

> **Amaç**: Meyer turnike sisteminizden gelen ham giriş/çıkış kayıtlarını işleyip, mola limitlerini hesaplamak, anomalileri tespit etmek ve detaylı Excel raporları oluşturmak.

---

## 📋 İçindekiler

- [Sistem Hakkında](#sistem-hakkında)
- [Teknoloji Yığını](#teknoloji-yığını)
- [Kurulum](#kurulum)
- [Konfigürasyon](#konfigürasyon)
- [Proje Yapısı](#proje-yapısı)
- [Mimarı ve İş Akışı](#mimarı-ve-iş-akışı)
- [Dosya Açıklamaları](#dosya-açıklamaları)
- [Kullanım Örnekleri](#kullanım-örnekleri)

---

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

### Temel Özellikleri

| Özellik | Açıklama |
|---------|----------|
| **Otomatik Veri Çekme** | SQL Server'dan Meyer Pool tablosundan kayıtları sorgular |
| **Mola Hesaplaması** | Vardiya, görev, çalışma saati bazında mola limitlerini dinamik hesaplar |
| **Çift Okutma Tespiti** | Aynı anda iki kez okutmayı (4 saniye içinde) algılar ve işaretler |
| **Anomali Analizi** | Anormal mola kullanımını ve tutarsızlıkları otomatik bulur |
| **Excel Raporları** | Özet ve detaylı formatda indirilebilir raporlar |
| **Kullanıcı Yönetimi** | Admin paneli ile kullanıcı, şifre ve oturum yönetimi |
| **Performans Optimizasyonu** | 1 aylık veri ~30x daha hızlı işlenir (caching + groupby) |

---

## 🛠️ Teknoloji Yığını

### Backend
- **FastAPI** - Modern Python web framework
- **SQLAlchemy** - ORM ve SQL sorgulaması
- **Pandas** - Veri işleme ve analiz
- **openpyxl** - Excel dosyası oluşturma
- **Pydantic** - Veri validasyonu

### Veritabanı
- **SQL Server** - Microsoft SQL Server (Meyer sistem)
- **ODBC Driver 17** - Bağlantı sürücüsü

### Frontend
- **HTML5 / CSS3** - Arayüz yapısı
- **JavaScript (Vanilla)** - Dinamik işlemler
- **Chart.js** - İstatistik grafikleri

### Deployment
- **NSSM** - Windows servisi olarak çalıştırma
- **Uvicorn** - ASGI sunucusu

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

**requirements.txt** (örnek):
```
fastapi==0.104.1
sqlalchemy==2.0.23
pandas==2.1.3
openpyxl==3.1.2
uvicorn==0.24.0
pydantic==2.5.0
pydantic-settings==2.1.0
scikit-learn==1.3.2
pyodbc==4.0.35
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

**Windows servisi olarak (NSSM ile):**
```bash
# Servisi kur
nssm install "Mola Takip" "C:\path\to\python.exe" "C:\path\to\service_mola_takip.py"

# Servisi başlat
nssm start "Mola Takip"
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

## 🔌 API Endpoints

### Kimlik Doğrulama

#### POST `/api/login`
```json
{
  "username": "admin",
  "password": "admin123"
}
```
**Yanıt:**
```json
{
  "token": "eyJ0...",
  "display_name": "Sistem Yöneticisi",
  "role": "admin"
}
```

#### POST `/api/logout`
```
Header: Authorization: Bearer {token}
```

---

### Analiz Endpoints

#### POST `/gorevler`
Belirtilen tarih aralığında tüm görev isimlerini döner (analiz yapılmaksızın).

**İstek:**
```json
{
  "baslangic_tarih": "2025-02-01",
  "bitis_tarih": "2025-02-28"
}
```

**Yanıt:**
```json
["GENEL OPERATÖR", "BAKIM", "SÜRÜCÜ", ...]
```

#### POST `/analiz`
Mola analizi yaparak detaylı sonuçları döner.

**İstek:**
```json
{
  "baslangic_tarih": "2025-02-01",
  "bitis_tarih": "2025-02-28",
  "secili_gorevler": ["GENEL OPERATÖR", "BAKIM"]
}
```

**Yanıt:**
```json
[
  {
    "ad_soyad": "AHMET YILMAZ",
    "gorev": "GENEL OPERATÖR",
    "tarih": "2025-02-01",
    "vardiya": "Gündüz",
    "calisma_dk": 480,
    "mola_dk": 45,
    "izin_verilen_mola_dk": 45,
    "asim_dk": 0,
    "anomali": false,
    "cift_okutma": false,
    "cift_okutma_listesi": [],
    "hareketler": [
      {"zaman": "08:00", "tip": "giris", "sure": 480, "stip": "calisma", "cift": false},
      {"zaman": "12:00", "tip": "cikis", "sure": 45, "stip": "mola", "cift": false},
      ...
    ]
  },
  ...
]
```

#### POST `/analiz/excel`
Özet Excel dosyası indir (kişi başına bir satır).

**Yanıt:** `mola_ozet_2025-02-01_2025-02-28.xlsx`

#### POST `/analiz/excel-detay`
Detaylı Excel dosyası indir (İK formatı).

**Yanıt:** `mola_detay_2025-02-01_2025-02-28.xlsx`

---

### Admin Endpoints

#### GET `/api/admin/status`
Sistem durumunu kontrol et.

**Yanıt:**
```json
{
  "ok": true,
  "api": "aktif",
  "database": "bağlı",
  "user_count": 5,
  "session_count": 2
}
```

#### GET `/api/admin/users`
Tüm kullanıcıları listele (admin).

#### POST `/api/admin/users`
Yeni kullanıcı oluştur (admin).

```json
{
  "username": "yeni_user",
  "password": "1234",
  "display_name": "Yeni Kullanıcı",
  "role": "user"
}
```

#### PUT `/api/admin/users/{username}/password`
Kullanıcı şifresini güncelle (admin).

#### DELETE `/api/admin/users/{username}`
Kullanıcı sil (admin).

#### DELETE `/api/admin/active_sessions/{username}`
Kullanıcıyı çıkış yap (oturumunu sonlandır).

#### GET `/api/admin/session_logs`
Oturum loglarını görüntüle (admin).

#### GET `/api/admin/logs`
Uygulama loglarını görüntüle (admin).

#### GET `/saglik`
Sağlık kontrol (health check).

**Yanıt:**
```json
{
  "durum": "aktif",
  "veritabani": "bağlı"
}
```

---

## 📄 Dosya Açıklamaları

### 1. **app.py** (FastAPI Uygulaması)
**Görev:** Ana uygulama, tüm API endpoints'leri ve kimlik doğrulamayı yönetir.

**Ana Bölümler:**

| Bölüm | Görev |
|-------|-------|
| **Auth & Session** | Kullanıcı oturum yönetimi, token kontrol |
| **Admin Fonksiyon.** | Kullanıcı oluşturma, silme, şifre değiştirme |
| **Analiz Endpoints** | `/analiz`, `/gorevler`, Excel indir |
| **Frontend Dosyaları** | `static/` klasöründen HTML/JS sunma |

**Önemli Fonksiyonlar:**
- `login_user()` - Kimlik kontrolü ve token oluşturma
- `get_current_user()` - Aktif oturumu doğrula
- `analiz_yap()` - Mola analizi yap
- `analiz_excel_ozet()` - Özet Excel döndür
- `analiz_excel_detay()` - Detaylı Excel döndür

---

### 2. **hesaplama.py** (Mola Hesaplama Motoru)
**Görev:** Ham veri işlemesi, mola hesapları ve raporlama.

**Ana Fonksiyonlar:**

| Fonksiyon | Amacı |
|-----------|-------|
| `cift_okutma_tespit()` | Aynı günde, aynı tipte, 4 sn içindeki iki kaydı bulur |
| `ham_veri_isle()` | Meyer raw kayıtlarını çalışma/mola periyotlarına dönüştürür |
| `mola_hesapla()` | Kişi+görev+gün bazında mola özetini hesaplar |
| `excel_hazirla()` | Özet Excel raporu oluşturur |
| `excel_hazirla_detay()` | Detaylı Excel raporu oluşturur |
| `hareket_listesi_olustur()` | Bir kişinin giriş/çıkış timeline'ını oluşturur |

**Algoritma Özeti:**
1. Ham kayıtları tarih + kişi bazında grupla
2. Çift okutmaları tespit et ve işaretle
3. Art arda aynı tipte kayıtları temizle
4. Ardışık kayıt çiftlerinden periyotlar oluştur (Giriş→Çıkış=çalışma, Çıkış→Giriş=mola)
5. Vardiya belirle
6. Mola limitini hesapla
7. Anomali tespit et
8. Sonuçları birleştir

---

### 3. **models.py** (Veri Modelleri & Kurallar)
**Görev:** Vardiya tanımları, mola limitleri, veri yapıları.

**Önemli Tanımlar:**

| Tanım | Değer | Açıklama |
|-------|-------|----------|
| `VARDIYALAR` | list | Gündüz, Ara, Gece vardiyası tanımları |
| `MOLA_LIMIT_NORMAL` | 45 dk | Normal personel mola limiti |
| `MOLA_LIMIT_MESAI` | 60 dk | Mesai yapan personel mola limiti |
| `BAKIM_ELEKTRIK_GOREVLER` | list | Bakım/Elektrik görev listeleri |
| `CIFT_OKUTMA_ESIGI_DK` | 4 sn | Çift okutma tespiti için zaman farkı |

**Veri Sınıfları:**
- `MolaSonucu` - Kişi başına mola sonucu
- `CiftOkutma` - Çift okutma kaydı
- `HareketKaydi` - Tek bir giriş/çıkış hareketi

**Önemli Fonksiyonlar:**
- `vardiya_bul()` - Giriş saatine göre vardiya belirler
- `mola_limiti_hesapla()` - Vardiya ve çıkış saatine göre limit hesaplar
- `is_bakim_elektrik()` - Görevin bakım/elektrik kategorisinde olup olmadığını kontrol eder

---

### 4. **config.py** (Ayarlar & Bağlantı)
**Görev:** Ortam değişkenlerini oku, DB bağlantı URL'si oluştur.

**Önemli Sınıflar:**
- `Settings` - Pydantic BaseSettings, `.env` dosyasından okunur

**Bağlantı Bilgileri:**
```
DB_SERVER    - SQL Server adı/adresi
DB_PORT      - Port (varsayılan: 1433)
DB_NAME      - Veritabanı adı
DB_USER      - Kullanıcı adı
DB_PASSWORD  - Şifre
APP_ENV      - Ortam (production/development)
API_HOST     - API dinleme adresi
API_PORT     - API dinleme portu
```

---

### 5. **database.py** (Veri Tabanı Sorgulama)
**Görev:** Meyer tablosundan ham giriş/çıkış kayıtlarını çek, cache'le.

**Önemli Fonksiyonlar:**

| Fonksiyon | Amacı |
|-----------|-------|
| `get_engine()` | SQLAlchemy engine'i al (singleton) |
| `test_connection()` | DB bağlantısını test et |
| `fetch_raw_records()` | Tarih aralığında kayıtları çek |

**Cache Mekanizması:**
- TTL: 10 dakika
- Anahtar: `{start_date}_{end_date}`
- Maksimum 20 sorgu hafızada tutulur
- Daha eski sorgular otomatik silinir

**SQL Sorgusu:**
```sql
SELECT
    p.SicilID, s.Ad, s.Soyad, g.Ad AS Gorev,
    p.TerminalID, p.EventTime
FROM Pool p
INNER JOIN Sicil s ON s.ID = p.SicilID
LEFT JOIN cbo_Gorev g ON g.ID = s.Gorev
WHERE
    p.EventTime >= CAST(:start_date AS DATETIME)
    AND p.EventTime < DATEADD(day, 1, CAST(:end_date AS DATETIME))
    AND p.TerminalID IN (1013,1014,1015,1016,1043,1044,1045,1046,1047)
    AND (p.Deleted IS NULL OR p.Deleted = 0)
ORDER BY p.SicilID, p.EventTime
```

---

### 6. **app.js** (Frontend JavaScript)
**Görev:** Tarayıcıda UI mantığı, API iletişimi, etkileşim işlemleri.

**Ana Bölümler:**

| Bölüm | Görev |
|-------|-------|
| **Kimlik Doğrulama** | Giriş, çıkış, oturum kontrolü |
| **Analiz İşlemleri** | Tarih seçimi, filtreler, API çağrıları |
| **Tablo Görüntüsü** | Sonuçları tablo olarak göster |
| **İndir** | Excel özet/detay indir |
| **Admin Paneli** | Kullanıcı yönetimi, loglar |
| **Renkli İşaretleme** | Asım (sarı), Çift Okutma (mor), Anomali (kırmızı) |

**Önemli Fonksiyonlar:**
- `doLogin()` - Login formundan giriş yap
- `doLogout()` - Çıkış yap
- `analiz_yap()` - API'ye analiz isteği gönder
- `tablo_guncelle()` - Sonuçları tabloya ekle
- `excel_indir()` - Excel dosyası indir

---

### 7. **index.html** (Frontend HTML)
**Görev:** Web arayüzü yapısı, formlar, tablolar, admin paneli.

**Bölümleri:**

| Bölüm | İçerik |
|-------|--------|
| **Login Screen** | Kullanıcı/şifre giriş formu |
| **Analysis Form** | Tarih aralığı, görev filtreleri, analiz düğmesi |
| **Results Table** | Mola sonuçları tablosu, sıralama, filtreleme |
| **Admin Panel** | Kullanıcı listesi, oturum yönetimi, loglar |
| **Detail Modal** | Kişinin gün bazında hareketleri |

**Tasarım:**
- Responsive (mobil uyumlu)
- Koyu tema
- Kolay navigasyon

---

### 8. **service_mola_takip.py** (Windows Servisi)
**Görev:** NSSM tarafından çalıştırılan servisi başlat.

**Yapı:**
```python
def main():
    uvicorn.run(
        "app:app",
        host="0.0.0.0",
        port=8501,
        reload=False,
        workers=1,
    )

if __name__ == "__main__":
    main()
```

**Kullanım (NSSM ile):**
```bash
nssm install "Mola Takip" "C:\path\to\python.exe" "service_mola_takip.py"
nssm start "Mola Takip"
nssm stop "Mola Takip"
```

---

### 9. **.env** (Ortam Değişkenleri)
**Görev:** Duyarlı ayarları depolamak.

**Örnek:**
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

> ⚠️ **GIT'E EKLEMEYİN!** `.gitignore` dosyasına eklenmelidir.

---

### 10. **.gitignore** (Git Ignore Kuralları)
**Görev:** Depo'ya eklenmeyecek dosyaları belirtmek.

**Örnek:**
```
.env
venv/
__pycache__/
*.pyc
*.xlsx
data/
.DS_Store
```

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

## 🚀 Performans İpuçları

### 1 Aylık Veri (~30,000 satır)
- **Önceki:** ~30 saniye (tüm veriyi her işlemde tarıyor)
- **Şimdi:** ~1 saniye (groupby + caching ile)

### Optimizasyonlar Yapıldı:

1. **GroupBy Caching** (`app.py`, line 717)
   ```python
   df_ham_gruplari = {
       pid: grup.reset_index(drop=True)
       for pid, grup in df_ham.groupby("PersonelID")
   }
   ```

2. **DB Caching** (`database.py`)
   ```python
   _cache[cache_key] = (df.copy(), now)  # 10 dakika TTL
   ```

3. **Connection Pooling** (`database.py`)
   ```python
   pool_size=10, max_overflow=20, pool_timeout=60
   ```

---

## 🐛 Sorun Giderme

### 1. "Veritabanı bağlantı hatası"

**Çözüm:**
- `.env` dosyasındaki DB bilgilerini kontrol et
- SQL Server'ın çalışmakta olduğundan emin ol
- ODBC Driver 17'nin yüklü olduğundan emin ol:
  ```bash
  odbcinst -j  # Linux
  ```

### 2. "Oturum süresi doldu"

**Çözüm:**
- Token'ın 30 dakika geçerlidir
- Tekrar giriş yap
- `/api/admin/sessions/{username}` ile oturumu temizle

### 3. "Kayıt bulunamadı"

**Çözüm:**
- Tarih aralığını kontrol et
- Görevlerin uygun olduğundan emin ol
- Meyer Pool tablosunda o tarihlerde kayıtlar olduğundan emin ol

### 4. "Excel dosyası oluşturulamadı"

**Çözüm:**
- `/tmp` klasöründe yazma izni ol
- Disk alanı yeterli ol
- openpyxl ve pandas güncel olduğundan emin ol

---

## 📞 İletişim & Destek

- **GitHub Issues:** [Hata bildir](https://github.com/yourusername/mola-takip-sistemi/issues)
- **Email:** your.email@example.com
- **Telefon:** +90 123 456 7890

---

## 📜 Lisans

Bu proje **MIT Lisansı** altında dağıtılmaktadır.

Daha fazla bilgi için `LICENSE` dosyasına bakın.

---

## 🙏 Teşekkürler

- Meyer Turnike Sistemi
- FastAPI Topluluğu
- Pandas Geliştiricileri
- Açık kaynaklı tüm katkıda bulunanlar

---

**Son Güncelleme:** Şubat 2025  
**Sürüm:** 1.0.0  
**Durum:** Production Ready ✅
