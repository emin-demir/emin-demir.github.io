---
title: "All Enumerating Subdomains Tools"
date: 2024-03-21
categories: [Siber Güvenlik, Recon]
tags: [subdomain, recon, enumeration, tools]
---

# Açıklama:

Bu yazıda, en çok kullanılan Recon araçlarını sizler için derledim. Ancak burada araçların nasıl kurulacağını veya nasıl kullanılacağını anlatmayacağım; zaten bu bilgiler, ilgili araçların GitHub sayfalarında yer alıyor. Bu yazıda, bir el kitabı gibi, araçların ne işe yaradığını, nasıl çalıştığını ve parametrelerini açıklayarak aktif veya pasif tarama yapabiliyor mu gibi sorulara yanıt vermeye çalıştım. Ayrıca araçların çalışma şekli hakkında fikir vermesi için *help* menülerini de ekledim.

Bu listeyi hazırlamamın temel sebebi, Bug Bounty’de *Recon nasıl yapılır?* diye araştırma yaparken onlarca farklı araçla karşılaşmış olmam. Bu durum, hem kafa karışıklığına hem de tatminsizliğe yol açıyordu. Zamanla, parça parça hepsinin ne olduğunu öğrendikçe GitHub’a ve araçlara daha fazla hâkim oldum. Hatta bir aracın deposuna katkıda bulundum.

Aşağıda, yalnızca subdomain’leri sıralayan araçları, en beğendiğimden en az beğendiğime doğru sıraladım. DNS çözümleyici ve DNS brute force yapan araçları ise listelemedim; çünkü neredeyse tamamı çalışırken DNS sorgu limitine takılıyor ve bağlantımı koparıyordu.

# Subdomain araçları ve Açıklamaları :

# **1. Chaos:**

### Nedir?

Bir web sitesine ve cli aracına sahiptir. DNS veri setleri ile oluşturulmuş çok geniş kapsamlı subdomain tarayıcısıdır. Ayrıyeten nuclei, naabu, Cloud ile beraber çalışma özellikleri ile donatılmış bir araçtır. Benim için Tier 1 de olma sebebi ise çeşitli API veya brute-force gibi uğraşma olmadan çoğu aracın bulduğu subdomainlerden daha fazla subdomaini hazır olarak sunmasıdır. 

### Ek bilgilendirme:

- İnternet sitesi var ve çok modern yapılmış [https://cloud.projectdiscovery.io/](https://cloud.projectdiscovery.io/)
- Normalde diğer araçlarla taradığımda 5 bine yakın subdomain buldu ama sitede 16 bin adet subdomain var yazıyor
- Nuclei ve Naabu ile subdomainlerin tarama akabinde yapılabilir.

 `export PDCP_API_KEY=API_KEY`  # API anahtarını terminalde bir çevre değişkeni olarak tanımlıyoruz.

- Hazır Taranmış domainlerin subdomainlerini otomatik olarak indirebiliyorsunuz.
- **Cloud Entegrasyonu:** Chaos veritabanı, Cloud’a bağlanarak, sürekli güncellenen subdomain bilgilerini sağlar.
- **API Kullanımı :** Chaos API ile sorguları aynı şekilde terminalde de yapılabiliyor
- Ücretsiz hali 60 kullanıma kadar kısıtlı ama zaten subdomain olarak çoğu taramada hazır subdomain listeleri var.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-key string` | ProjectDiscovery için API anahtarı. | `-key your_api_key` |
| `-d string` | subdomain aramak için hedef domain. | `-d example.com` |
| `-count` | Belirtilen domain için istatistikleri gösterir. | `-count` |
| `-silent` | Çıkışı sessiz moda alır. | `-silent` |
| `-o string` | Çıkışı yazdırmak için dosya yolu. | `-o output.txt` |
| `-dL string` | subdomain aramak için domain listesi. | `-dL domains.txt` |
| `-json` | Çıkışı JSON formatında verir. | `-json` |
| `-v`, `-verbose` | Ayrıntılı çıktı modu | `-v` veya `-verbose` |
| `-up`, `-update` | Chaos'u en son sürüme günceller. | `-up` |
| `-duc` | Otomatik güncellemeyi devre dışı bırakır. | `-disable-update-check` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image.png)

---

# **2. OneForAll:**

### Nedir ?

- 6 certificate Modülü, 6 baseline testing Modülleri,  2 web crawler Modülü, 25 DNS datasets Modülleri, 6 DNS queries modülü 6 Threat intelligence Modülü, 16 search engines modülü ile toplamda 67 adet modülü birleştiren Güçlü bir araçtır. Piyasada bulunan bütün modüllerin birleştirip ardından kullanımı basitleştirmeye çalışmak kadar zor bir şey yok ama adamlar bunu başarmışlar. Listede 2. sırada olmasının sebebi de bu. Diğer araçlara ihtiyacınız kalmıyor.

### Artıları:

### Eksileri :

- Çoklu Kaynak Taraması yaptığından en fazla veriyi sağlıyor.
- Diğer toollar'a yeterince güçlü, hızlı, kullanıcı dostu ve Sürekli Destek olmadığından dolayı bu tool geliştirilmiş.
- Python Kullanır. Geliştirilebilir.

- API anahtarlarına ihtiyaç duyuyor.
- Zaman açısından Bütün Modüller çalıştırıldığında çok verimsiz olabiliyor.
- Servis değişirse Modülün güncellenmesi gerekiyor. Yoksa Hatalara sebebiyet verebiliyor.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-d` | Taranacak hedef alan adını belirtir. | `-d example.com` |
| `--alive` | Sadece çalışan (alive) subdomain'leri filtreler. | `--alive` |
| `--fmt` | Çıktı formatını belirler. | `--fmt json` |
| `--level` | Recursive derinlik seviyesi belirler. | `--level 2` |
| `--ports` | Belirtilen portlara tarama yapar. | `--ports 80,443` |
| `--skip-whois` | Whois sorgularını atlar. | `--skip-whois` |
| `--dns` | DNS sorgularını etkinleştirir. | `--dns` |
| `--path` | Çıktının kaydedileceği dizini belirtir. | `--path ./results/` |
| `--disable` | Belirtilen modülleri devre dışı bırakır. | `--disable spyse` |
| `--source` | Özel olarak kullanılacak kaynakları seçer. | `--source crtsh,censys` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image1.png)

