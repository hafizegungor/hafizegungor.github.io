---
layout: post
title: "Bir Enkazı Mimariye Dönüştürmek: Legacy Projelerde Modernizasyon Stratejileri"
image: 01.jpg
date: 2026-02-25 17:50:18 +0200
tags: [legacy, architecture]
categories: productivity
---
7 yıllık mobil geliştirme kariyerimde onlarca farklı projeye dokundum. Bazıları sıfırdan (greenfield) heyecanla başladığımız, bazıları ise "bunu kim yazmış?" dedirten, dokunmaya korkulan devasa "legacy" (miras) projelerdi.
Kıdemli bir mühendis olmanın getirdiği en büyük sorumluluklardan biri, o "korkulan" kod yığınını, iş süreçlerini aksatmadan modern, test edilebilir ve sürdürülebilir bir yapıya dönüştürmektir.
Eski projeler genellikle platformun artık "ikinci plana" attığı dillerle yazılmıştır ve legacy projelerin en büyük sorunu, iş mantığının (business logic) UI katmanına sızmış olmasıdır.

Modernizasyon sürecine profesyonel bir yol haritası (roadmap) ile başlamak, projenin sürdürülebilirliği ve ekibin motivasyonu için kritiktir. Plansız yapılan her refactor, sonunda daha büyük bir teknik borç dağına dönüşür.

İşte adım adım uygulayabileceğimiz Modernizasyon Stratejisi:

### 1. Mevcut Durum Analizi (Audit)
Her şeyden önce, mevcut projenin röntgenini çekmeliyiz.

* Dependency Graph: Hangi kütüphaneler artık desteklenmiyor (deprecated)?

* Tight Coupling Check: Hangi sınıflar birbirine çok bağımlı? (Örneğin: Activity/ViewController içinde doğrudan API çağrısı yapılıyor mu?)

* Build Performance: Projenin derlenme süresi ne kadar? Modülerleşme ihtiyacını buradan ölçebiliriz.

### 2. Altyapı Hazırlığı (Foundation)
Modern kodun üzerine oturacağı "temiz" zemini hazırlıyoruz.

* Dependency Injection (DI) Kurulumu: Android için Hilt, iOS için Swinject gibi bir yapı kurarak nesne üretimini merkezileştirmek. Bu, test yazabilmemiz için ön şarttır.

* Version Catalog / SPM: Bağımlılık yönetimini modernize etmek. Tüm kütüphane versiyonlarını tek bir merkezden yönetilir hale getirmek.

* Linter & Formatter: Kod standartlarını otomatize etmek (SwiftLint, Ktlint).

### 3. Katmanlı Mimariye Geçiş (Refactoring)
"Big Bang Rewrite" (her şeyi silip baştan yazma) yerine Strangler Fig yaklaşımını uyguluyoruz:

* Domain Katmanı: İş mantığını UI'dan koparıp UseCase veya Interactor seviyesine taşıyoruz.

* Data Katmanı: API çağrılarını ve Local DB işlemlerini bir Repository arkasına gizliyoruz. UI, verinin nereden geldiğini bilmemeli.

* ViewModel/State: Veriyi UI'a "push" etmek yerine, UI'ın "observe" edeceği bir State yapısı (StateFlow veya @Published) kurguluyoruz.

### 4. UI Modernizasyonu (Hybrid Approach)
Tüm uygulamayı bir anda Compose veya SwiftUI'a geçiremezsiniz.

* New Features First: Gelen her yeni ekranı mutlaka modern declarative framework ile yazıyoruz.

* Interoperability: Eski ekranların içine küçük ComposeView veya UIHostingController ekleyerek, parça parça bileşenleri modernize ediyoruz.

### 5. Test ve CI/CD Entegrasyonu
Kodun "emniyet kemeri" testtir.

* Unit Tests: Özellikle Domain ve Data katmanı için test yazarak iş mantığını sağlama alıyoruz.

* Automation: Her commit'te çalışan bir CI/CD pipeline (GitHub Actions, Bitrise vb.) kurarak regresyon hatalarının önüne geçiyoruz.

Peki, mesela Java'dan Kotlin'e, MVVM'den MVI'ya geçerken gemiyi batırmadan rotayı nasıl değiştiririz?

## 1. Hype Değil, Stratejik Karar
Modernizasyon bir "hype" yada popülerlik takibi olarak değil de bir Senior olarak kendimize ve ekibimize şu sorular sorulmalı:
* Mevcut yapı yeni özellik ekleme hızımızı (velocity) ne kadar yavaşlatıyor?

* Bakım maliyeti (maintenance cost), yeniden yazım maliyetinden yüksek mi?

* Crash oranlarımızın ne kadarı mimari eksikliklerden kaynaklanıyor?
  
