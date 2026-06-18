# 📘 LCD Termostat Projesi — Konu Anlatımı

> **Kaynak Dosya:** [lcdTermostat.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/lcdTermostat.pbp)
> **Konu:** LCD + ADC + Seri port termostat, özel karakter ile durum gösterimi

---

## 📌 1. Bu Kod Ne Yapıyor?

1. **ADC** ile oda sıcaklığını okur
2. **LCD ekranda** sıcaklık ve termostat değerini gösterir
3. Seri porttan **Space** tuşuna basılırsa termostat ayarlama moduna geçer
4. Sıcaklık ≥ termostat ise **güneş simgesi** (klima açık), değilse **boş simge** gösterir

---

## 📌 2. Özel Karakterler (3 Adet)

```basic
LCDOUT $FE,$40, 0, 0, 14, 10, 14, 0, 0, 0     ' Karakter 0: küçük kutucuk (LED sönük)
LCDOUT $FE,$48, 7, 5, 7, 0, 0, 0, 0, 0         ' Karakter 1: derece simgesi (°)
LCDOUT $FE,$50, 4, 21, 14, 27, 14, 21, 4, 0     ' Karakter 2: güneş (LED yanık/klima açık)
```

| Karakter | Simge | Kullanım |
|:---:|:---:|:---|
| 0 | ○ (boş) | Klima kapalı |
| 1 | ° (derece) | Sıcaklık birimi |
| 2 | ☀ (güneş) | Klima açık |

---

## 📌 3. HSERIN ile Timeout ve WAIT Kullanımı ⭐

```basic
hserin 100, devam, [wait(" ")]
```

Bu satır **çok önemli** ve birkaç özelliği birleştirir:

| Parametre | Açıklama |
|:---|:---|
| `100` | 100 x 10ms = **1 saniye** bekle |
| `devam` | Süre dolunca `devam` etiketine git |
| `wait(" ")` | **Sadece boşluk (space)** karakteri bekle, başka tuşu yoksay |

### WAIT Değiştiricisi

```basic
hserin [wait("OK"), x]    ' "OK" stringi gelene kadar bekle, sonra x'i oku
hserin [wait(" ")]        ' Sadece boşluk gelene kadar bekle
```

> [!IMPORTANT]
> `WAIT` bir **filtre** görevi görür. Belirtilen karakter/string gelene kadar diğer verileri **yoksayar**. Timeout ile birlikte kullanıldığında: "1 saniye içinde space basılmazsa devam et" demektir.

---

## 📌 4. LCD'de Durum Gösterimi

```basic
if i >= j then
    lcdout $fe, 142, 2     ' Pozisyon 142'ye güneş simgesi yaz
else
    lcdout $fe, 142, 0     ' Pozisyon 142'ye boş simge yaz
endif
```

- `i` = oda sıcaklığı (ADC'den okunan)
- `j` = termostat değeri (kullanıcının ayarladığı)
- `142` = 1. satırın 14. pozisyonu ($80 + 14 = $8E = 142)

Bu, LCD'nin sağ üst köşesinde küçük bir durum ikonu gösterir.

---

## 📌 5. LCD'de Veri Gösterimi

```basic
yaz:
    portA.0 = 1                              ' ADC disable
    trisB = 0                                ' PORTB çıkış
    lcdout $fe, 1                            ' Ekranı temizle
    lcdout "sicaklik:", #i, 1, "C"           ' 1. satır: sıcaklık + derece simgesi
    lcdout $fe, 192, "termostat:", #j, 1, "C"  ' 2. satır: termostat + derece simgesi
    pause 100
return
```

Ekran görünümü:
```
┌──────────────────┐
│sicaklik:25°C    ☀│
│termostat:22°C    │
└──────────────────┘
```

> [!NOTE]
> `1` (tırnaksız) → özel karakter 1 (derece simgesi °) gösterir. `"C"` → normal C harfi.

---

## 📌 6. Termostat Ayarlama (Seri Port)

```basic
ayarla:
    hserout["termostat degerini gir:", 13, 10]
    hserin[dec j]          ' Decimal sayı olarak oku
    hserout["deger: ", dec j, 13, 10]
return
```

- `DEC j` → Gelen veriyi **decimal (ondalık) sayı** olarak yorumla
- Kullanıcı "25" yazarsa `j = 25` olur
- `DEC` olmadan, sadece ilk karakterin ASCII değeri alınırdı

> [!IMPORTANT]
> `HSERIN [dec j]` ile `HSERIN [j]` farkı:
> - `[dec j]` → "25" yazarsanız j = **25** (sayı olarak)
> - `[j]` → "25" yazarsanız j = **50** (sadece '2' nin ASCII değeri)

---

## 📌 7. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **WAIT değiştiricisi** | Belirli karakter gelene kadar bekle |
| **Timeout + label** | Süre dolunca etikete git |
| **DEC değiştiricisi** | Sayısal girdi okuma |
| **Özel karakter** | CGRAM ile tanımlanır, numarayla çağrılır |
| **$FE, 142** | Belirli pozisyona yazdırma |
| **TRIS dinamik** | ADC okuma/LCD yazma geçişi |
| **LCD + ADC + HSER** | Üç donanımın entegrasyonu |