---

# **3. Github-Subdomains:**

### Nedir?

Github API desteği ile github repolarını araştırarak ana domainin alt domainlerini bulmak için oluşturulmuş bir araçtır. Diğerlerinin aksine 2-3 aracı birleştirmek yerine kendisine özgü yapısıyla ortaya çıkıyor.

### Artıları:

### Eksileri :

- Github API Tokeni kullanarak arama yapar. Kodlarda herhangi bir subdomain referansı arar.(config. veya env) gibi
- Sadece subdomainleri değil potansiyel kaynak dosyalarında raporluyor.
- Manuel olarak yapılamayacak bir hızda alakalı olan bütün repoları gezer ve geniş bir analiz yapar.
- Kolay Entegrasyon yapılabilir Pipe Line destekler.

- Github API'sına ihtiyaç duyar
- Sıkıntıları : API Günlük sınırı var. / Private Repolara bakamıyor / Hiç alakasız şeyleri subdomain olarak alabilir.
- Verileri aldıktan sonra sağlam bir filtreleme gerekebilir saf veriyle işlem yapmak işleri zorlaştırabilir.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-d` | Ana domaini belirtir. | `github-subdomains -d example.com` |
| `-t` | GitHub API token.  | `github-subdomains -t your_github_token` |
| `-o` | Çıktıyı yazdırmak için dosya yolu. . | `github-subdomains -o output.txt` |
| `-silent` |  Sessiz mod. yalnızca önemli sonuçlar. | `github-subdomains -silent` |
| `-v` | Ayrıntılı çıktıyı etkinleştirir.  | `github-subdomains -v` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image2.png)

---

# **4. Findomain:**

### Nedir ?

Çeşitli meşhur araçlarla(OWASP Amass, sublist3r, assetfinder ) kuşatılmış. API desteğini verdikten sonra gerisini kendi halleden bir araçtır. Kendi Database'ini oluşturup, Webhook'lar ile ( telegram api, discord api ) birleştirip kendi subdomain veri tabanını oluşturmak ve bunu otomatik olarak bildirecek bir sistem yapmak insana başarmışlık hissini gayet iyi veriyor.

### Artıları:

- Kendi Veri tabanını rahat bir şekilde oluşturabiliyor ve bu sayede monitoring yapabiliyorsun. [https://blog.findomain.app/subdomains-enumeration-what-is-how-to-do-it-monitoring-automation-using-webhooks-and-5e0a0c6d9127](https://blog.findomain.app/subdomains-enumeration-what-is-how-to-do-it-monitoring-automation-using-webhooks-and-5e0a0c6d9127) Bunun için güzel bir rehber.
- Web-hook olarak Discord, Telegram, E-mail 'i destekliyor.
- Geniş Parametreleri sayesinde çok hızlı bir şekilde çok az bir cpu ve ram harcayacak çoğu subdomaini elde edebiliyor.
- DNS Desteği de var çözümleme ve brute-force yapabiliyor

### Eksileri :

- Veri tabanı ve web hooklar hakkında bilgi edinmek lazım. Kolay değil bazen kafa karıştırabiliyor.
- En son geçen sene commit almış. Değişen sistemlere ayak uyduramadığında hatalar ortaya çıkabiliyor.

| **Parametre** | **Açıklama** | **Örnek Kullanım** |
| --- | --- | --- |
| `-c, --config config-file` | Yapılandırma dosyasını belirtir. Varsayılan olarak `findomain`  | `--config "/path/config.json"` |
| `-m, --monitoring-flag` | İzleme modunu etkinleştirir. Bulunan subdomainleri takip eder ve bildirir. | `--monitoring-flag` |
| `--external-subdomains` | Harici kaynaklardan (örneğin amass, subfinder) alt alan adlarını alır | `--external-subdomains` |
| `-t, --target <target>` | Hedef alan adını belirtir. | `--target example.com` |
| `-o, --output` | Çıktıyı otomatik oluşturulan bir dosyaya kaydeder  | `--output` `example.com.txt` |
| `-u, --unique-output file` | Tüm çıktıyı belirttiğiniz bir dosyaya kaydeder. | `--unique-output results.txt` |
| `-i, --ip` | Alt alan adlarının IP adreslerini gösterir veya dosyaya yazar. | `--ip` |
| `--http-status` | Alt alan adlarının HTTP durum kodlarını kontrol eder. | `--http-status` |
| `--query-database` | Daha önce keşfedilmiş alt alan adlarını veritabanında sorgular. | `--query-database` |
| `--reset-database` | Veri tabanını sıfırlar. Tüm veriler silinir. | `--reset-database` |
| `--sandbox` | Chrome/Chromium sandbox modunu etkinleştirir. Root ile kullanma | `--sandbox` |
| `--http-timeout <timeout>` | HTTP durum kontrolü için zaman aşımı süresi (saniye). Varsayılan 5. | `--http-timeout 10` |
| `--http-retries <retries>` | HTTP durum kontrolü için deneme sayısı. Varsayılan 1. | `--http-retries 3` |
| `--resolvers <file>` | DNS çözümleyici. IP adresleri listesini içeren dosya belirtir.  | `--resolvers /path/r.txt` |
| `--pscan` | Port tarayıcısını etkinleştirir. | `--pscan` |
| `-j, --jobname <name>` | Farklı hedefleri aynı iş adıyla ilişkilendirmek için bir veritabanı iş adı belirtir. | `--jobname "daily-scan"` |
| `--postgres-user -host - password - port` | PostgreSQL kullanıcı adını , Parolasını, bilgisayar adını , portunu belirtir | `--postgres-user domain_user` |
| `--postgres-database db` | PostgreSQL veritabanı adını belirtir. | `--postgres-database f_db` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image3.png)

![image.png](/assets/images/all-enumerating-subdomains-tools/image4.png)

---

# **5.Sublist3r:**

### Nedir?

Findomain gibi API destekleri ile belirli site veya veri kayıtlarından dönen subdomainleri alıyor. 

### Artıları:

- Pasif keşif yapar, Hafif minimalisttir. Arama motorlarını ve VirusTotal, ThreatCrowd, Netcraft gibi API'lerden ve DNS sunucularından alt alan adı bilgisi toplar.
- Topladığı alan adlarını dns kayıtlarına göre çözümler yalnızca geçerli olanları döndürür. Son zamanlarda Subbrute ve kelime listesi ile dns sorgularını kullanan alan adı keşfini yapıyor.
- V2 si var https://github.com/hxlxmjxbbxs/sublist3rV2 Burada DNSDumpster sorununa da el atmaya çalıştım. Syntax hatalarını ve dnsdumpster'in çalışma şekli form'dan api'a geçiş yaptığı için projeyi forklayıp düzelttim.
- Python altyapılı olduğundan kolay bir şekilde geliştirilebilir oluyor

### Eksileri :

- python 3.6 sürümünü destekliyor sonraki sürümlerde hatalar verebiliyor.
- Eski proje en son 4 sene önce son commit almış
- subbrute eklentisi DNS sorgu limitlerine takılıyor.
- Google Eklentisi hata veriyor. Bot yazılım olduğunu algılıyor.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| -d --domain | Alan adını belirterek alt alan taraması yapma | `sublist3r -d example.com` |
| -b --bruteforce | Kaba kuvvet saldırısı ile alt alan taraması | `sublist3r -d example.com -b` |
| -p --ports | Belirtilen portları tarama | `sublist3r -d example.com -p 80,443` |
| -v --verbose | Detaylı çıktı alma | `sublist3r -d example.com -v` |
| -t --threads | Eşzamanlı işlem sayısını belirleme | `sublist3r -d example.com -t 20` |
| -e --engines | Kullanılacak arama motorlarını belirleme | `sublist3r -d example.com -e google,yahoo` |
| -o --output | Sonuçları dosyaya kaydetme | `sublist3r -d example.com -o sonuclar.txt` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image5.png)

---

# **6. Amass :**

### Nedir?

OWASP tarafından geliştirilen güçlü bir alt alan adı keşif ve haritalama aracıdır. Pasif ve aktif keşif tekniklerini birleştirerek kapsamlı bir veri toplama sunar. Çoklu modül desteğine sahiptir. API , Whois, DNS vb sorguları ile çeşitli yöntemleri mevcuttur. Çıktıları ayrıyeten elden geçirmek gerekebilir. Diğerlerinden biraz daha işlemesi zordur.

### Pasif Veri Toplama :

- **DNS çözümleyiciler**: Alt alan adlarını ve IP'leri bulmak için kullanılır.
- **WHOIS verileri**: Alan adı kayıt bilgilerini analiz eder.
- **Sertifika şeffaflık logları**: SSL/TLS sertifikalarındaki alan adlarını kontrol eder.
- **API Entegrasyonları**: SecurityTrails, VirusTotal, Shodan gibi hizmetlerle entegredir.
- Bir çok çeşit subdomain araştırma kombinasyonları oluşturmak lazım.
    
    ### **Amass İntel :** Bilgi Toplama
    
    Hedef ağ veya alan hakkında bilgi toplamak için kullanılır. 
    
    | **Parametre** | **Açıklama** |
    | --- | --- |
    | `-d DOMAIN` | Hedef domain |
    | `-whois` | Reverse WHOIS sorgusu |
    | `-active` | Sertifika adını yakalama girişimi yapar |
    | `-asn VALUE` | ASN numarası belirtir : Enum |
    | `-addr VALUE` | IP adreslerini belirtir ÖRN:  192.168.1.1-254 : Enum |
    | `-cidr VALUE` | CIDR belirtir : 192.168.1.0/24,10.0.0.0/16 : Enum |
    | `-ip` | Keşfedilen IP adreslerini gösterir |
    | `-df VALUE` | Domain listesini bir dosyadan okur |
    | `-ef VALUE` | Hariç tutulacak veri kaynakları |
    | `-include VALUE` | Belirli veri kaynaklarını dahil eder |
    | `-exclude VALUE` | Belirli veri kaynaklarını hariç tutar |
    | `-timeout VALUE` | Zaman limiti belirler |
    | `-list` | Kullanılabilir veri kaynaklarını listeler |
    | `-demo` | Sansürlü bir çıktı oluşturur |
    | `-r` or `-rf` | Dns çözümleyici kullanma  |
    
    ![image.png](/assets/images/all-enumerating-subdomains-tools/image6.png)
    

### **Aktif Veri Toplama :**

- **DNS zone transfer denemeleri**: Alan adı sunucularından detaylı bilgi çeker.
- **DNS brute-forcing**: Alt alan adlarını brute-force yöntemiyle keşfeder.
- **Port taramaları**: Açık portları ve çalışan hizmetleri bulur
- Bu toolun kullanımı Basit değil. Ne istediğini bilen ve istediği sonucu çıktıdan ayırabilecek insanlara hitap ediyor.
- Birçok modülü içerisinde bulundurduğundan çok kapsamlı ve geniş bir tarama yapıyor.

### Amass Enum : Keşif ve Ağ haritalandırma

Hedef domain ve IP aralığı üzerinde detaylı bir keşif çalışması yapar. Amass'ın en güçlü özelliklerini barındırır.

| **Parametre** | **Açıklama** |
| --- | --- |
| `-d DOMAIN` |  Keşif yapılacak hedef domain |
| `-brute` | Brute-force ile alt alan adı araması |
| `-o` | Çıktıyı bir dosyaya kaydeder |
| `-p Value` | Hedef portları kontrol eder |
| `-dir` | Çıkış dosyalarının kaydedileceği dizin |
| `-config` | YAML dosyasını kullanarak özelleştirme |
| `-alts` |  Alan adı varyasyonları üretir |
| `-aw VALUE` |  Özel wordlist dosyası kullanır |
| `-iface` | Trafiği belirli bir ağ arayüzünden gönderir |
| `-nf VALUE` | Bilinen subdomain'leri dışlar |
| `-max-depth Value` | Alt alan adı brute-force derinliği |
| `-dns-qps Value` | DNS sorguları/saniye limiti : İntel |
| `-scripts Value` | ADS scriptleri kullanır |
| `-passive` | Pasif Keşfi çalıştırır |
| `-active` | Alan transferi ve sertifika ismi yakalamayı etkinleştirir. |

![image.png](/assets/images/all-enumerating-subdomains-tools/image7.png)

# **7. TheHarverster:**

### Nedir?

Pasif tarama ile düşük bir güç tüketimiyle beraber birçok çıktı vermeyi amaçlar. Çoklu Modül desteğine sahip. 

### Artıları:

- Pasif Taraması oldukça geniş bir araçtır.
- ScreenShoot alabilir.
- Farklı Modüllerden birçok veri desteği sağlanıyor
- Aşağıda Bulunan Modüllere sahiptir:

Anubis, BeVigil, Baidu, BinaryEdge, Bing, BingAPI, Brave, BufferOverrun, Censys, CertSpotter, CriminalIP, Crtsh, DNSDumpster, DuckDuckGo, FullHunt, GitHub-Code, HackerTarget, Hunter, HunterHow, Intelx, Netlas, Onyphe, OTX, PentestTools, ProjectDiscovery, RapidDNS, RocketReach, SecurityTrails, Shodan, SiteDossier, SubdomainCenter, SubdomainFinderC99, ThreatMiner, Tomba, URLScan, Vhost, VirusTotal, Yahoo, ZoomEye.

### Eksileri :

- Bazı modüllerin API'ları paralıdır.
- Art Arda çalıştırmak bazı güvenlik duvarları tarafından işaretlenebilir.(Örn: Linkedin)
- Çalıştırırken hatalar verdi. sürekli kullanımda en optimal ayarı öğrenmek lazım.
- Pipe Line desteği yok

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-d, --domain` | Araştırılacak şirket adı veya domain. | `theHarvester -d example.com` |
| `-l, --limit` | Arama sonuçlarının sınırını belirler (varsayılan: 500). | `theHarvester -d example.com -l 100` |
| `-S, --start` | Başlangıç sonucu numarası (varsayılan: 0). | `theHarvester -d example.com -S 10` |
| `-p, --proxies` | proxy bilgilerini  `/root/.proxies.yaml` dosyasına girin. | `theHarvester -p` |
| `-s, --shodan` | Bulunan ana ip addreslerini sorgulamak için Shodan'ı kullanır. | `theHarvester -d example.com -s` |
| `--screenshot` | domainlerin ekran görüntüsünü alır. çıktı dizinini belirtilmesi lazım. | `theHarvester --screenshot ./screenshots` |
| `-v, --virtual-host` | DNS çözümlemesi ile virutal-host'ları arar | `theHarvester -d example.com -v` |
| `-e, --dns-server` | DNS çözümlemesi için kullanılacak DNS sunucusunu belirtir. | `theHarvester -d example.com -e 8.8.8.8` |
| `-t, --take-over` | Takeover ( Hala kullanılıyor mu) Kontrolü yapar | `theHarvester -d example.com -t` |
| `-r, --dns-resolve` | Resolver listesiyle veya belirli resolverlarla DNS çözümlemesi yapar. | `theHarvester -d example.com -r` |
| `-n, --dns-lookup` | DNS sunucusu sorgulamasını etkinleştirir (varsayılan: False). | `theHarvester -d example.com -n` |
| `-c, --dns-brute` | Domain üzerinde DNS brute force yapar. | `theHarvester -d example.com -c` |
| `-f, --filename` | Sonuçları XML ve JSON dosyasına kaydeder. | `theHarvester -d example.com -f results.xml` |
| `-b, --source` | Kullanılacak kaynakları belirtir (ör. `bing`, `crtsh`, vb.). | `theHarvester -d example.com -b bing,crtsh` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image8.png)

