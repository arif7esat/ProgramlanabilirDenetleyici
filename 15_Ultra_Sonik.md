# 📘 Ultrasonik Sensör ile Mesafe Ölçme — Konu Anlatımı

> **Kaynak Dosya:** [ultra_sonik.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/ultra_sonik.pbp)
> **Konu:** HC-SR04 ile temel mesafe ölçme ve LCD'ye yazdırma

---

## 📌 1. Bu Kod Ne Yapıyor?

Ultrasonik sensörle sürekli **mesafe ölçer** ve LCD ekranda gösterir. BoyOlcme'nin daha basit halidir.

```
┌──────────────────┐
│mesafe: 15 cm     │
│                  │
└──────────────────┘
```

---

## 📌 2. Temel Yapı

```basic
symbol trigger = portB.7      ' Tetikleme pini
symbol echo = portB.6         ' Yankı pini
salinim var word              ' Salınım süresi
mesafe var word               ' Mesafe (cm)
sonuc_cm con 6                ' Dönüşüm katsayısı
trigger = 0                   ' Trigger başlangıçta LOW

label:
    gosub oku                  ' Mesafe oku
    lcdout $fe, 1              ' Ekranı temizle
    lcdout "mesafe: ", #mesafe, " cm"   ' Yazdır
    pause 100
goto label

oku:
    pulsout trigger, 1         ' 10µs tetikleme darbesi
    pulsin echo, 1, salinim    ' Yankı süresini ölç
    mesafe = (salinim / sonuc_cm)
    pause 10
return
```

Bu kodun her satırını önceki konu anlatımlarında (03_BoyOlcme) detaylıca açıklamıştık.

---

## 📌 3. Sınav İçin Bu Dosyadan Hatırlanacaklar

| Konu | Hatırla |
|:---|:---|
| **PULSOUT** | Trigger'a darbe gönder |
| **PULSIN** | Echo'dan süre ölç |
| **CON** | Sabit tanımla |
| **WORD** | 16 bit değişken (mesafe için gerekli) |
| **`#mesafe`** | Sayı olarak LCD'ye yazdır |
| **PAUSE 100** | Ölçümler arası bekleme |
| **trigger = 0** | Başlangıçta LOW (sensör hazır) |
