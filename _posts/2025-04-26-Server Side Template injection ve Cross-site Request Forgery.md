---
title: "Server Side Template injection ve Cross-site Request Forgery"
date: 2025-04-26
categories: [Siber Güvenlik, HandBook]
tags: [SSTI, CSTI, Handbook, Learn]
---

# Server Side Template İnjection Nedir ?
SSTI zafiyetini tanımadan önce bilmemiz gereken bir şey var. Bu zafiyetin oluşmasındaki temel bileşen nedir?

## Template Engines (Şablon Motoru)

Şablon motorları, önceden oluşturulmuş yapıların (template), dinamik verilerle birleştirilerek web uygulamalarında yanıtlar oluşturmak için kullanılır. Popüler şablon motorlarından bazıları **Jinja** ve **Twig**'dir.

Şablon motorları tipik olarak iki bileşene ihtiyaç duyar: biri şablonun kendisi, diğeri ise şablonun içerisine yerleştirilecek **value** (değer). Bu değerler bir string, sayısal veri veya bir dosya olabilir ve genellikle `key-value` (anahtar-değer) çiftleri olarak sağlanır. Şablon motoru, şablon içinde belirtilen anahtara karşılık gelen değeri yerleştirir. Bu işleme **rendering** (işleme/render etme) denir.

Örnek olarak bir Jinja2 şablon motorunda string alanını dinamik olarak nasıl değiştirebildiğimize bakalım:

```Jinja2
Hello {{ name }}
```

Bu şablon sadece `name` verisini alır. Örneğin `name = "Ahmet"` olarak sağlandığında, şablon motoru bu değeri yerine yerleştirerek "Hello Ahmet" sonucunu üretir. Bu, temel bir şablon motoru kullanım örneğidir.

Birçok modern şablon motoru yalnızca değer yerleştirmeyi değil; koşullar, döngüler gibi programlama dillerinin sunduğu daha karmaşık yapıları da destekler. Örneğin:

```Jinja2
{% for name in names %}
Hello {{ name }}!
{% endfor %}
```

Eğer `names = ["vautia", "21y4d", "Pedant"]` şeklinde bir veri girilirse, şablon çıktısı şu şekilde olur:

```nginx
Hello vautia!
Hello 21y4d!
Hello Pedant!
```
Bu şekilde, şablon motorları sayesinde statik içerikler yerine dinamik olarak üretilmiş çıktılar elde edebiliriz.

## Server-side Template Injection

## SSTI Nedir?

Önceki bölümde anlattığımız gibi, şablon motorları sunucu tarafında çalışan yapılardır. Kullanıcıdan gelen verilerle, belirli bir şablon formatına göre dinamik içerikler üretirler. Ancak bu mekanizma, kötüye kullanıma açık olabilir. Eğer saldırgan, uygulamanın işlediği veri alanlarına zararlı bir kod enjekte edebilirse, şablon motoru bu kodu yorumlayabilir ve çalıştırabilir. Bu durum, saldırganın sunucu üzerinde kod çalıştırmasına, hatta sistemi tamamen ele geçirmesine neden olabilir.

## SSTI Tespiti

Bir SSTI zafiyetinden faydalanmadan önce, bu zafiyetin gerçekten var olup olmadığını tespit etmek oldukça önemlidir. Ayrıca, sömürme süreci büyük ölçüde kullanılan şablon motoruna bağlıdır. Çünkü her şablon motoru farklı bir söz dizimi (syntax) kullanır. Bu nedenle, hedef uygulamanın hangi şablon motorunu kullandığını tespit etmek gerekir.

Şablon motorunu tanımlamak için kullanılan yaygın yöntemlerden biri, hata mesajları tetikleyerek söz dizimi hakkında ipuçları edinmektir. Aşağıdaki gibi rastgele ancak tipik SSTI yapıları içeren bir test girdisiyle başlayabilirsiniz:

```Jinja2
${{<%[%'"}}%\.
```