# **8. AssetFinder:**

Nedir? 

Tomnomnom'un geliştirmiş olduğu bir araçtır. Sublist3r ve Findomain ile aynı işi yapıyor.

### Artıları:

- Kullanımı Aşırı basit. Pasif tarama yapar.
- Diğer Araçlarla kolay bir şekilde kullanılabilir.
- Kullanımı Aşırı Basit API'ları ekle siteyi yaz yeter.
- —subs-only parametresi var.

### Eksileri :

- Son commit yine 4 yıl önce atılmış
- Hatalar verebiliyor.

Sade tarama hiçbir API eklemedim.

![image.png](/assets/images/all-enumerating-subdomains-tools/image9.png)

---

# **9. Gau:**

### Nedir?

GetAllUrls açıklamasıyla olan gau bir domain için çeşitli kaynaklardan Url toplamak amacıyla kullanılan bir tooldur. Subdomain, Endpoint veya saldırı yüzeyini genişletmek için kullanılabilir ama özellikle endpoint keşfi için kullanılıyor.

### Artıları:

- Tam anlamıyla subdomain searcher aracı değildir. - -subs parametresi ile biraz daha aktif oluyor.
- Hızlı Hafif Çalışması,  Pipeline kullanımı ( httpx, nuclei ) Basitliği ile önplana çıkıyor.

