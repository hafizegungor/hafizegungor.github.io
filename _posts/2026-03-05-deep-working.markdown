---
layout: post
title: "Deep Working"
image: 05.jpg
date: 2026-03-05 17:50:18 +0200
tags: [deep work, focus]
categories: productivity
---

> *Sürekli bölünmelerin olduğu bir ortamda kafamızdaki karmaşık algoritmalar üzerine nasıl odaklanılır?*

---

Slack'te bir bildirim geliyor. Okumadan geçiyorsunuz. Beş dakika sonra bir toplantı daveti. Toplantı biter, dönüyorsunuz koda — ama o an neredeydiniz? O algoritmanın hangi adımındaydınız? Yeniden başlamak gerekiyor.

Bu döngüyü günde pek çok kez yaşıyoruz, yaşıyorsunuz?

---

## Sorunun Gerçek Boyutu

Cal Newport'un tanımıyla *deep work*, bilişsel olarak zorlu görevlerde dikkat dağılması olmadan çalışmaktır. Geliştirme yaparken bu tanım özellikle acı verici çünkü işimizin en değerli kısmı — karmaşık bir animasyon sistemi kurmak, bellek sızıntısı ayıklamak, bir rendering pipeline tasarlamak — tam da bu tür konsantrasyon gerektirir.

Ama modern iş ortamı buna tam zıt şekilde tasarlanmıştır.

Araştırmalar derin odaktan kopmanın zihinsel maliyetinin sadece o anla sınırlı olmadığını gösteriyor. Her kesintinin ardından zihninizin önceki odak seviyesine dönmesi 20-25 dakika alıyor. Günde 8 saat çalışan bir mühendis, ortalama olarak 3-4 saat gerçek deep work yapabiliyor — geri kalanı ya bölünme ya da toparlanma zamanı.

Mobil geliştiriciler için bu rakam daha da düşük. Neden?

**Platform çoğulluğu.** iOS ve Android'i aynı anda takip etmek, iki farklı ekosistem, iki farklı dil, iki farklı hata mekaniği demektir. Zihninizde sürekli birden fazla mental model tutmak zorunda kalıyorsunuz.

**Cihaz fragmentasyonu.** "Neden Xiaomi'de çalışmıyor?" sorusu sizi anında başka bir probleme çekiyor.

**Emülatör/cihaz döngüsü.** Derleme süreleri, her bölünmeyle birleşince, odak penceresini parçalıyor.

---

## "Sadece Bildirimleri Kapat" Yeterli Değil

Bu tavsiyeyi duymuşsunuzdur. Doğrudur ama yetersizdir.

Bildirimleri kapattığınızda dış bölünmeleri engellemiş olursunuz. Ama iç bölünmeler — "acaba o MR merge edildi mi?", "toplantıda ne söylenecekti?" — hâlâ oradadır. Zihin, bitmemiş işler için bir alarm sistemi kurar. Buna psikolojide *Zeigarnik Etkisi* denmekte: tamamlanmamış görevler dikkatimizi işgal eder.

Gerçek deep work, hem dış hem de iç bölünmelere karşı sistemli bir savunma gerektirir.

---

## Sistem Kurmak: Pratikte Çalışan Şeyler

### 1. "Deep Work Bloğu" Takviminizde Gerçek Bir Slot Olmalı

Soyut bir niyet değil, takvimde görünen, başkasının üzerine toplantı atamayacağı bir blok. Benim için bu sabahın ilk iki saatiydi — 09:00-11:00, hiçbir şey giremez.

Bu bloğu korumak için ekibinizle açık bir anlaşma yapmanız gerekiyor. "Bu saatler arasında acil olmayan her şey bekleyebilir" bir politika, bireysel bir tercih değil.

### 2. Göreve Girmeden Önce "Kapanış Ritüeli"

Bu benim en çok değer verdiğim alışkanlık.

Deep work başlamadan önce beş dakika harcayarak şunları yapıyorum: açık tüm browser tab'larını kapatıyorum, o anki zihinsel "bitmemiş işleri" bir yere not ediyorum, ve ne üzerine çalışacağımı tek bir cümleyle yazıyorum.

```
# Örnek deep work hedefi
Görev: RecyclerView scroll performansı — 60fps altına düşen frame'lerin kaynağını bul
Süre: 2 saat
Başarı kriteri: Profiler'da bottleneck'i tespit et, bir hipotez yaz
```

Bu küçük adım zihinsel geçişi dramatik biçimde hızlandırıyor. Nereye gideceğinizi bilmeden yola çıkmak, odaklanmayı imkânsız kılar.

### 3. Derleme Sürelerini Odak Düşmanı Olarak Görün

Mobil geliştirmede en sinsi deep work katili derleme süreleridir.

60 saniyelik bir derleme süresinde ne oluyor? Twitter açılıyor. Slack kontrol ediliyor. Geri döndüğünüzde zihin başka yerdedir.

Bu soruna karşı aldığım önlemler:

- **Incremental build yapılandırması** — tüm modülleri değil, değişen modülü build et
- **Build önbelleği** — Gradle/Xcode build cache'ini doğru yapılandır
- **Modüler mimari'nin odak faydası** — büyük monolith projeleri build sürelerini artırır; modülerleştirme hem mimari hem odak problemidir

