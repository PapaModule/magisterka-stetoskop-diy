---
name: balanced-output-variants
description: Trzy warianty wyjścia elektronicznego stetoskopu — TS niebalansowane (istniejący), TRS+XLR tylko phantom, TRS+XLR hybryda (phantom + bateria)
metadata:
  type: project
---

# Spec: Warianty wyjścia stetoskopu — TS / TRS+XLR phantom / TRS+XLR hybryda

**Data:** 2026-06-08  
**Projekt:** `PapaModule/magisterka-stetoskop-diy`  
**Status:** zatwierdzony do implementacji

---

## Kontekst i cel

Istniejący projekt (README.md, w pełni zweryfikowany elektrycznie) opisuje jeden wariant:
wyjście TS 6.35mm niebalansowane, zasilanie 2×18650 Li-ion. Pasmo 20–800 Hz i niska podłoga
szumów to wymagania magisterki cHiFi-GAN, ale ta sama architektura nadaje się też np. jako
geofon/mikrofon kontaktowy z dłuższymi kablami do interfejsu studyjnego — stąd potrzeba
wariantów z wyjściem zbalansowanym i zasilaniem phantom power.

**Cel:** rozszerzyć dokumentację projektu o dwa dodatkowe warianty wyjścia/zasilania,
tak żeby README opisywało trzy kompletne build-y (BOM, schemat, layout, weryfikacja)
z identyczną obudową i układem panelu.

---

## Wspólny rdzeń (niezmieniony we wszystkich wariantach)

```
WM-61A → R_pull(2,2kΩ→V+) → RV1(10kΩ, dzielnik, trim na panelu)
       → C_in(22µF NP) → R_in(909Ω) → U1A(NE5532, inwert.×33, 30dB)
       → C_inter(22µF NP) → R_in(909Ω) → U1B(NE5532, inwert.×33, 30dB)
       → węzeł N4 (wyjście rdzenia, 60dB całkowite)

Zasilanie rdzenia: single-supply, VMID = V+/2
  (R_VMID×2 + C_VMID; pin3 U1A i pin5 U1B do VMID)
Wejście kapsuły: TS 3.5mm jack (tip=sygnał+zasilanie, sleeve=GND)
```

Wszystkie wcześniej zweryfikowane parametry (f_HPF, obciążenie kapsuły, gain, headroom,
analiza DC-bias, polaryzacja kondensatorów, R_bleed) dotyczą rdzenia i pozostają bez zmian.

---

## Opcja 1 — TS niebalansowane, zasilanie bateryjne (istniejąca)

### Końcówka wyjściowa
```
N4 → C_out(22µF NP) → R_out(100Ω) → TS 6.35mm (tip=sygnał, sleeve=GND)
                     ↘ R_bleed(100kΩ) → GND
```

### Zasilanie
- 2× ogniwo Li-ion 18650 (2S, 6,0–8,4V), BMS 2S, ładowarka TP5100 + USB-C
- V+ = V_bat (6,0–8,4V), VMID = V+/2

### Komponenty charakterystyczne dla opcji 1
| Ref | Opis | Wartość |
|-----|------|---------|
| U1 | Dual op-amp (gain ×33 + ×33) | NE5532N (DIP-8) |
| J_out | Gniazdo wyjściowe | TS 6.35mm mono |
| BT1 | Ogniwa Li-ion | 2× 18650 (koszyczek 2S) |
| U_BMS | BMS 2S | np. HX-2S-A10 |
| U_CHG | Ładowarka Li-ion | TP5100 + USB-C |

### Parametry
| Parametr | Wartość |
|---|---|
| Impedancja wyjściowa | ~100 Ω |
| Max długość kabla | ~5 m (niebalansowany, zależy od środowiska EMI) |
| Zasilanie | autonomiczne, ~300 h |

---

## Opcja 2 — TRS+XLR zbalansowane, zasilanie wyłącznie phantom 48V

### Zasada działania

Phantom power 48V (IEC 61938) dostarczane przez interfejs przez dwa rezystory 6,81 kΩ
(po jednym na pin 2 i pin 3 XLR) jest pobierane przez odbiornik phantom w urządzeniu
i regulowane do lokalnego V+ (~9V) dla op-ampów. Audio wychodzi jako sygnał różnicowy
(hot na pin 2 XLR / tip TRS, cold na pin 3 XLR / ring TRS) przez dwa niezależne
kondensatory sprzęgające izolujące DC phantom od toru audio.

### Odbiornik phantom (zasilanie)
```
XLR pin2 → L1(100µH choke) ─┐
XLR pin3 → L2(100µH choke) ─┴─→ V_raw (~14V przy 10mA)
                                   ├─ C_f1(100µF/63V) → GND         [filtr]
                                   └─ LDO(MCP1700-5.0, Vdrop≤0,2V) → V+ = 5V
                                        └─ C_f2(100nF ceramiczny) → GND [decouple]
XLR pin1 → GND (masa sygnałowa)
```