### Eksileri :

- Wayback Machine'den kaynaklar geldiği için Url'lerin ayrıştırılması ve düzenlenmesi lazım. Bazı Urller aktif olmayabilir.

| **Parametre** | **Açıklama** | **Örnek** | **Parametre**  | **Açıklama** | **Örnek** |
| --- | --- | --- | --- | --- | --- |
| `--blacklist` | Atlanacak uzantılar listesi | `gau --blacklist svg,png` | `--config` | Alternatif yapılandırma dosyası | `gau --config $HOME/.config/gau.toml` |
| `--fc` | Filtrelenecek durum kodları listesi | `gau --fc 404,302` | `--from` | Tarihten itibaren URL çek | `gau --from 202101` |
| `--ft` | Filtrelenecek MIME türleri listesi | `gau --ft text/plain` | `--fp` | Aynı endpoint'in farklı parametresini kaldır | `gau --fp` |
| `--json` | Çıktıyı JSON formatında yazdır | `gau --json` | `--mc` | Eşleştirilecek durum kodları listesi | `gau --mc 200,500` |
| `--mt` | Eşleştirilecek MIME türleri listesi | `gau --mt text/html/json` | `--o` | Sonuçların yazılacağı dosya | `gau --o out.txt` |
| `--providers` | Kullanılacak sağlayıcılar  | `gau --providers wayback` | `--proxy` | Kullanılacak HTTP proxy | `gau --proxy http://proxy.com:8080` |
| `--retries` | HTTP istemcisi için tekrar deneme sayısı | `gau --retries 10` | `--timeout` | HTTP istemcisi için zaman aşımı (saniye) | `gau --timeout 60` |
| `--subs` | Hedef domain'in alt alan adlarını dahil et | `gau example.com --subs` | `--threads` | Başlatılacak iş parçacığı sayısı | `gau example.com --threads` |
| `--to` | Belirtilen tarihe kadar URL çek | `gau example.com --to 202101` | `--verbose` | Ayrıntılı çıktıyı göster | `gau --verbose example.com` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image10.png)

