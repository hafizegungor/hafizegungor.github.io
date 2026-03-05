---
layout: post
title: "LGTM Code Review"
image: 04.jpg
date: 2026-03-25
categories: productivity
tags: [engineering, code-review, team-culture, best-practices]
---

# Code Review Sanatı

İlk pull request'imi review ettiğimde, bir arkadaşım bana sadece şunu söylemişti: *"Her satırı sanki sen yazmışsın gibi oku."* O günden bu yana yüzlerce review yaptım. Ve şunu öğrendim: code review, hata bulmakla değil — anlayışla başlar.

---

## Neden Çoğu Code Review Başarısız Olur?

Çoğu ekipte code review bir formalitedir. PR açılır, biri hızlıca göz atar, "LGTM 👍" yazar, merge edilir. Bu yaklaşımın bir adı var: *rubber-stamp review*. Onay damgasını basmak. Ne güvenlik açığı yakalar, ne mimari hatayı, ne de gelecekteki teknik borcu.

Öte yanda bir de karşı ekstrem vardır: her satıra yorum yazan, "bu değişken adı neden `x`?" diye soran, review'ı micro-management aracına dönüştüren yaklaşım. Bu da ekibi yıldırır, PR boyutlarını küçültür, iletişimi korkuya döndürür.

> *"Code review, kod hakkında değil — mühendisler hakkındadır. İyi bir review insanı büyütür. Kötü bir review ise tam tersi..."*

---

## Reviewer Olmadan Önce: Zihin Çerçevesi

PR'ı açmadan önce kendinize şunu sorun: *"Burada ne değişti, ve neden?"* Description'ı okuyun. Yoksa isteyin. Bağlam olmadan review yapmak, röntgen filmini raporsuz yorumlamak gibidir.

1. **Değişkeni anlayın** — Hangi iş problemi çözülüyor? Hangi ticket? Hangi trade-off kabul edildi?
2. **Büyük resmi görün** — Dosyaları tek tek değil, değişimi bir bütün olarak okuyun. Önce diff'in tamamına bakın, sonra detaya inin.
3. **Niyeti iyi farz edin** — Kötü kod, çoğunlukla kötü niyetten değil; bağlam eksikliğinden ya da zaman baskısından gelir.

---

## Neye Bakmalısınız?

Bir PR'ı incelediğimde zihnimde bir öncelik hiyerarşisi var. Önem sırasına göre:

### 1. Doğruluk (Correctness)

Kod, iddia ettiği şeyi yapıyor mu? Edge case'ler düşünülmüş mü? Sınır değerleri, boş listeler, null referanslar, concurrent erişim — bunlar "bariz" değil, çoğu production bug'ının kaynağıdır.

```python
# ❌ Tehlikeli: liste boş olabilir
def get_first_admin(users):
    return [u for u in users if u.is_admin][0]

# ✅ Güvenli: boş liste için None döner
def get_first_admin(users):
    admins = [u for u in users if u.is_admin]
    return admins[0] if admins else None
```

### 2. Güvenlik (Security)

Kullanıcıdan gelen veri sanitize ediliyor mu? SQL injection, XSS, path traversal, insecure deserialization — bunları otomatik araçlara bırakmak yetmez. Mantık seviyesindeki güvenlik açıklarını sadece insan gözü yakalar.

### 3. Mimari Uyum

Bu değişiklik, ekibin kabul ettiği pattern'lara uyuyor mu? Yeni bir servis layer mı açıldı gereksiz yere? Mevcut bir abstraction delineated mı? Bugünün pragmatik çözümü, yarının spaghetti code'u olabilir.

### 4. Performans

N+1 query var mı? Loop içinde ağ çağrısı yapılıyor mu? Büyük liste bellekte tutulup sonra bir kez kullanılıyor mu?

```python
# ❌ N+1 Problem — her order için ayrı DB sorgusu
for order in orders:
    user = db.get_user(order.user_id)
    print(user.name)

# ✅ Eager Loading — tek sorguda hepsini çek
user_ids = [o.user_id for o in orders]
users = db.get_users_bulk(user_ids)
user_map = {u.id: u for u in users}
```

### 5. Test Kalitesi