## 2. "Big Bang Rewrite" Tuzağından Kaçınmak
Çoğu junior veya mid-level arkadaşın düştüğü en büyük hata: "Her şeyi silelim, baştan yazalım, bir ay sürer." 7 yılın bana öğrettiği tek bir gerçek var: Asla bir ay sürmez.
O eski kodun içinde, yılların getirdiği, kimsenin hatırlamadığı ama hayati önem taşıyan "edge case" çözümleri gizlidir.
### Strangler Fig Pattern (Boğucu İncir Stratejisi)
Modernizasyonda en güvenli yol budur. Tıpkı bir incir ağacının konak ağacı yavaş yavaş sarması gibi:

Yeni Özellikler: Her yeni ekranı veya özelliği yeni mimariyle (Kotlin + MVI + Compose/ViewBinding) yazılmalı.
Bridge (Köprü) Katmanı: Eski ve yeni dünyayı birbirine bağlayan sağlam Interface yapıları kurulmalıdır.

Özetle bu strateji mevcut yapının üzerine modern katmanlar inşa ederek, eski fonksiyonları zamanla devre dışı bırakmak.

## 3. Emniyet Kemeri (Testler)
Kodun içine girmeden önce, sistemin şu an nasıl çalıştığını belgelemelisiniz.
Refactor edilen her modülün en az %70-80 oranında test kapsamına alınmasını sağlamak olası regresyon riskini yönetmek için önemlidir.

### Regression Testing:
En kritik 5 akışınızı (Örn: Login, Sepet, Ödeme) belirleyin.

### UI ve Integration Testleri:
Kod değişse bile bu akışların bozulmadığını kanıtlayan otomatik testleriniz yoksa, modernizasyon bir kumardır.
### CI/CD Pipeline: 
Modernizasyon sürecinde kod kalitesini korumak için static code analysis (Linter, SonarQube) araçlarını devreye almak.
Modernizasyon yaparken sadece dili veya mimariyi değil, Threading ve Nullability gibi dilden dile değişen temel davranışları da normalize etmelisiniz.
MVVM'de her UI bileşeni için ayrı bir LiveData veya StateFlow tanımlamak (Örn: isLoading, errorMessage, data),
proje büyüdükçe "State Explosion" dediğimiz duruma yol açar.
Senior seviyesinde bir dönüşüm için MVI (Model-View-Intent) ile bu durumu tek bir ViewState altında topluyoruz.
## (Refactor Öncesi vs. Sonrası)
### MVVM (Eski Yaklaşım):
```kotlin class ProductViewModel : ViewModel() {
    val isLoading = MutableLiveData<Boolean>()
    val productList = MutableLiveData<List<Product>>()
    val error = MutableLiveData<String>()
    
    // Her birini ayrı ayrı observe etmek zorundasınız
}
 MVI (Modern Yaklaşım):
 data class ProductViewState(
    val isLoading: Boolean = false,
    val products: List<Product> = emptyList(),
    val error: String? = null
)

 class ProductViewModel : ViewModel() {
    private val _state = MutableStateFlow(ProductViewState())
    val state: StateFlow<ProductViewState> = _state.asStateFlow()

    // UI sadece 'state'i dinler ve tek bir yerden beslenir
}
```

Bu yapıya geçmek, özellikle asenkron işlemlerde (Side Effects) hangi verinin hangi durumda olduğunu takip etmeyi oldukça kolaylaştırıyor. 
Üstelik Unidirectional Data Flow (UDF) prensibi sayesinde hata ayıklama (debugging) süremiz yarı yarıya azalıyor.

* Modernizasyonun en sancılı ama en ödüllü kısmı 👉 DI (Dependency Injection) yapısını güncellemektir. 
Yıllar önce Dagger 2'nin o karmaşık Component ve Module yapısıyla boğuşurken, bugün Hilt ile hayatımız çok daha kolay bir hale geldi.
Eğer projenizde hiç DI yoksa veya çok karmaşıksa, önce ViewModel seviyesinde Hilt entegrasyonu yapın. Bu, kodun test edilebilirliğini %50 artıracaktır.

### Görsel Bir Devrim: XML'den Jetpack Compose'a Geçiş
Eski bir projede binlerce satırlık XML dosyalarını bir anda çöpe atamazsınız. Ancak modernizasyon stratejimizin bir parçası olarak Interoperability (Birlikte Çalışabilirlik) özelliğini kullanmalıyız.

* ComposeView Kullanımı: Mevcut bir XML layout'un içine sadece yeni eklenen bir butonu veya listeyi Compose ile yazarak başlayın.

* Temalandırma (Theming): XML tabanlı MaterialComponents temanızı, Compose MaterialTheme ile eşleştirin (Bridge). Böylece uygulamanın yarısı XML yarısı Compose olsa bile kullanıcı farkı anlamaz.
``` // Eski XML içinde Compose kullanmak bu kadar basit:
binding.composeView.setContent {
    MatherialTheme {
        ProductDetailScreen(uiState)
    }
}
```

Legacy bir projeyi modernize etmek, bir antikayı restore etmek gibidir; orijinaline saygı duyarken onu modern dünyanın standartlarına (Hilt, MVI, Compose) hazırlarsınız.
