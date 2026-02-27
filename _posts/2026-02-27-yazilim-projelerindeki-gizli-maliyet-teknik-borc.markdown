---
layout: post
title: "Yazılım Projelerindeki Gizli Maliyet: Teknik Borç (Technical Debt)"
image: 03.jpg
date: 2026-02-27
categories: productivity
tags: [technical debt, architecture, software]
---

## ⛓️ 1. Teknik Borç (Technical Debt) Nedir?

Ward Cunningham bu terimi ortaya attığında, bunu bir "finansal borç" analojisi olarak kurguladı.
Borçlanmak, bugün yapamayacağınız bir şeyi yapmanıza olanak tanır, ancak geri ödemezseniz faiziyle sizi batırır.

Teknik borç, yazılım geliştirme sürecinde bilinçli ya da farkında olmadan kalite standartlarından ödün verilmesi sonucu ortaya çıkan eksikliklerin zamanla birikmesidir. 
Bu birikim, tıpkı finansal borç gibi “faiz” etkisi yaratarak ekibin geliştirme hızını azaltır, esnekliğini kısıtlar ve ilerleyen aşamalarda daha fazla efor gerektirir.
---


## 📉 2. Teknik Borç Nasıl Ortaya Çıkar?

Teknik borç bilinçli veya bilinçsiz şekilde birçok nedenden doğabilir:

🟡 Bilinçli Borç

Hızlı girdi sağlama amacıyla test yazmamak, tasarım kalıplarını atlamak, gereksiz kopyalama gibi bilinçli tercihler.
Bu tür borç stratejik olabilir, ancak planlı geri ödeme gerektirir.

🔴 Kazara / Eskimiş Tasarım Borcu

Zamanla ihtiyaçlar, ortam ve platform değiştiğinde mevcut mimari eskiyebilir — bu da borç yaratır.

⚠️ Bit Rot (Code Rot)

Kod tabanı zaman içerisinde çürür; tutarsızlıklar, karmaşık yapılar ve anlaşılması zor kod parçaları ortaya çıkar.

## 🧠 3. Teknik Borcun Etkileri Nelerdir?

Teknik borç eğer ele alınmazsa, aşağıdaki sonuçları doğurur:

🚦 Azalan Geliştirme Hızı

Kod karmaşıklaştıkça yeni özellik eklemek zorlaşır, hata düzeltmek gecikir — velocity düşer.

🧩 Artan Bakım Maliyeti

Yeni gelen mühendisler mevcut çerçeveyi çözmek için ekstra zaman harcar; bu da toplam sahip olma maliyetini yükseltir.

🛠 Zayıf Test Kapsamı

Eksik testler, refactor seferberliklerinde regresyon riskini artırır.

🍜 Kod Entropisi
Entropi nedir? Entropi, en temel anlamıyla bir sistemdeki düzensizlik, belirsizlik veya dağınıklık miktarını ifade eden bir kavramdır.
Yazılım geliştikçe düzensizlik yani entropi artar; bu, sistemin evrimine karşı çalışan bir mekanizmadır.

## 🔄 4. Teknik Borç Ödenmezse Ne Olur?

** Teknik borç uzun süre göz ardı edilirse: **

* Ekibin üretim hızı giderek azalır.

* Yeni özellikler karmaşıklaşır.

* Regresyon riskleri artar.

* Codebase “bağışıklık kazanmış bozulma” yaşar.

Martin Fowler’un çizdiği metaforik üçgen, yazılımda borç biriktikçe sistemin karmaşıklığının yükseldiğini ve değişim maliyetinin arttığını görsel olarak ortaya koyar.


🛠 5. Teknik Borç Neden ve Nasıl Yönetilmelidir?
Teknik borcu yönetmek, sadece yazılımcıların değil, ürün ve yöneticilerin de sorumluluğudur. Sadece “temiz kod yazmak” değil, aynı zamanda kurumsal hedeflerle uyumlu karar almak gerekir.

