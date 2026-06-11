# Stetoskop — trzy warianty wyjścia w README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rozbudować README.md o trzy warianty budowy (Opcja 1 = istniejąca TS/bateria, Opcja 2 = XLR+TRS/phantom, Opcja 3 = XLR+TRS/hybryda), tabele porównawcze, layout veroboard MCP6004 DIP-14 i trzy oddzielne kosztorysy.

**Architecture:** Plik `README.md` rozbudowujemy in-place — dodajemy sekcje Opcja 1/2/3, opakowując istniejącą treść jako Opcja 1. Źródłem technicznym jest `docs/superpowers/specs/2026-06-08-balanced-output-variants-design.md` (rev 6). Żadnych nowych plików — tylko edycja README.

**Tech Stack:** Markdown, veroboard THT, MCP6004-I/P DIP-14, MCP1703-5002E, BAT85, BZX55C9V1.

---

## Pliki do edycji

| Plik | Akcja |
|---|---|
| `README.md` | Modyfikacja — 7 zadań poniżej |
| `docs/superpowers/specs/2026-06-08-balanced-output-variants-design.md` | Tylko czytanie (źródło prawdy) |

---

## Zadanie 1: Nagłówek "Warianty budowy" + tabela porównawcza

**Pliki:**
- Modyfikuj: `README.md` (po pierwszym akapicie "Projekt akcesorium do magisterki…", przed sekcją "## Specyfikacja docelowa")

- [ ] **Krok 1: Wstaw sekcję "Warianty budowy" przed "## Specyfikacja docelowa"**

Znajdź w README linię:
```
## Specyfikacja docelowa
```

Przed nią wstaw:

```markdown
## Warianty budowy

Projekt oferuje trzy warianty budowy współdzielące tę samą kapsułkę WM-61A i rdzeń wzmacniający (60 dB, dwa stopnie ×33). Różnią się wyjściem i źródłem zasilania.

| Parametr | Opcja 1 — TS, bateria | Opcja 2 — XLR+TRS, phantom | Opcja 3 — XLR+TRS, hybryda |
|---|---|---|---|
| Wyjście | TS 6.35mm (niebalansowane) | XLR 3-pin + TRS 6.35mm (zbalansowane) | XLR 3-pin + TRS 6.35mm (zbalansowane) |
| Zasilanie | 2×18650 Li-ion (USB-C ładowanie) | Phantom 48V (IEC 61938) | Phantom 48V **lub** 2×18650 (auto-priorytet) |
| IC | NE5532N (DIP-8) | MCP6004-I/P (DIP-14, quad) | MCP6004-I/P (DIP-14, quad) |
| Max długość kabla | ~5 m | >50 m | >50 m |
| CMRR | — | ≥60 dB | ≥60 dB |
| USB-C ładowanie | ✓ | — | ✓ |
| Autonomia bez phantom | ~300 h | 0 h | ~300 h |
| Obudowa | Hammond 1590BB | Hammond 1590BB | Hammond 1590BB |
| Koszt orientacyjny | ~159–259 zł | ~135–217 zł | ~196–320 zł |

Opcja 1 opisana jest w pełni poniżej. Opcje 2 i 3 dodane są jako oddzielne sekcje na końcu dokumentu.

---
```

- [ ] **Krok 2: Dodaj nagłówek "Opcja 1" przed "## Specyfikacja docelowa"**

Bezpośrednio przed `## Specyfikacja docelowa` wstaw:

```markdown
## Opcja 1 — TS 6.35mm, zasilanie bateryjne 2×18650

```

- [ ] **Krok 3: Commit**

```bash
git add README.md
git commit -m "docs: sekcja Warianty budowy + tabela porównawcza + nagłówek Opcja 1"
```

---

## Zadanie 2: Aktualizacja obudowy 1590B → 1590BB i diagram panelu

**Pliki:**
- Modyfikuj: `README.md`

Uzasadnienie: Hammond 1590B (92×38×31mm) jest za ciasna dla Neutrik NC3FBH (gniazdo XLR). Wszystkie trzy warianty używają jednej obudowy Hammond 1590BB (119×94×34mm) dla spójności panelu.

- [ ] **Krok 1: Zastąp wszystkie wystąpienia "1590B" (nie "1590BB") na "1590BB"**

```bash
sed -i '' 's/Hammond 1590B\b/Hammond 1590BB/g' README.md
```

Zweryfikuj:
```bash
grep -n "1590B" README.md
```
Oczekiwany wynik: wszystkie trafienia zawierają "1590BB".

- [ ] **Krok 2: Zastąp diagram montażu preamp boxa**

Znajdź blok zaczynający się od:
```
  Widok od góry (panel wieczka):
  ┌───────────────────────────────────────┐
```
i kończący się na ostatniej linii `└` bloku.

Zastąp całą tę sekcję:
```markdown
  Panel przedni (Hammond 1590BB, 119×94mm):
  ┌──────────────────────┐
  │  [TS 3.5mm IN]       │
  │  [GAIN TRIM ▽]       │
  │  [LED]               │
  └──────────────────────┘

  Panel tylny:
  ┌──────────────────────────────┐
  │  [USB-C]    [ON/OFF]         │
  │  [TS 6.35mm OUT †]           │
  └──────────────────────────────┘

  † tylko Opcja 1. Opcje 2/3 mają TRS 6.35mm + XLR zamiast TS 6.35mm i brak USB-C (Opcja 2).
```

