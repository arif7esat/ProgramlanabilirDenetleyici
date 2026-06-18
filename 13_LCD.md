# 📘 LCD Ekran Kullanımı — Konu Anlatımı

> **Kaynak Dosya:** [lcd.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/lcd.pbp)
> **Konu:** LCD tanımları, LCDOUT komutları, özel karakter oluşturma (CGRAM)

---

## 📌 1. Bu Kod Ne Yapıyor?

1. LCD ekranda **özel karakterler** (yüz gülen, insan figürü) tanımlar
2. Seri porttan gelen karakterleri **LCD'ye yazdırır**
3. Geri sayım örneği ve "boom" efekti içerir (yorum satırlarında)

---

## 📌 2. LCD Tanımları (Her Projede Aynı)

```basic
DEFINE LCD_DREG PORTD       ' Veri pinleri: PORTD
DEFINE LCD_DBIT 0           ' Veri başlangıç bit: 0
DEFINE LCD_RSREG PORTE      ' RS pini: PORTE.0
DEFINE LCD_RSBIT 0
DEFINE LCD_EREG PORTE       ' Enable pini: PORTE.2
DEFINE LCD_EBIT 2
DEFINE LCD_BITS 8           ' 8 bit modu
TRISE = %000                ' PORTE tamamı çıkış
PortE.1 = 0                 ' R/W pini = 0 (yazma modu)
ADCON1 = 7                  ' Dijital mod
PAUSE 100                   ' LCD'nin açılması için bekle
```

> [!IMPORTANT]
> LCD'nin başlaması için en az **100ms bekleme** gerekir! `PAUSE 100` yazmazsanız LCD düzgün çalışmayabilir.

---

## 📌 3. LCDOUT Komutları — Tam Referans

| Komut | Açıklama |
|:---|:---|
| `LCDOUT $FE, 1` | **Ekranı temizle** |
| `LCDOUT $FE, $80` | İmleci **1. satır başına** getir |
| `LCDOUT $FE, $C0` | İmleci **2. satır başına** getir |
| `LCDOUT $FE, $94` | İmleci **3. satır başına** getir (4x20 LCD) |
| `LCDOUT $FE, $D4` | İmleci **4. satır başına** getir (4x20 LCD) |
| `LCDOUT "metin"` | Sabit metin yazdır |
| `LCDOUT #değişken` | Değişkenin sayısal değerini yazdır |
| `LCDOUT değişken` | Değişkenin ASCII karakterini yazdır |

### İmleç Pozisyonları (16x2 LCD)

```
1. Satır: $80 $81 $82 $83 $84 $85 $86 $87 $88 $89 $8A $8B $8C $8D $8E $8F
2. Satır: $C0 $C1 $C2 $C3 $C4 $C5 $C6 $C7 $C8 $C9 $CA $CB $CC $CD $CE $CF
```

> [!TIP]
> `$C0` = 192 (decimal). İkisi de aynı şeyi yapar: `$FE, $C0` = `$FE, 192`

---

## 📌 4. Özel Karakter Oluşturma (CGRAM) ⭐

LCD'de **8 adet özel karakter** (0-7) tanımlanabilir. Her karakter 5x8 pikseldir.

```basic
LCDOUT $FE, $40, 0, 10, 0, 17, 17, 17, 31, 0    ' Karakter 0: gülen yüz
LCDOUT $FE, $48, 31, 17, 10, 4, 10, 31, 0, 0     ' Karakter 1: insan figürü
```

### CGRAM Adresleri

| Karakter No | Başlangıç Adresi |
|:---:|:---:|
| 0 | `$40` |
| 1 | `$48` |
| 2 | `$50` |
| 3 | `$58` |
| 4 | `$60` |
| 5 | `$68` |
| 6 | `$70` |
| 7 | `$78` |

### Piksel Haritası Örneği (Gülen Yüz)

```
$FE, $40 ile Karakter 0 tanımla:
Satır verileri: 0, 10, 0, 17, 17, 17, 31, 0

Satır 1:  00000  (0)   →  .....
Satır 2:  01010  (10)  →  .X.X.  (gözler)
Satır 3:  00000  (0)   →  .....
Satır 4:  10001  (17)  →  X...X  (ağız köşeleri)
Satır 5:  10001  (17)  →  X...X
Satır 6:  10001  (17)  →  X...X
Satır 7:  11111  (31)  →  XXXXX  (ağız altı)
Satır 8:  00000  (0)   →  .....
```

### Özel Karakteri Ekranda Gösterme

```basic
LCDOUT 0       ' Karakter 0'ı göster (gülen yüz)
LCDOUT 1       ' Karakter 1'i göster (insan)
```

> [!IMPORTANT]
> Özel karakterler `0`-`7` arası numaralarla çağrılır. `LCDOUT 0` yazdığınızda tanımlanan özel karakter görünür. `LCDOUT "0"` yazsaydınız **sıfır rakamı** görünürdü. Tırnak işareti farkına dikkat!

---

## 📌 5. Seri Porttan LCD'ye Yazdırma

```basic
label:
    hserout["karakter girin: ", 13, 10]   ' Bilgisayara mesaj gönder
    hserin [i]                             ' Bir karakter al
    hserout[i]                             ' Aldığını geri gönder (echo)
    lcdout $fe, 1                          ' LCD temizle
    lcdout $fe, $80, i                     ' LCD'ye yazdır
    pause 500
goto label
```

Bu örnekte bilgisayardan girilen her karakter LCD ekranda gösterilir.

---

## 📌 6. Geri Sayım Örneği (Yorumda)

```basic
' for i = 10 to 0 step -1      ' 10'dan 0'a geri say
'     lcdout $fe, 1             ' Ekranı temizle
'     lcdout $fe, $80, #i       ' Sayıyı yazdır
'     pause 100
' next i
' lcdout $fe, 1
' lcdout $fe, $C0, "boom  ", 0, " ", 1   ' "boom" + özel karakterler
```

---

## 📌 7. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **$FE, 1** | Ekranı temizle |
| **$FE, $80** | 1. satır başı |
| **$FE, $C0** | 2. satır başı |
| **#değişken** | Sayı olarak yazdır |
| **değişken** (tırnaksız) | ASCII karakter olarak yazdır |
| **CGRAM** | $40-$78 arası, 8 özel karakter |
| **5x8 piksel** | Her satır 5 bit, 8 satır |
| **PAUSE 100** | LCD başlangıç bekleme |
| **PortE.1=0** | R/W pini yazma modunda |