# DNS çözümleyiciler ve DNS brute force araçları :

# **1.Knock:**

### Nedir?

Python3 ile geliştirilmiş, pasif subdomain taramaları yapan bir araçtır.

### Artıları:

- DNS Zone'larına Brute force yapar
- Dns Kayıtlarını otomatik bypasslar.
- python-dnspython kütüphanesini kullanıyor
- Sistemdeki Default Dns konfigürasyonuna göre tarar 8.8.8.8 gibi
- `google`, `duckduckgo`, ile `virustotal` ile subdomain belirler

### Eksileri:

- Pipe Line uygun değildir.
- Çıktıların dikkatlice bakılması lazım ayrıştırmak zahmetli olabilir.
- Dns Sorgu Limiti

Bunu araştırırken öğrendiğim bir bilgi  sudo ln -s /opt/my_app/knock.py /usr/local/bin/knock ile sembolik bir link oluşturursanız knock yazarak çalıştırabilirsiniz.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| -d, --domain | Analiz edilecek domain | `knockpy -d example.com` |
| -f, --file | Domain listesi içeren dosya | `knockpy -f domains.txt` |
| --dns | Özel DNS sunucusu kullanımı | `knockpy -d example.com --dns 8.8.8.8` |
| --timeout | Zaman aşımı süresi belirleme | `knockpy -d example.com --timeout 10` |
| --threads | Thread sayısını belirleme | `knockpy -d example.com --threads 20` |
| --recon | Subdomain keşfi yapar | `knockpy -d example.com --recon` |
| --bruteforce | Subdomain kaba kuvvet taraması | `knockpy -d example.com --bruteforce` |
| --wordlist | wordlist kullanımı (bruteforce) | `knockpy -d example.com --bruteforce --wordlist list.txt` |
| --wildcard | Wildcard testi yapar | `knockpy -d example.com --wildcard` |
| --json | JSON formatında çıktı | `knockpy -d example.com --json` |
| --save | Raporu klasöre kaydetme | `knockpy -d example.com --save reports` |
| --report | Kaydedilmiş raporu gösterir | `knockpy --report saved_report` |
| Tam Tarama: | Olmayabilir Kontrol et  | `knockpy -d domain.com --recon --bruteforce` |
| API anahtarı : | Olmayabilir Kontrol et  | `export API_KEY_VIRUSTOTAL=your-virustotal-api-key` |
| Rapor Kayıt : | `knockpy -d domain.com`  | `--recon --bruteforce --save report` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image11.png)