- [ ] **Krok 3: Zaktualizuj BOM wiersz obudowy**

Znajdź: `| Obudowa | Hammond 1590B lub equiv. | metalowa, mieści 2×18650 |`
Zastąp: `| Obudowa | Hammond 1590BB (119×94×34mm) | metalowa; jeden rozmiar dla wszystkich wariantów |`

- [ ] **Krok 4: Zaktualizuj sekcję "Decyzje projektowe" o TS/phantom**

Znajdź nagłówek:
```
### Dlaczego wyjście niebalansowane TS zamiast XLR / phantom 48V?
```

Zastąp nagłówek i akapit:
```markdown
### Dlaczego Opcja 1 ma wyjście niebalansowane TS zamiast XLR / phantom 48V?
Opcja 1 celuje w prostotę budowy i autonomię (~300 h). Przy odległości stetoskop–interfejs ≤ 2–5 m zakłócenia na kablu TS są pomijalne. NE5532 (DIP-8) pobiera ~8 mA — na granicy normy phantom IEC 61938 (~10 mA per urządzenie). Zasilanie bateryjne eliminuje zależność od interfejsu z phantom power.

Opcja 2 i 3 zapewniają wyjście zbalansowane XLR+TRS z CMRR ≥60 dB i pracą na kablach >50 m — opisane w dedykowanych sekcjach.
```

- [ ] **Krok 5: Commit**

```bash
git add README.md
git commit -m "docs: obudowa 1590B→1590BB, diagram panelu Opcji 1, decyzje projektowe"
```

---

## Zadanie 3: Opcja 2 — phantom receiver + balanced driver (schemat i weryfikacja)

**Pliki:**
- Modyfikuj: `README.md` (wstaw przed `## Powiązane projekty`)

- [ ] **Krok 1: Wstaw pełną sekcję Opcji 2 przed "## Powiązane projekty"**

Znajdź linię `## Powiązane projekty` i wstaw przed nią:

````markdown
---

## Opcja 2 — XLR+TRS zbalansowane, zasilanie wyłącznie phantom 48V

### Architektura

```
[Interfejs audio]         [Preamp box — Opcja 2]                    [Interfejs audio]
  Phantom 48V       XLR   ┌──────────────────────────────────────┐
  R_ph=6,81kΩ ──────────>│ R_dc1/R_dc2 (6,81kΩ 0,1%) → V_raw   │
                           │ Z1(BZX55C9V1 9V) + C_f1 + LDO 5V    │
[Głowica]         TS      │ MCP6004 quad DIP-14:                  │
  WM-61A     3.5mm────>   │  U1A×33 → U1B×33 (60 dB rdzeń)       │──XLR pin2/TRS tip──>
                           │  U2A (follower HOT)                   │
                           │  U2B (inwerter COLD ×-1)              │──XLR pin3/TRS ring─>
                           └──────────────────────────────────────┘
```

Wspólny rdzeń (WM-61A → U1A×33 → U1B×33) taki sam jak w Opcji 1. IC zmienia się z NE5532N (DIP-8) na MCP6004-I/P (DIP-14, quad) — dwa dodatkowe wzmacniacze obsługują balanced driver. Zasilanie pochodzi wyłącznie z phantom 48V.

### Phantom receiver — jak to działa

Interfejs wysyła 48V przez R_ph=6,81kΩ na XLR pin2 i pin3. Urządzenie podłącza te piny przez R_dc1/R_dc2=6,81kΩ do wspólnego węzła V_raw:

```
48V → R_ph1(6,81kΩ) → XLR pin2 → R_dc1(6,81kΩ, 0,1%) ─┐
48V → R_ph2(6,81kΩ) → XLR pin3 → R_dc2(6,81kΩ, 0,1%) ─┴─→ V_raw
                                                              ├─ Z1: BZX55C9V1 (9V, 500mW) → GND
                                                              ├─ C_f1: 100µF / 25V elektrolit → GND
                                                              ├─ R_pull_2: 22kΩ → węzeł kapsuły WM-61A
                                                              └─ MCP1703-5002E (LDO, Vin_max=16V) → V+ = 5V
                                                                   └─ C_f2: 100nF ceramiczny → GND
XLR pin1 → GND
```

**Napięcie Thevenin V_th = 48V.** Przy braku obciążenia V_raw nie ma ścieżki do GND → I = 0 → V_raw = 48V. Zener BZX55C9V1 klamruje V_raw do 9V — 1V margines do WM-61A Vs_max=10V.

**Prąd dostępny:** I = (48 − 9) / 6,81kΩ = **5,72 mA**.

**Dlaczego R_dc = 6,81kΩ (nie cewki):** Cewka 100µH @ 20Hz ma Z = 0,013Ω — praktycznie zwiera audio do masy AC. Cewka izolująca 20Hz musiałaby mieć >1H — nierealny DIY. Rezystor R_dc = 6,81kΩ (dopasowany do R_ph) poprawia CMRR i powoduje tylko −0,33 dB straty poziomu audio.

