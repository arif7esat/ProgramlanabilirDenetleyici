# 📘 Tüm Motor Sürme Modları — Konu Anlatımı

> **Kaynak Dosya:** [butunMotorlar.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/butunMotorlar.pbp)
> **Konu:** Tek fazlı, çift fazlı ve yarım adım modlarının karşılaştırması

---

## 📌 1. Bu Kod Ne Yapıyor?

Bu dosya üç farklı step motor sürme modunu **karşılaştırmalı** olarak gösterir. Yorum satırlarıyla (label1, label2, label3) üç mod ayrılmış.

---

## 📌 2. Üç Sürme Modunun Karşılaştırması

### 🔹 Mod 1: Tek Fazlı (Wave Drive) — `label1`

```basic
' Tek fazlı: bir seferde sadece 1 bobin aktif
portB = %00000001 : pause 500     ' A bobini
portB = %00000010 : pause 500     ' B bobini
portB = %00000100 : pause 500     ' C bobini
portB = %00001000 : pause 500     ' D bobini
```

**Açılar:** -45° → 45° → 135° → 225° → 315°

### 🔹 Mod 2: Çift Fazlı / Tam Adım (Full Step) — `label2`

```basic
' Çift fazlı: bir seferde 2 bobin aktif
portB = %00000011 : pause 500     ' A+B bobinleri  (3)
portB = %00000110 : pause 500     ' B+C bobinleri  (6)
portB = %00001100 : pause 500     ' C+D bobinleri  (12)
portB = %00001001 : pause 500     ' D+A bobinleri  (9)
```

**Açılar:** 0° → 90° → 180° → 270° → 360°

### 🔹 Mod 3: Yarım Adım (Half Step) — `label3` ⭐

```basic
' Yarım adım: tek ve çift fazlı sırayla
portB = %00000001 : pause 500     ' A        (1)
portB = %00000011 : pause 500     ' A+B      (3)
portB = %00000010 : pause 500     ' B        (2)
portB = %00000110 : pause 500     ' B+C      (6)
portB = %00000100 : pause 500     ' C        (4)
portB = %00001100 : pause 500     ' C+D      (12)
portB = %00001000 : pause 500     ' D        (8)
portB = %00001001 : pause 500     ' D+A      (9)
```

**Açılar:** -45° → 0° → 45° → 90° → 135° → 180° → ...

> [!IMPORTANT]
> Yarım adım = Tek fazlı + Çift fazlı **sırayla** uygulanır. Her adımda **3.75°** döner (7.5° / 2). Daha hassas kontrol sağlar!

---

## 📌 3. Büyük Karşılaştırma Tablosu ⭐⭐⭐

| Özellik | Tek Fazlı | Çift Fazlı (Tam Adım) | Yarım Adım |
|:---|:---:|:---:|:---:|
| Aktif Bobin | 1 | 2 | 1 veya 2 (değişken) |
| Adım Açısı | 7.5° | 7.5° | **3.75°** |
| 1 Tur Adım Sayısı | 48 | 48 | **96** |
| Tork | Düşük | **Yüksek** | Orta |
| Güç Tüketimi | Düşük | Yüksek | Orta |
| Hassasiyet | Normal | Normal | **Yüksek** |
| Değer Dizisi | 1,2,4,8 | 3,6,12,9 | **1,3,2,6,4,12,8,9** |
| Sürme Yöntemi | Bit kaydırma | Dizi | Dizi |

> [!CAUTION]
> Sınavda **"üç modu karşılaştırın"** sorusu çıkabilir! Bu tabloyu iyi öğrenin.

---

## 📌 4. Adım Dizileri Görsel Karşılaştırma

```
TEK FAZLI:    A → B → C → D → A → ...
              1   2   4   8

ÇİFT FAZLI:  AB → BC → CD → DA → AB → ...
              3    6    12   9

YARIM ADIM:  A → AB → B → BC → C → CD → D → DA → A → ...
             1   3    2   6    4   12   8   9
```

---

## 📌 5. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **Tek fazlı** | 1 bobin, düşük tork, 1-2-4-8 |
| **Çift fazlı** | 2 bobin, yüksek tork, 3-6-12-9 |
| **Yarım adım** | Alternating, hassas, 1-3-2-6-4-12-8-9 |
| **Adım açısı** | Tek/çift: 7.5°, yarım: 3.75° |
| **Tam tur** | Tek/çift: 48 adım, yarım: 96 adım |
| **En yüksek tork** | Çift fazlı (tam adım) |
| **En hassas** | Yarım adım |
