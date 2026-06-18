# 📘 Karmaşık Proje: Motor + Tur Sayma + Display (ZOR) — Konu Anlatımı

> **Kaynak Dosya:** [ZOR6.5.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/ZOR6.5.pbp)
> **Konu:** Seri port motor kontrolü + ileri/geri tur sayma + net tur hesabı + negatif gösterim

---

## 📌 1. Bu Kod Ne Yapıyor?

Bu, tüm konuların **bir arada** kullanıldığı en kapsamlı projedir:
1. Seri porttan `d` tuşu → motor **ileri**, `a` tuşu → motor **geri**
2. Her 52 adımda bir tur sayar (**ayrı ayrı**: ileri tur, geri tur)
3. **Net tur** = ileri tur - geri tur hesaplar
4. Geri tur fazlaysa **"N0"** (negatif, başlangıç noktasının gerisinde) gösterir
5. Sonucu 7-segment display'de gösterir

---

## 📌 2. İleri/Geri Tur Sayma Mantığı

```basic
sayacIleri var byte    ' İleri tur sayacı
sayacGeri var byte     ' Geri tur sayacı
c var byte             ' İleri adım sayacı (0-51)
d var byte             ' Geri adım sayacı (0-51)

' İleri motor çalıştığında:
c = c + 1
if c = 51 then c = 0 : sayacIleri = sayacIleri + 1 : gosub hesapla

' Geri motor çalıştığında:
d = d + 1
if d = 51 then d = 0 : sayacGeri = sayacGeri + 1 : gosub hesapla
```

### Tur Tamamlama Mantığı
- Her motor adımında adım sayacı (`c` veya `d`) 1 artar
- 52 adım olunca (0-51) → adım sayacı sıfırlanır, tur sayacı 1 artar
- Sonra `hesapla` alt programı çağrılır

---

## 📌 3. Net Tur Hesaplama

```basic
hesapla:
    if sayacIleri >= sayacGeri then
        sayac = sayacIleri - sayacGeri    ' Net pozitif tur
        gosub yaz                          ' Sayıyı göster
    else
        goto yaz1                          ' Negatif → "N0" göster
    endif
return
```

| İleri Tur | Geri Tur | Net | Gösterim |
|:---:|:---:|:---:|:---:|
| 5 | 3 | 2 | `02` |
| 3 | 3 | 0 | `00` |
| 3 | 5 | -2 | `N0` |

> [!IMPORTANT]
> PBP'de **byte** tipi negatif sayı tutamaz (0-255 arası). Bu yüzden geri tur daha fazlaysa sayısal çıkarma yapılamaz. Bunun yerine "N0" (negatif bölge) gösterilir.

---

## 📌 4. Negatif Bölge Gösterimi (N0)

```basic
b var byte[2]
b[0] = %00110111    ' "N" harfinin segment kodu
b[1] = %00111111    ' "0" rakamının segment kodu

yaz1:
    for j = 0 to 24
        portA.0 = 0 : portA.1 = 1
        portB = b[1]     ' Birler: "0"
        pause 10
        portA.0 = 1 : portA.1 = 0
        portB = b[0]     ' Onlar: "N"
        pause 10
    next j
    hserin 10, yaz1, [x]       ' Yeni tuş bekle
    if x = "d" then goto label  ' d basılırsa normal moda dön
goto yaz1
```

### "N" Harfinin Segment Kodu

```
  ─── a ───        a=1, b=1, c=1
 |         |       d=0, e=1, f=1
 f         b       g=0
 |         |
  ─── g ───   →   %00110111 = 55
 |         |
 e         c
 |         |
  ─── d ───
```

> [!NOTE]
> 7-segment display'de sadece rakamlar değil, **bazı harfler** de gösterilebilir: A, b, C, d, E, F, H, L, n, o, P, U vb. Ama tüm harfler gösterilemez (M, W, K gibi).

---

## 📌 5. Seri Port Motor Kontrolü

```basic
label:
    hserin 10, devam, [x]
    devam:
    if x = "d" then
        gosub ileri : x = " "    ' Basılı tutunca ileri
        c = c + 1
        if c = 51 then c = 0 : sayacIleri = sayacIleri + 1 : gosub hesapla
    endif

    if x = "a" then
        gosub geri : x = " "     ' Basılı tutunca geri
        d = d + 1
        if d = 51 then d = 0 : sayacGeri = sayacGeri + 1 : gosub hesapla
    endif
goto label
```

---

## 📌 6. Motor İleri/Geri (portA sıfırlama ile)

```basic
ileri:
    portA = 0                    ' ⚠️ Önce portA'yı sıfırla (display'i kapat)
    pulsout portA.2, 100
    i = i + 1
    if i = 4 then i = 0
    portB = motor[i] : pause 10
return

geri:
    portA = 0
    pulsout portA.2, 100
    if i = 0 then i = 4
    portB = motor[i] : pause 10
    i = i - 1
return
```

> [!WARNING]
> Motor çalıştırmadan önce `portA = 0` yapılır. Bu, display multiplexing pinlerini kapatarak **display ile motor çakışmasını** önler.

---

## 📌 7. Bu Projede Kullanılan Tüm Kavramlar

| Kavram | Bu Projede |
|:---|:---|
| Seri haberleşme | `HSERIN`, `HSEROUT` |
| Step motor (tam adım) | 3-6-12-9 dizisi |
| 7-segment display | Segment kodları + multiplexing |
| DIG operatörü | Basamak ayırma |
| Sayaç mantığı | İleri/geri ayrı sayaçlar |
| Koşullu dallanma | IF-THEN-ELSE |
| Port paylaşımı | Motor + display aynı PORTB |
| Timeout | `hserin 10, devam, [x]` |
| Özel segment kodu | "N" harfi: %00110111 |

---

## 📌 8. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **Net tur** | İleri tur - geri tur |
| **Negatif kontrol** | byte negatif olamaz → N0 göster |
| **Port çakışması** | Motor öncesi portA=0 |
| **52 adım = 1 tur** | Adım sayacı 51'e ulaşınca |
| **x = " "** | Tuşu sıfırla (basılı tutma modu) |
| **Entegre proje** | HSER + Motor + Display birlikte |