**Napięcie DC na C_hot/C_cold:** I_gałąź = I_total / 2 = 2,86 mA → V_pin = 48 − 2,86 mA × 6,81kΩ = **28,5V DC**. Po stronie urządzenia VMID ≈ 2,5V DC. Przez kondensatory C_hot/C_cold przepływa **~26V DC** — dlatego wymagane znamionowanie **≥63V** (25V lub 35V ulega uszkodzeniu przy pierwszym podłączeniu phantom).

### Balanced driver

MCP6004 pin 4 = VDD = V+. MCP6004 pin 11 = VSS = GND.  
U1A/U1B = stopnie ×33. U2A/U2B = balanced driver.

```
N4 → C_out(22µF NP) → SIG_OUT

SIG_OUT → U2A (voltage follower):
    U2A IN+ (pin 10) = SIG_OUT
    U2A IN- (pin  9) = U2A OUT (pin 8)   ← sprzężenie ujemne
    U2A OUT → R_ser_hot(100Ω) → C_hot(10µF/63V NP) → XLR pin2 + TRS tip

SIG_OUT → U2B (inwerter ×-1):
    R_U2B_in(10kΩ, 0,1%) → U2B IN- (pin 13)
    R_U2B_fb(10kΩ, 0,1%) → U2B IN- (pin 13) ← U2B OUT (pin 14)
    U2B IN+ (pin 12) → VMID
    U2B OUT → R_ser_cold(100Ω) → C_cold(10µF/63V NP) → XLR pin3 + TRS ring

XLR pin1 = TRS sleeve = GND
```

CMRR ≥ 60 dB: R_U2B_in = R_U2B_fb = 10kΩ **0,1%**. R_dc1 = R_dc2 = 6,81kΩ **0,1%**.

**C_VMID (obowiązkowo):** VMID generowane przez dzielnik 470kΩ||470kΩ = 235kΩ. Bez C_VMID tętnienie V+ sprzęga się przez VMID do wejść odwracających wszystkich czterech wzmacniaczy. Dodać **C_VMID = 10µF NP / 25V** z VMID → GND (fc = 0,07Hz).

### Weryfikacja — budżet prądowy phantom

| Komponent | I_max |
|---|---|
| R_pull_2 (22kΩ, V_raw=9V) | 0,409 mA |
| MCP6004 quad (170µA/amp × 4 @ 5V, max) | 0,680 mA |
| MCP1703 (LDO quiescent) | 0,120 mA |
| R_VMID (470kΩ×2, V+=5V) | 0,005 mA |
| **Razem** | **1,214 mA** |
| **I_available (zener 9V)** | **5,720 mA** |
| **Margines worst-case** | **79 %** |

### Parametry Opcji 2

| Parametr | Wartość |
|---|---|
| Zasilanie | Phantom 48V (IEC 61938) |
| V_raw (zener BZX55C9V1) | 9V — margines 1V do WM-61A Vs_max=10V |
| V+ (op-ampy MCP6004) | 5V (MCP1703-5002E LDO) |
| I_available z phantomu | 5,72 mA |
| I_total worst-case | 1,21 mA (margines **79 %**) |
| CMRR | ≥ 60 dB |
| DC przez C_hot/C_cold | ~26V → min **63V rating** |
| f_HPF wyjście | 1,6 Hz (−0,01 dB @ 20 Hz) |
| Max długość kabla | >50 m (XLR ekranowany) |
````

- [ ] **Krok 2: Commit**

```bash
git add README.md
git commit -m "docs: Opcja 2 — phantom receiver, balanced driver, weryfikacja budżetu"
```

---

## Zadanie 4: Opcja 2 — BOM i layout veroboard MCP6004 DIP-14

**Pliki:**
- Modyfikuj: `README.md` (dołącz bezpośrednio po sekcji z Zadania 3, przed `## Opcja 3`)

- [ ] **Krok 1: Wstaw BOM Opcji 2 (po "### Parametry Opcji 2", przed `---`)**

````markdown
### BOM — Opcja 2

#### Elektronika