Testler var mı? Ama daha önemlisi: testler gerçekten bir şeyi test ediyor mu? Her zaman geçen, hiçbir şeyi yakalamayan testler false confidence yaratır.

---

## Yorum Yazma Sanatı

İyi bir review yorumu üç şeyi yapar: problemi net tanımlar, neden problem olduğunu açıklar, ve mümkünse bir alternatif önerir. "Bu yanlış" bir yorum değil — bir iddiadır.

> **💡 Altın Kural:** Kodu eleştirin, insanı değil. "Sen bunu yanlış yapmışsın" yerine "Bu pattern şu nedenle sorun çıkarabilir" deyin. Fark küçük görünür, etkisi büyüktür.

Yorumlarınızı önceliklendirin — her şey aynı ağırlıkta değil:

| Öncelik | Kategori | Anlamı |
|---------|----------|--------|
| 🔴 | **Blocking** | Merge edilmemeli. Güvenlik açığı, mantık hatası, veri kaybı riski. |
| 🟡 | **Suggestion** | Öneriyorum ama zorunlu değil. Daha iyi bir pattern, daha okunabilir bir yaklaşım. |
| ⚪ | **Nit** | Küçük stil meselesi. "Alırsan memnun olurum ama almasan da olur." |
| 🟢 | **Appreciation** | İyi bir şey görünce de söyle. Sadece hata aramak insanı söndürür. |

---

## Şimdiye kadar öğrendiklerim

### Büyük PR'lar kötü PR'lardır

2000 satır değişiklik içeren bir PR'ı kimse düzgün review edemez. Beyni yorar, gözden kaçan şeyler artar. Mümkün olduğunda küçük, odaklı PR'lar isteyin. Bu aynı zamanda rollback'i kolaylaştırır.

### Asenkron iletişimin limitleri vardır

Yorumlar gidip geliyor, thread uzuyor, konu karışıyor. Beş yorumdan sonra hâlâ anlaşamıyorsanız, beş dakikalık bir call hepsini çözer. "Bu yorumu yaz, o yorumu yaz" döngüsü mühendislik zamanının en verimsiz kullanımıdır.

### Review kültürü, ekip kültürünü yansıtır

Ekibinizde yorumlar sert ve kişisel mi? Tabi her yapılan yorumu kişisel algılamamak gerekir ama eğer öyleyse de o ekipte psikolojik olarak birbirine güven yoktur. Yorumlar yüzeysel ve anlamsız mı? O ekipte sorumluluk duygusu yoktur. Review'larınıza bakın — ekip sağlığınızı görürsünüz.

### PR author'ının da sorumlulukları var

Review sadece reviewer'ın işi değil. PR açan kişi iyi bir description yazar, kritik kararları önceden not eder, review'ı kolaylaştıracak context sağlar. "Açıktır zaten" diye düşünülen şeyler çoğunlukla açık değildir.

### Approve etmek cesaret ister

Bazı mühendisler hiç approve etmez, ya da her zaman "changes requested" bırakır. Bu, sorumluluktan kaçmaktır. İyi bir reviewer hem "bu merge edilmemeli" diyebilmeli, hem de güvenle "bu hazır, gönderebilirsin" diyebilmelidir.

> *"En iyi code review, review bitmeden önce başlar — kodu yazarken kendinize 'bunu bir başkası okuyacak' diye düşünmekle."*

---

## Son Söz: Bir Ritüel Olarak Review

Code review, kalite kontrol mekanizmasından fazlasıdır. İyi yapıldığında bilgi transferinin en doğal yoludur. Junior bir mühendis, senior bir mühendisten aldığı yorumda bir yıllık deneyimi birkaç satırda öğrenebilir. Senior mühendis ise bazen junior'ın taze bakışından bir şeyler öğrenir.

Bunu bir ritüel olarak görün. Dikkati, saygıyı ve merakı hak eden bir ritüel. Çünkü sonunda incelediğiniz satırlar — siz de dahil — hepimizin üretimde bakacağı kod.

---

> **📌 Özet:** Doğruluk → Güvenlik → Mimari → Performans → Test. Önce bağlamı anlayın. Kodu eleştirin, insanı değil. Yorumlarınızı önceliklendirin. Küçük PR'ları tercih edin. Ve iyi işi takdir etmeyi unutmayın.
