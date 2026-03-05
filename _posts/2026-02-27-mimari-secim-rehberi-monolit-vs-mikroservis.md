---
layout: post
title: "Monolith, Microservices, Modular Monolith"
image: 02.jpg
date: 2026-02-27
categories: productivity
tags: [legacy, architecture]
---

Her zaman söylediğim bir şey var yazılım mimarisi, popüler trendleri takip etmek değil, projenin kısıtlarını ve hedeflerini dengelemektir. Bir Senior Engineer olarak öğrendiğim en belirgin ders: **"Mikroservisler bir performans çözümü değil, bir ölçekleme ve organizasyon çözümüdür."** Bu yazıda, modern sistem tasarımındaki üç ana yaklaşımı teknik derinliğiyle analiz ediyoruz.

---

<img width="1024" height="1024" alt="Image" src="https://github.com/user-attachments/assets/2caf1219-3d8e-441a-adc4-b85ad6b41972" />

## 1. Monolithic Architecture: The Single Unit
Monolitik yapılar, tüm iş mantığının tek bir süreç (process) içinde çalıştığı sistemlerdir.



### Teknik Avantajlar
* **Low Latency:** Modüller arası iletişim bellek içi (in-memory) gerçekleşir; network overhead yoktur.
* **Transactional Integrity:** Veritabanı seviyesinde ACID garantisi sağlamak çok basittir. Dağıtık transaction (Saga pattern vb.) karmaşası yoktur.
* **Simplified Testing:** Uçtan uca (E2E) testler tek bir ortamda kolayca koşturulur.

### Dezavantajlar & Teknik Borç
* **Tight Coupling:** Bileşenler zamanla birbirine "leak" eder. Bir modüldeki bellek sızıntısı (memory leak) tüm sistemi çökertir.
* **Resource Inefficiency:** Sadece CPU yoğun bir modülü ölçeklemek için tüm uygulamayı replicate etmeniz gerekir.

---

## 2. Microservices: Distributed Complexity
Uygulamayı ağ (network) üzerinden haberleşen, bağımsız deploy edilebilir servislere bölme sanatıdır.



### Teknik Avantajlar
* **Independent Scalability:** Talebin yoğun olduğu servisi (örn. Payment) dikeyde, diğerlerini yatayda bağımsızca ölçekleyebilirsiniz.
* **Fault Isolation:** "Bulkhead" prensibi sayesinde, bir servisin çökmesi tüm sistemi aşağı çekmez.
* **Tech Stack Agnostic:** Her servis ihtiyacına göre farklı dillerde (Go, Rust, Node.js) yazılabilir.

### Operasyonel Gerçekler (Cons)
* **Network Reliability:** "Fallacy of distributed computing" (dağıtık hesaplama yanılgıları) devreye girer. Paket kayıpları ve timeout'lar yönetilmelidir.
* **Data Consistency:** Her servisin kendi DB'si olduğu için "Eventual Consistency" kaçınılmazdır.
* **Observability Burden:** Distributed tracing (Jaeger, Zipkin) olmadan sistemdeki bir hatanın kök nedenini bulmak imkansızdır.

---

## 3. Modular Monolith: The Strategic Middle Ground
Senior mühendislerin favori "escape hatch"i (kaçış noktası) budur. Kod, mantıksal olarak mikroservisler gibi izole edilmiş modüllere ayrılmıştır ancak tek bir deployment unit (artifact) olarak paketlenir.



### Neden "Altın Orta"?
* **Bounded Contexts:** İş mantığı, Domain Driven Design (DDD) prensipleriyle net sınırlarla ayrılır.
* **No Network Overhead:** Modüller arası iletişim hala bellek içindedir ancak arayüzler (Interfaces) üzerinden yapılır.
* **Zero Infrastructure Complexity:** Kubernetes cluster'ları veya karmaşık servis ağları (Service Mesh) gerektirmez.

---

## Karşılaştırmalı Teknik Analiz

| Kriter | Monolith | Modular Monolith | Microservices |
| :--- | :--- | :--- | :--- |
| **Deployment Dağıtımı** | Kolay (Tekil) | Kolay (Tekil) | Zor (Çoklu/Orkestrasyon) |
| **Modüller Arası İletişim** | Bellek İçi | Interface/Event Bus | HTTP/gRPC/Message Queue |
| **Veri Tutarlılığı** | Strong (ACID) | Strong (ACID) | Eventual Consistency |
| **Geliştirme Hızı (Initial)**| Çok Hızlı | Orta/Hızlı | Yavaş (Infra Setup) |
| **Hata İzolasyonu** | Yok | Kısmi | Tam |

---

## Ne Zaman Hangisi?

1.  **Monolith:** Eğer tek bir ekipseniz ve saniyede milyonlarca istek almıyorsanız, monolitten vazgeçmeyin. "Evolvability" (evrilebilirlik) her zaman "scalability"den (ölçeklenebilirlik) önce gelir.
2.  **Modular Monolith:** Mimari disiplin istiyorsanız ancak altyapı maliyetlerine (hem maddi hem personel) hazır değilseniz en mantıklı yoldur. Mikroservislere geçiş için en güvenli köprüdür.
3.  **Microservices:** Eğer ekibiniz 20-30 kişiyi geçtiyse ve insanlar birbirinin koduna dokunmadan geliştirme yapamıyorsa (organizasyonel darboğaz), mikroservislere geçme zamanı gelmiştir.

**Unutmayın:** Mikroservisler, teknik bir gereklilikten ziyade organizasyonel bir ihtiyaçtır. Conway Yasası'nı (Conway's Law) asla göz ardı etmeyin.