| Poz. | Element | Wartość/typ | Ilość |
|---|---|---|---|
| U1 | Quad op-amp (2 stopnie gain + balanced driver) | **MCP6004-I/P (DIP-14)** + podstawka DIP-14 | 1 |
| VR1 | LDO 5V (Vin_max=16V — wymagany dla ~9V z zenera) | **MCP1703-5002E/TO (TO-92)** | 1 |
| Z1 | Zener ochronny V_raw (WM-61A Vs_max=10V → 9V+1V zapas) | **BZX55C9V1 (9V, 500mW, SOD-80)** | 1 |
| R_dc1, R_dc2 | DC tap phantom (symetryczne z R_ph, CMRR) | **6,81kΩ, 0,1%, E96** | 2 |
| R_U2B_in, R_U2B_fb | Inwerter U2B (CMRR ≥60dB — parowanie 0,1%) | **10kΩ, 0,1%** | 2 |
| R_pull_2 | Zasilanie kapsuły (V_raw=9V → Vdd_cap=3,5V) | 22kΩ, 1% | 1 |
| R_in | Wejściowy każdego stopnia gain | 909Ω, E96, 1% | 2 |
| R_fb | Sprzężenie zwrotne każdego stopnia (gain=33×) | 30,1kΩ, E96, 1% | 2 |
| R_ser_hot, R_ser_cold | Izolacja wyjść HOT/COLD (stabilność do kapacytancji kabla) | 100Ω, 1% | 2 |
| R_VMID | Dzielnik wirtualnej masy | 470kΩ, 1% | 2 |
| R_bleed | DC=0V na SIG_OUT (anty-pop) | 100kΩ, 1% | 1 |
| C_f1 | Bulk filter V_raw | 100µF / 25V elektrolit | 1 |
| C_f2 | Decouple LDO wyjście | 100nF ceramiczny | 1 |
| C_in, C_inter | Sprzęgające wejście/międzystopniowy | 22µF / 16V **NP** elektrolit | 2 |
| C_out | Sprzęganie rdzenia → SIG_OUT | 22µF / 16V **NP** elektrolit | 1 |
| C_hot, C_cold | DC block HOT/COLD — **~26V DC!** | **10µF / 63V NP** elektrolit | 2 |
| C_VMID | Bypass VMID (fc=0,07Hz, PSRR) | 10µF / 25V **NP** elektrolit | 1 |
| C_decouple | Odsprzęganie V+ blisko U1 pin4 | 100nF ceramiczny | 1 |
| RV1 | Gain trim (panel) | 10kΩ, liniowy, mono | 1 |
| J_XLR | Gniazdo XLR żeńskie, panel | **Neutrik NC3FBH** | 1 |
| J_TRS | Gniazdo TRS 6.35mm, panel | stereo 6.35mm | 1 |
| J_IN | Gniazdo wejściowe kapsuły | TS 3.5mm mono, panel | 1 |

#### Mechanika

| Element | Uwagi |
|---|---|
| Hammond 1590BB (119×94×34mm) | metalowa, jeden rozmiar dla wszystkich wariantów |
| Veroboard ~5×7cm | DIP-14 zajmuje 7 kolumn + obwód phantom po tej samej płytce |
| Kabel głowicy TS 3.5mm, 1,5m, ekranowany | |
| Wtyczka TS 3.5mm (na kabel głowicy) | |

### Layout veroboard — Opcja 2 (MCP6004 DIP-14)

#### Przypisanie pinów MCP6004-I/P w tym układzie

| Pin | Sygnał | Wzmacniacz |
|---|---|---|
| 1 | OUT_A | U1A — wyjście stopnia 1 (N2) |
| 2 | IN-_A | U1A — wejście odwracające |
| 3 | IN+_A | U1A — wejście nieodwracające (VMID) |
| **4** | **VDD = V+** | **zasilanie +5V** |
| 5 | IN+_B | U1B — wejście nieodwracające (VMID) |
| 6 | IN-_B | U1B — wejście odwracające |
| 7 | OUT_B | U1B — wyjście stopnia 2 (N4) |
| 8 | OUT_C | U2A — wyjście voltage follower (HOT pre-Rser) |
| 9 | IN-_C | U2A — wejście odwracające (= pin8, follower) |
| 10 | IN+_C | U2A — wejście nieodwracające (SIG_OUT) |
| **11** | **VSS = GND** | **punkt star-ground** |
| 12 | IN+_D | U2B — wejście nieodwracające (VMID) |
| 13 | IN-_D | U2B — wejście odwracające |
| 14 | OUT_D | U2B — wyjście inwerter COLD |

> **Zweryfikuj pinout w datasheecie MCP6004-I/P przed montażem.** Powyższy odpowiada standardowej kolejności quad op-amp DIP-14 (LM324/TL074 style). Pin 4 = VDD, pin 11 = VSS.

#### Przecięcia pasków miedzi — DIP-14 (7 wymaganych)

DIP-14 okracza 7 kolumn (A–G). Każdy pasek łączy pin górnego rzędu (1–7) z odpowiednim pinem dolnego rzędu (14–8). Wymagane **7 przecięć** — jedno na kolumnę:

```
Kolumna:  A    B    C    D    E    F    G
          │    │    │    │    │    │    │
pasek ────┼────┼────┼────┼────┼────┼────┼──[IC body]──┼────┼────┼────┼────┼────┼────┼──
          │    │    │    │    │    │    │              │    │    │    │    │    │    │
DIP-14: pin1 pin2 pin3 pin4 pin5 pin6 pin7          pin8 pin9 p10 p11 p12 p13 p14
        N2  IN-A VMID V+  VMID IN-B  N4            HOT  IN-  SIG GND VMID IN-D COLD
                                                    pre  C+   OUT           pre

✂ = przecięcie paska między rzędem pin1-7 a pin8-14, w każdej z 7 kolumn A–G
```

> **Częsty błąd:** DIP-14 wymaga 7 przecięć (vs 4 dla DIP-8 NE5532). Zweryfikować miernikiem
> ciągłości KAŻDĄ z par: **1–14, 2–13, 3–12, 4–11, 5–10, 6–9, 7–8** — każda musi być **rozwarta**
> przed włożeniem układu w podstawkę.

#### Netlist Opcji 2 — 40 połączeń

Węzły: `CAP`=kapsuła, `N1`–`N4`=wewnętrzne, `SIG_OUT`=za C_out, `VMID`=V+/2, `V_raw`=9V po zenerze, `V+`=5V z LDO, `HOT`/`COLD`=XLR+TRS wyjście.

