---
name: balanced-output-variants
description: Trzy warianty wyjścia elektronicznego stetoskopu — TS niebalansowane (istniejący), TRS+XLR tylko phantom, TRS+XLR hybryda (phantom + bateria)
metadata:
  type: project
---

# Spec: Warianty wyjścia stetoskopu — TS / TRS+XLR phantom / TRS+XLR hybryda

**Data:** 2026-06-08 (rev 3: 2026-06-11)**  
**Projekt:** `PapaModule/magisterka-stetoskop-diy`  
**Status:** zweryfikowany, zatwierdzony do implementacji

---

## Historia poprawek

| Rev | Zmiana |
|---|---|
| 1 | Pierwotny draft |
| 2 | Korekta 3 blokerów: L→R_dc, MCP1700→MCP1703, TL072→MCP6004 |
| 3 | Korekta błędu Thevenin V_th=24V→48V; usunięcie ślepych zaułków analizy; C_hot/cold 1µF→10µF; dokumentacja 41,5V DC na kondensatorach wyjściowych |

---

## Wspólny rdzeń (niezmieniony we wszystkich wariantach)

```
WM-61A → R_pull(→V+) → RV1(10kΩ, dzielnik, trim na panelu)
       → C_in(22µF NP) → R_in(909Ω) → U1A(inwert.×33, 30dB)
       → C_inter(22µF NP) → R_in(909Ω) → U1B(inwert.×33, 30dB)
       → węzeł N4 (wyjście rdzenia, 60dB całkowite)

Zasilanie: single-supply, VMID = V+/2
Wejście kapsuły: TS 3.5mm (tip=sygnał+zasilanie, sleeve=GND)
```

Opcja 1: NE5532 (DIP-8), R_pull=2,2kΩ→V+.  
Opcje 2 i 3: MCP6004 (DIP-14, quad), R_pull_2=33kΩ→V_raw.

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

**I_available przy zenerze 15V:**
```
I = (V_th − V_zener) / R_th = (48 − 15) / 6,81k = 4,84 mA
```

**Dlaczego R_dc = 6,81kΩ (dopasowanie do R_ph):**
- Symetria R_dc = R_ph → poprawa CMRR odbiornika phantom.
- Strata audio na R_dc: op-amp (Zout≈100Ω) widzi na pinie XLR obciążenie
  10kΩ‖6,81kΩ‖6,81kΩ = 2,52kΩ → poziom audio = 2,52k/(100+2,52k) = **−0,33dB** ✓

**Dlaczego nie cewki:**  
100µH @ 20Hz: Z = 2π×20×100µH = 0,013Ω → zwiera audio do AC-masy C_f1.
Cewka izolująca 20Hz musiałaby mieć >1H — nierealny komponent DIY.

### Zener ochronny — konieczny

Przy I_load→0: V_raw→48V → niszczy MCP1703 (Vin_max=16V). Zener BZX55C15 (15V)
klamruje V_raw. Przy pełnym obciążeniu 1,18mA:
```
I_zener = 4,84 − 1,18 = 3,66mA,   P_zener = 15V × 3,66mA = 54,9mW  (< 500mW) ✓
Margines prądowy: (4,84 − 1,18) / 4,84 = 76% ✓
```

### Napięcie DC na C_hot / C_cold — krytyczne

Przy I_load=1,18mA (0,59mA na gałąź):
```
V_pin2 = 48 − 0,59mA × 6,81kΩ = 43,98V ≈ 44V DC (po stronie kabla/interfejsu)
V_op-amp_out ≈ VMID = 2,5V DC (po stronie urządzenia)
ΔV przez C_hot / C_cold = 44 − 2,5 = 41,5V DC
```
> **Kondensatory C_hot i C_cold muszą być znamionowane na ≥63V.**
> Użycie kondensatorów 25V (przy nieuwadze na etapie zakupu) kończy się natychmiastowym
> uszkodzeniem przy pierwszym podłączeniu do interfejsu z phantom power.

### C_hot / C_cold: 10µF / 63V NP (zmiana względem draftu)

