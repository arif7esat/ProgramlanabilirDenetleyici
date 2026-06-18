# 📘 Motor Sürücü (Label Tabanlı) — Konu Anlatımı

> **Kaynak Dosya:** [MotorSurucu.pbp](file:///c:/Users/Aleyna/Desktop/denetleyici/MotorSurucu.pbp)
> **Konu:** Etiket tabanlı ileri-geri motor sürme, GOTO ile döngü oluşturma

---

## 📌 1. Bu Kod Ne Yapıyor?

Step motoru **ileri-geri (ping-pong)** hareket ettirir. İleri yönde 4 adım, sonra geri yönde 4 adım, sürekli tekrar.

Bu önceki MOTOR.pbp'den farklı olarak **FOR döngüsü kullanmaz**, sadece **GOTO + etiket** ile çalışır.

---

## 📌 2. İki Etiketli Yapı

```basic
label1:
    pulsout portA.2, 100
    if i = 16 then goto label2    ' 16'ya ulaştıysa geri dön
    portB = i : pause 500
    i = i << 1                     ' Sola kaydır (ileri)
goto label1

label2:
    pulsout portA.2, 100
    i = i >> 1                     ' Sağa kaydır (geri)
    portB = i : pause 500
    if i = 1 then goto label1      ' 1'e ulaştıysa ileri dön
goto label2
```

### Akış:

```
label1 (ileri): 1 → 2 → 4 → 8 → 16 olunca → label2'ye git
label2 (geri):  8 → 4 → 2 → 1 olunca → label1'e git
```

> [!NOTE]
> Bu yaklaşım GOSUB/RETURN yerine GOTO kullanır. Daha basit ama daha az yapılandırılmıştır. Sınavda "Bu kodu FOR döngüsüyle yeniden yazın" şeklinde soru gelebilir.

---

## 📌 3. FOR-NEXT vs GOTO Karşılaştırma

| Özellik | FOR-NEXT | GOTO + Etiket |
|:---|:---|:---|
| Okunabilirlik | ✅ Daha temiz | ❌ Karmaşık |
| Esneklik | Sabit tekrar sayısı | Koşula bağlı |
| Alt program | GOSUB ile kullanılır | Doğrudan dallanma |
| Kullanım | Belirli tur sayısı | Sınır kontrolü |

---

## 📌 4. Sınav İçin Dikkat Noktaları

| Konu | Hatırla |
|:---|:---|
| **İki etiketli yapı** | İleri (label1) ve geri (label2) arasında zıplama |
| **GOTO vs GOSUB** | GOTO geri dönmez, GOSUB geri döner |
| **Taşma kontrolü** | i=16 → geri dön, i=1 → ileri dön |
| **Sıra farkı** | label1'de önce portB sonra kaydır, label2'de önce kaydır sonra portB |
