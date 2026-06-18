# 🎯 Programlanabilir Denetleyici — Sınav Hazırlık Dokümanı

> **Sınav Formatı:**
> - 1 adet **kod yazma** sorusu (ultrasonik — senaryo verilir, kodu baştan yaz)
> - Diğer sorular: **"Bu çıktıyı/çalışmayı üreten kod hangi şıktadır?"** veya **"Bu kodun çıktısı nedir?"** formatında
> **Konular:** Ultrasonik Sensör | Step Motor | Termometre+Termostat | Display/LCD

---

# 📋 BÖLÜM 1: KRİTİK KAVRAM ÖZETLERİ

## 🔧 Temel PBP Komutları

| Komut | Açıklama |
|:---|:---|
| `DEFINE LOADER_USED 1` | Bootloader tanımı (her programda) |
| `ADCON1 = 7` | Tüm pinler dijital |
| `TRIS_ = 0` | Port çıkış (0=O=Output) |
| `TRIS_ = %11111111` | Port giriş (1=I=Input) |
| `VAR BYTE` | 8-bit değişken (0-255) |
| `VAR WORD` | 16-bit değişken (0-65535) |
| `CON` | Sabit tanımlama (değişmez) |
| `SYMBOL` | Pine isim verme |
| `PAUSE 500` | 500ms bekleme |
| `GOTO label` | Etikete dallan (geri dönmez) |
| `GOSUB / RETURN` | Alt program çağır/dön |
| `FOR i=0 TO 47 ... NEXT i` | Döngü |
| `WHILE koşul ... WEND` | Koşullu döngü |
| `IF-THEN-ELSE-ENDIF` | Koşul |
| `DIG 0` / `DIG 1` | Birler / onlar basamağı |
| `i << 1` / `i >> 1` | Sola/sağa bit kaydırma |
| `PULSOUT pin, süre` | Darbe gönder (süre × 10µs) |
| `PULSIN pin, durum, değişken` | Darbe süresini ölç |

## ⚙️ Step Motor Modları (Ezber!)

| Mod | Aktif Bobin | Değerler | Adım Açısı | Tam Tur |
|:---|:---:|:---|:---:|:---:|
| **Tek Fazlı** | 1 | **1, 2, 4, 8** | 7.5° | 48 adım |
| **Tam Adım** | 2 | **3, 6, 12, 9** | 7.5° | 48 adım |
| **Yarım Adım** | 1-2 | **1,3,2,6,4,12,8,9** | 3.75° | 96 adım |

## 📺 LCD Komutları

| Komut | İşlev |
|:---|:---|
| `LCDOUT $FE, 1` | Ekranı temizle |
| `LCDOUT $FE, $80` | 1. satır başı |
| `LCDOUT $FE, $C0` (= 192) | 2. satır başı |
| `LCDOUT "metin"` | Sabit metin |
| `LCDOUT #degisken` | Sayısal değer yazdır |
| `LCDOUT degisken` | ASCII karakter yazdır |
| `LCDOUT 0` | Özel karakter 0'ı göster |

## 🔢 7-Segment Kodları

| Rakam | Binary | Decimal |
|:---:|:---:|:---:|
| 0 | `%00111111` | 63 |
| 1 | `%00000110` | 6 |
| 2 | `%01011011` | 91 |
| 3 | `%01001111` | 79 |
| 4 | `%01100110` | 102 |
| 5 | `%01101101` | 109 |
| 6 | `%01111101` | 125 |
| 7 | `%00000111` | 7 |
| 8 | `%01111111` | 127 |
| 9 | `%01101111` | 111 |

---

# 📝 BÖLÜM 2: SINAV SORULARI

---

## 🔵 SORU 1 — ULTRASONİK KOD YAZMA (Senaryo) ⭐⭐⭐

### Senaryo:
> Bir fabrikanın konveyör bandında ilerleyen ürünlerin yüksekliğini ölçen bir sistem tasarlanacaktır. Ultrasonik sensör (HC-SR04), konveyörün **30 cm üzerinde** tavana monte edilmiştir. Sensör, altından geçen ürünün üst yüzeyine olan mesafeyi ölçer. Ürün yüksekliği = 30 - ölçülen mesafe formülüyle hesaplanır. Sonuç LCD ekranın 1. satırında `"mesafe: XX cm"`, 2. satırında `"yukseklik: XX cm"` formatında gösterilmelidir.
>
> - Trigger: PORTB.7, Echo: PORTB.6
> - LCD: 8-bit mod, PORTD üzerinden bağlı
> - Dönüşüm katsayısı: 6
> - Sürekli ölçüm yapılacak

### ✅ ÇÖZÜM:

```basic
DEFINE LOADER_USED 1

' LCD Tanımları
DEFINE LCD_DREG PORTD
DEFINE LCD_DBIT 0
DEFINE LCD_RSREG PORTE
DEFINE LCD_RSBIT 0
DEFINE LCD_EREG PORTE
DEFINE LCD_EBIT 2
DEFINE LCD_BITS 8
TRISE = %000
PortE.1 = 0
ADCON1 = 7
PAUSE 100

' Pin Tanımları
symbol trigger = portB.7
symbol echo = portB.6

' Değişkenler
salinim var word
mesafe var word
yukseklik var word
tavan con 30
sonuc_cm con 6

' Başlangıç
trigger = 0

' Ana Döngü
label:
    gosub oku
    yukseklik = tavan - mesafe
    lcdout $fe, 1
    lcdout "mesafe: ", #mesafe, " cm"
    lcdout $fe, $C0, "yukseklik: ", #yukseklik, " cm"
    pause 100
goto label

' Alt Program
oku:
    pulsout trigger, 1
    pulsin echo, 1, salinim
    mesafe = (salinim / sonuc_cm)
    pause 10
return
```

### 🧠 Kodda Dikkat Edilecek Kritik Noktalar:

| # | Nokta | Neden Önemli? |
|:---:|:---|:---|
| 1 | `DEFINE LOADER_USED 1` | İlk satır, her zaman yaz |
| 2 | LCD tanım bloğu (7 satır) | Ezbere bil |
| 3 | `ADCON1 = 7` | Dijital mod, unutursan çalışmaz |
| 4 | `PAUSE 100` | LCD'nin açılması için bekle |
| 5 | `salinim var word` | WORD lazım (255'i aşar) |
| 6 | `sonuc_cm con 6` | CON = sabit, VAR değil |
| 7 | `trigger = 0` | Başlangıçta LOW |
| 8 | `pulsout trigger, 1` | 10µs darbe gönder |
| 9 | `pulsin echo, 1, salinim` | HIGH süresini ölç |
| 10 | `#mesafe` | # = sayı olarak yazdır |
| 11 | `$fe, $C0` | 2. satıra geç |

---

## 🔵 SORU 2 — ULTRASONİK SENARYO VARYASYONU ⭐⭐

### Senaryo:
> Bir otopark giriş bariyerinde ultrasonik sensörle araç mesafesi ölçülmektedir. Mesafe 20 cm'den az olduğunda LCD'de `"YAKIN!"` uyarısı, aksi halde `"mesafe: XX cm"` gösterilecektir. Ayrıca mesafe 20 cm'den az olduğunda PORTA.4'e bağlı LED yanacak, aksi halde sönecektir.

### ✅ ÇÖZÜM:

```basic
DEFINE LOADER_USED 1
DEFINE LCD_DREG PORTD
DEFINE LCD_DBIT 0
DEFINE LCD_RSREG PORTE
DEFINE LCD_RSBIT 0
DEFINE LCD_EREG PORTE
DEFINE LCD_EBIT 2
DEFINE LCD_BITS 8
TRISE = %000
PortE.1 = 0
ADCON1 = 7
PAUSE 100

symbol trigger = portB.7
symbol echo = portB.6
symbol led = portA.4

salinim var word
mesafe var word
sonuc_cm con 6
trigger = 0
TRISA = 0

label:
    gosub oku
    lcdout $fe, 1
    if mesafe < 20 then
        lcdout "YAKIN!"
        led = 1
    else
        lcdout "mesafe: ", #mesafe, " cm"
        led = 0
    endif
    pause 100
goto label

oku:
    pulsout trigger, 1
    pulsin echo, 1, salinim
    mesafe = (salinim / sonuc_cm)
    pause 10
return
```

---

## 🟡 SORU 3 — STEP MOTOR: "Bu çalışmayı sağlayan kod hangisidir?" ⭐⭐

### Senaryo:
> Bir step motor **tam adım (full step)** modunda çalışarak **ileri yönde sürekli** dönmektedir. Motor dizisi kullanılarak sürülmekte ve `PORTB` çıkışları sırasıyla **3 → 6 → 12 → 9 → 3 → 6 → ...** şeklinde tekrar etmektedir.
>
> **Bu çalışmayı sağlayan kod hangisidir?**

**A)**
```basic
i var byte
i = 1
label:
    portB = i : pause 50
    i = i << 1
    if i = 16 then i = 1
goto label
```

**B)**
```basic
motor var byte[4]
motor[1]=3 : motor[2]=6 : motor[3]=12 : motor[4]=9
j var byte : j = 0
label:
    j = j + 1
    if j = 5 then j = 1
    portB = motor[j] : pause 50
goto label
```

**C)**
```basic
i var byte
i = 1
label:
    portB = i : pause 50
    i = i >> 1
    if i = 0 then i = 8
goto label
```