**Analiza prądowa:**
- Źródło phantom: V_th=48V, R_th=6,81kΩ‖6,81kΩ=3,405kΩ
- Max prąd (zwarcie): 48V / 3,405kΩ ≈ 14,1 mA
- Przy poborze 10mA: V_raw = 48V − 10mA×3,405kΩ = **14V** → LDO → 5V ✓
- Zapas LDO: 14V − 5V = 9V (>Vdropout MCP1700=0,2V) ✓

**Dlaczego V+=5V zamiast wyższego napięcia:**
Ujednolicone V+=5V dla opcji 2 i 3 umożliwia identyczny schemat zasilania w obu wariantach:
w opcji 3 bateria po diodzie daje 5,7–8,1V na wejściu LDO — wystarczy dla MCP1700-5.0
(dropout 0,2V, min Vin=5,2V). LM7809 (dropout 2V) nie działałby z baterii (min Vin=11V).

**Wymaganie: pobór całkowity układu ≤10mA przy V+=5V.**

NE5532N pobiera ~8mA/dual (datasheet TI), dwa układy U1+U2 = 16mA — przekracza limit.

> **Wybór IC dla opcji 2 i 3:** zamiast NE5532 użyć **TL072CP**
> (Icc ≈ 1,4mA/op-amp = 2,8mA/dual, dwa układy = ~6mA — mieści się w 10mA).
> Kompromis: TL072 ma wyższy szum wejściowy (~18 nV/√Hz vs ~5 nV/√Hz NE5532),
> ale dla sygnałów serca (pasmo 800Hz, wymagana SNR ~40dB) to akceptowalne.
> Headroom przy V+=5V, VMID=2,5V: TL072 swing ~±1V, max input rdzenia przed clippingiem
> = 1V/1089 ≈ 0,92mV — wystarczające dla sygnałów z WM-61A (~88µV przy 55dB SPL).
> Alternatywa wysokiej klasy: OPA2134 (Icc=2,5mA/dual, szum 8nV/√Hz) — droższy.
> **Decyzja:** TL072CP jako domyślny dla opcji 2 i 3; opcja na OPA2134 dla wersji
> "low-noise" opisana jako footnote w BOM.

### Balanced driver (końcówka wyjściowa)
```
N4 → C_out(22µF NP) → SIG_OUT
     SIG_OUT → U2A (unity gain, non-inverting)  → R_ser_hot(100Ω) → C_hot(1µF NP/63V) → XLR pin2 + TRS tip
     SIG_OUT → U2B (unity gain, inverting ×−1)  → R_ser_cold(100Ω) → C_cold(1µF NP/63V) → XLR pin3 + TRS ring
     XLR pin1 = TRS sleeve = GND
```

**Weryfikacja filtra na wyjściu zbalansowanym:**
C_hot/C_cold (1µF) z typowym obciążeniem interfejsu (10kΩ):
f_HPF = 1/(2π × 10kΩ × 1µF) ≈ **16 Hz** — poniżej 20Hz ✓

**Dopasowanie rezystorów U2B (CMRR):**
Inwerter U2B: gain = −R_fb/R_in = −1, więc R_fb = R_in.
Użyć 10kΩ, **tolerancja 0,1%** (zamiast 1%) → CMRR ≈ 60dB (vs ~40dB przy 1%).
Dotyczy tylko 2 rezystorów w U2B — różnica kosztu marginalna.

### Komponenty charakterystyczne dla opcji 2
| Ref | Opis | Wartość |
|-----|------|---------|
| U1 | Dual op-amp gain stages | TL072CP (DIP-8) |
| U2 | Dual op-amp balanced driver | TL072CP (DIP-8) |
| L1, L2 | Dławiki odbiornika phantom | 100 µH, THT, prąd ≥50mA |
| VR1 | Regulator napięcia LDO 5V | MCP1700-5002E/TO (TO-92, Vdrop=0,2V) |
| C_f1 | Kondensator filtrujący phantom | 100 µF / 63V elektrolit |
| C_f2 | Decouple LDO | 100 nF ceramiczny |
| C_hot | Kondensator blokujący DC (hot) | 1 µF / 63V NP elektrolit |
| C_cold | Kondensator blokujący DC (cold) | 1 µF / 63V NP elektrolit |
| R_ser_hot | Rezystor szeregowy (hot) | 100 Ω, 1% |
| R_ser_cold | Rezystor szeregowy (cold) | 100 Ω, 1% |
| R_U2B_in | Rezystor wejściowy inwertera | 10 kΩ, **0,1%** |
| R_U2B_fb | Rezystor sprzężenia inwertera | 10 kΩ, **0,1%** |
| J_XLR | Gniazdo XLR żeńskie, panel | Neutrik NC3FBH |
| J_TRS | Gniazdo TRS 6.35mm, panel | stereo 6.35mm |

### Parametry opcji 2
| Parametr | Wartość |
|---|---|
| Zasilanie | Phantom 48V (IEC 61938), pobór ≤10mA |
| V+ lokalny | 5V (LDO MCP1700-5.0) |
| CMRR | ≥60dB (przy 0,1% rezystorach U2B) |
| Impedancja wyjściowa | ~100Ω (hot), ~100Ω (cold) |
| f_HPF wyjście | ~16Hz (C_hot/cold + 10kΩ load) |
| Max długość kabla | >50m (XLR ekranowany) |
| Autonomia | ∞ (zasilanie z interfejsu) |

