# 📘 Seri Haberleşme (HSER) + Motor Kontrol — Konu Anlatımı

> **Kaynak Dosya:** [A_Ve_D_Ve_S_Motor.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/A_Ve_D_Ve_S_Motor.pbp)
> **Konu:** Seri port üzerinden bilgisayardan komut alıp step motor yön kontrolü

---

## 📌 1. Bu Kod Ne Yapıyor?

Bilgisayardan seri port (UART) üzerinden **klavye tuşlarına** basarak bir **step motoru** kontrol eder:

| Tuş | İşlev |
|:---:|:---|
| `w` | Motoru **sürekli ileri** döndürür |
| `s` | Motoru **sürekli geri** döndürür |
| `a` | **Basılı tutulduğu sürece** ileri döner, bırakınca durur |
| `d` | **Basılı tutulduğu sürece** geri döner, bırakınca durur |

---

## 📌 2. Seri Haberleşme (UART / HSER) Nedir?

UART (Universal Asynchronous Receiver Transmitter), PIC ile bilgisayar arasında **veri alışverişi** yapmak için kullanılan bir iletişim protokolüdür.

### Seri Haberleşme Tanımları

```basic
DEFINE HSER_RCSTA 90h     ' Alıcı (Receiver) durumu - alıcıyı aktif eder
DEFINE HSER_TXSTA 24h     ' Verici (Transmitter) durumu - vericiyi aktif eder
DEFINE HSER_BAUD 9600     ' Baud Rate (iletişim hızı) = 9600 bit/saniye
DEFINE HSER_SPBRG 25      ' Baud rate oluşturucu değeri
DEFINE HSER_CLROERR 1     ' Hata olursa otomatik temizle
```

> [!IMPORTANT]
> **Baud Rate** = Saniyede gönderilen bit sayısı. Her iki cihaz (PIC ve bilgisayar) **aynı baud rate** kullanmalıdır. Genellikle **9600** kullanılır.

### HSER Komutları

| Komut | Açıklama | Örnek |
|:---|:---|:---|
| `HSEROUT [...]` | PIC'ten bilgisayara veri **gönder** | `hserout["Merhaba",13,10]` |
| `HSERIN [...]` | Bilgisayardan PIC'e veri **al** | `hserin [x]` |
| `HSERIN timeout,label,[x]` | Belirli süre bekle, veri gelmezse label'a git | `hserin 10,devam,[x]` |

- `13,10` → Satır sonu (CR+LF, yeni satıra geç)
- `timeout` → Bekleme süresi (x10 ms cinsinden). `10` = 100ms bekle.

---

## 📌 3. Bit Kaydırma Operatörleri (`<<` ve `>>`)

Bu kodun **en kritik kısmı** bit kaydırma operatörleridir:

```basic
i = i << 1    ' Sola kaydır (ileri yön)
i = i >> 1    ' Sağa kaydır (geri yön)
```

### Sola Kaydırma (`<<`) — İleri Yön

```
Başlangıç:  i = %00000001  (1)
1. adım:    i = %00000010  (2)   ← sola 1 bit kaydı
2. adım:    i = %00000100  (4)
3. adım:    i = %00001000  (8)
4. adım:    i = %00010000  (16)  ← 16 olunca i=1'e sıfırla
```

### Sağa Kaydırma (`>>`) — Geri Yön

```
Başlangıç:  i = %00001000  (8)
1. adım:    i = %00000100  (4)   ← sağa 1 bit kaydı
2. adım:    i = %00000010  (2)
3. adım:    i = %00000001  (1)
4. adım:    i = %00000000  (0)   ← 0 olunca i=8'e sıfırla
```

> [!TIP]
> **Sola kaydırma = 2 ile çarpma**, **Sağa kaydırma = 2'ye bölme** demektir. Bu step motorun bobinlerini sırayla aktif etmek için kullanılır.

---

## 📌 4. PULSOUT Komutu

```basic
PULSOUT portA.2, 100
```

- Belirtilen pinden **kısa bir darbe (pulse)** gönderir.
- `portA.2` = darbenin gönderileceği pin
- `100` = darbe süresi (x10 µs = 1000 µs = 1 ms)
- Motor sürücü entegresine (ULN2003 vb.) **adım sinyali** göndermek için kullanılır.

---

## 📌 5. GOSUB ve RETURN — Alt Program Yapısı

```basic
gosub ileri    ' "ileri" alt programına git
...
ileri:
    ' kodlar
return          ' çağrıldığın yere geri dön
```

| Komut | Açıklama |
|:---|:---|
| `GOSUB etiket` | Alt programa dallan |
| `RETURN` | Alt programdan geri dön |
| `GOTO etiket` | Etilete git (geri dönmez!) |

> [!WARNING]
> **GOSUB vs GOTO farkı çok önemli!**
> - `GOSUB` → Alt programa gider, `RETURN` ile **geri döner**
> - `GOTO` → Etikete gider, **geri dönmez!**
> Sınavda bu fark sorulabilir.

---

## 📌 6. `w`/`s` vs `a`/`d` Farkı

```basic
if x= "w" then gosub ileri     ' x temizlenMEZ → döngüde hep "w" kalır → sürekli döner
if x= "a" then gosub ileri : x=" "  ' x temizlenİR → bir kez çalışır, tekrar tuş bekler
```

- `w`/`s`: Tuşa bir kez basınca motor **sürekli** o yönde döner (x değişkeni değişmez)
- `a`/`d`: Tuşa **basılı tutmalısınız**, bırakınca durur (x her seferinde sıfırlanır)

> [!NOTE]
> `:` (iki nokta) PBP'de **aynı satırda iki komut yazmak** için kullanılır:
> `gosub ileri : x=" "` → önce ileri'yi çağır, sonra x'i temizle

---

## 📌 7. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **HSER tanımları** | RCSTA, TXSTA, BAUD hep aynı kalıp |
| **HSEROUT** | PIC → Bilgisayar (veri gönder) |
| **HSERIN** | Bilgisayar → PIC (veri al) |
| **Timeout** | `hserin 10,devam,[x]` → 100ms bekle, gelmezse devam'a git |
| **`<<` sola kaydır** | 2 ile çarpma, ileri yön |
| **`>>` sağa kaydır** | 2'ye bölme, geri yön |
| **GOSUB/RETURN** | Alt program, geri döner |
| **GOTO** | Geri dönmez! |
| **PULSOUT** | Motor sürücüye adım sinyali |