**D)**
```basic
motor var byte[8]
motor[1]=1:motor[2]=3:motor[3]=2:motor[4]=6
motor[5]=4:motor[6]=12:motor[7]=8:motor[8]=9
j var byte : j = 0
label:
    j = j + 1
    if j = 9 then j = 1
    portB = motor[j] : pause 50
goto label
```

### ✅ CEVAP: **B**

| Şık | Ne yapıyor? | Neden yanlış/doğru? |
|:---:|:---|:---|
| A | Tek fazlı ileri (1→2→4→8→1...) | ❌ Çıktı: 1,2,4,8 — tek fazlı |
| **B** | **Tam adım ileri (3→6→12→9→3...)** | ✅ Doğru dizilim ve döngüsel kontrol |
| C | Tek fazlı geri (8→4→2→1→8...) | ❌ Geri yön + tek fazlı |
| D | Yarım adım (1,3,2,6,4,12,8,9) | ❌ 8 adımlık yarım adım dizisi |

---

## 🟡 SORU 4 — STEP MOTOR: "Bu kodun PORTB çıktısı nedir?" ⭐⭐

### Soru:
> Aşağıdaki kod çalıştığında, `geri` alt programı **5 kez** çağrılıyor. `j` başlangıç değeri `4`. PORTB çıkışlarını sırasıyla yazınız.

```basic
motor var byte[4]
motor[1]=3 : motor[2]=6 : motor[3]=12 : motor[4]=9
j var byte : j = 4

geri:
    j = j - 1
    if j = 0 then j = 4
    portB = motor[j] : pause 50
return
```

### ✅ ÇÖZÜM:

| Çağrı | j (giriş) | j - 1 | if kontrol | portB = motor[j] |
|:---:|:---:|:---:|:---:|:---:|
| 1 | 4 | 3 | — | motor[3] = **12** |
| 2 | 3 | 2 | — | motor[2] = **6** |
| 3 | 2 | 1 | — | motor[1] = **3** |
| 4 | 1 | 0 | 0→j=4 | motor[4] = **9** |
| 5 | 4 | 3 | — | motor[3] = **12** |

**Çıktı: 12 → 6 → 3 → 9 → 12**

> Geri yön = dizinin tersten okunması. İleri: 3→6→12→9, Geri: 9→12→6→3 (ters döngü)

---

## 🟡 SORU 5 — STEP MOTOR: "Bu senaryoyu gerçekleştiren kod hangisidir?" ⭐⭐

### Senaryo:
> Bir step motor **tek fazlı** modda **ileri** yönde **48 adım** (360°) döndükten sonra **geri** yönde **48 adım** döner ve bu işlemi sürekli tekrarlar. Motor bit kaydırma yöntemiyle sürülmektedir.
>
> **Bu çalışmayı gerçekleştiren kod hangisidir?**

**A)**
```basic
i var byte : j var byte : i = 1
label:
    for j = 0 to 47
        gosub ileri
    next j
    for j = 0 to 47
        gosub geri
    next j
goto label

ileri:
    pulsout portA.2, 100
    i = i << 1
    if i = 16 then i = 1
    portB = i : pause 300
return

geri:
    pulsout portA.2, 100
    i = i >> 1
    if i = 0 then i = 8
    portB = i : pause 300
return
```

**B)**
```basic
i var byte : i = 1
label:
    pulsout portA.2, 100
    i = i << 1
    if i = 16 then i = 1
    portB = i : pause 300
goto label
```

**C)**
```basic
i var byte : j var byte : i = 1
label:
    for j = 0 to 47
        gosub ileri
    next j
goto label

ileri:
    pulsout portA.2, 100
    i = i << 1
    if i = 16 then i = 1
    portB = i : pause 300
return
```

**D)**
```basic
i var byte : j var byte : i = 1
label:
    for j = 0 to 95
        gosub ileri
    next j
goto label

ileri:
    pulsout portA.2, 100
    i = i << 1
    if i = 16 then i = 1
    portB = i : pause 300
return
```

### ✅ CEVAP: **A**

| Şık | Açıklama |
|:---:|:---|
| **A** | ✅ 48 adım ileri + 48 adım geri + sonsuz döngü (doğru!) |
| B | ❌ Sadece sürekli ileri (geri yok, FOR yok) |
| C | ❌ Sadece ileri 48 adım (geri yok) |
| D | ❌ 96 adım sadece ileri (yarım adım karışıklığı) |

---

## 🟡 SORU 6 — STEP MOTOR: "Hangisi doğrudur?" ⭐

### Soru:
> Aşağıdakilerden hangisi **yarım adım** step motor sürme modu için **doğrudur**?

