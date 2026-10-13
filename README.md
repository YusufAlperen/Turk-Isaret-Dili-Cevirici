# Türk İşaret Dili Çevirici

**Android mobil cihazlar üzerinde çalışan kameradan alınan anlık görüntüden Türk İşaret Dili harf ve kelimelerini tanıyan uygulama ve kullanılan modellerin eğitim dosyaları.**

## Projenin kapsamı

- **İşaretten metne:** Kamera görüntüsünden el/vücut landmark'larının çıkarılması ve harf/kelime modelinin tahminlerinin metin arayüzünde sunulması.
- **Metinden işarete:** Türkçe metnin kelimelere ayrılması ve eşleşen yerel işaret videolarının oynatma listesi halinde sunulması.
- **Dil işleme:** Türkçe kelimeleri çözümleme ve belirli kelime dizilerinden cümle oluşturma için yardımcı kodlar.
- **Model hazırlama:** Harf sınıflandırması için iskelet görüntüsü hazırlama ve CNN eğitimi; kelime tanıma için video karelerinden landmark çıkarma, normalizasyon ve veri artırma betikleri.

## Depo yapısı ve dosya açıklamaları

### `harfModel/` — Harf tanıma ve eğitim

| Dosya | Açıklama |
|---|---|
| `harfModelEgitimi.ipynb` | 128×128 renkli iskelet görüntülerini klasör etiketlerinden yükler, veriyi eğitim/doğrulama kümelerine ayırır ve CNN tabanlı harf sınıflandırıcısını eğitir. Döndürme, öteleme ve yakınlaştırma veri artırımı; `ModelCheckpoint` ve `EarlyStopping` kullanımı içerir. |
| `harfModeli.keras` | Notebook tarafından kaydedilen Keras harf sınıflandırma modeli. |
| `veriSetindenLandmarkCikarma.ipynb` | Türk İşaret Dili veri setindeki görüntülerden MediaPipe Hands ile el landmark'larını çıkarıp sayısal özellik ve etiket verileri oluşturmak için hazırlanmış notebook. |
| `sayisalVerilerdeOnIsleme.py` | Sayısal landmark CSV dosyasını işler. İki elle yapılması gereken harflerde iki elin de mevcut olduğu örnekleri tutarak filtrelenmiş veri seti üretir. |
| `iskeletGoruntuleriEldeEtme.py` | CSV içindeki 126 el landmark özelliğini 128×128 renkli iskelet görüntülerine dönüştürür. Sol ve sağ el bağlantılarını farklı renklerle çizer ve görüntüleri sınıf klasörlerine kaydeder. |
| `labels.txt` | Harf modelinin sınıf sırasını belirtir. `del`, `nothing` ve `space` gibi özel sınıfları da içerir. |

### `kelimeModel/` — Kelime modeli için veri hazırlama

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

#### Android yapılandırması ve arayüz kaynakları

| Dosya / klasör | Açıklama |
|---|---|
| `app/build.gradle.kts` | Android modülünün derleme yapılandırması ve bağımlılık tanımları. |
| `app/src/main/AndroidManifest.xml` | Uygulama bileşenleri ve gerekli Android izinlerinin tanımı. |
| `app/src/main/res/layout/activity_main.xml` | Ana ekranın yerleşimi. |
| `app/src/main/res/layout/fragment_sign_to_text.xml` | İşaretten metne ekranının arayüz yerleşimi. |
| `app/src/main/res/layout/fragment_text_to_sign.xml` | Metinden işarete ekranının arayüz yerleşimi. |
| `app/src/main/res/menu/bottom_nav_menu.xml` | Alt gezinme menüsünün öğeleri. |
| `app/src/main/res/values/strings.xml` | Arayüz metinleri. |
| `app/src/main/res/values/colors.xml` | Renk tanımları. |
| `app/src/main/res/values/themes.xml` ve `values-night/themes.xml` | Gündüz/gece tema tanımları. |
| `app/src/main/res/color/bottom_nav_color.xml` | Alt gezinme bileşeni için renk durumu tanımı. |
| `app/src/main/res/drawable/` | Başlatıcı simgesinin ön plan ve arka plan çizimleri. |
| `app/src/main/res/mipmap-*/` | Farklı yoğunluklar için uygulama ve yuvarlak uygulama simgeleri. |
| `app/src/main/res/xml/backup_rules.xml` | Android yedekleme kuralları. |
| `app/src/main/res/xml/data_extraction_rules.xml` | Android veri çıkarımı/yedekleme kuralları. |
| `app/src/androidTest/java/com/example/signtr2/ExampleInstrumentedTest.kt` | Android cihaz/çalışma zamanı ortamında yürütülmek üzere oluşturulmuş örnek enstrümantasyon testi. |
| `app/proguard-rules.pro` | R8/ProGuard küçültme ve kod koruma kuralları için dosya. |
| `app/.gitignore`, `mobilUygulama/.gitignore`, `.gitignore` | Git tarafından izlenmemesi gereken dosya ve klasör kalıpları. |
| `.idea/` | Android Studio/IntelliJ proje ve yerel geliştirme ortamı ayarları. Bunlar modelin çalışma mantığı değil, IDE yapılandırmasıdır. |
| `.kotlin/errors/` | Geliştirme sırasında oluşmuş Kotlin derleme hata günlükleri; uygulamanın iş mantığının parçası değildir. |

