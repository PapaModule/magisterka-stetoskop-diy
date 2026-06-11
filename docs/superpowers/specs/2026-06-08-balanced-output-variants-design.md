---
name: balanced-output-variants
description: Trzy warianty wyjścia elektronicznego stetoskopu — TS niebalansowane (istniejący), TRS+XLR tylko phantom, TRS+XLR hybryda (phantom + bateria)
metadata:
  type: project
---

# Spec: Warianty wyjścia stetoskopu — TS / TRS+XLR phantom / TRS+XLR hybryda

**Data:** 2026-06-08 (rev 8: 2026-06-11)**  
**Projekt:** `PapaModule/magisterka-stetoskop-diy`  
**Status:** zweryfikowany, zatwierdzony do implementacji

---

## Historia poprawek

| Rev | Zmiana |
|---|---|
| 1 | Pierwotny draft |
| 2 | Korekta 3 blokerów: L→R_dc, MCP1700→MCP1703, TL072→MCP6004 |
| 3 | Korekta błędu Thevenin V_th=24V→48V; usunięcie ślepych zaułków analizy; C_hot/cold 1µF→10µF; dokumentacja DC na kondensatorach wyjściowych |
| 4 | Zener 15V→9V (V_raw=15V przekraczało WM-61A Vs_max=10V); korekta V_pin=28,5V (nie 44V); R_dc 1%→0,1%; R_pull_2 33kΩ→22kΩ |
| 5 | Korekta stałej V_bus_phantom 14,7V→8,7V w opisie Opt3 (stała z ery zenera 15V); Icc MCP6004 600→680µA (datasheet max @5V: 170µA/wzmacniacz×4); adnotacja o marginesie diode-OR 0,6V |
| 6 | Dodanie C_VMID = 10µF NP do rdzenia wspólnego (odsprzęganie szyny VMID) |
| 7 | RV1 przeniesiony między U1A a U1B (klasyczna topologia niskoszumowa); R_fb2 30,1kΩ→49,9kΩ; wzmocnienie 60→65 dB; nowe ostrzeżenie RV1 bottom→VMID; analiza HPF vs pozycja suwaka (−1,5 do −2,4 dB @ 20Hz) |
| 8 | RV1 usunięty całkowicie — kapsuła przy ciele, clip nierealny; gain stały 65 dB; HPF stały −1,4 dB @ 20 Hz (oba stopnie f_c=7,96 Hz); wzmocnienie w interfejsie audio zastępuje trymer |

---

## Wspólny rdzeń (niezmieniony we wszystkich wariantach)

```
WM-61A → R_pull(→V+) → C_in(22µF NP) → R_in1(909Ω) → U1A(inwert.×33, 30,4dB)
       → C_inter(22µF NP) → R_in2(909Ω) → U1B(inwert.×55, 34,8dB)
       → węzeł N4 (wyjście rdzenia, 65dB całkowite)

Zasilanie: single-supply, VMID = V+/2
Wejście kapsuły: TS 3.5mm (tip=sygnał+zasilanie, sleeve=GND)
Regulacja poziomu: wyłącznie w interfejsie audio (brak trymera w preampie)
```

Opcja 1: NE5532 (DIP-8), R_pull=2,2kΩ→V+.  
Opcje 2 i 3: MCP6004 (DIP-14, quad), R_pull_2=22kΩ→V_raw.

### Wzmocnienie i dobór R_fb

| Stopień | R_in | R_fb | Wzmocnienie |
|---|---|---|---|
| U1A (stopień 1) | 909 Ω | 30,1 kΩ, E96, 1% | ×33,1 = 30,4 dB |
| U1B (stopień 2) | 909 Ω | **49,9 kΩ, E96, 1%** | ×54,9 = 34,8 dB |
| **Łącznie** | | | **×1817 = 65,2 dB** |

Uzasadnienie 65 dB: dataset cHiFi-GAN zawiera zaszumione nagrania z tonami S3/S4
i szmerami klas I-II (SPL ~40–55 dB). Przy 65 dB: V_out = 54–173 mV → SNR ≈20 dB
dla słabych dźwięków. Clip nierealny: kapsuła WM-61A siedzi przy ciele bezpośrednio
(w dzwonku stetoskopu) → SPL przy kapsule ograniczony do zakresu stetoskopowego
(typowo 50–75 dB). Próg clipu: ~78 dB SPL — poza realnym zakresem zastosowania.