- **A)** Aynı anda her zaman 2 bobin aktiftir, adım açısı 7.5°'dir
- **B)** Aynı anda her zaman 1 bobin aktiftir, adım açısı 3.75°'dir
- **C)** Tek ve çift fazlı sırayla uygulanır, adım açısı 3.75°'dir, 1 tur = 96 adım ✅
- **D)** Bobin dizilimi 3-6-12-9'dur, 1 tur = 48 adımdır

### ✅ CEVAP: **C**

| Şık | Analiz |
|:---:|:---|
| A | ❌ "Her zaman 2 bobin" → tam adım, yarım adım değil |
| B | ❌ "Her zaman 1 bobin" → tek fazlı |
| **C** | ✅ Yarım adım: tek+çift sırayla, 3.75°, 96 adım |
| D | ❌ 3-6-12-9 → tam adım, 48 adım → tam adım |

---

## 🟡 SORU 7 — TERMOMETRE: "Bu çıktıyı üreten kod hangisidir?" ⭐⭐

### Senaryo:
> Bir sıcaklık ölçme sisteminde harici ADC entegresi PORTB'ye bağlıdır. ADC'den okunan değer **7-segment display'de** gösterilecektir. Aynı PORTB hem ADC okuma (giriş) hem display yazma (çıkış) için kullanılmaktadır.
>
> **Bu çalışmayı doğru şekilde gerçekleştiren kod hangisidir?**

**A)**
```basic
oku:
    portA.0 = 0
    TRISB = %11111111
    i = portB
return

yaz:
    portA.0 = 1
    TRISB = 0
    birler = i dig 0
    onlar = i dig 1
    for j = 0 to 50
        portA.1 = 1
        portB = a[birler]
        pause 10
        portA.1 = 0
        portB = a[onlar]
        pause 10
    next j
return
```

**B)**
```basic
oku:
    portA.0 = 1
    TRISB = 0
    i = portB
return

yaz:
    portA.0 = 0
    TRISB = %11111111
    birler = i dig 0
    onlar = i dig 1
    portB = a[birler]
    pause 10
    portB = a[onlar]
    pause 10
return
```

**C)**
```basic
oku:
    portA.0 = 0
    TRISB = 0
    i = portB
return

yaz:
    portA.0 = 1
    TRISB = %11111111
    birler = i dig 0
    portB = a[birler]
return
```

**D)**
```basic
oku:
    TRISB = %11111111
    i = portB
return

yaz:
    TRISB = 0
    birler = i dig 0
    onlar = i dig 1
    for j = 0 to 50
        portA.1 = 1
        portB = a[birler]
        pause 10
        portA.1 = 0
        portB = a[onlar]
        pause 10
    next j
return
```

### ✅ CEVAP: **A**

| Şık | Analiz |
|:---:|:---|
| **A** | ✅ Okuma: ADC enable(portA.0=0) + TRISB giriş + oku. Yazma: ADC disable(portA.0=1) + TRISB çıkış + multiplexing |
| B | ❌ Ters! Okumada TRISB=0 (çıkış) yanlış. ADC enable/disable ters |
| C | ❌ Okumada TRISB=0 yanlış. Yazmada TRISB=%11111111 (giriş) yanlış. Multiplexing yok |
| D | ❌ portA.0 (ADC enable/disable) hiç kontrol edilmiyor → çakışma olur |

> **Kritik kural:** Okurken → `portA.0=0` (ADC enable) + `TRISB=255` (giriş). Yazarken → `portA.0=1` (ADC disable) + `TRISB=0` (çıkış).

---

## 🟡 SORU 8 — TERMOSTAT: "Bu senaryonun kodunu hangi şık karşılar?" ⭐⭐

### Senaryo:
> Bir termostat sisteminde oda sıcaklığı ADC ile okunmakta ve butonlarla termostat değeri ayarlanmaktadır. **"Ayarla" butonuna basıldığında** (aktif düşük, basılınca 0) termostat ayarlama moduna geçilir. Bu modda **"Artır" butonuna** her basıldığında termostat değeri 1 artar. Oda sıcaklığı termostat değerine **eşit veya üstünde** ise LED yanar (klima açılır).
>
> **Termostat karşılaştırma ve LED kontrol bloğu aşağıdakilerden hangisidir?**

**A)**
```basic
if i > termostatDeger then
    led = 1
else
    led = 0
endif
```

**B)**
```basic
if i >= termostatDeger then
    led = 0
else
    led = 1
endif
```

**C)**
```basic
if i < termostatDeger then
    led = 0
else
    led = 1
endif
```

**D)**
```basic
if i >= termostatDeger then
    led = 1
else
    led = 0
endif
```

### ✅ CEVAP: **B**

