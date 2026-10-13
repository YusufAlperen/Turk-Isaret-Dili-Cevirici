# Türk İşaret Dili Çevirici

**Android mobil cihazlar üzerinde çalışan kameradan alınan anlık görüntüden Türk İşaret Dili harf ve kelimelerini tanıyan uygulama ve kullanılan modellerin eğitim dosyaları.**

## Projenin kapsamı

- **İşaretten metne:** Kamera görüntüsünden el/vücut landmark'larının çıkarılması ve harf/kelime modelinin tahminlerinin metin arayüzünde sunulması.
- **Metinden işarete:** Türkçe metnin kelimelere ayrılması ve eşleşen yerel işaret videolarının oynatma listesi halinde sunulması.
- **Dil işleme:** Türkçe kelimeleri çözümleme ve belirli kelime dizilerinden cümle oluşturma için yardımcı kodlar.
- **Model hazırlama:** Harf sınıflandırması için iskelet görüntüsü hazırlama ve CNN eğitimi; kelime tanıma için video karelerinden landmark çıkarma, normalizasyon ve veri artırma betikleri.

## Depo yapısı ve dosya açıklamaları

### `harfModel/` — Harf tanıma ve eğitim
**Kullanılan Veri Seti = https://www.kaggle.com/datasets/berkaykocaoglu/tr-sign-language**
| Dosya | Açıklama |
|---|---|
| `harfModelEgitimi.ipynb` | 128×128 renkli iskelet görüntülerini klasör etiketlerinden yükler, veriyi eğitim/doğrulama kümelerine ayırır ve CNN tabanlı harf sınıflandırıcısını eğitir. Döndürme, öteleme ve yakınlaştırma veri artırımı; `ModelCheckpoint` ve `EarlyStopping` kullanımı içerir. |
| `harfModeli.keras` | Notebook tarafından kaydedilen Keras harf sınıflandırma modeli. |
| `veriSetindenLandmarkCikarma.ipynb` | Türk İşaret Dili veri setindeki görüntülerden MediaPipe Hands ile el landmark'larını çıkarıp sayısal özellik ve etiket verileri oluşturmak için hazırlanmış notebook. |
| `sayisalVerilerdeOnIsleme.py` | Sayısal landmark CSV dosyasını işler. İki elle yapılması gereken harflerde iki elin de mevcut olduğu örnekleri tutarak filtrelenmiş veri seti üretir. |
| `iskeletGoruntuleriEldeEtme.py` | CSV içindeki 126 el landmark özelliğini 128×128 renkli iskelet görüntülerine dönüştürür. Sol ve sağ el bağlantılarını farklı renklerle çizer ve görüntüleri sınıf klasörlerine kaydeder. |
| `labels.txt` | Harf modelinin sınıf sırasını belirtir. `del`, `nothing` ve `space` gibi özel sınıfları da içerir. |

### `kelimeModel/` — Kelime modeli için veri hazırlama
**Kullanılan Veriseti = https://chalearnlap.cvc.uab.es/dataset/40/description/**
| Dosya | Açıklama |
|---|---|
| `ozellikCikarma.py` | Kelime videolarından örneklenmiş 30 kare seçer; her kare için MediaPipe Pose ve Hand Landmarker kullanarak gövde ve iki el landmark'larını birleştirir. Çıktıları `X` ve `y` NumPy dizileri olarak kaydeder. |
| `normalizasyon.py` | Kare başına 225 boyutlu ham landmark verisinden gövdenin üst kısmı ve iki el için 201 özellik seçer. Koordinatları burun konumu ve omuzlar arası mesafeye göre normalize eder. |
| `gurultuEkleme.py` | Özellik dizilerine küçük rastgele gürültü ekleyerek veri artırımı yapar; özgün ve artırılmış örnekleri birleştirip karıştırır. |

### `mobilUygulama/` — Android uygulaması

#### Kotlin kaynak dosyaları

| Dosya | Açıklama |
|---|---|
| `app/src/main/java/com/example/signtr2/MainActivity.kt` | Uygulamanın ana Activity'si ve temel ekran/navigasyon yapısı. |
| `app/src/main/java/com/example/signtr2/SignToTextFragment.kt` | İşaretten metne ekranını yönetir. Kamera akışı, MediaPipe landmark çıkarımı, TensorFlow Lite model yükleme/çıkarımı, harf ve kelime tahminleri, aday tahminler, metin oluşturma ve metni seslendirme işlevlerini bir araya getirir. |
| `app/src/main/java/com/example/signtr2/TextToSignFragment.kt` | Metinden işarete ekranını yönetir. Metin ve sesli giriş, video varlıklarının indekslenmesi, kelime-video eşleştirmesi ve oynatma listesi kontrollerini içerir. |
| `app/src/main/java/com/example/signtr2/TurkishLemmatizer.kt` | Girilen Türkçe cümleyi çözümleyerek kelimeleri işaret video etiketleriyle eşleştirmeye yardımcı olur. |
| `app/src/main/java/com/example/signtr2/NLG.kt` | Kelime dizilerinden daha düzenli Türkçe cümleler üretmek için fiil çekimi, olumsuzluk, zaman ve bazı isim hâli işlemlerini içeren kural tabanlı dil üretim kodu. |

## Sonuçlar ve doğruluk

### Harf sınıflandırma modeli

`harfModelEgitimi.ipynb` dosyasındaki kayıtlı eğitim günlüğünde:

- Girdi boyutu: **128 × 128 × 3** renkli görüntü.
- Veri ayrımı: **%80 eğitim / %20 doğrulama** (`validation_split=0.2`).
- Eğitim günlüğünde görülen en yüksek doğrulama doğruluğu: **%94,28** (27. epoch).
- Aynı epoch'taki doğrulama kaybı (`val_loss`): **0,2163**.
- Eğitim günlüğü 34. epoch'a kadar kayıt içeriyor; yapılandırmada üst sınır 50 epoch ve erken durdurma ayarı bulunuyor.

## Teknoloji özeti

- **Kotlin / Android:** Mobil uygulama ve arayüz.
- **TensorFlow / Keras:** Harf modeli eğitimi.
- **TensorFlow Lite:** Mobil cihazda model çıkarımı.
- **MediaPipe:** El, vücut ve bütüncül landmark çıkarımı.
- **OpenCV / NumPy / pandas:** Görüntü işleme ve sayısal veri hazırlama.
- **Python / Jupyter Notebook:** Model veri hattı ve deneyler.