| SPL przy kapsule | V_out @ stały gain 65 dB | Ocena |
|---|---|---|
| 40 dB | ~29 mV | użyteczny (S3/S4 słabe) |
| 50 dB | ~91 mV | dobry |
| 60 dB | ~289 mV | bardzo dobry |
| 75 dB | ~1,62 V | linia nominalna |
| 78 dB | ~2,7 V peak | próg clipu (NE5532, V+=8,4V) — poza realnym zakresem |

### Brak trymera gain (uzasadnienie rev8)

Potencjometr gain trim usunięty. Analiza ryzyka clipu (Opus):

| Źródło ryzyka | Ocena |
|---|---|
| Clip od dźwięków serca | Nierealny — kapsuła przy ciele, SPL ograniczony przez akustykę stetoskopu |
| Clip od tarcia mechanicznego | Impulsowy, nie sinusoidalny — krótkookresowy, nie degraduje nagrania |
| Regulacja poziomu | Interfejs audio (gain na wejściu LINE) — wystarczający |

Usunięcie trymera eliminuje:
- Ryzyko błędu okablowania (RV1 bottom → VMID, krytyczny dla single-supply)
- Szum mechaniczny ścieraka podczas nagrania
- Zmienność HPF w zależności od pozycji suwaka (−1,5 do −2,4 dB przy suwaku)

### HPF — analiza (rev8)

Bez trymera oba stopnie mają **identyczne parametry HPF**:

| Etap | C | R_eff | f_c |
|---|---|---|---|
| Wejście U1A | C_in = 22µF | R_in1 = 909 Ω | 7,96 Hz |
| Wejście U1B | C_inter = 22µF | R_in2 = 909 Ω | 7,96 Hz |

Kaskada dwóch identycznych HPF: łączny −3 dB = 7,96 × √2 ≈ **11,3 Hz**,
tłumienie @ 20 Hz = **−1,4 dB (stały, niezależny od żadnej regulacji)**.
S3 (25–50 Hz) i S4 (20–30 Hz) zachowane. Poprawa względem rev7 (−1,5 do −2,4 dB
zależnie od suwaka) — HPF jest teraz przewidywalny i nie zmienia się w trakcie nagrania.

### C_VMID — obowiązkowo we wszystkich wariantach

VMID generowane przez dzielnik R_VMID (470kΩ||470kΩ = 235kΩ). Bez kondensatora
odsprzęgającego tętnienie V+ sprzęga się przez VMID do wejść odwracających obu
stopni wzmocnienia i inwertera cold (U2B). Przy 65dB wzmocnienia i PSRR MCP6004
~70dB @ 1kHz wpływ jest pomijalny, ale przy NF jest widoczny na niskich
częstotliwościach (PSRR MCP6004 spada do ~40dB @ 100Hz).

```
VMID → C_VMID (10µF NP, 25V) → GND
```

C_VMID tworzy z R_VMID filtr LP: fc = 1/(2π × 235k × 10µF) = **0,07Hz** — praktycznie
DC przy każdej częstotliwości audio. Dodać do każdej płytki veroboard.

---

## Opcja 1 — TS niebalansowane, zasilanie bateryjne (istniejąca, bez zmian)

```
N4 → C_out(22µF NP) → R_out(100Ω) → TS 6.35mm
                     ↘ R_bleed(100kΩ) → GND
Zasilanie: 2×18650 (2S, 6,0–8,4V) + BMS + TP5100 + USB-C.
IC: 1× NE5532N (DIP-8).
```

| Parametr | Wartość |
|---|---|
| Impedancja wyjściowa | ~100Ω |
| Max długość kabla | ~5m (niebalansowany) |
| Autonomia | ~300h |

---

## Opcja 2 — TRS+XLR zbalansowane, zasilanie wyłącznie phantom 48V

### Phantom receiver — analiza Thevenin

Obwód phantom (dwie równoległe gałęzie od interfejsu do V_raw):
```
48V → R_ph1(6,81kΩ) → pin2_XLR → R_dc1(6,81kΩ) → V_raw
48V → R_ph2(6,81kΩ) → pin3_XLR → R_dc2(6,81kΩ) → V_raw
```

