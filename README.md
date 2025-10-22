# ING Hubs Türkiye Datathon — Churn Prediction

Bu repo, ING Hubs Türkiye Datathon için hazırlanmış bir churn tahmin projesinin çalışma dizinidir. Amaç, müşterilerin referans tarihinden sonraki 6 aylık dönemde churn (erime) olasılıklarını tahmin etmektir.

Not: Veri dosyaları (ör. `data/*.csv`) gizlilik / yarışma kuralları gereği repoya dahil edilmemiştir — README'de veri şeması ve nasıl kullanılacağı açıklanmıştır.

## Kısa Özet
- Problem: Müşterilerin 6 aylık dönemde churn olasılıklarını tahmin etmek.
- Modele giriş verileri: müşteri demografileri, aylık işlem geçmişinin özetleri ve referans tarihleri.
- Hedef: İkili sınıflandırma (churn/no-churn) ve yarışma metriğine (Gini, Recall@10%, Lift@10% ağırlıklı skor) göre optimizasyon.

## Dosya/Dizin Yapısı
- `notebooks/01_eda.ipynb` — Veri keşfi ve temel temizleme.
- `notebooks/02_feature_engineering.ipynb` — Rolling/aggregate feature oluşturma, imputation, winsorize ve feature tip ayarları.
- `notebooks/03_modeling.ipynb` — Modelleme: baseline modeller, CatBoost hiperparam aramaları, final fit ve submission.
- `data/` — (Yerel) CSV dosyaları burada olmalı; *bu repo'ya dahil edilmemiştir*.
  - `customer_history.csv`, `customers.csv`, `reference_data.csv`, `reference_data_test.csv`, vs.
- `scripts/` — Yardımcı scriptler (ör. `impute_cv.py`) — veri-temizleme ve quick-CV için.
- `models/` — Kaydedilmiş imputer/ modeller (ör. `imputer_knn.joblib`) (oluşturulur çalışma sırasında).
- `outputs/` — Üretilen artefaktlar (submission, raporlar, feature CSV'leri).
- `metrik.py` — Yarışma metriği (ing_hubs_datathon_metric) ve yardımcı fonksiyonlar.

## Veri (özet) — repoya dahil değil
Aşağıdaki dosyalar yarışmada sağlanan verileri temsil eder (örnek isimler):
- `customer_history.csv` — müşterilerin aylık olarak özetlenmiş işlem geçmişini içerir.
- `customers.csv` — demografik bilgiler.
- `reference_data.csv` — train referans tarihleri ve label (churn).
- `reference_data_test.csv` — test referans tarihleri.
- `sample_submission.csv` — gönderim formatı örneği.

Repo'ya veri koymayacaksanız, bu CSV'leri lokal `data/` altına koyup notebook'lardaki yolların (`../data/*.csv`) doğru olduğundan emin olun.

## Hızlı Başlangıç (Windows PowerShell)
Aşağıda en kısa çalışma akışı verilmiştir.

1) Ortam oluşturun ve bağımlılıkları kurun

```powershell
cd "C:\Users\MSI\OneDrive\Desktop\ing"
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2) Gerekli verileri `data/` klasörüne koyun (lokal, repoda yok)

3) Feature engineering notebook'u çalıştırın (önerilen sıralama)
- `notebooks/01_eda.ipynb` (veri temizliği)
- `notebooks/02_feature_engineering.ipynb` (feature üretimi ve imputation)

4) Modeling
- `notebooks/03_modeling.ipynb` içinde model eğitim ve CV hücrelerini çalıştırın.

Alternatif: hızlı script ile impute/CV test
```powershell
python .\scripts\impute_cv.py
```

## Önerilen Deneme Akışı (competition-ready)
1. `02_feature_engineering.ipynb` ile feature'ları üretin (`train_features_fast.csv` ve `test_features_fast.csv` üretilir).
2. Baseline (LightGBM / LogisticRegression) ile hızlı CV yapın.
3. KNN/Iterative imputer deneyin (script hücresinde zaten karşılaştırma kodu var).
4. CatBoost ile hiperparam araması — önce küçük bir örnek/short-iterations ile (ör. `iterations=500`, `od_wait=50`, `n_iter=8`) ön arama, ardından en iyi paramlarla full fit.
5. Final modellemede yarışma metriğini (ING metric) kullanarak seçim yapın — `metrik.py` içindeki fonksiyonları kullanın.

## Yarışma Metriği
Kaggle değerlendirme metriği üç ölçütün ağırlıklı toplamıdır:
- Gini (40%) — Gini = 2 * ROC AUC - 1
- Recall@10% (30%)
- Lift@10% (30%)

Bazı notebook hücreleri `metrik.py` içindeki `ing_hubs_datathon_metric` fonksiyonunu çağıracak biçimde yazılmıştır. Model seçiminde (CV ve holdout) AUC yerine **öncelikle yarışma metriğini** kullanmanız önerilir.

## GPU / Colab Notları
- CatBoost/LightGBM GPU hızlandırması için Colab kullanacaksanız `Runtime > Change runtime type > GPU` seçin. CatBoost GPU ayarı: `CatBoostClassifier(task_type="GPU", devices='0', ...)`.
- TPU bir avantaj sağlamaz; karar-ağaçlı yöntemler TPU desteklemez.
- GPU modunda bazı metrikler (ör. AUC) her iterasyonda hesaplanamayabilir; CatBoost uyarısıyla `metric_period`=5 gibi bir varsayılan değer gösterir, bu normaldir.

## Güvenlik & Veri Gizliliği
- Gerçek müşteri verisi içeriyorsa kesinlikle repoya push etmeyin. Veriyi lokal `data/` dizinine koyun ve `.gitignore`'a ekleyin.
- Model artefaktlarında gizli veri veya PII bırakmamaya dikkat edin.

## Repro ve Notlar
- `requirements.txt` içinde temel paketler listelenmiştir. Ortamı kurduktan sonra notebook hücrelerini sırayla çalıştırın.
- Hiperparam aramaları büyük veri üzerinde çok zaman alır — önce küçük örnekleme ile ön arama yapın.