# **2. DNSRecon:**

- Subdomainleri Zone transfer veya  DNS sorgularıyla araştıran bir sistemdir.
- Bruteforce yöntemleri de mevcut.
- python tabanlı olduğu için kolayca özelleştirilebilir.
- Standart bir DNS ile tarama aracı

| **Parametre** | **Açıklama** | **Örnek Kullanım** |
| --- | --- | --- |
| `-d` | Hedef domain adı | `dnsrecon -d example.com` |
| `-t` | Sorgu türü (AXFR, zonewalk, std, brute vb.) | `dnsrecon -d example.com -t std` |
| `-z` | Zone transfer testi | `dnsrecon -d example.com -z` |
| `-D` | Alt alan adlarını brute force için liste | `dnsrecon -d example.com -D subdomains.txt` |
| `-r` | IP aralığı üzerinde ters DNS taraması | `dnsrecon -r 192.168.1.0/24` |
| `-s` | Servis keşfi | `dnsrecon -d example.com -s` |
| `-j` | Çıkışları JSON formatında kaydetme | `dnsrecon -d example.com -j output.json` |
| `-c` | DNS keşfi sırasında kullanılacak çözümleyici | `dnsrecon -d example.com -c 8.8.8.8` |

# **3. Puredns:**

- Hızlı Brute force dns çözümleme
- Massdns ile çoklu dns çözümleme işlemi
- WildCard tespiti, wildcard filtreleme işi
- Yardımcı Wordlistleri kendileri sunuyorlar zaten
- Gayet iyi çalışıyor gibi gözüküyor. Ne kadar kaldığını vs de gösteriyor.
- resolve, bruteforce, -q quiet parametreleri var sadece. basit ama sağlam bir şey gibi duruyor.