1µF dawało f_HPF = 15,9Hz → −2,2dB @ 20Hz — niezgodne z resztą projektu (rdzeń:
~−0,1dB @ 20Hz). Z 10µF: f_HPF = 1/(2π×10k×10µF) = **1,6Hz** → −0,01dB @ 20Hz ✓

### Schemat finalny — phantom receiver

```
XLR pin2 → R_dc1(6,81kΩ, 1%) ─┐
XLR pin3 → R_dc2(6,81kΩ, 1%) ─┴─→ V_raw
                                     ├─ Z1: BZX55C15 (15V, 500mW) → GND
                                     ├─ C_f1: 100µF / 25V elektrolit → GND
                                     ├─ R_pull_2: 33kΩ → węzeł_CAP (kapsuła WM-61A)
                                     └─ MCP1703-5002E (LDO, Vin_max=16V) → V+ = 5V
                                          └─ C_f2: 100nF ceramiczny → GND
XLR pin1 → GND
```

R_pull_2=33kΩ z V_raw=15V: Vdd_kapsuły = 15−0,25mA×33k = **6,75V ∈ [1V,10V]** ✓  
Pobór R_pull_2 z V_raw: 15/33k = 0,455mA (zaliczany do budżetu poniżej).

### Balanced driver (końcówka wyjściowa)

MCP6004: wzmacniacze 1+2 = U1A/U1B (stopnie ×33), wzmacniacze 3+4 = U2A/U2B (driver).

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
| R_pull_2 (33kΩ, V_raw=15V) | 0,455mA |
| MCP6004 quad (Icc worst-case) | 0,600mA |
| MCP1703 (LDO quiescent) | 0,120mA |
| R_VMID (470kΩ×2, V+=5V) | 0,005mA |
| **Razem** | **1,180mA** |
| **I_available** | **4,840mA** |
| **Margines worst-case** | **76%** |

### Komponenty opcji 2

| Ref | Opis | Wartość |
|-----|------|---------|
| U1 | Quad op-amp — gain ×33/×33 + balanced driver | **MCP6004-I/P (DIP-14)** |
| R_dc1, R_dc2 | DC tap phantom (dopasowane) | **6,81kΩ, 1%, E96** |
| Z1 | Zener ochronny V_raw | **BZX55C15 (15V, 500mW)** |
| C_f1 | Bulk filter V_raw | 100µF / 25V elektrolit |
| C_f2 | Decouple LDO | 100nF ceramiczny |
| VR1 | LDO 5V (Vin max 16V) | **MCP1703-5002E/TO (TO-92)** |
| R_pull_2 | Zasilanie kapsuły z V_raw | **33kΩ, 1%** |
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
| I_available z phantomu | 4,84mA |
| I_total worst-case | 1,18mA (margines **76%**) |
| V_raw (kapsuła + LDO_in) | 15V (zener) |
| V+ (op-ampy) | 5V (MCP1703) |
| CMRR | ≥60dB (R_U2B 0,1%) |
| DC przez C_hot/cold | **~41,5V → min 63V rating** |
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
                                                              ├─ Z1: BZX55C15(15V)→GND
                                                              ├─ C_bus: 100µF/25V→GND
                                                              ├─ R_pull_2(33kΩ)→kapsuła
                                                              └─ MCP1703→V+=5V
TP5100 + USB-C → ładowanie baterii (niezależnie od phantom)
```

### Weryfikacja V_bus

| Źródło | V_bus | LDO Vin | Margines do Vin_max=16V |
|---|---|---|---|
| Phantom (light load, clamped) | 15−0,3 = **14,7V** | ✓ | 1,3V |
| Bateria pełna (8,4V) | 8,4−0,3 = **8,1V** | ✓ | 7,9V |
| Bateria min (6,0V) | 6,0−0,3 = **5,7V** | dropout@1,2mA≈5mV → Vout=5,7V≈5V ✓ | 695mV |

Priorytet automatyczny: V_bus_phantom (14,7V) >> V_bus_bat_max (8,1V) → D1 prowadzi,
D2 zablokowana. Przełączenie phantom→bateria: bufor C_bus (100µF), brak glitchu audio.

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