| # | Połączenie | Uwagi |
|---|---|---|
| 1 | TS 3.5mm IN (tip) → CAP | sygnał z głowicy |
| 2 | TS 3.5mm IN (sleeve) → GND | masa sygnałowa |
| 3 | CAP → R_pull_2 (22kΩ) → V_raw | zasilanie kapsuły FET |
| 4 | CAP → RV1 terminal górny | bezpośrednie AC+DC |
| 5 | RV1 terminal dolny → GND | dzielnik napięcia |
| 6 | RV1 suwak → C_in (+) | AC-sprzężenie PO trymerze |
| 7 | C_in (−) → R_in (909Ω) → N1 | wejście U1A |
| 8 | N1 → R_fb (30,1kΩ) → N2 | sprzężenie zwrotne U1A |
| 9 | U1A IN+ (pin3) → VMID | bias nieodwracający |
| 10 | U1A OUT (pin1) = N2 | wyjście stopnia 1 |
| 11 | N2 → C_inter (+) | sprzęgacz międzystopniowy |
| 12 | C_inter (−) → R_in (909Ω) → N3 | wejście U1B |
| 13 | N3 → R_fb (30,1kΩ) → N4 | sprzężenie zwrotne U1B |
| 14 | U1B IN+ (pin5) → VMID | bias nieodwracający |
| 15 | U1B OUT (pin7) = N4 | wyjście stopnia 2 |
| 16 | N4 → C_out (+) | sprzęgacz wyjściowy |
| 17 | C_out (−) = SIG_OUT | wyjście rdzenia |
| 18 | SIG_OUT → R_bleed (100kΩ) → GND | anty-pop |
| 19 | SIG_OUT → U2A IN+ (pin10) | wejście voltage follower HOT |
| 20 | U2A IN- (pin9) = U2A OUT (pin8) | sprzężenie ujemne follower |
| 21 | U2A OUT (pin8) → R_ser_hot (100Ω) → C_hot (+) | HOT przed DC-blokiem |
| 22 | C_hot (−) → XLR pin2 + TRS tip | wyjście HOT |
| 23 | SIG_OUT → R_U2B_in (10kΩ, 0,1%) → U2B IN- (pin13) | wejście inwertera COLD |
| 24 | U2B OUT (pin14) → R_U2B_fb (10kΩ, 0,1%) → U2B IN- (pin13) | sprzężenie inwertera |
| 25 | U2B IN+ (pin12) → VMID | ref inwertera ×-1 |
| 26 | U2B OUT (pin14) → R_ser_cold (100Ω) → C_cold (+) | COLD przed DC-blokiem |
| 27 | C_cold (−) → XLR pin3 + TRS ring | wyjście COLD |
| 28 | XLR pin1 = TRS sleeve = GND | masa sygnałowa wyjścia |
| 29 | XLR pin2 → R_dc1 (6,81kΩ, 0,1%) → V_raw | phantom DC tap HOT |
| 30 | XLR pin3 → R_dc2 (6,81kΩ, 0,1%) → V_raw | phantom DC tap COLD |
| 31 | V_raw → Z1 (BZX55C9V1, 9V, 500mW) → GND | ochrona Vs_max WM-61A |
| 32 | V_raw → C_f1 (100µF/25V) → GND | bulk filter |
| 33 | V_raw → MCP1703 Vin | wejście LDO |
| 34 | MCP1703 Vout = V+ | 5V dla MCP6004 i VMID |
| 35 | V+ → C_f2 (100nF) → GND | decouple LDO |
| 36 | VMID = dzielnik R_VMID(470kΩ) + R_VMID(470kΩ) z V+ → GND | wirtualna masa |
| 37 | VMID → C_VMID (10µF NP / 25V) → GND | bypass VMID, fc=0,07Hz |
| 38 | U1 pin4 (VDD) → V+; C_decouple (100nF) → GND blisko pinu | odsprzęganie zasilania |
| 39 | U1 pin11 (VSS) → GND — **punkt star-ground** | wszystkie GND zbiegają tu |
| 40 | Obudowa → star-ground (jeden punkt) | ekranowanie EMI |

#### Kolejność montażu — Opcja 2