| Durum | Strateji | Operasyonel Karşılığı(Action Item) |
| :--- | :--- | :--- | 
| **Borcu Kaydet** | Görünmez olan borç yönetilemez. Teknik borcu kodun içinde ve iş takip sisteminde somutlaştırın. | Koda // TODO: [TECH-DEBT] - [ISSUE-ID] yorumu ekleyin. Backlog'da bu borcu "Technical Debt" etiketiyle bir task'e dönüştürün. |
| **Faizi Takip Et** | Borcun "faizi", o kodun geliştirme hızını ne kadar yavaşlattığıdır. Hatalar ve "developer friction" bu maliyeti belirler. | Change Failure Rate ve Lead Time for Changes metriklerini izleyin. Sık dokunulan ama her seferinde "kırılan" modülleri refactor listesinin başına alın.| 
| **Geri Ödeme Planı** |Borç birikmesine izin vermek mühendislik iflasıdır. Temizlik süreci projenin doğal bir parçası olmalıdır. | Her sprint kapasitesinin %10-20'sini "Maintenance & Refactor" kalemine rezerve edin. Bu süreyi Product Manager'a bir "pazarlık payı" olarak değil, sistem sağlığı için "zorunluluk" olarak sunun. | 
| **Bilinçli Borçlanma**| Borçlanmak bir mühendislik hatası değil, ticari bir karardır. Riski ve vadeyi iş birimleriyle paylaşın. |Karar anında şeffaf olun: "Bu mimari bizi Go-Live'a 2 hafta erken taşır ama 3 ay sonra ölçeklenme sorunları başlar. Bu borcu alıyor muyuz?" onayını yazılı/sözlü alın. | 

### Teknik borç yönetiminin faydaları:

* Değişime daha hızlı uyum sağlama

* Kod kalitesinde kalıcı iyileşme

* Yeni ekip üyelerinin projeye hızlı adapte olması

* Daha az kritik hata ve daha az sürpriz regresyon
---

## 📏 6. Teknik Borç Nasıl Ölçülür?

Teknik borç doğrudan sayılarla ölçülmez, ancak göstergeleri vardır:

🔹 Kod karmaşıklığı ve döngüsel bağımlılıklar
🔹 Test kapsamı ve otomasyon eksikliği
🔹 Refactor ihtiyacı artan bileşenler
🔹 Build süreleri ve CI başarısızlıkları

Statik analiz araçlarıyla bu göstergeler zaman içinde izlenebilir.

## 📊 7. Teknik Borçla Mücadele Stratejileri
🔁 Planlı Refactoring

Sadece “aceleyle yapılan her şeyi temizlemek” değil, sistematik ve risk odaklı refactor planı.

* 📦 Test Otomasyonu

Unit, integration ve end-to-end testler, regresyon riskini düşürür.

* 📋 Borç İşlerinin Sprint’e Dahil Edilmesi

Teknik borç maddelerini backlog’da görünür kılmak ve planlı şekilde çözmek.

* 📊 Süreç ve Performans Ölçümü

Kod analiz araçları, CI pipeline metrikleri ile borcun etkisini izlemek.

## 🌟 Ne Zaman Mükemmeliyetçi Olunmalı?
Burada yapacağınız "hızlı" hareketler, gelecekteki mühendislik kapasitenizi (engineering velocity) sıfırlayabilir.

* **Distributed Systems Boundaries (Servis Sınırları):** 2026'nın en büyük teknik borcu "Distributed Monolith"tir. Servis sınırlarını (Bounded Contexts) yanlış çizmek, kodun kirli olmasından bin kat daha tehlikelidir. Sınırlar yanlışsa, mikroservislerin getirdiği esneklik yerine ağ gecikmesi ve bağımlılık cehennemi (dependency hell) satın alırsınız.

* **Schema & Contract Design:** API kontratları ve veritabanı şemaları sistemin "evlilik sözleşmesi" gibidir. Bir kez public olan veya başka bir servisin bağımlı olduğu bir kontratı değiştirmek, tüm organizasyona "borç faizi" ödetmektir.