**V_th = 48V.**  
Przy otwartym zacisku V_raw nie ma ścieżki do GND. KCL: I₁+I₂=0, co przy
obu gałęziach zasilanych z 48V zachodzi tylko gdy I₁=I₂=0 → V_raw=48V.
*(Błąd częsty: wzór V_th=48×R_dc/(R_ph+R_dc)=24V dotyczy dzielnika R_dc→GND.
Tu R_dc idzie do węzła wyjściowego V_raw, nie do GND.)*

**R_th = 6,81kΩ.**  
Ze źródłami skróconymi: (R_dc1+R_ph1)||(R_dc2+R_ph2) = 13,62k||13,62k = 6,81kΩ.

**I_available przy zenerze 9V:**
```
I = (V_th − V_zener) / R_th = (48 − 9) / 6,81k = 5,72 mA
```

**Dlaczego R_dc = 6,81kΩ (dopasowanie do R_ph):**
- Symetria R_dc = R_ph → poprawa CMRR odbiornika phantom.
- Tolerancja **0,1%** (jak R_U2B) dla zachowania tej symetrii — ta sama filozofia
  precyzji co w balanced driverze. Dotyczy tylko 2 rezystorów, koszt marginalny.
- Strata audio na R_dc: op-amp (Zout≈100Ω) widzi na pinie XLR obciążenie
  10kΩ‖6,81kΩ‖6,81kΩ = 2,52kΩ → poziom audio = 2,52k/(100+2,52k) = **−0,33dB** ✓

**Dlaczego nie cewki:**  
100µH @ 20Hz: Z = 2π×20×100µH = 0,013Ω → zwiera audio do AC-masy C_f1.
Cewka izolująca 20Hz musiałaby mieć >1H — nierealny komponent DIY.

### Zener ochronny — konieczny

Przy I_load→0: V_raw→48V → niszczy MCP1703 (Vin_max=16V). Zener BZX55C9V1 (9V)
klamruje V_raw do 9V — w spec WM-61A (Vs_max=10V) z 1V zapasem.
Przy pełnym obciążeniu 1,13mA:
```
I_zener = 5,72 − 1,21 = 4,51mA,   P_zener = 9V × 4,51mA = 40,6mW  (< 500mW) ✓
Margines prądowy: (5,72 − 1,21) / 5,72 = 79% ✓
```

> **Dlaczego 9V (nie 15V):** WM-61A datasheet: Vs_max = 10V. V_raw = 15V aplikowałoby
> 15V na drain kapsuły przez R_pull_2 — poza gwarantowanym zakresem operacji.
> 9V daje 1V margines do limitu i jednocześnie zwiększa I_available (5,72 vs 4,84mA).

### Napięcie DC na C_hot / C_cold — krytyczne

Prąd I_total=5,72mA dzieli się symetrycznie — 2,86mA na każdą gałąź (R_ph + R_dc):
```
V_pin2 = 48 − 2,86mA × 6,81kΩ = 48 − 19,48 = 28,5V DC (po stronie kabla/interfejsu)
V_op-amp_out ≈ VMID = 2,5V DC (po stronie urządzenia)
ΔV przez C_hot / C_cold = 28,5 − 2,5 = 26V DC
```

> **Kondensatory C_hot i C_cold muszą być znamionowane na ≥63V** (26V DC + 37V zapasu).
> Użycie kondensatorów 25V lub 35V kończy się natychmiastowym uszkodzeniem przy
> pierwszym podłączeniu do interfejsu z phantom power.
>
> *(Poprzedni draft podawał błędnie 44V / 41,5V — tamta kalkulacja uwzględniała tylko
> I_load=1,18mA na gałąź, a pomijała I_zener=4,59mA który też płynie przez R_ph.
> Poprawnie: I_gałąź = I_total/2 = 5,72/2 = 2,86mA.)*

### C_hot / C_cold: 10µF / 63V NP (zmiana względem draftu)

1µF dawało f_HPF = 15,9Hz → −2,2dB @ 20Hz — niezgodne z resztą projektu (rdzeń:
~−0,1dB @ 20Hz). Z 10µF: f_HPF = 1/(2π×10k×10µF) = **1,6Hz** → −0,01dB @ 20Hz ✓

### Schemat finalny — phantom receiver

```
XLR pin2 → R_dc1(6,81kΩ, 0,1%) ─┐
XLR pin3 → R_dc2(6,81kΩ, 0,1%) ─┴─→ V_raw
                                      ├─ Z1: BZX55C9V1 (9V, 500mW) → GND
                                      ├─ C_f1: 100µF / 25V elektrolit → GND
                                      ├─ R_pull_2: 22kΩ → węzeł_CAP (kapsuła WM-61A)
                                      └─ MCP1703-5002E (LDO, Vin_max=16V) → V+ = 5V
                                           └─ C_f2: 100nF ceramiczny → GND
XLR pin1 → GND
```

