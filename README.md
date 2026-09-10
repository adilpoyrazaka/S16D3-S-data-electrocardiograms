# Electrocardiograms

## Veri setini indirin

Veri seti [buradan](https://d32aokrjazspmn.cloudfront.net/materials/ML_Electrocardiograms_dataset.csv) edinilebilir. Aşağıdaki komutlarla indirip `02-Electrocardiograms` dizinindeki `data` klasörüne kaydedelim:

```bash
curl https://d32aokrjazspmn.cloudfront.net/materials/ML_Electrocardiograms_dataset.csv > data/electrocardiograms.csv
```

## Veri seti

- Veri setinin her gözlemi, bir hastanın electrocardiogram (ECG)'ından alınan sayısal olarak temsil edilmiş kalp atışıdır.
- `target` ikili değerlidir ve kalp atışının kardiyovasküler hastalık riski altında olup olmadığını tanımlar [1] veya değildir [0].

## Alıştırma

🎯 Göreviniz kardiyovasküler hastalık riski altındaki kalp atışlarını işaretlemektir. Şunları yapacaksınız:

- Veri setinin sınıf dengesini araştırın
- İki modeli değerlendirin ve karşılaştırın: KNN ve LogisticRegression
- Modellerin performansları hakkında içgörü elde etmek için Confusion matrix ve Classification report kullanın
- Uygun metriğe dayalı olarak optimal modeli seçin

Alıştırmaya başlamak için `jupyter notebook`'ta `Electrocardiograms.ipynb`'yi açın ve talimatları takip edin.

🚀 Sıra sizde!


## Sonuçlar ve Yorum

Veri seti: 19.565 kalp atışı, 187 ölçüm noktası + `target`.
Tüm metrikler 5-fold cross-validation ile hesaplanmıştır.

### 1. Sınıf dengesi

| Sınıf | Gözlem | Oran |
|---|---|---|
| Sağlıklı (0) | 18.117 | %92,60 |
| Riskli (1) | 1.448 | %7,40 |

Dağılım kasıtlı olarak korunmuştur: gerçek popülasyonda da kalp hastalığı
riski taşıyanlar azınlıktadır. Dengeleme yapmak yerine model seçimini bu
gerçekliğe göre uyarlıyoruz.

Kritik sonuç: hiçbir şey öğrenmeyip her atışa "sağlıklı" diyen bir model
bile **%92,6 accuracy** alır. Bu yüzden accuracy bu problemde tek başına
anlamsız bir metriktir.

### 2. Logistic Regression

| Metrik | Değer |
|---|---|
| Accuracy | 0,9388 |
| Recall | 0,3266 |
| Precision | 0,6821 |
| F1 | 0,4408 |

Accuracy %93,9 ile kulağa iyi geliyor, ama yukarıdaki naif taban çizgisinin
sadece 1,3 puan üstünde. Recall 0,327: model riskli hastaların **üçte
ikisini kaçırıyor**.

### 3. Confusion matrix (holdout, %30 test)

|  | Tahmin: sağlıklı | Tahmin: riskli |
|---|---|---|
| **Gerçek: sağlıklı** | 5374 | 62 |
| **Gerçek: riskli** | 287 | 147 |

Sol alt hücre (287 kaçırılan riskli hasta) sağ alttan (147 doğru işaretlenen)
neredeyse iki kat büyük. Buna karşılık yanlış alarm sadece 62 tane. Model
"riskli" dediğinde çoğunlukla haklı, ama nadiren "riskli" diyor —
dengesiz veride doğrusal karar sınırının çoğunluk sınıfına kayması bu.

Tıbbi tarama bağlamında maliyet asimetriktir: yanlış alarm ek bir tetkik
demektir, kaçırılan hasta tedavisiz kalmak demektir. Bu yüzden optimize
edilecek metrik **recall**'dır.

### 4. KNN vs Logistic Regression

| Metrik | LogisticRegression | KNN (k=5) |
|---|---|---|
| Accuracy | 0,9388 | **0,9854** |
| Recall | 0,3266 | **0,8577** |
| Precision | 0,6821 | **0,9402** |
| F1 | 0,4408 | **0,8971** |

KNN dört metrikte de üstün — bir takas söz konusu değil. Recall 2,6 kat
artıyor: 1448 riskli gözlemin LogReg'de ~975'i kaçarken KNN'de ~206'sı
kaçıyor.

Neden: LogisticRegression tüm veriye tek bir doğrusal karar sınırı çiziyor
ve dengesizlik bu sınırı çoğunluk sınıfına doğru itiyor. KNN'in böyle
küresel bir sınırı yok; her tahmin en yakın 5 komşuya bakılarak yerel
olarak veriliyor. Azınlık sınıfı seyrek olsa da 187 boyutlu uzayda kendi
kümelerini oluşturuyor ve KNN bunları yakalayabiliyor.

**Seçilen model: KNN.**

### 5. Yeni hasta tahmini

Tam veri setine yeniden eğitilen KNN modeli, ikinci görüş için gelen
hastanın kalp atışını **"at risk"** olarak sınıflandırdı.