1. Wykonać **7 przecięć** w obszarze DIP-14 (kolumny A–G, między rzędami pinów).
2. **Zweryfikować miernikiem** — 7 par (1-14, 2-13, 3-12, 4-11, 5-10, 6-9, 7-8) musi być rozwarcie.
3. Zamontować podstawkę DIP-14 (bez układu scalonego).
4. Wlutować TO-92 (MCP1703) i SOD-80 (BZX55C9V1).
5. Wlutować rezystory 0,1% jako pierwsze (R_dc1/R_dc2, R_U2B_in/fb), następnie resztę rezystorów.
6. Wlutować kondensatory ceramiczne (C_f2, C_decouple).
7. Wlutować kondensatory elektrolityczne: C_f1 (100µF/25V, spolaryzowany), C_VMID (10µF NP), C_in/C_inter/C_out (22µF NP), C_hot/C_cold (10µF/**63V** NP — **uwaga na napięcie!**).
8. Połączenia zewnętrzne: RV1, J_IN, J_XLR (Neutrik NC3FBH), J_TRS.
9. **Ciągłość całego netlistu miernikiem PRZED włożeniem MCP6004 w podstawkę.**
10. Pierwsze włączenie (XLR do interfejsu z phantom 48V): zmierzyć V_raw ≈ 9V, V+ ≈ 5V, VMID ≈ 2,5V. Jeśli V_raw > 10V — **STOP**, sprawdzić Z1.
````

- [ ] **Krok 2: Commit**

```bash
git add README.md
git commit -m "docs: Opcja 2 — BOM, netlist 40 połączeń, veroboard DIP-14, montaż"
```

---

## Zadanie 5: Opcja 3 — hybryda (phantom + bateria)

**Pliki:**
- Modyfikuj: `README.md` (wstaw po sekcji Opcji 2, przed `## Powiązane projekty`)

- [ ] **Krok 1: Wstaw sekcję Opcji 3**

Znajdź `## Powiązane projekty` i wstaw przed nią:

````markdown
---

## Opcja 3 — XLR+TRS zbalansowane, zasilanie hybrydowe (phantom + bateria 2×18650)

Opcja 3 jest identyczna z Opcją 2 **z wyjątkiem sekcji zasilania**: diode-OR łączy V_raw phantom z V_bat (bateria 2×18650) przed zenerem i LDO. Układ wzmacniający, balanced driver, gniazda XLR+TRS, BOM elektroniki — wszystko bez zmian.

### Schemat zasilania hybrydowego (delta względem Opcji 2)

```
Phantom: R_dc1/R_dc2 → V_raw_ph → D1 (BAT85, Vf=0,3V) ─┐
Bateria: 2×18650 + BMS 2S → V_bat  → D2 (BAT85, Vf=0,3V) ─┴─→ V_bus
                                                              ├─ Z1: BZX55C9V1 (9V) → GND
                                                              ├─ C_bus: 100µF / 25V → GND
                                                              ├─ R_pull_2 (22kΩ) → kapsuła
                                                              └─ MCP1703-5002E → V+ = 5V
TP5100 + USB-C → ładowanie baterii (niezależnie od phantom)
```

### Weryfikacja V_bus — priorytety

| Źródło | V_bus po diodzie Schottky | LDO Vin | Margines do Vin_max=16V |
|---|---|---|---|
| Phantom (zener klamped) | 9 − 0,3 = **8,7V** | ✓ | 7,3V |
| Bateria pełna (8,4V) | 8,4 − 0,3 = **8,1V** | ✓ | 7,9V |
| Bateria minimalna (6,0V) | 6,0 − 0,3 = **5,7V** | dropout ≈5mV → Vout≈5V ✓ | 695mV |

**Priorytet automatyczny:** V_bus_phantom = 8,7V > V_bus_bat_max = 8,1V → D1 prowadzi, D2 zablokowana. Przełączenie phantom→bateria przy odłączeniu XLR: bufor C_bus (100µF) eliminuje glitch audio.

> **Margines diode-OR:** różnica V_bus wynosi **0,6V**. Prąd wsteczny BAT85 w zakresie temperatur roboczych (−20…+60°C) wynosi <1µA — pomijalny wobec I_total=1,21mA. Wariant konserwatywny: D2 = 1N4148 (Vf≈0,6V) → V_bus_bat=7,8V, margines 0,9V.

### Ładowanie baterii przez phantom

Phantom 48V **nie ładuje** baterii bezpośrednio — linia phantom jest odizolowana zenarem i D1. TP5100 (zasilany USB-C 5V) ładuje baterie niezależnie od stanu phantom. Podłączenie phantom nie wpływa na cykl ładowania.

### BOM — komponenty dodatkowe Opcji 3 (delta od Opcji 2)

| Ref | Opis | Wartość |
|---|---|---|
| D1 | Dioda priorytetowa phantom | BAT85 Schottky |
| D2 | Dioda priorytetowa bateria | BAT85 Schottky (lub 1N4148 — konserwatywnie) |
| C_bus | Bufor szyny V_bus (zastępuje C_f1 z Opcji 2) | 100µF / 25V elektrolit |
| BT1 | Ogniwa Li-ion | 2× 18650 markowe (Samsung/Molicel — **nie no-name**) |
| U_BMS | BMS 2S 8.4V | np. HX-2S-A10 (ochrona nad/pod-napięciowa) |
| U_CHG | Ładowarka Li-ion 2S | TP5100 |
| J_USB | Gniazdo ładowania | USB-C + ew. PD trigger CH224K |
| SW1 | Przełącznik ON/OFF | dźwigniowy, panel mount |

> Wszystkie pozostałe komponenty — identyczne z Opcją 2.

### Różnice montażowe Opcji 3 vs Opcji 2

- D1/D2 (BAT85): katoda do V_bus, anoda do V_raw_ph/V_bat.
- C_bus zajmuje pozycję C_f1 z Opcji 2 (ta sama topologiczna pozycja bufor/filtr).
- Koszyczek 2×18650 + BMS po stronie opozytnej do veroboard (unikaj bliskości TP5100 i MCP6004).
- BMS GND = GND układu (wspólna masa), BMS V+ = V_bat do D2 anody.
- TP5100 i USB-C na panelu tylnym.

### Parametry Opcji 3

| Parametr | Wartość |
|---|---|
| Zasilanie | Phantom 48V (priorytet) lub 2×18650 (auto-fallback) |
| V_bus | 8,7V (phantom) / 8,1–5,7V (bateria) |
| V+ (op-ampy) | 5V (MCP1703-5002E LDO) |
| Margines prądowy | 79% (identyczny z Opcją 2) |
| CMRR | ≥ 60 dB |
| DC przez C_hot/C_cold | ~26V → min 63V rating |
| Autonomia bez phantom | ~300h (2×18650, zależnie od ogniw) |
| USB-C ładowanie | ✓ (TP5100, niezależnie od phantom) |
````

- [ ] **Krok 2: Commit**

```bash
git add README.md
git commit -m "docs: Opcja 3 — zasilanie hybrydowe, diode-OR, BOM delta, parametry"
```

---

## Zadanie 6: Kosztorys trójwariantowy

**Pliki:**
- Modyfikuj: `README.md` — zastąp istniejącą sekcję `## Kosztorys`

- [ ] **Krok 1: Zastąp całą sekcję "## Kosztorys" poniższą treścią**

Znajdź `## Kosztorys (orientacyjny, do potwierdzenia cen na TME/Botland)` i zastąp cały blok aż do następnego `##`:

```markdown
## Kosztorys orientacyjny

Ceny do potwierdzenia na TME.eu / Botland.com.pl. Wysyłka TME darmowa od ~100 zł. Kabel wyjściowy do interfejsu pominięty.

### Opcja 1 — TS, bateria 2×18650

| Komponent | Koszt |
|---|---|
| Stetoskop (używany) | 20–35 zł |
| Kapsułka Panasonic WM-61A | 5–8 zł |
| 1× NE5532N (DIP-8) + podstawka | 2–3 zł |
| Obudowa Hammond 1590BB | 35–55 zł |
| Gniazdo TS 3.5mm (panel, mono) | 4–6 zł |
| Gniazdo TS 6.35mm (panel) | 4–6 zł |
| Potencjometr 10kΩ + pokrętło | 6–10 zł |
| 2× ogniwo 18650 (Samsung/Molicel) | 30–50 zł |
| Koszyczek 2×18650 + BMS 2S | 15–25 zł |
| Moduł TP5100 (2S) + gniazdo USB-C | 12–20 zł |
| Przełącznik ON/OFF | 3–5 zł |
| Rezystory, kondensatory (BOM Opt1) | 10–15 zł |
| Płytka veroboard | 5–8 zł |
| Kabel TS 3.5mm głowicy (1,5m) + wtyczka | 8–13 zł |
| **Razem** | **~159–259 zł** |

### Opcja 2 — XLR+TRS, phantom

| Komponent | Koszt |
|---|---|
| Stetoskop (używany) | 20–35 zł |
| Kapsułka Panasonic WM-61A | 5–8 zł |
| 1× MCP6004-I/P (DIP-14) + podstawka | 4–6 zł |
| 1× MCP1703-5002E (TO-92) | 2–3 zł |
| 1× BZX55C9V1 (zener 9V, SOD-80) | 1–2 zł |
| Obudowa Hammond 1590BB | 35–55 zł |
| Gniazdo TS 3.5mm (panel, mono) | 4–6 zł |
| Gniazdo TRS 6.35mm (panel, stereo) | 5–8 zł |
| Gniazdo XLR żeńskie Neutrik NC3FBH | 15–25 zł |
| Potencjometr 10kΩ + pokrętło | 6–10 zł |
| Rezystory 0,1% (R_dc×2, R_U2B×2) | 8–12 zł |
| Rezystory 1%, kondensatory (BOM Opt2) | 12–18 zł |
| Kondensatory 10µF/63V NP (C_hot, C_cold) | 4–6 zł |
| Płytka veroboard (większa — DIP-14) | 6–10 zł |
| Kabel TS 3.5mm głowicy (1,5m) + wtyczka | 8–13 zł |
| **Razem** | **~135–217 zł** |

> Opcja 2 jest najtańsza — brak ogniw 18650, BMS i TP5100. Wymaga interfejsu audio z phantom 48V (standard od klasy Focusrite Scarlett Solo).

### Opcja 3 — XLR+TRS, hybryda

Baza = Opcja 2, dodatkowe komponenty:

| Komponent dodatkowy | Koszt |
|---|---|
| 2× ogniwo 18650 (Samsung/Molicel) | 30–50 zł |
| Koszyczek 2×18650 + BMS 2S | 15–25 zł |
| Moduł TP5100 (2S) + gniazdo USB-C | 12–20 zł |
| 2× BAT85 Schottky | 2–4 zł |
| Przełącznik ON/OFF | 3–5 zł |
| **Razem Opcja 3** | **~197–321 zł** |
```

- [ ] **Krok 2: Commit**

```bash
git add README.md
git commit -m "docs: kosztorys trójwariantowy (Opcja 1/2/3 oddzielne tabele)"
```

---

## Zadanie 7: Aktualizacja statusu projektu

**Pliki:**
- Modyfikuj: `README.md`

- [ ] **Krok 1: Zastąp sekcję "## Status projektu"**

Znajdź `## Status projektu` i zastąp całą sekcję aż do `## Powiązane projekty`:

```markdown
## Status projektu

### Wspólne (wszystkie warianty)
- [x] Architektura modułowa (głowica TS 3.5mm + preamp box)
- [x] Rdzeń wzmacniający 60 dB (dwa stopnie ×33, NE5532 / MCP6004)
- [x] Filtr HPF dobrany pod tony S3/S4 (C=22µF, −1,3 dB @ 20Hz)
- [x] Trymer wzmocnienia RV1 na wejściu (brak ryzyka clippingu)
- [x] Wartości R_in=909Ω, R_fb=30,1kΩ E96 1% (gain=33,11×/stopień, 60,80 dB)
- [x] C_VMID = 10µF NP w rdzeniu wspólnym (bypass VMID, fc=0,07Hz)
- [x] Obudowa Hammond 1590BB — jeden rozmiar dla wszystkich wariantów
- [x] Spec wariantów (rev 6): `docs/superpowers/specs/2026-06-08-balanced-output-variants-design.md`

### Opcja 1 — TS 6.35mm, bateria 2×18650
- [x] Schemat elektryczny (NE5532N DIP-8, zweryfikowany)
- [x] BOM z kosztorysem (~159–259 zł)
- [x] Layout veroboard (netlist 24 połączeń, 4 przecięcia DIP-8, kolejność montażu)
- [ ] Zakup komponentów (weryfikacja cen TME/Botland)
- [ ] Budowa i test: SNR vs dataset mediana (normal = −2,7 dB)

### Opcja 2 — XLR+TRS, phantom 48V
- [x] Spec phantom receiver (V_th=48V, zener 9V, I_avail=5,72mA, margines 79%)
- [x] Spec balanced driver (U2A follower + U2B inwerter ×-1, CMRR ≥60dB)
- [x] Weryfikacja budżetu prądowego (1,21mA / 5,72mA worst-case)
- [x] BOM (MCP6004-I/P, MCP1703, BZX55C9V1, C_hot/cold 63V, R 0,1%)
- [x] Layout veroboard (netlist 40 połączeń, 7 przecięć DIP-14, kolejność montażu)
- [ ] Zakup komponentów
- [ ] Budowa i test (V_raw≈9V, V+≈5V, VMID≈2,5V przy uruchomieniu; pomiar CMRR)

### Opcja 3 — XLR+TRS, hybryda phantom+bateria
- [x] Spec zasilania hybrydowego (diode-OR BAT85, V_bus 8,7V/8,1V/5,7V)
- [x] Weryfikacja marginesu diode-OR (0,6V; BAT85 I_r <1µA w temp. roboczej)
- [x] BOM delta (D1/D2 BAT85, C_bus, 2×18650, BMS 2S, TP5100)
- [ ] Zakup komponentów
- [ ] Budowa i test (auto-przełączanie phantom→bateria przy odłączeniu XLR)

```

- [ ] **Krok 2: Finalny commit**

```bash
git add README.md
git commit -m "docs: status projektu z checklistami per wariant (Opt1/2/3) + C_VMID, 1590BB"
```

- [ ] **Krok 3: Push do GitHub**

```bash
git push
```

---

## Self-review

### Pokrycie wymagań spec

| Wymaganie | Zadanie |
|---|---|
| Sekcja "Warianty budowy" + tabela | Zadanie 1 |
| Opcja 1 opakowana nagłówkiem | Zadanie 1 |
| Obudowa 1590B → 1590BB wszędzie | Zadanie 2 |
| Diagram panelu 1590BB | Zadanie 2 |
| Opcja 2 — phantom receiver (schemat, V_th, V_pin 28,5V) | Zadanie 3 |
| Opcja 2 — balanced driver (U2A/U2B, C_hot/cold 63V, CMRR) | Zadanie 3 |
| Opcja 2 — budżet prądowy (1,21mA / 5,72mA, margines 79%) | Zadanie 3 |
| Opcja 2 — BOM (MCP6004, MCP1703, BZX55C9V1, R 0,1%) | Zadanie 4 |
| Opcja 2 — veroboard DIP-14 (netlist 40, 7 przecięć) | Zadanie 4 |
| Opcja 3 — zasilanie hybrydowe (diode-OR, tabela V_bus) | Zadanie 5 |
| Opcja 3 — BOM delta | Zadanie 5 |
| Kosztorys: 3 oddzielne szacunki | Zadanie 6 |
| Status projektu z checklistami per wariant | Zadanie 7 |
| C_VMID = 10µF NP w rdzeniu wspólnym (spec rev6) | Zadanie 3 + 7 |

### Spójność nazw i wartości

- `V_raw` / `V_bus` — konsekwentnie rozróżniane (V_raw=Opcja2, V_bus=Opcja3) ✓
- Pin DIP-14: pin4=VDD=V+, pin11=VSS=GND (star-ground) ✓
- `C_hot/C_cold 10µF/63V NP` — spójne we wszystkich sekcjach ✓
- `R_dc1/R_dc2 6,81kΩ 0,1%` — spójne ✓
- `I_total 1,214mA → margines 79%` — spójne z budżetem prądowym w spec rev6 ✓
- Netlist Opcji 2 = 40 połączeń (vs 24 Opcji 1) — różnica: phantom receiver (29-35) + balanced driver (19-28) ✓