R_pull_2=22kΩ z V_raw=9V: Vdd_kapsuły = 9−0,25mA×22k = **3,5V ∈ [1V,10V]** ✓  
Pobór R_pull_2 z V_raw: 9/22k = 0,409mA (zaliczany do budżetu poniżej).

### Balanced driver (końcówka wyjściowa)

MCP6004: wzmacniacze 1+2 = U1A/U1B (stopnie ×33/×55), wzmacniacze 3+4 = U2A/U2B (driver).

```
N4 → C_out(22µF NP) → SIG_OUT

SIG_OUT → U2A (voltage follower: in+=SIG_OUT, in−=out)
        → R_ser_hot(100Ω) → C_hot(10µF/63V NP) → XLR pin2 + TRS tip

SIG_OUT → U2B (inwerter ×−1):
    R_U2B_in(10kΩ, 0,1%) → U2B in−
    R_U2B_fb(10kΩ, 0,1%) → U2B in− ← U2B out
    U2B in+ → VMID
        → R_ser_cold(100Ω) → C_cold(10µF/63V NP) → XLR pin3 + TRS ring

XLR pin1 = TRS sleeve = GND
```

CMRR: R_U2B_in = R_U2B_fb = 10kΩ, **0,1%** → CMRR ≥ 60dB.

### Budżet prądowy — weryfikacja worst-case

| Komponent | I_max |
|---|---|
| R_pull_2 (22kΩ, V_raw=9V) | 0,409mA |
| MCP6004 quad (Icc worst-case, 170µA/amp×4 @5V) | 0,680mA |
| MCP1703 (LDO quiescent) | 0,120mA |
| R_VMID (470kΩ×2, V+=5V) | 0,005mA |
| **Razem** | **1,214mA** |
| **I_available** | **5,720mA** |
| **Margines worst-case** | **79%** |

### Komponenty opcji 2

| Ref | Opis | Wartość |
|-----|------|---------|
| U1 | Quad op-amp — gain ×33/×33 + balanced driver | **MCP6004-I/P (DIP-14)** |
| R_dc1, R_dc2 | DC tap phantom (dopasowane do R_ph) | **6,81kΩ, 0,1%, E96** |
| Z1 | Zener ochronny V_raw (Vs_max WM-61A=10V → 9V+1V zapas) | **BZX55C9V1 (9V, 500mW)** |
| C_f1 | Bulk filter V_raw | 100µF / 25V elektrolit |
| C_f2 | Decouple LDO | 100nF ceramiczny |
| VR1 | LDO 5V (Vin max 16V) | **MCP1703-5002E/TO (TO-92)** |
| R_pull_2 | Zasilanie kapsuły z V_raw=9V | **22kΩ, 1%** |
| C_hot | DC block + sprzęganie hot | **10µF / 63V NP** elektrolit |
| C_cold | DC block + sprzęganie cold | **10µF / 63V NP** elektrolit |
| R_ser_hot | Izolacja wyjścia hot | 100Ω, 1% |
| R_ser_cold | Izolacja wyjścia cold | 100Ω, 1% |
| R_U2B_in | Inwerter U2B — wejście | **10kΩ, 0,1%** |
| R_U2B_fb | Inwerter U2B — sprzężenie | **10kΩ, 0,1%** |
| J_XLR | Gniazdo XLR żeńskie, panel | Neutrik NC3FBH |
| J_TRS | Gniazdo TRS 6.35mm, panel | stereo 6.35mm |

### Parametry opcji 2

| Parametr | Wartość |
|---|---|
| Zasilanie | Phantom 48V (IEC 61938) |
| I_available z phantomu | 5,72mA |
| I_total worst-case | 1,21mA (margines **79%**) |
| V_raw (kapsuła + LDO_in) | 9V (zener BZX55C9V1, margines 1V do WM-61A Vs_max=10V) |
| V+ (op-ampy) | 5V (MCP1703) |
| CMRR | ≥60dB (R_dc i R_U2B oba 0,1%) |
| DC przez C_hot/cold | **~26V → min 63V rating** |
| f_HPF wyjście | 1,6Hz (−0,01dB @ 20Hz) |
| Szum op-ampa RTI | ~1µV RMS — dominuje szum kapsuły ~4µV; różnica vs NE5532: 0,2dB |
| Max długość kabla | >50m (XLR ekranowany) |