| Şık | Analiz |
|:---:|:---|
| A | ❌ `>` değil `>=` olmalı. led=1 → LED söner (ters!) |
| **B** | ✅ `>=` doğru. led=0 → LED yanar (aktif düşük). led=1 → LED söner |
| C | ❌ Karşılaştırma ters: `<` yerine `>=` olmalı |
| D | ❌ LED mantığı ters: led=1 → söner olmalıydı ama yanar diyor |

> **Tuzak:** Aktif düşük LED → `led = 0` yanar, `led = 1` söner. Sezgiye ters!

---

## 🟡 SORU 9 — TERMOSTAT: "Bu kodun çıktısı nedir?" ⭐⭐

### Soru:
> Aşağıdaki kodda sıcaklık değeri `i = 22`, termostat değeri `termostatDeger = 25` ise, `basla` butonuna basılı **değildir**. Programın çıktısını yazınız.

```basic
label:
    gosub oku
    gosub yaz
    while basla = 0
        gosub termostat
    wend
    if i >= termostatDeger then
        led = 0
    else
        led = 1
    endif
goto label
```

### ✅ ÇÖZÜM:

```
1. gosub oku → ADC'den i = 22 okunur
2. gosub yaz → Display'de "22" gösterilir
3. basla = 0 mı? → HAYIR (buton basılı DEĞİL, basla = 1)
   → WHILE döngüsüne GİRMEZ (termostat ayarlanmaz)
4. i >= termostatDeger → 22 >= 25 → YANLIŞ (FALSE)
   → ELSE bloğu çalışır → led = 1 → LED SÖNER (klima kapalı)
5. goto label → tekrar başa dön

SONUÇ:
- Display'de: 22
- LED: SÖNÜK (oda soğuk, klima kapalı)
```

---

## 🟡 SORU 10 — TERMOSTAT SENARYO VARYASYONU ⭐

### Soru:
> Aynı kodda `i = 30`, `termostatDeger = 25` ve `basla` butonuna basılı **değilse**, çıktı ne olur?

### ✅ ÇÖZÜM:

```
1. gosub oku → i = 30
2. gosub yaz → Display'de "30" gösterilir
3. basla = 0 mı? → HAYIR → WHILE'a girmez
4. 30 >= 25 → DOĞRU → led = 0 → LED YANAR (klima açıldı!)
5. goto label

SONUÇ:
- Display'de: 30
- LED: YANIK (oda sıcak, klima açık)
```

---

## 🟡 SORU 11 — DISPLAY: "Display'de 73 sayısını gösteren kod hangisidir?" ⭐⭐

### Senaryo:
> 7-segment display üzerinde **73** sayısını gösteren multiplexing kodu yazılacaktır. İki adet display kullanılmaktadır. Display seçimi `portA.0` ve `portA.1` ile yapılmaktadır.
>
> **Doğru kodu hangi şık verir?**

**A)**
```basic
birler = 73 dig 0       ' birler = 3
onlar = 73 dig 1        ' onlar = 7
for j = 0 to 24
    portA.0 = 1 : portA.1 = 0
    portB = a[birler]
    pause 10
    portA.0 = 0 : portA.1 = 1
    portB = a[onlar]
    pause 10
next j
```

**B)**
```basic
birler = 73 dig 1       ' birler = 7  ← DIG TERS!
onlar = 73 dig 0        ' onlar = 3
for j = 0 to 24
    portA.0 = 1 : portA.1 = 0
    portB = a[birler]
    pause 10
    portA.0 = 0 : portA.1 = 1
    portB = a[onlar]
    pause 10
next j
```

**C)**
```basic
birler = 73 dig 0
onlar = 73 dig 1
portB = a[birler]
pause 10
portB = a[onlar]
pause 10
```

**D)**
```basic
birler = 73 dig 0
onlar = 73 dig 1
for j = 0 to 24
    portA.0 = 1
    portB = a[onlar]
    pause 10
    portA.0 = 0
    portB = a[birler]
    pause 10
next j
```

### ✅ CEVAP: **A**

| Şık | Analiz |
|:---:|:---|
| **A** | ✅ DIG 0=birler(3), DIG 1=onlar(7). Multiplexing döngüsü doğru |
| B | ❌ DIG 0 ve DIG 1 ters kullanılmış → "37" gösterir |
| C | ❌ Multiplexing yok (FOR döngüsü yok, portA seçimi yok) → sadece son rakam görünür |
| D | ❌ portA.1 hiç kontrol edilmiyor → display seçimi eksik, karışık görüntü |

---

## 🟡 SORU 12 — LCD: "LCD'de bu çıktıyı veren kod hangisidir?" ⭐⭐

### Senaryo:
> LCD ekranda aşağıdaki görüntü elde edilmek istenmektedir:
> ```
> ┌──────────────────┐
> │sicaklik:25°C     │
> │termostat:22°C    │
> └──────────────────┘
> ```
> Özel karakter 1 = derece simgesi (°) olarak tanımlıdır.
> `i = 25` (sıcaklık), `j = 22` (termostat değeri)
>
> **Bu görüntüyü veren kod hangisidir?**