SQL injection'da `'` karakteri nasıl test amacıyla kullanılıyorsa, SSTI'de de benzer şekilde özel karakterler veya yapılarla sistemin davranışı gözlemlenir. Eğer sunucu bu giriş karşısında bir hata (örneğin HTTP 500) döndürürse, SSTI'den şüphelenilebilir.

Bir diğer yaygın yöntem ise farklı şablon motorlarının söz dizimlerini kullanarak matematiksel işlemleri enjekte etmektir. Bu işlemler başarılı bir şekilde değerlendirilirse, hangi motorun kullanıldığını anlamak mümkün olur.
Aşağıda yer alan görselde, farklı şablon motorlarının nasıl yanıtlar ürettiğine dair bir karar ağacı gösterilmektedir:
![[Pasted image 20250429163208.png]]

Örneğin; eğer `{{7*'7'}}` ifadesi 49 döndürüyorsa bu büyük olasılıkla Twig kullanıldığını gösterir. Ancak çıktı `7777777` ise, Jinja2 motoru kullanılıyor olabilir. Bu tür testler ile şablon motorunun türü belirlenebilir.


## Zafiyetin Sömürülmesi
Şablon motorunu doğru çalıştırabilmek için öncelikle sözdiziminin nasıl çalıştığını öğrenmek iyi bir fikirdir. Basit bir sözdizimi yerleştirme ile bir zafiyeti hızlıca tetikleyebilirsiniz. Örneğin, bir web uygulaması hata mesajlarını doğrudan kullanıcıdan gelen URL parametresine göre oluşturuyorsa, bu alan SSTI için potansiyel bir aday olabilir.

Aşağıdaki gibi bir test yapılabilir:
![[Pasted image 20250429175637.png]]
SSTI zafiyetini tetiklemek için aritmetik işlemlerin karşılığına bakıyoruz. 
![[Pasted image 20250429180027.png]]
Eğer çıktı "49" olarak dönerse, Uygulamanın hata vermesini sağlayarak verdiği hataya göre hangi şablon motorunu kullandığını öğrenebiliriz;

![[Pasted image 20250429180335.png]]
Sistemin verdiği hata bakıp Ruby üzerinde çalıştığını anlıyoruz. aşağıdaki komut gibi bir kod parçası çalıştırılarak **uzaktan komut çalıştırma (RCE)** denenebilir:
`<%= system("whoami") %> `
Bu şekilde sistemde çalışan kullanıcı bilgisi görüntülenebilir. Eğer bu komut başarılı olursa, saldırgan artık hedef sistem üzerinde keyfi komutlar çalıştırabilir, zararlı yazılım yükleyebilir veya kalıcı erişim sağlayabilir.

### SSTI Zafiyetinin Etkileri

Bazı durumlarda, SSTI zafiyetleri sistem üzerinde doğrudan bir risk oluşturmaz. Ancak çoğu senaryoda bu zafiyetin etkileri oldukça yıkıcı olabilir. Bir saldırgan:

- Uzaktan komut çalıştırabilir (Remote Code Execution),
- Sunucunun tam kontrolünü ele geçirebilir,
- Dahili sistemlere pivot yapabilir (lateral movement),
- Hassas dosyalara erişebilir,
- Yetkisiz veri sızıntılarına yol açabilir.

Bu nedenle SSTI, özellikle modern web uygulamalarında kritik öneme sahip bir güvenlik açığıdır.

### SSTI Zafiyetleri Nasıl Önlenir?

SSTI zafiyetlerini önlemenin en etkili yolu, kullanıcı tarafından sağlanan verilerin şablon motoru tarafından doğrudan işlenmesini tamamen devre dışı bırakmaktır. Yani, kullanıcı girdilerinin şablon mantığı içerisinde yorumlanmaması sağlanmalıdır.

Ancak bazı iş gereksinimlerinden dolayı bu durum her zaman mümkün olmayabilir. Böyle senaryolarda, aşağıdaki önlemler alınarak riskler minimize edilebilir:

- **Güvenli Şablon Kullanımı:** Şablon motoru, yalnızca geliştirici tarafından belirlenmiş değişkenleri işleyecek şekilde yapılandırılmalı ve kullanıcı girdileri asla doğrudan `render` işlemlerine sokulmamalıdır.
- **Girdi Doğrulama ve Filtreleme:** Kullanıcıdan gelen tüm veriler, mümkünse beyaz liste yöntemiyle doğrulanmalı; şablon motoruna özel kontrol karakterleri (`{{`, `{%`, `<%` gibi) filtrelenmelidir.
- **Şablon Motoru Özelliklerinin Sınırlandırılması:** Kullanılmayan veya riskli işlevler (örneğin `eval`, `exec`, `system` gibi fonksiyonlara erişim) devre dışı bırakılmalıdır. Bazı motorlar güvenli modlar sunar.
- **Uygulama İzolasyonu:** Web uygulaması, mümkünse ayrıcalıksız bir kullanıcı altında ve sınırlı yetkilere sahip bir ortamda çalıştırılmalıdır. En etkili yöntemlerden biri, uygulamayı bir **Docker kapsayıcısı** içinde çalıştırarak, yürütme ortamını izole etmektir. Böylece bir sömürü durumunda saldırının kapsayıcı dışına yayılması engellenmiş olur.
- **Güncellemeler ve Güvenlik Yama Yönetimi:** Şablon motorları ve uygulama bağımlılıkları düzenli olarak güncellenmeli, bilinen zafiyetlere karşı yamalar uygulanmalıdır.

Bu önlemlerle SSTI zafiyetlerinin sömürülmesi büyük ölçüde zorlaştırılabilir. Ancak her zaman "en iyi güvenlik, gereksiz saldırı yüzeyini ortadan kaldırmaktır" ilkesi geçerlidir. Kullanıcı verilerinin yorumlanması gereken durumlarda ekstra dikkat gösterilmelidir.