* **Compliance & Security:** KVKK/GDPR veya SOC2 uyumluluğunda borçlanamazsınız. Buradaki açıklar şirketin iflasına yol açabilir; bu konular "technical debt" değil, "legal liability"dir.
* 
* **Core Business Logic:** Uygulamanın kalbi olan, sürekli değişecek ve üzerine ekleme yapılacak alanlarda borçlanmak intihardır.

* **Güvenlik ve Veri Bütünlüğü:** "Hızlı olsun diye SQL injection'a açık bıraktık" diyemezsiniz. Bazı konuların "borcu" olmaz.

* **Yüksek Trafikli Bileşenler:** Verimlilik hataları ölçeklendiğinde devasa altyapı maliyetlerine dönüşür.

* **Public API'lar:** Bir kez dışarıya açtığınız bir interface'i değiştirmek, teknik borcu kullanıcılarınıza yansıtmak demektir.

## Ne Zaman "Hızlı ve Kirli" (Quick & Dirty)?
Hızlı kod yazmak her zaman tembellik değildir; bazen en profesyonel tercihtir. Aşağıdaki durumlarda "kirli" kod kabul edilebilir:

* **Pazar Doğrulaması (MVP):** Bir fikrin tutup tutmayacağını bilmiyorsanız, mükemmel bir mikroservis mimarisi kurmak kaynak israfıdır. Önce değer yaratıp yaratmadığınızı görün.

* **Kritik Deadline ve Fırsat Maliyeti:** Eğer bir kampanya dönemini kaçırmak şirkete milyonlara mal olacaksa, o "hard-coded" konfigürasyonu oraya koyun.

* **Kullan-At Kodlar (Disposable Code):** Bir defalık veri göçü yapacak bir script yazıyorsanız, SOLID prensipleriyle vakit kaybetmeyin.

* ** Hotfixes:** Sistem ayaktayken ve her saniye para kaybedilirken, zarif bir çözüm değil, kanamayı durduracak bir bandaj gerekir.

"Hızlı ve Kirli" (Quick & Dirty) kod, bir mühendislik hatası değil, yerine göre bir finansal araçtır.
2024-2026 yılları arasında büyük ölçekli sistemlerden örnekler verecek olursak:

* **Pazar Zamanlaması (Time-to-Market):** OpenAI’ın o1 modelini piyasaya sürme sürecindeki gibi; eğer rakibinizden önce "inference-heavy" bir özellik çıkarmanız gerekiyorsa, mükemmel bir autoscaling yerine başlangıçta over-provisioned (fazladan kapasiteli) ama verimsiz çalışan bir yapı kurmak "doğru" karardır.

* **Disposable Features (Kullan-At Özellikler):** A/B testi yapılan bir algoritma için karmaşık bir Design Pattern uygulamayın. Kodun %90 olasılıkla 2 hafta sonra silineceği bir senaryoda yapılan "temiz kod" yatırımı, aslında bir negatif değerdir.

* **Legacy Data Migration:** Bir kerelik çalışacak veri göçü scriptlerinde DRY (Don't Repeat Yourself) prensibi için kasmayın. Kodun okunabilir olması, tekrar kullanılabilir olmasından daha kritiktir.

Teknik borç, yazılım ekiplerinin kaçınılmaz bir gerçeğidir.
Doğru yönetildiğinde yazılımın evrimsel kapasitesini artırır; göz ardı edildiğinde ise yavaşlatır, karmaşıklaştırır ve sürdürülemez kılar.
Mükemmeliyetçilik overengineering'e dönüşebilir, aşırı hızlı kod yazmak ise ürün kalitesizliğine yol açar. 
Senior seviyesindeki bir mühendis, "Clean Code" kitabındaki kuralları ezbere bilen değil; hangi kuralın ne zaman çiğnenebileceğine karar verebilen kişidir.

Müşteri temiz koda para ödemez, çalışan bir çözüme öder. Ancak kirli kod yani birikmiş teknik borç ve faizleri, çözümün çalışmaya devam etmesini engeller.