**A)**
```basic
lcdout $fe, 1
lcdout "sicaklik:", #i, 1, "C"
lcdout $fe, 192, "termostat:", #j, 1, "C"
```

**B)**
```basic
lcdout $fe, 1
lcdout "sicaklik:", i, "1", "C"
lcdout $fe, 192, "termostat:", j, "1", "C"
```

**C)**
```basic
lcdout $fe, 1
lcdout "sicaklik:", #i, "°C"
lcdout $fe, $C0, "termostat:", #j, "°C"
```

**D)**
```basic
lcdout $fe, 1
lcdout "sicaklik:", #i, #1, "C"
lcdout $fe, 192, "termostat:", #j, #1, "C"
```

### ✅ CEVAP: **A**

| Şık | Analiz |
|:---:|:---|
| **A** | ✅ `#i` → sayı(25). `1` (tırnaksız) → özel karakter 1 (°). `"C"` → normal C |
| B | ❌ `i` (tırnaksız #'sız) → ASCII karakter yazdırır, sayı değil. `"1"` → 1 rakamı, özel karakter değil |
| C | ❌ `"°C"` → LCD özel karakter değil, normal string. ° karakteri LCD'de yok |
| D | ❌ `#1` → "1" sayısını yazdırır, özel karakteri göstermez |

> **Tuzak:** `1` (tırnaksız, #'sız) = özel karakter 1. `"1"` = "1" metni. `#1` = 1 sayısı. Üçü farklı!

---

## 🟡 SORU 13 — LCD: "Bu kodun LCD çıktısı nedir?" ⭐

### Soru:
> Aşağıdaki kod çalıştırıldığında LCD ekranda ne görünür? (`mesafe = 18`)

```basic
lcdout $fe, 1
lcdout $fe, $80, "uzaklik:"
lcdout $fe, $C0, "mesafe: ", #mesafe, " cm"
```

### ✅ ÇÖZÜM:

```
┌──────────────────┐
│uzaklik:          │  ← 1. satır ($80)
│mesafe: 18 cm     │  ← 2. satır ($C0)
└──────────────────┘
```

- `$fe, 1` → ekran temizlenir
- `$fe, $80` → imleç 1. satır başına (zaten orada ama açıkça belirtilmiş)
- `$fe, $C0` → imleç 2. satır başına
- `#mesafe` → 18 (sayı olarak)

---

## 🟡 SORU 14 — WHILE-WEND: "Bu döngünün çalışma şekli hangisidir?" ⭐

### Soru:
> Aşağıdaki kodda `basla` butonuna **3 saniye boyunca basılı tutulursa** ve bu süre içinde `artir` butonuna **2 kez** basılırsa, `termostatDeger` kaç olur? (Başlangıçta `termostatDeger = 20`)

```basic
while basla = 0
    while artir = 0
        termostatDeger = termostatDeger + 1
        pause 500
    wend
wend
```

### ✅ ÇÖZÜM:

```
- basla = 0 (basılı) → dış WHILE'a gir
- artir'a her basışta termostatDeger 1 artar
- artir basılı tutulurken PAUSE 500 (0.5s) bekler
- 2 kez basıldı → termostatDeger = 20 + 2 = 22

CEVAP: termostatDeger = 22
```

> İç WHILE: artir butonuna basılıyken çalışır. Dış WHILE: basla butonuna basılıyken çalışır.

---

## 🟡 SORU 15 — BİT KAYDIRMA: "Bu kodun çıktısı nedir?" ⭐⭐

### Soru:
> Aşağıdaki kodda `i = 4` başlangıç değeriyle `label` döngüsüne girilir. İlk 5 adımda PORTB'ye yazılan değerleri sırasıyla yazınız.

```basic
i var byte : i = 4
label:
    portB = i : pause 500
    i = i << 1
    if i = 16 then i = 1
goto label
```

### ✅ ÇÖZÜM:

| Adım | portB (yazdırılan) | Sonra i << 1 | if kontrol |
|:---:|:---:|:---:|:---|
| 1 | **4** (%00000100) | 8 | — |
| 2 | **8** (%00001000) | 16 | 16→i=1 |
| 3 | **1** (%00000001) | 2 | — |
| 4 | **2** (%00000010) | 4 | — |
| 5 | **4** (%00000100) | 8 | — |

**Çıktı: 4 → 8 → 1 → 2 → 4**

> Dikkat: Başlangıç i=4 olduğu için döngü 1'den değil 4'ten başlıyor!

---

## 🟡 SORU 16 — GÖZ SENSÖRÜ: "Bu çalışmayı sağlayan kod hangisidir?" ⭐⭐

### Senaryo:
> Bir göz sensörü (portA.4) ile step motor kontrol edilmektedir. Sensöre **ışık geldiğinde** motor **ileri** döner, **ışık gelmediğinde** motor **geri** döner. Motor tek fazlı olarak sürülmektedir.
>
> **Bu çalışmayı sağlayan kod hangisidir?**

**A)**
```basic
symbol goz = portA.4
label:
    if goz = 1 then
        gosub ileri
    else
        gosub geri
    endif
goto label
```

**B)**
```basic
symbol goz = portA.4
label:
    if goz = 0 then
        gosub ileri
    else
        gosub geri
    endif
goto label
```

