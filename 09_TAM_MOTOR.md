# 📘 Tam Adım (Full Step) Motor Sürme — Konu Anlatımı

> **Kaynak Dosya:** [TAM_MOTOR.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/TAM_MOTOR.pbp)
> **Konu:** Çift fazlı (tam adım) step motor sürme, dizi tabanlı bobin kontrolü

---

## 📌 1. Bu Kod Ne Yapıyor?

Step motoru **tam adım (full step)** modunda sürer. Tek fazlıdan farkı: **aynı anda 2 bobin aktif** olur → daha fazla tork sağlar.

---

## 📌 2. Tek Fazlı vs Tam Adım (Çift Fazlı) Karşılaştırma

### Tek Fazlı (Wave Drive) — Önceki kodlar
```
Adım 1: %00000001  → sadece A bobini
Adım 2: %00000010  → sadece B bobini
Adım 3: %00000100  → sadece C bobini
Adım 4: %00001000  → sadece D bobini
```

### Tam Adım (Full Step) — Bu kod ⭐
```
Adım 1: %00000011  (3)  → A + B bobinleri
Adım 2: %00000110  (6)  → B + C bobinleri
Adım 3: %00001100  (12) → C + D bobinleri
Adım 4: %00001001  (9)  → D + A bobinleri
```

| Mod | Aynı Anda Aktif Bobin | Tork | Adım Açısı |
|:---|:---:|:---:|:---:|
| Tek Fazlı | 1 | Düşük | 7.5° |
| Tam Adım | 2 | **Yüksek** | 7.5° |
| Yarım Adım | 1-2 (değişken) | Orta | 3.75° |

> [!IMPORTANT]
> **Tam adımda aynı anda 2 bobin aktiftir!** Bu daha fazla tork (döndürme gücü) sağlar. Sınavda tek fazlı ile tam adım farkı sorulabilir.

---

## 📌 3. Dizi ile Bobin Dizilimi

```basic
i var byte[4]
i[1] = 3      ' %00000011 → A+B
i[2] = 6      ' %00000110 → B+C
i[3] = 12     ' %00001100 → C+D
i[4] = 9      ' %00001001 → D+A
```

| İndeks | Değer | Binary | Aktif Bobinler |
|:---:|:---:|:---:|:---|
| i[1] | 3 | `%00000011` | A + B |
| i[2] | 6 | `%00000110` | B + C |
| i[3] | 12 | `%00001100` | C + D |
| i[4] | 9 | `%00001001` | D + A |

> [!TIP]
> **3-6-12-9** sırasını ezberleyin! Sınavda çıkabilir. Bu sıra tam adım step motor için standart bobin dizilimidir.

---

## 📌 4. İleri ve Geri Yön (Dizi İndeksi ile)

```basic
ileri:
    pulsout portA.2, 100
    j = j + 1               ' İndeksi 1 artır
    if j = 5 then j = 1     ' 5 olunca 1'e dön (döngüsel)
    portB = i[j] : pause 50
return

geri:
    pulsout portA.2, 100
    j = j - 1               ' İndeksi 1 azalt
    if j = 0 then j = 4     ' 0 olunca 4'e dön (döngüsel)
    portB = i[j] : pause 50
return
```

### İleri yön sırası:
```
j: 1 → 2 → 3 → 4 → 1 → 2 → ...
Değer: 3 → 6 → 12 → 9 → 3 → ...
```

### Geri yön sırası:
```
j: 4 → 3 → 2 → 1 → 4 → 3 → ...
Değer: 9 → 12 → 6 → 3 → 9 → ...
```

> [!NOTE]
> Tek fazlı yöntemde **bit kaydırma** (`<<`, `>>`) kullanılıyordu. Tam adımda ise **dizi indeksini artırma/azaltma** yöntemi kullanılıyor. Çünkü tam adım değerleri (3,6,12,9) bit kaydırmayla oluşturulamaz.

---

## 📌 5. Bit Kaydırma vs Dizi Yöntemi Karşılaştırma

| Özellik | Bit Kaydırma (`<<`, `>>`) | Dizi İndeksi |
|:---|:---|:---|
| Uygun mod | Tek fazlı | Tam adım, yarım adım |
| Değerler | 1,2,4,8 (2'nin katları) | 3,6,12,9 (herhangi) |
| Esneklik | Düşük | **Yüksek** |
| Kullanım | `i = i << 1` | `j = j + 1; portB = motor[j]` |

---

## 📌 6. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **Tam adım** | Aynı anda 2 bobin aktif |
| **3-6-12-9** | Tam adım bobin dizilimi |
| **Dizi yöntemi** | İndeks ile bobin seçimi |
| **48 adım = 360°** | 7.5° adım açısı |
| **İleri** | İndeks artır (j+1) |
| **Geri** | İndeks azalt (j-1) |
| **Döngüsel kontrol** | j=5→j=1, j=0→j=4 |
