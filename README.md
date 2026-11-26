# 🎵 Spotify Tracks Dataset ile Popülerlik Skoru Tahmini (FET445)

**Veri Madenciliği (FET445)** dersi kapsamında hazırlanan bu projede, şarkıların dijital ve akustik ses özellikleri ile müzik türü kullanılarak Spotify üzerindeki güncel popülerlik skorunun (`popularity`) **regresyon** yöntemleriyle tahmin edilmesi amaçlanmaktadır.

---

## 👥 Ekip Bilgileri — SoundData

| Ad Soyad | Öğrenci Numarası | Sorumluluk Alanı |
| :--- | :--- | :--- |
| **Umut Torun** | `[Öğrenci No]` | Veri hazırlığı, EDA, Ridge Regresyon ve LightGBM Regressor |
| `[2. Üye Ad Soyad — varsa]` | `[Öğrenci No]` | Öznitelik mühendisliği, Random Forest Regressor ve hata analizi |

---

## 🎯 Araştırma Sorusu
> *"Bir şarkının dijital ve akustik ses özellikleri (dans edilebilirlik, enerji, ses yüksekliği, akustiklik, tempo vb.) ile müzik türü kullanılarak Spotify üzerindeki güncel popülerlik skoru ne doğrulukla tahmin edilebilir?"*

---

## 📊 Veri Seti Özeti
* **Kaynak:** [Spotify Tracks Dataset (Kaggle — Maharshi Pandya)](https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset)
* **Lisans:** CC0: Public Domain
* **Boyut:** 114.000 satır, 20 sütun (yinelenen `track_id` kayıtları tekilleştirildiğinde ~89.741 benzersiz parça)
* **Hedef Değişken (`popularity`):** `0` ile `100` arasında tam sayı popülerlik skoru

### Kullanılan Öznitelikler
* **Sayısal (10):** `duration_ms`, `danceability`, `energy`, `loudness`, `speechiness`, `acousticness`, `instrumentalness`, `liveness`, `valence`, `tempo`
* **Kategorik (3):** `track_genre`, `key`, `time_signature`
* **İkili (2):** `explicit`, `mode`
* **Çıkarılan Sütunlar:** `Unnamed: 0`, `track_id` (kimlik bilgisi), `track_name`, `album_name` (yüksek kardinaliteli serbest metin), `artists` (dolaylı veri sızıntısı ve yüksek kardinalite riski)

---

## ⚙️ Yöntem ve Modelleme Pipeline'ı
Veri sızıntısını önlemek amacıyla tüm ön işleme adımları `scikit-learn` `Pipeline` ve `ColumnTransformer` yapısı içinde yalnızca eğitim katmanlarına uygulanmaktadır:

1. **Veri Temizleme ve Ön İşleme:** Yinelenen `track_id` kayıtlarının tekilleştirilmesi, eksik değerlerin doldurulması (`SimpleImputer`), kategorik değişkenler için `OneHotEncoder` ve sayısal değişkenler için `StandardScaler`.
2. **Modeller:**
   * **Temel Model (Baseline):** `DummyRegressor` (Eğitim kümesi ortalaması)
   * **Doğrusal Model:** `Ridge Regression` (L2 düzenlileştirmesi)
   * **Ağaç Topluluğu (Bagging):** `Random Forest Regressor`
   * **Gradyan Artırma (Boosting):** `LightGBM Regressor`
3. **Değerlendirme Stratejisi:**
   * %80 Eğitim / %20 Test ayrımı (`random_state=42`)
   * Eğitim kümesi üzerinde **5-Katlı Çapraz Doğrulama (5-Fold CV)** ve `RandomizedSearchCV` ile hiperparametre optimizasyonu
   * **Metrikler:** `MAE`, `RMSE` ve $R^2$

---

## 📁 Depo Yapısı (Planlanan)

```text
FET445_SoundData_SpotifyTracksDataset/
├── docs/                  # Proje önerisi ve final raporu PDF dosyaları
├── notebooks/             # Keşifsel Veri Analizi (EDA) ve modelleme notebook'ları
├── src/                   # Veri ön işleme, eğitim ve değerlendirme Python betikleri
├── .gitignore             # Python ve veri dosyası filtreleri
└── README.md              # Proje dokümantasyonu
```

---

## 📚 Kaynakça
1. M. Pandya, *"Spotify Tracks Dataset,"* Kaggle, 2022. Erişim: https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset
2. F. Pedregosa vd., *"Scikit-learn: Machine Learning in Python,"* Journal of Machine Learning Research, c. 12, ss. 2825–2830, 2011.
3. G. Ke vd., *"LightGBM: A Highly Efficient Gradient Boosting Decision Tree,"* NeurIPS, c. 30, ss. 3146–3154, 2017.