**C)**
```basic
symbol goz = portA.4
label:
    if goz = 1 then
        gosub geri
    else
        gosub ileri
    endif
goto label
```

**D)**
```basic
symbol goz = portA.4
label:
    while goz = 1
        gosub ileri
    wend
goto label
```

### ✅ CEVAP: **A**

| Şık | Analiz |
|:---:|:---|
| **A** | ✅ goz=1 (ışık var) → ileri, goz=0 (ışık yok) → geri. ELSE ile her iki durumda motor hareket eder |
| B | ❌ Ters! goz=0 (ışık yok) → ileri demiş |
| C | ❌ Ters! goz=1 (ışık var) → geri demiş |
| D | ❌ Sadece ışık varken ileri döner, ışık yokken hiçbir şey yapmaz (geri yok) |

---

## 🟡 SORU 17 — LCD ÖZEL KARAKTER: "Doğru tanım hangisidir?" ⭐

### Soru:
> LCD'de özel karakter **0** olarak aşağıdaki şekil tanımlanmak isteniyor:
> ```
> .....   (0)
> .X.X.   (10)
> .....   (0)
> X...X   (17)
> X...X   (17)
> X...X   (17)
> XXXXX   (31)
> .....   (0)
> ```
> **Doğru LCDOUT komutu hangisidir?**

**A)**
```basic
LCDOUT $FE, $40, 0, 10, 0, 17, 17, 17, 31, 0
```

**B)**
```basic
LCDOUT $FE, $48, 0, 10, 0, 17, 17, 17, 31, 0
```

**C)**
```basic
LCDOUT $FE, $40, 0, 10, 17, 0, 17, 17, 31, 0
```

**D)**
```basic
LCDOUT $FE, $40, 31, 10, 0, 17, 17, 17, 0, 0
```

### ✅ CEVAP: **A**

| Şık | Analiz |
|:---:|:---|
| **A** | ✅ Karakter 0 adresi ($40) doğru. Satır verileri sırasıyla doğru: 0,10,0,17,17,17,31,0 |
| B | ❌ $48 = karakter **1**'in adresi, karakter 0 değil |
| C | ❌ Satır 3 ve 4 yer değiştirmiş (0 ile 17 ters) |
| D | ❌ Satır sırası tamamen karışmış (31 üstte değil altta olmalı) |

> CGRAM adresleri: Karakter 0=$40, 1=$48, 2=$50, 3=$58, 4=$60, 5=$68, 6=$70, 7=$78

---

## 🟡 SORU 18 — HSERIN-WAIT: "Bu kodun çalışma şekli hangisidir?" ⭐

### Soru:
> Aşağıdaki komut ne yapar?
> ```basic
> hserin 100, devam, [wait(" ")]
> ```

- **A)** 100 byte veri okur, devam etiketine gider
- **B)** 1 saniye boyunca space tuşu bekler; space gelirse devam eder, gelmezse `devam` etiketine atlar ✅
- **C)** 100ms boyunca herhangi bir tuş bekler
- **D)** Space tuşuna basılana kadar sonsuza kadar bekler

### ✅ CEVAP: **B**

```
100 × 10ms = 1000ms = 1 saniye timeout
wait(" ") → sadece space karakteri kabul et, diğerlerini yoksay
devam → timeout dolunca dallanılacak etiket
```

---

## 🔵 SORU 19 — LCD+TERMOSTAT BİLEŞİK: "LCD çıktısı ne olur?" ⭐⭐

### Soru:
> Aşağıdaki LCD termostat kodunda `i = 18` (oda sıcaklığı), `j = 22` (termostat) ise LCD ekranda ne görünür ve durum simgesi (karakter 0=boş kutu, karakter 2=güneş) hangisi olur?

```basic
lcdout $fe, 1
lcdout "sicaklik:", #i, 1, "C"
lcdout $fe, 192, "termostat:", #j, 1, "C"
if i >= j then
    lcdout $fe, 142, 2
else
    lcdout $fe, 142, 0
endif
```

### ✅ ÇÖZÜM:

