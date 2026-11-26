# 🎵 Spotify Tracks Dataset ile Popülerlik Skoru Tahmini (FET445)

Veri Madenciliği (FET445) dersi kapsamında hazırlanan bu projede, şarkıların dijital ve akustik ses özellikleri ile müzik türü kullanılarak Spotify üzerindeki güncel popülerlik skorunun (`popularity`) **regresyon** yöntemleriyle tahmin edilmesi amaçlanmaktadır.

📄 **Proje önerisi:** [`docs/SoundData_FET445_ProjectOutline.pdf`](docs/SoundData_FET445_ProjectOutline.pdf)

## 👥 Ekip Bilgileri — SoundData

| Ad Soyad | Öğrenci Numarası | Sorumluluk Alanı |
|---|---|---|
| Umut Torun | 23040101063 | Veri hazırlığı ve EDA, Ridge Regresyon, LightGBM Regressor; raporun 4., 5. ve 7. bölümleri |
| Onur Şengün | 23040101044 | Öznitelik mühendisliği, Random Forest Regressor, model karşılaştırması ve hata analizi; raporun 6. ve 9. bölümleri |
| Ortak | — | Problem tanımı, değerlendirme tasarımı, sınırlılıklar, README ve sunum videosu |

İş yükü iki üye arasında eşit paylaşılmıştır. Her üye kendi yazdığı notebook'ları depoya kendi hesabından yükler.

## 🎯 Araştırma Sorusu

> "Bir şarkının dijital ve akustik ses özellikleri ile müzik türü kullanılarak Spotify üzerindeki güncel popülerlik skoru ne doğrulukla tahmin edilebilir?"

## 📊 Veri Seti Özeti

- **Kaynak:** [Spotify Tracks Dataset (Kaggle — Maharshi Pandya)](https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset)
- **Lisans:** CC0: Public Domain
- **Boyut:** 114.000 satır, 20 sütun (yinelenen `track_id` kayıtları tekilleştirildiğinde ~89.741 benzersiz parça)
- **Hedef Değişken (`popularity`):** 0 ile 100 arasında tam sayı popülerlik skoru

### Kullanılan Öznitelikler (15)

- **Sayısal (10):** `duration_ms`, `danceability`, `energy`, `loudness`, `speechiness`, `acousticness`, `instrumentalness`, `liveness`, `valence`, `tempo`
- **Kategorik (3):** `track_genre`, `key`, `time_signature`
- **İkili (2):** `explicit`, `mode`
- **Çıkarılan Sütunlar:** `Unnamed: 0`, `track_id` (kimlik bilgisi), `track_name`, `album_name` (yüksek kardinaliteli serbest metin), `artists` (model şarkıyı değil sanatçıyı ezberler; yalnızca bölmede gruplama anahtarı olarak kullanılır)

### Veriyi edinme

1. Kaggle sayfasından `dataset.csv` dosyasını indirin.
2. Depodaki `data/` klasörüne `data/dataset.csv` olarak kaydedin. Veri dosyası depoya yüklenmez.

## ⚙️ Yöntem ve Modelleme Pipeline'ı

Veri sızıntısını önlemek amacıyla veriden öğrenilen tüm ön işleme adımları scikit-learn `Pipeline` ve `ColumnTransformer` yapısı içinde, her çapraz doğrulama katmanında **yalnızca eğitim verisinden** öğrenilir:

- **Veri Temizleme ve Ön İşleme:** Yinelenen `track_id` kayıtlarının tekilleştirilmesi; geçersiz değerlerin (ör. `tempo = 0`) eksik sayılması; eksik değerlerin doldurulması (`SimpleImputer`: sayısal → medyan, kategorik → en sık değer); kategorik değişkenler için `OneHotEncoder(handle_unknown="ignore")`; sayısal değişkenler için `StandardScaler`.
- **Modeller:**
  - **Temel Model (Baseline):** `DummyRegressor` (eğitim kümesi ortalaması)
  - **Doğrusal Model:** Ridge Regression (L2 düzenlileştirmesi)
  - **Ağaç Topluluğu (Bagging):** Random Forest Regressor
  - **Gradyan Artırma (Boosting):** LightGBM Regressor
- **Değerlendirme Stratejisi:**
  - Aynı şarkı farklı albümlerde farklı `track_id` ile de yer alabildiğinden veri, **sanatçı + şarkı adına göre gruplu** olarak %80 eğitim / %20 test şeklinde ayrılır (`GroupShuffleSplit`, `random_state=42`). Aynı şarkının kopyaları hiçbir zaman hem eğitimde hem testte bulunmaz.
  - Test kümesi yalnızca en sonda **bir kez** kullanılır.
  - Eğitim kümesi üzerinde aynı gruplamayla 5-katlı çapraz doğrulama (`GroupKFold`) ve `RandomizedSearchCV` ile hiperparametre optimizasyonu; sonuçlar ortalama ± standart sapma olarak raporlanır.
  - **Metrikler:** MAE, RMSE ve R²

## 📁 Depo Yapısı (Planlanan)

```
FET445_SoundData_SpotifyTracksDataset/
├── data/                  # dataset.csv buraya konur (depoya yüklenmez)
├── docs/                  # Proje önerisi ve final raporu PDF dosyaları
├── notebooks/             # Keşifsel Veri Analizi (EDA) ve modelleme notebook'ları
├── src/                   # Veri ön işleme, eğitim ve değerlendirme Python betikleri
├── .gitignore             # Python ve veri dosyası filtreleri
└── README.md              # Proje dokümantasyonu
```

## 📚 Kaynakça

1. M. Pandya, "Spotify Tracks Dataset," Kaggle, 2022. Erişim: https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset
2. F. Pedregosa vd., "Scikit-learn: Machine Learning in Python," *Journal of Machine Learning Research*, c. 12, ss. 2825–2830, 2011.
3. G. Ke vd., "LightGBM: A Highly Efficient Gradient Boosting Decision Tree," *NeurIPS*, c. 30, ss. 3146–3154, 2017.