#### `app/src/main/assets/` — Modeller, etiketler ve videolar

| Dosya / klasör | Açıklama |
|---|---|
| `harf_modeli.tflite` | Mobil uygulamada harf sınıflandırması için kullanılan TensorFlow Lite modeli. |
| `en_iyi_model.tflite` | Mobil uygulamada kelime tanıma için kullanılan TensorFlow Lite modeli. Dosya adı tek başına modelin bağımsız testte en iyi sonucu verdiğini kanıtlamaz. |
| `hand_landmarker.task` | MediaPipe el landmark'larını tespit etmek için kullanılan görev modeli. |
| `holistic_landmarker.task` | Uygulamanın bütüncül landmark çıkarımı akışında kullanılan MediaPipe görev modeli. |
| `pose_landmarker.task` | Vücut pozu landmark'larını çıkarmak için kullanılan MediaPipe görev modeli. |
| `labels_harf.txt` | Harf TFLite modelinin çıktı indekslerini harf/özel sınıf adlarıyla eşleştiren etiket listesi. |
| `labels_kelime.txt` | Kelime TFLite modelinin çıktı indekslerini kelime etiketleriyle eşleştiren etiket listesi. |
| `videos/*.mp4` | Metinden işarete ekranında kelimelerle eşleştirilerek oynatılan işaret dili video varlıkları. Depoda 226 MP4 video dosyası bulunmaktadır. |

## Sonuçlar ve doğruluk

### Harf sınıflandırma modeli

`harfModelEgitimi.ipynb` dosyasındaki kayıtlı eğitim günlüğünde:

- Girdi boyutu: **128 × 128 × 3** renkli görüntü.
- Veri ayrımı: **%80 eğitim / %20 doğrulama** (`validation_split=0.2`).
- Eğitim günlüğünde görülen en yüksek doğrulama doğruluğu: **%94,28** (27. epoch).
- Aynı epoch'taki doğrulama kaybı (`val_loss`): **0,2163**.
- Eğitim günlüğü 34. epoch'a kadar kayıt içeriyor; yapılandırmada üst sınır 50 epoch ve erken durdurma ayarı bulunuyor.

Bu değer, notebook'taki doğrulama bölümünde ölçülen sınıflandırma başarısıdır. Ayrı ve bağımsız bir test kümesinde ölçülmüş sonuç olarak yorumlanmamalıdır. Ayrıca modelin gerçek ortamda farklı kişiler, kamera açıları, ışık koşulları ve arka planlar altında aynı doğruluğu vereceği sonucunu tek başına göstermez.

### Kelime sınıflandırma modeli

Depoda kelime modeline yönelik özellik çıkarımı, normalizasyon ve veri artırma betikleri ile uygulamada kullanılan `en_iyi_model.tflite` dosyası yer alıyor. Ancak depoda kelime modelinin test/doğrulama sonuçlarını, karışıklık matrisini veya doğruluk değerini doğrulayacak bir eğitim günlüğü ya da değerlendirme raporu bulunmuyor. Bu nedenle kelime tanıma için **kanıtlanmış bir yüzde doğruluk belirtilemiyor**.

### Uygulama düzeyindeki sonuçlar

Kod ve varlıklar iki akışın tasarlandığını gösteriyor: işaret tahminiyle metin üretimi ve metinden kelime videoları oluşturma. Fakat model doğruluğu ile uçtan uca çeviri başarısı aynı ölçüt değildir. Uygulamanın genel çeviri doğruluğu, gecikmesi veya kullanıcı başarısı hakkında sayısal bir iddia sunmak için ayrı test protokolü ve ölçüm sonuçları gerekir.

## Bilinen kapsam ve sınırlamalar

- Harf etiket dosyası Latin harflerinin bir alt kümesini ve `del`, `nothing`, `space` gibi özel sınıfları içerir; bu nedenle Türk alfabesindeki her harfin desteklendiği varsayılmamalıdır.
- Metinden işarete akışı mevcut video etiketleriyle eşleştirmeye dayanır. Her Türkçe cümle için dilbilgisel ve anlamsal açıdan eksiksiz Türk İşaret Dili çevirisi sağladığı söylenemez.
- İşaret dilleri yalnızca el şekillerinden oluşmaz; hareket, konum, yüz ifadesi ve bağlam da önemlidir.
- Veri artırımı veya doğrulama doğruluğu, tek başına gerçek dünya performansını kanıtlamaz.

## Teknoloji özeti

- **Kotlin / Android:** Mobil uygulama ve arayüz.
- **TensorFlow / Keras:** Harf modeli eğitimi.
- **TensorFlow Lite:** Mobil cihazda model çıkarımı.
- **MediaPipe:** El, vücut ve bütüncül landmark çıkarımı.
- **OpenCV / NumPy / pandas:** Görüntü işleme ve sayısal veri hazırlama.
- **Python / Jupyter Notebook:** Model veri hattı ve deneyler.