---

## Opcja 3 — TRS+XLR zbalansowane, zasilanie hybrydowe (phantom + bateria)

Identyczna końcówka i phantom receiver jak opcja 2. Diode-OR łączy V_raw phantomu
z V_bat przed zenerem i LDO.

### Schemat zasilania hybrydowego

```
Phantom: R_dc1/R_dc2 → V_raw_ph → D1(BAT85, Vf=0,3V) ─┐
Bateria: 2×18650+BMS → V_bat                → D2(BAT85) ─┴─→ V_bus
                                                              ├─ Z1: BZX55C9V1(9V)→GND
                                                              ├─ C_bus: 100µF/25V→GND
                                                              ├─ R_pull_2(22kΩ)→kapsuła
                                                              └─ MCP1703→V+=5V
TP5100 + USB-C → ładowanie baterii (niezależnie od phantom)
```

### Weryfikacja V_bus

| Źródło | V_bus | LDO Vin | Margines do Vin_max=16V |
|---|---|---|---|
| Phantom (light load, clamped) | 9−0,3 = **8,7V** | ✓ | 7,3V |
| Bateria pełna (8,4V) | 8,4−0,3 = **8,1V** | ✓ | 7,9V |
| Bateria min (6,0V) | 6,0−0,3 = **5,7V** | dropout@1,2mA≈5mV → Vout=5,7V≈5V ✓ | 695mV |

Priorytet automatyczny: V_bus_phantom (8,7V) > V_bus_bat_max (8,1V) → D1 prowadzi,
D2 zablokowana. Przełączenie phantom→bateria: bufor C_bus (100µF), brak glitchu audio.

> **Uwaga — cienki margines diode-OR:** różnica V_bus wynosi tylko **0,6V**.
> BAT85 prąd wsteczny (I_r) w temperaturze pokojowej: <200nA — pomijalny.
> Przy +85°C I_r rośnie do ~µA (typowy wzrost 100–1000× dla Schottky), nadal
> pomijalny wobec I_total=1,21mA. Priorytet działa prawidłowo w zakresie temperatur
> roboczych (−20…+60°C dla zastosowania stetoskopowego).
> Alternatywa (konserwatywna): D2 = 1N4148 (Vf≈0,6V) → V_bus_bat=7,8V, margines 0,9V.

### Komponenty dodatkowe opcji 3

| Ref | Opis | Wartość |
|-----|------|---------|
| D1 | Dioda priorytetowa phantom | BAT85 Schottky |
| D2 | Dioda priorytetowa bateria | BAT85 Schottky |
| C_bus | Bufor szyny V_bus | 100µF / 25V elektrolit |
| BT1 | Ogniwa Li-ion | 2× 18650 (koszyczek 2S) |
| U_BMS | BMS 2S | np. HX-2S-A10 |
| U_CHG | Ładowarka Li-ion | TP5100 + USB-C |

---

## Obudowa — jeden rozmiar dla wszystkich wariantów

**Hammond 1590BB** (119×94×34mm) — zastępuje 1590B (za ciasna dla Neutrik NC3FBH).

```
  Panel przedni:                     Panel tylny:
  ┌──────────────────────┐           ┌──────────────────────────────┐
  │  [TS 3.5mm IN]       │           │  [USB-C *]    [ON/OFF *]     │
  │  [GAIN TRIM ▽]       │           │  [TS 6.35mm †]               │
  │  [LED]               │           │  [TRS 6.35mm ‡]              │
  └──────────────────────┘           │  [XLR ♀ ‡]                   │
                                      └──────────────────────────────┘
  *opcje 1 i 3  †opcja 1  ‡opcje 2 i 3
```

---

## Plan rozszerzenia README.md

1. Sekcja "Warianty budowy" — tabela porównawcza 3 wariantów
2. Opcja 1 — istniejąca dokumentacja opakowana (bez zmian)
3. Opcja 2 — schemat, weryfikacja, BOM, layout veroboard (MCP6004 DIP-14)
4. Opcja 3 — delta od opcji 2 + tabela weryfikacji zasilania hybrydowego
5. Obudowa: 1590B → 1590BB + diagram panelu
6. Kosztorys: trzy oddzielne szacunki