---

## Opcja 3 — TRS+XLR zbalansowane, zasilanie hybrydowe (phantom + bateria)

### Zasada działania

Identyczna końcówka wyjściowa jak opcja 2. Zasilanie: dwa źródła przez diody Schottky'ego
do wspólnej szyny V+ — **automatyczny priorytet** bez ręcznego przełączania.

### Schemat zasilania hybrydowego
```
Phantom receiver → V_raw(~14V) → D1(BAT85, Vf≈0,3V) ─┐
2×18650 + BMS → V_bat(6,0–8,4V) → D2(BAT85, Vf≈0,3V) ─┴─→ V_bus
                                                               ├─ C_bus(100µF/25V) → GND
                                                               └─ LDO MCP1700-5.0 → V+ = 5V (jak opcja 2)
TP5100 + USB-C → ładowanie baterii (niezależnie od phantom)
```

**Priorytety i weryfikacja napięć:**
- Phantom podłączony: V_raw−Vf ≈ 13,7V → LDO wejście 13,7V → V+=5V ✓
- Phantom odłączony: V_bat−Vf ≈ 5,7–8,1V → LDO wejście min 5,7V, MCP1700 dropout=0,2V → min Vin=5,2V ✓
- D1 przewodzi gdy phantom aktywny (13,7V > max V_bat 8,1V po D2) → D2 zablokowana
- Przełączenie phantom→bateria: płynne dzięki C_bus (100µF bufor)
- Bateria rozładowuje się **tylko** gdy phantom niedostępny

### Komponenty dodatkowe (ponad opcję 2)
| Ref | Opis | Wartość |
|-----|------|---------|
| D1 | Dioda priorytetowa phantom | BAT85 Schottky (lub SS14) |
| D2 | Dioda priorytetowa bateria | BAT85 Schottky (lub SS14) |
| C_bus | Bufor szyny zasilania | 100 µF / 25V elektrolit |
| BT1 | Ogniwa Li-ion | 2× 18650 (koszyczek 2S) |
| U_BMS | BMS 2S | np. HX-2S-A10 |
| U_CHG | Ładowarka Li-ion | TP5100 + USB-C |

### Parametry opcji 3
| Parametr | Wartość |
|---|---|
| Zasilanie phantom | 48V IEC 61938, pobór ≤10mA |
| Zasilanie bateria | 2×18650 (2S), ~300h autonomii |
| Priorytet | automatyczny (phantom > bateria) |
| Przełączenie phantom→bateria | płynne, bufor C_bus eliminuje glitch |
| Wyjście | identyczne z opcją 2 |

---

## Obudowa — jeden rozmiar dla wszystkich wariantów

**Hammond 1590BB** (119×94×34mm, aluminium) — zastępuje 1590B (111×60×31mm, za ciasna dla XLR Neutrik).

### Układ panelu (spójny we wszystkich wariantach)
```
  Panel przedni:                     Panel tylny:
  ┌──────────────────────┐           ┌──────────────────────────────┐
  │  [TS 3.5mm IN]       │           │  [USB-C *]    [ON/OFF *]     │
  │  [GAIN TRIM ▽]       │           │  [TS 6.35mm †]               │
  │  [LED]               │           │  [TRS 6.35mm ‡]              │
  └──────────────────────┘           │  [XLR ♀ ‡]                   │
                                      └──────────────────────────────┘
  * tylko opcje 1 i 3 (mają baterię + USB-C do ładowania)
  † tylko opcja 1 (wyjście TS niebalansowane)
  ‡ tylko opcje 2 i 3 (wyjście zbalansowane TRS+XLR)
```

Gniazdo wejściowe (TS 3.5mm kapsuły), trymer GAIN i LED — **identyczna pozycja** we wszystkich.
Strona wyjściowa: opcja 1 ma TS tam, gdzie opcje 2/3 mają TRS+XLR — inne rozmiary otworów,
ale ta sama strona panelu i orientacja.

---

## Plan rozszerzenia README.md

1. **Nowa sekcja "Warianty budowy"** (zaraz po nagłówku i intro) — tabela porównawcza 3 wariantów
2. Obecna dokumentacja (schemat, BOM, layout) → **"## Opcja 1 — TS, bateria"**
3. Nowe sekcje **"## Opcja 2 — XLR+TRS, phantom"** i **"## Opcja 3 — XLR+TRS, hybryda"**:
   - schemat elektryczny (nowe sekcje)
   - wyniki weryfikacji elektrycznej
   - BOM (delta względem opcji 1 + tabela pełna)
   - layout veroboard (nowy)
   - status (nowe checkboxy)
4. **Obudowa** → aktualizacja z 1590B na 1590BB, nowy diagram panelu
5. **Kosztorys** → trzy osobne szacunki (opcja 1 najtańsza, opcja 3 najdroższa)