```
i = 18, j = 22
18 >= 22 → YANLIŞ → ELSE bloğu → lcdout $fe, 142, 0 → boş kutu simgesi

LCD Ekran:
┌──────────────────┐
│sicaklik:18°C    ○│  ← ° = özel karakter 1, ○ = özel karakter 0 (boş)
│termostat:22°C    │
└──────────────────┘

Durum: Oda sıcaklığı(18) < Termostat(22) → Klima KAPALI → boş kutu
```

---

## 🟡 SORU 20 — PORT PAYLAŞIMI: "Bu sorunu çözen yaklaşım hangisidir?" ⭐⭐

### Senaryo:
> Bir projede step motor ve 7-segment display **aynı PORTB'yi** paylaşmaktadır. Motor her turda 52 adım atıyor ve her tur sonunda tur sayısı display'de gösterilecektir. Ancak motor çalışırken display'de anlamsız veriler görünmektedir.
>
> **Bu sorunu çözen yaklaşım hangisidir?**

**A)** Motor ve display için ayrı portlar kullanılmalıdır (PORTB + PORTC)

**B)** Motor pozisyonu ayrı bir değişkende (`l`) saklanır. Display yazıldıktan sonra motor pozisyonu bu değişkenden geri yüklenir. ✅

**C)** Display'i sürekli kapalı tutup sadece motor durduğunda gösterilmelidir

**D)** TRISB sürekli değiştirilmeli, motor sürme sırasında TRISB=0, display sırasında TRISB=1 olmalıdır

### ✅ CEVAP: **B**

```basic
' Doğru yaklaşım:
label:
    i = l                    ' Motor pozisyonunu geri yükle
    for k = 0 to 51          ' 52 adım (1 tur)
        gosub ileri
    next k
    l = i                    ' Motor pozisyonunu kaydet
    sayac = sayac + 1
    gosub yaz                ' Display'e yazdır (PORTB'yi kullanır)
goto label
```

| Şık | Analiz |
|:---:|:---|
| A | ❌ Gerçek çözüm değil, donanım değişikliği gerektirir. Soruda aynı port paylaşımı isteniyor |
| **B** | ✅ Motor pozisyonunu `l` değişkeninde yedekle, display yazıldıktan sonra geri yükle |
| C | ❌ Display'i kapatmak çözüm değil, kullanıcı sürekli görmek ister |
| D | ❌ TRIS değiştirmek ADC için geçerli, motor+display durumu için geçersiz (ikisi de çıkış) |

---

# 📋 BÖLÜM 3: SON DAKİKA HATIRLAT­MALARI

## ⚡ Ultrasonik Kod Şablonu (Ezbere Yaz!)

```
1. DEFINE LOADER_USED 1
2. LCD tanım bloğu (DREG, DBIT, RSREG, RSBIT, EREG, EBIT, BITS)
3. TRISE=%000 / PortE.1=0 / ADCON1=7 / PAUSE 100
4. symbol trigger=portB.7 / symbol echo=portB.6
5. salinim var word / mesafe var word / sonuc_cm con 6
6. trigger=0
7. label: gosub oku / lcdout ... / goto label
8. oku: pulsout trigger,1 / pulsin echo,1,salinim / mesafe=salinim/sonuc_cm / return
```

## ⚡ Sık Çıkan Tuzaklar

| Tuzak | Doğru Bilgi |
|:---|:---|
| `led = 0` → LED söner | ❌ YANLIŞ! Aktif düşük: `led = 0` → LED **yanar** |
| `basla = 0` → buton bırakıldı | ❌ YANLIŞ! Aktif düşük: `basla = 0` → **basılı** |
| `LCDOUT 1` → "1" yazar | ❌ YANLIŞ! Tırnaksız sayı → **özel karakter** gösterir |
| `LCDOUT mesafe` → sayı yazar | ❌ YANLIŞ! `#` olmadan → **ASCII karakter** yazar |
| `DIG 0` → onlar basamağı | ❌ YANLIŞ! `DIG 0` = **birler**, `DIG 1` = **onlar** |
| Tam adım = 1-2-4-8 | ❌ YANLIŞ! Tam adım = **3-6-12-9**. 1-2-4-8 = tek fazlı |

## ⚡ Motor Değer Dizileri (Ezber!)

```
TEK FAZLI:   1 → 2 → 4 → 8         (bit kaydırma ile)
TAM ADIM:    3 → 6 → 12 → 9        (dizi ile)
YARIM ADIM:  1 → 3 → 2 → 6 → 4 → 12 → 8 → 9   (dizi ile)
```

## ⚡ Hızlı Formüller

```
360° / 7.5° = 48 adım (tek fazlı, tam adım)
360° / 3.75° = 96 adım (yarım adım)
mesafe = salinim / 6
karakter - 48 = rakam (ASCII dönüşüm)
$C0 = 192 (LCD 2. satır)
```
