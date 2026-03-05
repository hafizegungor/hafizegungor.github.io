---
layout: post
title: "Performans Optimizasyonu: Memory Leak, GPU Rendering ve Cold Start"
image: 06.jpg
date: 2025-03-05 17:50:18 +0200
tags: [performance, optimization, memory leak, gpu render, cold start, mobile]
categories: mobile
---

> **Bir Note:** Bu yazıyı yazmadan önce, production'da yanan 3 farklı uygulamayı düzelttim. Bunlardan birinde memory 6 saatte 2GB'a ulaşıyordu, birinde scroll her frame'de full repaint tetikliyordu, diğerinde ise kullanıcılar 8 saniye bekliyordu. Bu yazı o deneyimlerin distile edilmiş hali.

---

## İçindekiler

1. [Memory Leak Tespiti ve Çözümü](#memory-leak)
2. [GPU Rendering Sorunları](#gpu-rendering)
3. [Cold Start Optimizasyonu](#cold-start)
4. [Üçünü Birlikte Düşünmek: Holistic Performans Modeli](#holistic)

---

## 1. Memory Leak Tespiti ve Çözümü {#memory-leak}

### "Uygulama Bir Süre Sonra Yavaşlıyor" Sendromu

Memory leak, sinsi bir performans katildir. Anlık olarak yakalanmaz; saatler, bazen günler içinde birikir. Kullanıcılar genellikle şu semptomları bildirir:

- "Uygulamayı açık bıraktım, sonra dondu."
- "Birkaç saat sonra her şey yavaşlıyor."
- "Telefon ısınıyor ama bir şey yapmıyorum."

Bunların tamamı heap'in kontrolsüz büyüdüğünün işaretleri.

---

### Leak Nerede Başlar?

Memory leak'in üç ana kaynağı vardır:

#### 1.1 Referans Döngüleri (Reference Cycles)

JavaScript/TypeScript'te garbage collector genellikle akıllıdır, ancak döngüsel referanslar hâlâ tuzak kurabilir:

```javascript
// KÖTÜ: Closure içinde DOM referansı tutmak
function setupButton() {
  const button = document.getElementById('btn');
  const heavyData = loadHeavyData(); // 50MB veri

  button.addEventListener('click', () => {
    console.log(heavyData); // heavyData, listener kaldırılmadığı sürece GC edilmez
  });

  // button DOM'dan kaldırılsa bile heavyData bellekte kalır!
}
```

```javascript
// İYİ: WeakRef veya cleanup mekanizması
function setupButton() {
  const button = document.getElementById('btn');
  let heavyData = loadHeavyData();

  const handler = () => console.log(heavyData);
  button.addEventListener('click', handler);

  // Cleanup fonksiyonu dön
  return () => {
    button.removeEventListener('click', handler);
    heavyData = null; // Referansı kopar
  };
}
```

#### 1.2 Detached DOM Nodes

Bu, web uygulamalarında en sık karşılaştığım leak tipidir:

```javascript
// KÖTÜ: DOM'dan kaldırılan eleman hâlâ referansta tutuluyor
const cache = {};

function renderItem(id) {
  const el = document.createElement('div');
  cache[id] = el; // Bu referans asla temizlenmezse leak!
  document.body.appendChild(el);
}

function removeItem(id) {
  document.body.removeChild(cache[id]);
  // cache[id] hâlâ var! GC edemez.
  // FIX: delete cache[id];
}
```

Chrome DevTools'ta bunu tespit etmek için:
1. Memory tab → Heap Snapshot al
2. "Detached" filtresi uygula
3. DOM node'larının referans zincirini incele

#### 1.3 Timer ve Observer'lar

```javascript
// KÖTÜ: Component unmount'ta temizlenmeyen interval
class DataPoller extends React.Component {
  componentDidMount() {
    this.interval = setInterval(() => this.fetchData(), 5000);
  }
  // componentWillUnmount yok! Her remount'ta yeni interval açılır.
}

// İYİ:
class DataPoller extends React.Component {
  componentDidMount() {
    this.interval = setInterval(() => this.fetchData(), 5000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }
}

// Modern React hooks ile:
useEffect(() => {
  const interval = setInterval(() => fetchData(), 5000);
  return () => clearInterval(interval); // Cleanup
}, []);
```

---

### Profesyonel Tespit Araçları

#### Chrome DevTools Memory Profiling

**Allocation Timeline** yöntemi en güçlü araçtır:

```
1. DevTools → Memory → Allocation instrumentation on timeline
2. "Start" bas, uygulamayı normal kullan (5-10 dakika)
3. "Stop" bas
4. Timeline'da memory düşmüyorsa (sawtooth pattern yok) → leak var
```

Normal bir uygulamada heap şu şekilde görünür:
```
Memory (MB)
    ^
 50 |  /\  /\  /\   <- GC döngüleri (sawtooth pattern - NORMAL)
 30 | /  \/  \/  \
 10 |________________ → Zaman
```

Leak olan uygulamada:
```
Memory (MB)
    ^
200 |            /
150 |         /
100 |      /          <- Sürekli artış (PROBLEM!)
 50 |   /
 10 | /
    |________________ → Zaman
```

#### Node.js için Heap Profiling

```javascript
const v8 = require('v8');
const fs = require('fs');

// Heap snapshot al
function takeHeapSnapshot() {
  const snapshotStream = v8.writeHeapSnapshot();
  console.log(`Snapshot: ${snapshotStream}`);
}

// Her 60 saniyede snapshot al ve karşılaştır
let snapshotCount = 0;
setInterval(() => {
  if (snapshotCount < 3) {
    takeHeapSnapshot();
    snapshotCount++;
  }
}, 60000);
```

```bash
# clinic.js ile daha kapsamlı analiz
npm install -g clinic
clinic heapprofiler -- node app.js

# memwatch-next ile runtime monitoring
npm install memwatch-next
```

```javascript
const memwatch = require('memwatch-next');

memwatch.on('leak', (info) => {
  console.error('Memory leak detected:', info);
  // info: { start, end, growth, reason }
});

memwatch.on('stats', (stats) => {
  console.log('Heap stats:', stats.current_base);
});
```

---

### İleri Seviye: WeakMap ve WeakRef Kullanımı

```javascript
// Cache'i WeakMap ile yönet: key olan nesne GC edilirse otomatik temizlenir
const cache = new WeakMap();

function processElement(element) {
  if (cache.has(element)) {
    return cache.get(element);
  }

  const result = expensiveComputation(element);
  cache.set(element, result);
  return result;
}

// WeakRef ile soft referans
class ResourceManager {
  constructor(resource) {
    this.ref = new WeakRef(resource);
  }

  getResource() {
    const resource = this.ref.deref();
    if (!resource) {
      // GC tarafından temizlendi, yeniden oluştur
      return this.recreate();
    }
    return resource;
  }
}
```

---

## 2. GPU Rendering Sorunları {#gpu-rendering}

### "Scroll Takılıyor" Problemi ve Gerçek Nedeni

Kullanıcılar "scroll takılıyor" dediğinde, çoğu developer içgüdüsel olarak JavaScript'e bakar. Ama gerçek suçlu çoğunlukla GPU rendering pipeline'ıdır.

Tarayıcının render süreci şu adımlardan oluşur:

```
JavaScript → Style → Layout → Paint → Composite
    ↑            ↑       ↑        ↑         ↑
  ~ms          ~ms    Pahalı!  Pahalı!    Ucuz!
```

**Altın kural:** Mümkün olduğunca sadece **Composite** katmanında çalış.

---

### CSS Property Maliyetleri

Her CSS property değişimi farklı bir pipeline aşamasını tetikler:

```css
/* Layout tetikler (EN PAHALI) - tüm pipeline yeniden çalışır */
.element {
  width: 100px;    /* Layout */
  height: 100px;   /* Layout */
  margin: 10px;    /* Layout */
  padding: 5px;    /* Layout */
  top: 0;          /* Layout (position: relative/absolute ile) */
  font-size: 16px; /* Layout */
}

/* Paint tetikler (PAHALI) - sadece piksel yeniden çizilir */
.element {
  color: red;            /* Paint */
  background: blue;      /* Paint */
  border: 1px solid red; /* Paint */
  box-shadow: ...;       /* Paint */
}

/* Sadece Composite (UCUZ) - GPU katmanlarını birleştirir */
.element {
  transform: translateX(100px);  /* Composite only ✓ */
  opacity: 0.5;                  /* Composite only ✓ */
  will-change: transform;        /* GPU katmanı oluşturur ✓ */
}
```

#### Pratik Örnek: Animasyonları Düzeltmek

```css
/* KÖTÜ: Her frame'de layout tetikliyor */
@keyframes slideIn {
  from { left: -100px; }
  to   { left: 0; }
}

/* İYİ: Sadece composite, 60fps garantili */
@keyframes slideIn {
  from { transform: translateX(-100px); }
  to   { transform: translateX(0); }
}
```

```javascript
// KÖTÜ: JavaScript animasyonu, her frame layout thrashing
function animate() {
  element.style.left = (parseInt(element.style.left) + 1) + 'px';
  requestAnimationFrame(animate);
}

// İYİ: transform kullan
function animate() {
  currentX += 1;
  element.style.transform = `translateX(${currentX}px)`;
  requestAnimationFrame(animate);
}
```

---

### Layout Thrashing: Sessiz Performans Katili

Layout thrashing, bir frame içinde hem DOM okuma hem yazma yapıldığında oluşur:

```javascript
// KÖTÜ: Her iterasyonda layout force ediyor (reflow)
// 100 element için: 100 read + 100 write = 200 layout hesabı!
elements.forEach(el => {
  const height = el.offsetHeight; // DOM okuma → layout force
  el.style.height = (height + 10) + 'px'; // DOM yazma → layout invalidate
});

// İYİ: Tüm okumaları, tüm yazmalardan önce yap
// Sadece 2 layout hesabı!
const heights = elements.map(el => el.offsetHeight); // Toplu okuma
elements.forEach((el, i) => {
  el.style.height = (heights[i] + 10) + 'px'; // Toplu yazma
});
```

FastDOM kütüphanesi bu problemi sistematik çözer:

```javascript
import fastdom from 'fastdom';

// Okuma ve yazmaları otomatik batch'ler
fastdom.measure(() => {
  const height = element.offsetHeight;

  fastdom.mutate(() => {
    element.style.height = (height + 10) + 'px';
  });
});
```

---

### will-change: Doğru Kullanım

`will-change` yanlış kullanıldığında performansı artırmak yerine düşürür:

```css
/* YANLIŞ: Her şeye uygulama */
* {
  will-change: transform; /* GPU VRAM'i tüketir! */
}

/* YANLIŞ: Statik elementlere uygulama */
.header {
  will-change: transform; /* Header hiç animate edilmiyorsa gereksiz */
}

/* DOĞRU: Sadece aktif animasyon sırasında, JavaScript ile ekle */
```

```javascript
// Hover ile dinamik will-change
element.addEventListener('mouseenter', () => {
  element.style.willChange = 'transform';
});

element.addEventListener('animationend', () => {
  element.style.willChange = 'auto'; // Animasyon bitince kaldır!
});
```

---

### Chrome DevTools ile GPU Analizi

```
1. DevTools → Performance → Record
2. Uygulamayı kullan (scroll, animate)
3. Recording'i durdur
4. "Rendering" sekmesinde:
   - "Paint flashing" → Gereksiz repaint'leri gösterir (yeşil flash)
   - "Layer borders" → GPU katmanlarını gösterir (turuncu/mavi kenarlık)
   - "FPS meter" → Gerçek zamanlı frame rate
```

**Composite katmanları kontrol et:**

```javascript
// Chrome console'da layer sayısını öğren
// Çok fazla katman = aşırı GPU VRAM kullanımı
chrome.gpuBenchmarking.getGpuProcessInfo();
```

---

### Intersection Observer ile Lazy Rendering

Viewport dışındaki elementler için gereksiz render işlemini önle:

```javascript
// KÖTÜ: Tüm kartları aynı anda render et
const cards = data.map(item => renderCard(item));
container.append(...cards);

// İYİ: Sadece görünür olanları render et
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const placeholder = entry.target;
      const card = renderCard(placeholder.dataset.id);
      placeholder.replaceWith(card);
      observer.unobserve(placeholder);
    }
  });
}, {
  rootMargin: '100px', // Önceden yükle
  threshold: 0
});

// Önce placeholder'lar oluştur
data.forEach(item => {
  const placeholder = document.createElement('div');
  placeholder.dataset.id = item.id;
  placeholder.style.height = '200px'; // Layout shift önle
  container.appendChild(placeholder);
  observer.observe(placeholder);
});
```

---

## 3. Cold Start Optimizasyonu {#cold-start}

### Cold Start Nedir ve Neden Kritiktir?

Cold Start, uygulamanın tamamen kapalıyken ilk açılış sürecidir. Bu süreçte şunlar gerçekleşir:

```
Kullanıcı ikon'a tıklar
       ↓
OS: Process oluştur (fork/exec)
       ↓
Runtime: VM/JIT başlat (V8, JavaScriptCore, ART...)
       ↓
App: Bundle yükle ve parse et
       ↓
App: Modülleri initialize et
       ↓
App: İlk render
       ↓
Kullanıcı görür ← [Buradaki süre Cold Start'tır]
```

Google'ın araştırmasına göre, **53% of mobile users abandon a site that takes more than 3 seconds to load.** E-ticaret için her 100ms gecikme, dönüşümde %1 düşüş anlamına gelir.

---

### Web Uygulamaları için Cold Start Optimizasyonu

#### 3.1 Critical Rendering Path

```html
<!-- KÖTÜ: CSS ve JS head'de, render blocking -->
<head>
  <link rel="stylesheet" href="all-styles.css">  <!-- 500KB! -->
  <script src="app.bundle.js"></script>           <!-- 2MB! -->
</head>

<!-- İYİ: Critical CSS inline, diğerleri async -->
<head>
  <!-- Sadece above-the-fold için gerekli kritik CSS -->
  <style>
    /* ~14KB critical CSS - inline */
    body { margin: 0; font-family: system-ui; }
    .header { ... }
    .hero { ... }
  </style>

  <!-- Non-critical CSS async yükle -->
  <link rel="preload" href="full-styles.css" as="style"
        onload="this.onload=null;this.rel='stylesheet'">

  <!-- JS'i defer et -->
  <script src="app.bundle.js" defer></script>
</head>
```

#### 3.2 Code Splitting ve Tree Shaking

```javascript
// webpack.config.js - Agresif code splitting
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        // Route bazlı splitting
        routes: {
          test: /[\\/]src[\\/]pages[\\/]/,
          name(module) {
            return `route-${module.context.match(/[\\/]pages[\\/](.*)/)[1]}`;
          },
        }
      }
    }
  }
};

// React'ta lazy loading
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Profile = React.lazy(() => import('./pages/Profile'));

// Suspense ile birlikte
<Suspense fallback={<LoadingSkeleton />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile" element={<Profile />} />
  </Routes>
</Suspense>
```

#### 3.3 Resource Hints

```html
<!-- DNS Prefetch: DNS çözümlemesini önceden yap -->
<link rel="dns-prefetch" href="//api.example.com">

<!-- Preconnect: TCP + TLS handshake'i önceden yap -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://api.example.com" crossorigin>

<!-- Preload: Kritik kaynakları öncelikle yükle -->
<link rel="preload" href="hero-image.webp" as="image">
<link rel="preload" href="critical-font.woff2" as="font" type="font/woff2" crossorigin>

<!-- Prefetch: Sonraki sayfa için arka planda yükle -->
<link rel="prefetch" href="/next-page.js" as="script">
```

---

### React Native / Mobile Cold Start

Mobile native uygulamalarda cold start daha kritiktir çünkü:
- JS bundle parse edilmesi CPU-bound işlemdir
- Hermes JS engine'i önemli iyileştirme sağlar

```javascript
// metro.config.js - Bundle optimizasyonu
module.exports = {
  transformer: {
    // Hermes bytecode'a compile et
    hermesCommand: require('react-native/sdks/hermesc'),
    enableBabelRCLookup: false,
  },
  resolver: {
    // Kullanılmayan platform kodlarını dışla
    platforms: ['ios', 'android'],
  }
};
```

```javascript
// RAM Bundles ile lazy loading
import('./HeavyComponent').then(module => {
  // Component sadece gerektiğinde yüklenir
});

// Inline requires ile daha da optimize
// babel.config.js
module.exports = {
  plugins: [
    ['transform-inline-requires', {
      ignoredRequireNames: ['React', 'ReactNative']
    }]
  ]
};
```

#### iOS Specifics: Launch Screen Optimizasyonu

```swift
// AppDelegate.swift - Minimum iş yap
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    // KÖTÜ: Ağır işlemleri burada yapma
    // loadAllUserData()     // ❌
    // initializeDatabase()  // ❌ (async yap)
    // setupAnalytics()      // ❌ (defer et)

    // İYİ: Sadece kritik setup
    setupRootViewController() // ✓
    
    // Ağır işleri background thread'e at
    DispatchQueue.global(qos: .background).async {
      self.initializeDatabase()
    }

    return true
  }
}
```

#### Android Specifics: Application.onCreate Optimizasyonu

```kotlin
// Application.kt
class MyApplication : Application() {

  override fun onCreate() {
    super.onCreate()

    // StrictMode ile yavaş operasyonları tespit et (debug only)
    if (BuildConfig.DEBUG) {
      StrictMode.setThreadPolicy(
        StrictMode.ThreadPolicy.Builder()
          .detectDiskReads()
          .detectNetwork()
          .penaltyLog()
          .build()
      )
    }

    // Lazy initialization
    initializeCritical()
    
    // Non-critical'ı defer et
    Handler(Looper.getMainLooper()).postDelayed({
      initializeAnalytics()
      initializeNonCriticalSDKs()
    }, 1000) // İlk render'dan sonra
  }

  private fun initializeCritical() {
    // Sadece ilk frame için gerekli şeyler
    setupTheme()
    setupNavigation()
  }
}
```

---

### Service Worker ile Repeat Visit Optimizasyonu

Cold start'ı "warm start"a dönüştür:

```javascript
// service-worker.js
const CACHE_VERSION = 'v2';
const CRITICAL_CACHE = `critical-${CACHE_VERSION}`;

// Install: Kritik kaynakları önbelleğe al
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CRITICAL_CACHE).then(cache =>
      cache.addAll([
        '/',
        '/index.html',
        '/app.bundle.js',
        '/critical.css',
        '/fonts/primary.woff2',
      ])
    )
  );
  self.skipWaiting(); // Hemen aktive et
});

// Fetch: Cache-first stratejisi
self.addEventListener('fetch', event => {
  if (event.request.destination === 'document') {
    // HTML için stale-while-revalidate
    event.respondWith(
      caches.match(event.request).then(cached => {
        const fetchPromise = fetch(event.request).then(response => {
          caches.open(CRITICAL_CACHE).then(cache =>
            cache.put(event.request, response.clone())
          );
          return response;
        });
        return cached || fetchPromise; // Cache varsa hemen dön
      })
    );
  }
});
```

---

### Ölçüm: Core Web Vitals ile Cold Start Takibi

```javascript
// web-vitals kütüphanesi ile metrik toplama
import { getLCP, getFID, getCLS, getTTFB, getFCP } from 'web-vitals';

function sendToAnalytics({ name, value, id }) {
  const payload = {
    metric: name,
    value: Math.round(name === 'CLS' ? value * 1000 : value),
    id,
    url: window.location.href,
    timestamp: Date.now(),
  };

  // Beacon API ile ana thread'i bloklamadan gönder
  navigator.sendBeacon('/analytics', JSON.stringify(payload));
}

getLCP(sendToAnalytics);  // Largest Contentful Paint (hedef: <2.5s)
getFID(sendToAnalytics);  // First Input Delay (hedef: <100ms)
getCLS(sendToAnalytics);  // Cumulative Layout Shift (hedef: <0.1)
getTTFB(sendToAnalytics); // Time to First Byte (hedef: <600ms)
getFCP(sendToAnalytics);  // First Contentful Paint (hedef: <1.8s)
```

---

## 4. Üçünü Birlikte Düşünmek: Holistic Performans Modeli {#holistic}

Bu üç konu birbirinden bağımsız değildir. Şöyle bir senaryo düşün:

> Kullanıcı uygulamayı açar (Cold Start sorunu), scroll yapar (GPU Rendering sorunu), 2 saat kullanır (Memory Leak sorunu), uygulama çöker.

Kullanıcının deneyimi 3 ayrı problemin toplamıdır.

### Performans Bütçesi Belirle

```javascript
// performance-budget.json
{
  "bundles": [
    { "path": "dist/app.bundle.js", "maxSize": "200 kB" },
    { "path": "dist/vendors.js",    "maxSize": "150 kB" }
  ],
  "metrics": {
    "FCP":  { "target": 1800, "budget": 2500 },
    "LCP":  { "target": 2500, "budget": 4000 },
    "TTI":  { "target": 3500, "budget": 5000 },
    "heap": { "target": "50MB", "budget": "100MB" }
  }
}
```

### CI/CD'ye Performans Testleri Ekle

```yaml
# .github/workflows/performance.yml
name: Performance Budget Check

on: [pull_request]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          budgetPath: ./performance-budget.json
          uploadArtifacts: true

  bundle-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: preactjs/compressed-size-action@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          pattern: './dist/**/*.{js,css}'
```

### Monitoring: Production'da Gerçek Zamanlı İzle

```javascript
// Basit bir performance observer
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach(entry => {
    // Long tasks (50ms+) — kullanıcı interaction'ı bloklar
    if (entry.entryType === 'longtask') {
      console.warn(`Long task: ${entry.duration.toFixed(2)}ms`, {
        startTime: entry.startTime,
        attribution: entry.attribution
      });

      // Threshold aşıldıysa alert gönder
      if (entry.duration > 200) {
        reportToMonitoring('long_task', entry);
      }
    }
  });
});

observer.observe({ entryTypes: ['longtask', 'largest-contentful-paint'] });
```

---

## Sonuç: Performans Kültürü

Teknik optimizasyonlar kadar önemli olan şey, **performansı bir kültür haline getirmek**tir.

Birkaç pratik öneri:

1. **Her sprint'e performance ticket ekle.** Teknik borç gibi, performans borcu da birikmeden ödenir.

2. **"Yeterince hızlı" diye bir şey yoktur.** Metrik var, hedef var, bütçe var. Veri ile konuş.

3. **Kullanıcı cihazını simüle et.** MacBook Pro'nda geliştirip mid-range Android'de test et. [Chrome'da CPU throttling kullan: DevTools → Performance → CPU: 4x slowdown]

4. **Regresyon alarmları kur.** LCP 3 saniyeyi geçince Slack'e alert gitsin.

5. **Memory profiling'i düzenli yap.** Haftada bir 10 dakika. Leak'i production'da değil, geliştirme sürecinde yakala.

> **Son söz:** Performans optimizasyonu bir hedef değil, süreçtir. Sistemin çalışması yeterli değildir; *hızlı* çalışması gerekir.

---

*Bu yazı, real-world production deneyimleri ve aşağıdaki kaynaklardan yararlanılarak hazırlanmıştır:*
- *Chrome DevTools Documentation*
- *web.dev/performance (Google)*
- *React Native Performance Docs*
- *MDN Web Performance Guide*

---

**Etiketler:** `#performance` `#webdev` `#mobile` `#javascript` `#optimization` `#senior-dev`
