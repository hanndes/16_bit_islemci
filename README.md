# 🖥️ 16-Bit İşlemci Veri Yolu Tasarımı

Logisim ortamında tasarlanmış, özel komut setine (ISA) sahip 16-bit işlemci veri yolu (datapath) tasarımı. **Bilgisayar Organizasyonu ve Mimarisi (BLM22370)** dersi kapsamında geliştirilmiştir.

**Fatih Sultan Mehmet Vakıf Üniversitesi — Bilgisayar Mühendisliği, Mayıs 2026**

---

## 📐 Mimari Genel Bakış

| Özellik | Değer |
|---|---|
| Veri Yolu Genişliği | 16-bit |
| Yazmaç Sayısı | 8 adet (X0–X7) |
| Buyruk Boyutu | 16-bit |
| Tasarım Aracı | Logisim 2.7.1 |
| Aşamalar | Fetch → Decode → Execute → Memory → Write Back |

---

## 🔧 Veri Yolu Bileşenleri

- **Program Sayacı (PC)** — Her saat darbesiyle bir sonraki buyruk adresini tutar. Dallanma gerçekleşmediğinde PC+1, dallanmada hedef adrese güncellenir.
- **Buyruk Belleği (ROM)** — 16-bit adres/veri genişliğinde salt okunur bellek. PC çıkışı adres girişine bağlıdır.
- **Yazmaç Öbeği (Register File)** — X0–X7 arası 8 adet 16-bit yazmaç. Aynı anda 2 okuma, 1 yazma destekler. X0 daima 0'dır.
- **ALU** — ADD, SUB, AND, OR işlemlerini gerçekleştirir. Zero flag üretir.
- **Veri Belleği (RAM)** — MemRead/MemWrite sinyalleriyle yönetilen okuma/yazma belleği.
- **İşaret Genişletici (Sign Extender)** — 5-bit anlık değeri 16-bit'e işaret korumalı genişletir.
- **Kontrol Birimi (Control Unit)** — 5-bit opcode'a göre tüm kontrol sinyallerini üretir.

---

## 📋 Komut Seti (ISA)

Tüm komutlar 16-bit sabit uzunluktadır.

**Format:** `[15:11] Opcode (5) | [10:8] Hedef/Kaynak1 (3) | [7:5] Kaynak2 (3) | [4:0] Imm (5)`

### Aritmetik ve Mantık Komutları

| Komut | Örnek | Açıklama |
|---|---|---|
| ADDI | `ADDI X1, X0, #0` | Xd = Xs + imm |
| SUBI | `SUBI X6, X4, #5` | Xd = Xs - imm |
| ANDI | `ANDI Xd, Xs, #imm` | Xd = Xs AND imm |
| ORI  | `ORI  Xd, Xs, #imm` | Xd = Xs OR imm |

### Bellek Komutları

| Komut | Örnek | Açıklama |
|---|---|---|
| LDR | `LDR X4, [X2, #0]` | Xd = RAM[Xs + offset] |
| STR | `STR X1, [X0, #4]` | RAM[Xs + offset] = Xd |

### Dallanma Komutları

| Komut | Örnek | Açıklama |
|---|---|---|
| BEQ | `BEQ X0, X0, #0` | Xs1 == Xs2 ise PC += label/2 |
| BNE | `BNE X6, X0, #4` | Xs1 != Xs2 ise PC += label/2 |

> Dallanma hedef adresi: 5-bit label işaret genişletilip ÷2 yapılarak mevcut PC ile toplanır.

---

## 🎛️ Kontrol Sinyalleri

| Buyruk | RegWrite | AluSrc | MemToReg | MemRead | MemWrite | Branch | Cond |
|---|---|---|---|---|---|---|---|
| ADDI | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| SUBI | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| ANDI | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| ORI  | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| LDR  | 1 | 1 | 1 | 1 | 0 | 0 | 0 |
| STR  | 0 | 1 | 0 | 0 | 1 | 0 | 0 |
| BEQ  | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| BNE  | 0 | 0 | 0 | 0 | 0 | 1 | 1 |

