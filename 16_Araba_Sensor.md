# 📘 Araba Park Sensörü Projesi — Konu Anlatımı

> **Kaynak Dosya:** [araba_sensor.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/araba_sensor.pbp)
> **Konu:** Ultrasonik + LCD özel karakterler + SOUND komutu ile park sensörü

---

## 📌 1. Bu Kod Ne Yapıyor?

Arabanın arka park sensörü gibi çalışır:
1. Ultrasonik sensörle **mesafe ölçer**
2. Mesafeye göre LCD'de **bar grafik** gösterir (özel karakterler)
3. Mesafe azaldıkça **bip sesi** hızlanır (SOUND komutu)

```
┌──────────────────┐
│🚗_▏▎▍▌▋▊█       │  ← Bar grafik (mesafeye göre dolar)
│mesafe: 12 cm     │  ← Sayısal değer
└──────────────────┘
```

---

## 📌 2. LCD'de 8 Özel Karakter Tanımı

```basic
LCDOUT $FE,$40, 1, 3, 7, 31, 16, 23, 7, 7         ' 0: Araba ön
LCDOUT $FE,$48, 31, 4, 4, 31, 31, 31, 0, 0         ' 1: Araba orta
LCDOUT $FE,$50, 16, 8, 4, 31, 1, 29, 28, 28        ' 2: Araba arka
LCDOUT $FE,$58, 0, 0, 0, 0, 0, 31, 31, 0           ' 3: Bar ▏ (1/5)
LCDOUT $FE,$60, 0, 0, 0, 0, 31, 31, 31, 0          ' 4: Bar ▎ (2/5)
LCDOUT $FE,$68, 0, 0, 0, 31, 31, 31, 31, 0         ' 5: Bar ▍ (3/5)
LCDOUT $FE,$70, 0, 0, 31, 31, 31, 31, 31, 0        ' 6: Bar ▌ (4/5)
LCDOUT $FE,$78, 0, 31, 31, 31, 31, 31, 31, 0       ' 7: Bar █ (5/5)
```

LCD'nin 8 özel karakter kapasitesinin **tamamı** kullanılmış:
- Karakter 0-2: Araba şekli (3 parça)
- Karakter 3-7: Farklı doluluk seviyelerinde bar grafik

---

## 📌 3. SOUND Komutu ⭐

```basic
SOUND portA.5, [50, 10]    ' Pin, [frekans, süre]
```

| Parametre | Açıklama |
|:---|:---|
| `portA.5` | Buzzer'ın bağlı olduğu pin |
| `50` | Ses frekansı (0-127 arası) |
| `10` | Ses süresi (x 12ms) |

### Mesafeye Göre Ses Sıklığı

```basic
if mesafe <= 5  then SOUND portA.5, [50, 10]    ' Çok yakın → kısa aralık (hızlı bip)
if mesafe <= 10 then SOUND portA.5, [50, 20]    ' Yakın → orta aralık
if mesafe <= 15 then SOUND portA.5, [50, 30]    ' Orta mesafe
if mesafe <= 20 then SOUND portA.5, [50, 40]    ' Uzak
if mesafe <= 25 then SOUND portA.5, [50, 50]    ' Çok uzak → uzun aralık (yavaş bip)
if mesafe > 25  then SOUND portA.5, [5, 60]     ' Güvenli → farklı ton
```

> [!TIP]
> Süre artarsa bip **daha uzun** ve **daha seyrek** duyulur. Mesafe azaldıkça süre azalır → bip **hızlanır**. Tıpkı gerçek park sensörü gibi!

---

## 📌 4. Çoklu IF Koşulları ile Bar Grafik

```basic
if mesafe <= 5  then lcdout 0,1,2,"_"
if mesafe > 5  AND mesafe <= 10 then lcdout 0,1,2,"_",3
if mesafe > 10 AND mesafe <= 15 then lcdout 0,1,2,"_",3,4
if mesafe > 15 AND mesafe <= 20 then lcdout 0,1,2,"_",3,4,5
if mesafe > 20 AND mesafe <= 25 then lcdout 0,1,2,"_",3,4,5,6
if mesafe > 25 then lcdout 0,1,2,"_",3,4,5,6,7
```

| Mesafe | LCD Görünüm | Bar Doluluk |
|:---|:---|:---:|
| ≤ 5 cm | 🚗_ | Boş (çok yakın!) |
| 5-10 cm | 🚗_▏ | %20 |
| 10-15 cm | 🚗_▏▎ | %40 |
| 15-20 cm | 🚗_▏▎▍ | %60 |
| 20-25 cm | 🚗_▏▎▍▌ | %80 |
| > 25 cm | 🚗_▏▎▍▌▋ | %100 (güvenli) |

### AND Operatörü

```basic
if mesafe > 5 AND mesafe <= 10 then ...
```

- `AND` → **VE** mantıksal operatörü. Her iki koşul da doğru olmalı.
- Aralık kontrolü için kullanılır.

> [!NOTE]
> PBP'de mantıksal operatörler: `AND` (ve), `OR` (veya), `NOT` (değil)

---

## 📌 5. 2. Satırda Sayısal Mesafe

```basic
lcdout $fe, $C0, "mesafe: ", #mesafe, " cm"
```

Her ölçümde hem **görsel bar** (1. satır) hem **sayısal değer** (2. satır) güncellenir.

---

## 📌 6. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **SOUND** | `SOUND pin, [frekans, süre]` |
| **8 özel karakter** | LCD'de maksimum 8 tanımlanabilir (0-7) |
| **AND operatörü** | Aralık kontrolü: `x > 5 AND x <= 10` |
| **Çoklu IF** | Her mesafe aralığı ayrı IF ile kontrol |
| **Bar grafik** | Özel karakterler ile görsel gösterim |
| **CGRAM adresleri** | $40, $48, $50, $58, $60, $68, $70, $78 |
| **Park sensörü mantığı** | Mesafe azalır → bip hızlanır |