![image.png](/assets/images/all-enumerating-subdomains-tools/image12.png)

---

# **4. ShuffleDNS:**

### Nedir?

ProjectDiscovery'nin geliştirdiği bir araçtır.  Hızlı esnek Pipe Lie destekleyen bir yapısı var. Birden fazla modül destekliyor.

### Artıları:

- Pasif ve aktif DNS çözümleme yaparak subdomainleri hızlı ve verimli bir şekilde toplar ve doğrulamasını yapar.
- Hızlı bir DNS çözümleyicidir. Bilinen Subdomainlerin doğrulamasını yapar.
- DNS Veri tabanını kontrol ederek veyahut da İp adresini kullanarak DNS çözümlemesi yapar.
- Resolvers olarak Wordlist gerekli.
- Amass, Subfinder, dnsx ve birçok araçla kolay bir şekilde entegre şekilde çalışır

### Eksileri:

- Pasif taramalarındaki veriler bazen güncel olmayabiliyor.
- Çok fazla DNS çözümleyici sorgusu gönderildiğinde sorgu sınırı ile karşılaşılabiliyor.
- DNS resolvers'lerinin kalitesine göre çalışıyor.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-list` | Çözümlenecek subdomain listesinin dosya yolu. | `shuffledns -list subdomains.txt` |
| `-r` | DNS çözümleyici dosyasının yolu. | `shuffledns -r resolvers.txt` |
| `-d` | Hedef domain belirler. | `shuffledns -d example.com` |
| `-o` | Çıktıyı yazdırmak için dosya yolu. | `shuffledns -o output.txt` |
| `-silent` | Çıktıyı sessiz moda alır (yalnızca sonuçlar). | `shuffledns -silent` |
| `-t` | Paralel çalışan iş parçacığı sayısını ayarlar. | `shuffledns -t 50` |
| `-massdns` | MassDNS aracı ile entegre çalışır. | `shuffledns -massdns /path/massdns` |
| `-strict` | Çıktıda yalnızca çözümlenen subdomain'leri gösterir. | `shuffledns -strict` |
| `-json` | Çıktıyı JSON formatında verir. | `shuffledns -json` |
| `-timeout` | DNS çözümleme için zaman aşımı süresi (saniye). | `shuffledns -timeout 10` |
| `-ip` | Subdomain'lerin çözümlediği IP adreslerini listeler. | `shuffledns -ip` |
| `-w` | Passively çözümleme için Wordlist dosyası kullanır. | `shuffledns -w wordlist.txt` |
| `-retries` | Çözümleme sırasında tekrar deneme sayısını belirler. | `shuffledns -retries 5` |
| `-flush` | Çözümleyici önbelleğini temizler. | `shuffledns -flush` |
| `-v` | Ayrıntılı çıktıyı etkinleştirir. | `shuffledns -v` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image13.png)