---

## 🧪 Test Programı

Veri belleğindeki `{ 5, 7, 5, 5 }` dizisinde **5'e eşit olan elemanların sayısını** hesaplar ve sonucu `RAM[4]`'e yazar.

**Beklenen sonuç:** `RAM[4] = 3`

### Assembly Kodu

```asm
; Dizi: mem[0..3] = { 5, 7, 5, 5 }  →  Sonuç: mem[4] = X1

ADR 0:  ADDI X1, X0, #0     ; X1 = 0  (sayaç)
ADR 1:  ADDI X2, X0, #0     ; X2 = 0  (dizi indeksi)
ADR 2:  ADDI X3, X0, #4     ; X3 = 4  (dizi boyutu)
; --- LOOP ---
ADR 3:  LDR  X4, [X2, #0]   ; X4 = mem[X2]
ADR 4:  SUBI X6, X4, #5     ; X6 = X4 - 5
ADR 5:  BNE  X6, X0, #4     ; X6 ≠ 0 ise ADR 7'ye atla
ADR 6:  ADDI X1, X1, #1     ; sayaç++
ADR 7:  ADDI X2, X2, #1     ; indeks++
ADR 8:  BNE  X2, X3, #-10   ; X2 ≠ X3 ise ADR 3'e dön
; --- Sonucu Yaz ---
ADR 9:  STR  X1, [X0, #4]   ; mem[4] = X1
ADR 10: BEQ  X0, X0, #0     ; sonsuz döngü (dur)
```

### Makine Kodları

| ADR | HEX    | BINARY (16-bit)      | ASSEMBLY         |
|-----|--------|----------------------|------------------|
| 0   | 0x0100 | 00000 001 000 00000  | ADDI X1, X0, #0  |
| 1   | 0x0200 | 00000 010 000 00000  | ADDI X2, X0, #0  |
| 2   | 0x0304 | 00000 011 000 00100  | ADDI X3, X0, #4  |
| 3   | 0x2440 | 00100 100 010 00000  | LDR X4, [X2, #0] |
| 4   | 0x0E85 | 00001 110 100 00101  | SUBI X6, X4, #5  |
| 5   | 0x3E04 | 00111 110 000 00100  | BNE X6, X0, #4   |
| 6   | 0x0121 | 00000 001 001 00001  | ADDI X1, X1, #1  |
| 7   | 0x0241 | 00000 010 010 00001  | ADDI X2, X2, #1  |
| 8   | 0x3A76 | 00111 010 011 10110  | BNE X2, X3, #-10 |
| 9   | 0x2904 | 00101 001 000 00100  | STR X1, [X0, #4] |
| 10  | 0x3000 | 00110 000 000 00000  | BEQ X0, X0, #0   |

---

## 🚀 Nasıl Çalıştırılır

1. [Logisim 2.7.1](http://www.cburch.com/logisim/) uygulamasını indirin
2. `proje.circ` dosyasını **File → Open** ile açın
3. Sol panelden `main` devresine çift tıklayın
4. ROM bileşenine sağ tıklayıp **Load Image...** ile `mock_rom` dosyasını yükleyin
5. RAM bileşenine sağ tıklayıp **Load Image...** ile `mock_ram` dosyasını yükleyin
6. **Simulate → Reset Simulation** ile sıfırlayın
7. Adım adım: **Simulate → Step Simulation** | Otomatik: **Simulate → Ticks Enabled**
8. 11 buyruk sonunda `RAM[0x0004] = 0x0003` olduğunu doğrulayın

---

## 📁 Dosya Yapısı

```
├── proje.circ      # Logisim devre dosyası (main, RegisterFile, ALU, ControlUnit)
├── mock_ram        # Veri belleği — test dizisi { 5, 7, 5, 5 }
└── mock_rom        # Program belleği — 11 buyruk (binary)
```

---

## 👩‍💻 Geliştirici

**Hatice Hande Dereli** — [@hanndes](https://github.com/hanndes)  
Fatih Sultan Mehmet Vakıf Üniversitesi, Bilgisayar Mühendisliği