Build süresi 60 saniyeden 15 saniyeye indiğinde deep work kalitesi fark edilir biçimde artar. Sayısal bir kazanım değil, bilişsel bir kazanım.

### 4. Slack / Mesajlaşma İçin "Asenkron Norm" Oluşturun

Slack'in default kullanım şekli senkrondur: birisi yazar, hemen cevap beklenir. Bu norm kırılmalı.

Ekibimde şu ilkeleri yerleştirmeye çalıştım:

- "Acil" ile "önemli" ayrımı: gerçekten acil olan şeyler (production down, blokaj) hariç her şey asenkrondur
- Cevap penceresi: mesajlara 2-3 saat içinde cevap vermek yeterlidir
- Mesajlaşmaya ayrılan zaman: sabah, öğle, akşam — günde üç kontrol

İlk başta bu yaklaşıma "ulaşılamaz" damgası vuruluyor. Ama tutarlı şekilde uygulandığında ekip alışıyor. Hatta zamanla "ben de bunu yapabilir miyim?" diye soruyorlar.

### 5. Karmaşık Algoritmik İşi "Çözme Seansı" Olarak Çerçeveleme

Bir scroll performansı optimizasyonu veya karmaşık bir state machine yazmak, sadece kod yazmak değildir. Önce düşünmek gerekir.

Deep work bloğunu şöyle yapılandırıyorum:

```
[0-15 dk]   → Problemi kağıda yaz. Pseudocode, diyagram, notlar.
[15-75 dk]  → Kod yaz. Hiçbir şeyi kontrol etme.
[75-90 dk]  → Test et, gözlemle, notlar al.
[90-120 dk] → Refine et veya hipotezi güncelle.
```

Kağıda yazmak gereksiz görünür ama değildir. Problemi dile getirmek, zihnin onu "çözmüş gibi yapmasını" engeller. Çoğu zaman yazdıktan sonra problemi gerçekten anladığınızı fark edersiniz.

---

## Toplantılarla Barışmak (Ama Sınırını Çizmek)

Toplantıları tamamen ortadan kaldırmak ne mümkün ne de arzu edilir. Senkron iletişimin değeri var.

Ama toplantı yapısını şekillendirmek mümkün:

**Toplantı günleri.** Bazı mühendisler tüm toplantılarını haftanın belirli günlerine toplar — Salı ve Perşembe toplantı günleri, geri kalan günler deep work günleri. Bu radikal görünür ama deneyen ekiplerde production çıktısı artar.

**Stand-up formatı.** Günlük stand-up'ı 15 dakikadan fazla tutan, asenkrona çekiştiren her şeyi "ayrı konuşalım" ile bitirin. Stand-up bir raporlama aracı değil, blokajları görünür kılma aracıdır.

**Hazırlıksız toplantıya katılmama hakkı.** Ajandası olmayan toplantı daveti reddedilebilir. Bu bir kural olarak benimsenirse ekipte toplantı kalitesi artar.

---

## Dikkat Dağınıklığına Karşı Zihinsel Dayanıklılık

Sistem kurmak yeterli değil. Zihinsel bir kas da gerekiyor.

Deep work yapmayı öğrenmek, tıpkı ağır antrenman gibi kademeli bir süreçtir. İlk günler 90 dakika kesintisiz çalışmak zorlu gelir. Zihin kaçmak ister. Bu normal.

Benim için işe yarayan: fark ettiğimde geri dönmek. Zihin dağıldığında sert bir yargılamaya gerek yok — sadece geri dön. Bu refleks zamanla güçlenir.

Ayrıca şunu fark ettim: deep work'e giren zihni en çok yerinden eden, *gelmesi muhtemel kesintiye dair beklenti*dir. "Birazdan toplantı var" düşüncesi, toplantının kendisinden önce odağı bozar. Takvimi temizlemek bu yüzden bu kadar önemli.

---

## Somut Bir Haftalık Yapı

7 yılda yerleşen rutin şu şekle evrildi:

| Gün | Sabah | Öğleden Sonra |
|-----|-------|---------------|
| Pazartesi | Deep work blok (2 saat) | Toplantılar, PR review |
| Salı | Deep work blok (2 saat) | Teknik yazı, dokümantasyon |
| Çarşamba | Toplantı günü | Toplantı günü |
| Perşembe | Deep work blok (2 saat) | Mentoring, 1:1'ler |
| Cuma | Deep work blok (1.5 saat) | Haftayı kapat, öğrenme |

Çarşamba toplantı günü olduğu için diğer günler korunaklı. Bu yapı sabit değil — sprint döngüsüne, takıma, şirkete göre değişir. Ama bir yapı olması, her gün sıfırdan karar vermekten çok daha iyidir.

---

## Özetle

Bazı mühendisler deep work'ü kendini şımartmak gibi görür. "Asıl iş toplantılar ve koordinasyon, kod yazmak ikincil."

Bu bakış açısı tehlikelidir.

Karmaşık algoritmalar, iyi tasarlanmış mimari kararlar, ayıklanmış bir bellek sızıntısı — bunlar yüzeysel dikkatle üretilemez. Derin iş olmadan üretilen kod teknik borçtur. Ve teknik borç birikince ekibin tamamı yavaşlar.

Deep work'e yatırım yapmak bireysel bir tercih değil, uzun vadede takım hızını korumanın yoludur.

---