---

# **5. dnsX:**

### Nedir?

dnsx, retryabledns kütüphanesi ile tasarlanmış çok amaçlı bir DNS araç takımıdır. Birden fazla DNS sorgusunu, resolver.txt 'leri ve shuffledns gibi DNS joker karakter filtrelemesini vb. destekleyen bir araçtır. GO diliyle yazılmıştır.

### Artıları:

- Basit ve kullanışlı bir  yapısı var
- Brute-force, Özel DNS resolver listesini, TCP/UDP/DOH/DOT, Pipe Line yani stdin | stdout ve wildcard ( subdomainlerin varlığının sorgulanması )  destekliyor
- Çeşitli parametrelerle sorgu yapısı rahat şekillendirilebilir.

### Eksileri :

- DNS sorgu sınırı sıkıntısı yaşayabiliyior
- subdomain bulmak için çok sade bir sistem ve sadece dns araştırması için kullanılabilir.

| **Parametre** | **Açıklama** | **Örnek** |
| --- | --- | --- |
| `-a` , `-aaaa` , `-txt` | (A, AAAA, CNAME, NS, TXT, vb.). kaydını Teker teker sorgular. | `dnsx -a example.com` |
| `-recon` | Tüm DNS kayıtlarını sorgular (A, AAAA, CNAME, NS, TXT, vb.). | `dnsx -recon example.com` |
| `-r <resolver_file>` | Özel DNS resolver listesini kullanır. | `dnsx -r resolvers.txt example.com` |
| `-w <wordlist>` | Subdomain brute-force için kelime listesi kullanır. | `dnsx -w subdomains.txt example.com` |
| `-t <threads>` | Paralel iş parçacığı sayısını ayarlar. | `dnsx -t 50 example.com` |
| `-o <output_file>` | Sonuçları belirtilen dosyaya kaydeder. | `dnsx -o output.txt example.com` |
| `-j` | JSON formatında çıktı verir. | `dnsx -j example.com` |
| `-rcode <status_code>` | Belirtilen DNS durum koduna göre sonuçları filtreler. | `dnsx -rcode noerror example.com` |
| `-cdn` | CDN adını görüntüler. | `dnsx -cdn example.com` |
| `-asn` | ASN (Autonomous System Number) bilgilerini görüntüler. | `dnsx -asn example.com` |
| `-rl <rate_limit>` | DNS sorgu hızını limitler (saniye başına istek sayısı). | `dnsx -rl 10 example.com` |
| `-up` | dnsx aracını en son sürüme günceller. | `dnsx -up` |
| `-stats` | Tarama istatistiklerini görüntüler. | `dnsx -stats example.com` |
| `-silent` | Yalnızca sonuçları gösterir, diğer çıktıları gizler. | `dnsx -silent example.com` |
| `-version` | dnsx aracının sürümünü gösterir. | `dnsx -version` |

![image.png](/assets/images/all-enumerating-subdomains-tools/image14.png)