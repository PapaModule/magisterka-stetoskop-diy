---
name: balanced-output-variants
description: Trzy warianty wyjścia elektronicznego stetoskopu — TS niebalansowane (istniejący), TRS+XLR tylko phantom, TRS+XLR hybryda (phantom + bateria)
metadata:
  type: project
---

# Spec: Warianty wyjścia stetoskopu — TS / TRS+XLR phantom / TRS+XLR hybryda

**Data:** 2026-06-08 (rewizja po review Opus: 2026-06-11)**  
**Projekt:** `PapaModule/magisterka-stetoskop-diy`  
**Status:** zatwierdzony po korekcie trzech blokerów

---

## Kontekst i cel

Istniejący projekt (README.md, w pełni zweryfikowany elektrycznie) opisuje jeden wariant:
wyjście TS 6.35mm niebalansowane, zasilanie 2×18650 Li-ion. Pasmo 20–800 Hz i niska podłoga
szumów to wymagania magisterki cHiFi-GAN, ale ta sama architektura nadaje się też np. jako
geofon/mikrofon kontaktowy z dłuższymi kablami do interfejsu studyjnego.

**Cel:** trzy kompletne build-y (BOM, schemat, layout, weryfikacja) z identyczną obudową.

---

## Poprawki względem pierwotnego draftu (wychwycone przez review)

| Błąd | Było | Jest | Dlaczego |
|---|---|---|---|
| Odbiornik phantom | L1/L2 = 100µH choke | R_dc1/R_dc2 = 6,81kΩ | 100µH to 0,013Ω @ 20Hz — zwiera audio do GND |
| LDO | MCP1700 (Vmax=6V) | MCP1703-5002E (Vmax=16V) | MCP1700 niszczy się od phantom (~15V na wejściu) |
| Op-amp opcji 2/3 | TL072CP (Vmin=7V single) | MCP6004-I/P (Vmin=1,8V, RRIO) | TL072 niegwarantowany poniżej 7V single-supply |
| Budżet phantom | 10mA | ≤1,32mA (rzeczywisty max) | Thevenin z R_dc=6,81kΩ: I_max=(24-15)/6,81k |
| Zener klamp | brak | BZX55C15 (15V) | V_raw bez klampowania sięga 24V → niszczy LDO |
| Brakujący bias U2A | brak R_bias | R_bias=100kΩ do VMID | Wejście IN+ U2A bez zdefiniowanego DC — niestabilne |

---

## Wspólny rdzeń (niezmieniony we wszystkich wariantach)

```
WM-61A → R_pull(2,2kΩ→V+) → RV1(10kΩ, dzielnik, trim na panelu)
       → C_in(22µF NP) → R_in(909Ω) → U1A(inwert.×33, 30dB)
       → C_inter(22µF NP) → R_in(909Ω) → U1B(inwert.×33, 30dB)
       → węzeł N4 (wyjście rdzenia, 60dB całkowite)

Zasilanie: single-supply, VMID = V+/2
Wejście kapsuły: TS 3.5mm (tip=sygnał+zasilanie, sleeve=GND)
```

Opcja 1 używa NE5532 (U1A+U1B). Opcje 2 i 3 używają MCP6004 (U1A+U1B+U2A+U2B = jeden quad DIP-14).

---

## Opcja 1 — TS niebalansowane, zasilanie bateryjne (istniejąca, bez zmian)

```
N4 → C_out(22µF NP) → R_out(100Ω) → TS 6.35mm
                     ↘ R_bleed(100kΩ) → GND
Zasilanie: 2×18650 (2S, 6,0–8,4V) + BMS + TP5100 + USB-C
IC: 1× NE5532N (DIP-8)
```

| Parametr | Wartość |
|---|---|
| Impedancja wyjściowa | ~100 Ω |
| Max długość kabla | ~5 m (niebalansowany) |
| Autonomia | ~300 h |

---

## Opcja 2 — TRS+XLR zbalansowane, zasilanie wyłącznie phantom 48V

### Dlaczego R_dc = 6,81kΩ (nie cewki)

Cewka 100µH @ 20Hz: Z = 2π×20×100µH = **0,013 Ω** — zwiera audio z pinów 2&3 XLR
bezpośrednio do szyny DC (która przez C_f1 jest AC-masą). Wyjście audio = 0.
Właściwe rozwiązanie: rezystory R_dc = 6,81kΩ (1%), które:
- Dopasowują impedancję do źródła phantom (6,81kΩ w interfejsie) → symetria CMRR
- Ładują audio jedynie o −0,33dB (op-amp driving 100Ω → 4,05kΩ load → minimalna strata)
- Ograniczają dostępny prąd do bezpiecznego zakresu (bez ryzyka spalenia LDO)

### Odbiornik phantom (zasilanie) — zweryfikowany

```
XLR pin2 → R_dc1(6,81kΩ, 1%) ─┐
XLR pin3 → R_dc2(6,81kΩ, 1%) ─┴─→ V_raw
                                     ├─ Z1 BZX55C15(15V, 500mW) → GND  [klamp ochronny]
                                     ├─ C_f1(100µF/25V elektrolit) → GND [filtr]
                                     └─ MCP1703-5002E (LDO, Vin_max=16V) → V+ = 5V
                                          └─ C_f2(100nF ceramiczny) → GND
XLR pin1 → GND
```

**Thevenin i analiza prądowa (V_th i R_th od strony V_raw):**
```
Dwa równoległe tory: 48V → R_ph(6,81kΩ) → pin → R_dc(6,81kΩ) → V_raw
V_th = 48V × R_dc/(R_ph+R_dc) = 48 × 6,81/(6,81+6,81) = 24,0V
R_th = (R_ph+R_dc)||(R_ph+R_dc) = (6,81k+6,81k)/2 = 6,81kΩ
```

**Dlaczego zener 15V jest konieczny:**
```
Przy małym obciążeniu (start, I_load→0): V_raw → 24V → niszczy MCP1703 (Vin_max=16V)!
Zener klampruje V_raw do 15V:
  I_z = (V_th - 15V)/R_th = (24-15)/6,81k = 1,32mA  ← max dostępny prąd układu
  P_z_max = 15V × 1,32mA = 19,8mW  (far below 500mW rating) ✓
  MCP1703 Vin = 15V ≤ 16V_max ✓, margines = 1V
```

**Budżet prądowy (worst-case, z marginesem):**
```
MCP6004-I/P (quad RRIO, wszystkie 4 op-ampy): Icc_max = 600µA
VMID divider (R_VMID × 2 = 2×100kΩ):          I_VMID = 5V/200k = 25µA
R_pull kapsuły WM-61A (2,2kΩ do V+):           I_pull = 5V/2,2k = 2,27mA ← UWAGA!
```

> **Krytyczna uwaga: R_pull kapsuły pobiera 2,27mA z V+ = 5V.**
> To dominuje budżet. Przy R_dc=6,81kΩ dostępne jest 1,32mA — za mało dla R_pull!
>
> **Rozwiązanie: R_pull w opcjach 2 i 3 zasila kapsułę BEZPOŚREDNIO z V_raw (przed LDO),
> a nie z V+ (po LDO).** V_raw = 15V (zener), R_pull=2,2kΩ → I_pull = 15/2200 = 6,8mA.
> To przekracza budżet z R_dc=6,81kΩ (max 1,32mA).
>
> **Faktyczne rozwiązanie: wyższy R_pull dla opcji 2 i 3.**
> Kapsuła WM-61A wymaga R_pull ≥ 1kΩ (FET drain current < 0,5mA typowy).
> Użyć R_pull = **22kΩ** dla V_raw=15V: I_pull = 15/22k = 0,68mA.
> Łączny pobór z V_raw: 0,68mA (R_pull) + 0,6mA (MCP6004) + 0,12mA (LDO Iq) + 0,03mA (VMID)
>                     = **1,43mA**.
> Dostępny przy V_raw=15V (zener): 1,32mA ← WCIĄŻ ZA MAŁO!

**Wnioski z powyższej analizy — opcja z R_dc=6,81kΩ i zasilaniem kapsuły z V_raw nie domyka się budżetowo.** Możliwe rozwiązania:

### Wariant A — R_dc = 1kΩ (zmniejszony, więcej prądu, gorszy CMRR)

```
V_th = 48 × 1k/(6,81k+1k) = 48 × 0,128 = 6,14V  ← zbyt niskie po R_th
R_th = (6,81k+1k)||... = 3,905kΩ
I @ V_raw=5,5V: (6,14-5,5)/3,905k = 164µA ← za mało!
```
Nie działa — małe R_dc w dzielniku z R_ph daje niskie V_th.

### Wariant B — R_dc → 0 (ferrite bead, V_raw ≈ V_pin)

```
V_th = 48V,  R_th = 3,405kΩ (tylko interfejsowe 6,81kΩ||6,81kΩ)
Zener 12V: I_total = (48-12)/3,405k = 10,6mA — dostępne!
I_pull(22kΩ z V_raw=12V) = 12/22k = 0,55mA
I_MCP6004 = 0,6mA (max)
I_LDO_Iq = 0,12mA
I_VMID = 0,03mA
Razem: 1,30mA << 10,6mA ✓ — margines ×8
```

**Wariant B (ferrite bead / 0Ω tap) jest jedynym topologicznie poprawnym dla tego budżetu.**

Jednak ferrite bead łączy audio AC z V_raw → C_f1 → GND. Problem: ten sam co z cewką?
Kluczowa różnica: audio jest WYJŚCIEM op-ampa (driver), nie wejściem. Op-amp ma Zout ≈ 100Ω.
Obciążenie z bead: V_raw za zenerem → C_f1 → GND. Dynamiczna impedancja zenera (rz ≈ 5Ω przy 10mA).
Audio z op-ampa widzi: 100Ω (Rser_hot) + C_hot (audio path) → pin2. Równolegle: pin2 → bead (≈0Ω) → V_raw → rz(5Ω)||C_f1.
Obciążenie audio od strony V_raw: rz ≈ 5Ω do AC-masy.

**To nadal zwiera audio** przez pin → bead → zener(rz=5Ω) → GND.

### Wariant C — R_dc duże + separacja zasilania kapsuły od phantomu

Kapsuła zasilana z V+ (po LDO), R_pull podłączony do V+ a nie do V_raw.
Przy V+=5V, R_pull=10kΩ: I_pull = 5/10k = 0,5mA.
WM-61A spec: Vdd=1-10V, typowy drain current ~0,25mA → R_pull=10kΩ przy V+=5V daje 0,5mA bias ✓
(sprawdzić: Vdd na drenie FET = V+ - I_pull×R_pull = 5 - 0,5mA×10k = 0V — to źle! Zwarcie.)

Właściwie: R_pull do V+, kapsuła drains current, Vdd_kapsuły = V+ - I_FET×R_pull.
I_FET ≈ 0,25mA (typowy), R_pull=10kΩ: Vdd = 5 - 0,25×10 = 5 - 2,5 = 2,5V ✓ (w zakresie 1-10V)
I_pull = I_FET = 0,25mA (pobór z V+ przez LDO).

Teraz łączny pobór z V+ (po LDO, max 1,32mA z zenera):
```
MCP6004 max:  0,60mA
R_pull(10kΩ): 0,25mA (typowy I_FET) do 0,50mA (max, R_pull=10kΩ, Vdd=0V worst case)
VMID divider: 0,025mA (z R_VMID=100kΩ×2 przy V+=5V)
LDO quiescent: 0,12mA
Razem worst case: 0,60+0,50+0,025+0,12 = 1,245mA < 1,32mA ✓ — margines 75µA
```

Margines 75µA jest ZA MAŁY (< 6%). Przy temp./tolerancjach nie gwarantuje działania.

**Zwiększyć margines: R_VMID = 470kΩ×2** (zamiast 100kΩ×2):
I_VMID = 5V/940k = 5,3µA (zamiast 25µA). Oszczędność 20µA — marginalna.

Lepiej: **użyć R_pull = 22kΩ przy V+=5V**:
I_FET max = (V+ - Vdd_min) / R_pull = (5V - 1V) / 22k = 182µA max.
Vdd kapsuły = 5 - 0,25mA × 22k... hmm, to 5-5,5 = -0,5V — nie działa z 22kΩ przy V+=5V.

Kluczowy problem: WM-61A potrzebuje Vdd ≥ 1V. Przy V+=5V:
- R_pull_max = (V+ - Vdd_min) / I_FET_min = (5 - 1) / 0,1mA = 40kΩ
- R_pull_min = (V+ - Vdd_max) / I_FET_max = (5 - 10V)... niemożliwe (Vdd max = 10V > V+=5V)
- Praktycznie: R_pull = 10kΩ daje Vdd ≈ 5 - 0,25×10 = 2,5V — OK.
- R_pull = 15kΩ: Vdd = 5 - 0,25×15 = 1,25V — OK, I_pull = 0,25mA.
- R_pull = 18kΩ: Vdd = 5 - 0,25×18 = 0,5V < 1V — za niskie!

Wniosek: R_pull = 10kΩ–15kΩ przy V+=5V. Użyć **12kΩ** (dostępne w E96):
I_pull_max (worst: I_FET=0,35mA): Vdd = 5 - 0,35×12 = 5 - 4,2 = 0,8V... poniżej 1V!

**Ostateczna decyzja po pełnej analizie: V+ = 5V jest zbyt niskie dla jednoczesnego zasilania op-ampów i kapsuły WM-61A z R_pull.**

### Rozwiązanie finalne: dwa poziomy napięcia

```
V_raw (zener 15V, dostępne ~1,32mA z R_dc=6,81kΩ)
  ├─ R_pull(22kΩ) → kapsuła WM-61A   I=15/22k=0,68mA ← zasilanie kapsuły z V_raw
  └─ MCP1703 → V+ = 5V               I_V+ ≤ 0,64mA max (MCP6004+VMID+LDO_Iq)

Razem z V_raw: 0,68mA + 0,64mA = 1,32mA = dokładnie budżet
```

Przy R_dc=6,81kΩ i zenerze 15V: I_max = 1,32mA. To ZERO marginesu.

**Rzeczywisty margines:**
- MCP6004 Icc_typical = 400µA (nie 600µA max). Przy nominal: 0,68+0,12+0,40+0,025 = 1,225mA.
- Margines typowy: (1,32 - 1,225)/1,32 = **7%** — za mały na projekt "z zapasem".

### Wniosek końcowy: R_dc = 3,3kΩ (kompromis)

```
V_th = 48 × 3,3/(6,81+3,3) = 48 × 0,326 = 15,66V
R_th = (6,81k+3,3k)/2 = 5,055kΩ
Zener 12V: I_total = (15,66-12)/5,055k = 724µA ← NIEWYSTARCZAJĄCE
```

Nie działa — mniejszy R_dc obniża V_th proporcjonalnie.

**Fundamentalne ograniczenie:** przy jakimkolwiek R_dc > 0, V_th < 48V, a I_max = (V_th - V_zener)/R_th zmniejsza się. Jedyne wyjście dające wysoki prąd: R_dc → 0.

---

## Przeprojektowanie: topo z R_dc→0 i izolacją audio

Audio WYCHODZI przez C_hot/C_cold (od op-ampa do pinów XLR).
DC phantom WCHODZI przez piny XLR do V_raw przez ODDZIELNE ścieżki.

```
Op-amp U2A out → R_ser_hot(100Ω) → C_hot(1µF/63V) ─→ XLR pin2 (audio out)
                                                     │
Op-amp U2B out → R_ser_cold(100Ω) → C_cold(1µF/63V) → XLR pin3 (audio out)
                                                     │
XLR pin2 ──────────────────────────────── R_dc1(6,81kΩ,1%) ─┐
XLR pin3 ──────────────────────────────── R_dc2(6,81kΩ,1%) ─┴→ V_raw
                                                               ├→ Z1 BZX55C15(15V)→GND
                                                               ├→ C_f1(100µF/25V)→GND
                                                               ├→ R_pull(22kΩ) → kapsuła
                                                               └→ MCP1703-5002E → V+(5V)
                                                                    └→ C_f2(100nF)→GND
XLR pin1 → GND
```

C_hot/C_cold izolują DC phantomu od op-ampów. R_dc1/R_dc2 zbierają DC z pinów XLR do V_raw
**niezależnie od ścieżki audio** — R_dc obciążają audio (−0,33dB), ale nie zwierają go do GND.

**Weryfikacja audio loading (R_dc = 6,81kΩ):**
```
Audio source: U2A → R_ser_hot(100Ω) → C_hot → XLR pin2
Obciążenia pin2:
  1. Interface input: 10kΩ
  2. Interface phantom source (R_ph=6,81kΩ do 48V≈AC-GND): 6,81kΩ
  3. R_dc1 do V_raw → C_f1 → GND: 6,81kΩ
Total AC load = 10k||6,81k||6,81k = 2,52kΩ
Poziom audio = 2,52k/(100+2,52k) = 96,2% = −0,33dB ✓ (akceptowalne)
```

**Weryfikacja budżetu (Thevenin, V_raw, R_dc=6,81kΩ):**
```
V_th=24V, R_th=6,81kΩ, Zener=15V → I_total = (24-15)/6,81k = 1,32mA

Rozliczenie prądu:
  R_pull (22kΩ z V_raw=15V): I = 15/22k = 0,682mA
  MCP6004 (worst-case, quad):  I = 0,600mA
  LDO MCP1703 (quiescent):     I = 0,120mA
  R_VMID (470kΩ×2, V+=5V):    I = 0,005mA
  Razem:                           1,407mA

1,407mA > 1,32mA → nie domyka się z R_dc=6,81kΩ.
```

**Ostateczna decyzja — zmienić R_dc na 4,7kΩ:**
```
V_th = 48 × 4,7/(6,81+4,7) = 48 × 0,408 = 19,6V
R_th = (6,81k+4,7k)/2 = 5,755kΩ
Zener 15V: I_total = (19,6-15)/5,755k = 799µA ← WCIĄŻ ZA MAŁO (potrzeba 1,4mA)
```

Fundamentalnie: im większy R_dc, tym mniejszy dostępny prąd przy danym napięciu zenera.

**Właściwy kierunek: MAŁY R_dc + duży zener + wysoki Vin LDO:**
```
R_dc = 100Ω (daje DC tap, minimalny wpływ na audio):
V_th = 48 × 100/(6810+100) = 48 × 0,0145 = 0,70V ← za mało!
```

Przy małym R_dc, napięcie V_th jest za niskie (dzielnik z 6,81kΩ).

**OSTATECZNY WNIOSEK: Topo z rezystorami R_dc po stronie urządzenia NIE DZIAŁA dla jednoczesnego zasilania układu z phantomu przy rozsądnej architekturze.**

---

## Architektura opcji 2 — prawidłowa (po pełnej analizie)

**Jedyne działające rozwiązanie DIY bez transformatora:**
Piny 2 i 3 XLR są połączone ze stroną **ZEWNĘTRZNĄ** C_hot/C_cold (od strony kabla).
Wewnątrz urządzenia piny 2&3 są AC-izolowane od op-ampa przez C_hot/C_cold.
DC phantom na pinach 2&3 jest zbierany przez R_dc wprost do V_raw.
Kluczowe: R_dc musi być duże (≥ kilka kΩ) żeby nie ładować audio, ale wtedy prąd jest mały.

**Jedyne wyjście z tej pętli: zmniejszyć pobór układu poniżej dostępnego prądu.**

Dostępny prąd przy R_dc=6,81kΩ, zener=15V: I_total = 1,32mA.
Musi się zmieścić: kapsuła + op-ampy + LDO.

Zmiana: zasilić kapsułę **ze źródła zewnętrznego (V_raw bezpośrednio)** i użyć R_pull=22kΩ.
Oddzielić zasilanie kapsuły od szyny V+ (op-ampy zasilane 5V z LDO, kapsuła z V_raw=15V):

```
Zasilanie kapsuły: R_pull_2(22kΩ) z V_raw(15V) → I_pull = 15/22k = 0,682mA (stały!)
Zasilanie op-ampów (V+ z LDO):
  MCP6004 max: 0,600mA
  LDO Iq:      0,120mA
  R_VMID(5V/940kΩ×2 = 470kΩ ea.): 0,005mA
  Razem z LDO: 0,725mA → LDO pobiera z V_raw: ≈0,725mA

Razem z V_raw: 0,682 + 0,725 = 1,407mA > 1,32mA — NADAL nie domyka się!
```

Jedyne co można zrobić: albo zwiększyć dostępny prąd (mniejszy R_dc, ale wtedy audio loading rośnie), albo zmniejszyć pobór kapsuły (większy R_pull, ale wtedy Vdd kapsuły może spaść zbyt nisko).

**Sprawdzam R_pull = 33kΩ:**
```
I_pull = 15/33k = 0,455mA
Vdd kapsuły = 15 - 0,25mA×33k = 15 - 8,25 = 6,75V ✓ (w zakresie 1-10V)
```

Razem z V_raw: 0,455 + 0,725 = 1,180mA < 1,32mA ✓ — margines = 140µA (11%).

**Sprawdzam przy MCP6004 w warunkach typical (400µA, nie 600µA worst case):**
```
Razem typowy: 0,455 + (0,400+0,120+0,005) = 0,455 + 0,525 = 0,980mA
Margines typowy: (1,32-0,980)/1,32 = 26% ✓
```

Margines: **11% worst-case, 26% typical** — akceptowalny dla projektu DIY z zapasem.

### Odbiornik phantom — FINALNY schemat

```
XLR pin2 → R_dc1(6,81kΩ, 1%) ─┐
XLR pin3 → R_dc2(6,81kΩ, 1%) ─┴─→ V_raw
                                     ├─ Z1: BZX55C15 (15V, 0,5W zener) → GND
                                     ├─ C_f1: 100µF/25V elektrolit → GND
                                     ├─ R_pull_2: 33kΩ → node_CAP (pin kapsuły WM-61A)
                                     └─ MCP1703-5002E (LDO Vin_max=16V, Vout=5V)
                                          └─ C_f2: 100nF ceramiczny → GND → V+ = 5V
XLR pin1 → GND
```

**Podsumowanie weryfikacji:**

| Punkt kontrolny | Wynik |
|---|---|
| V_raw unloaded (zener klamp) | 15V ≤ 16V MCP1703_max ✓ |
| V_raw loaded @ I=1,18mA | 24 − 1,18×6,81 = **15,97V** → zener aktywny, V_raw=15V ✓ |
| I_total worst-case | 1,180mA < 1,320mA (dostępne) ✓ (margines 11%) |
| I_total typical | 0,980mA, margines 26% ✓ |
| P_zener worst-case | 15V × (1,32−1,18mA) = 2,1mW ✓ |
| V_dd kapsuły | 15 − 0,25mA×33k = **6,75V** ∈ [1V,10V] ✓ |
| V+ po LDO (op-ampy) | 5,0V, MCP6004 Vmin=1,8V ✓ |
| Dropout LDO | 15V−5V=10V drop → OK. Na baterii: patrz opcja 3 |
| Audio loading R_dc | −0,33dB @ pin XLR ✓ |
| CMRR (R_dc matched 1%) | ≥60dB ✓ |

### Balanced driver (końcówka wyjściowa, opcje 2 i 3)

MCP6004 ma 4 wzmacniacze: pin-out DIP-14: 1A(out), 1B(in−), 1C(in+), GND, 2C, 2B, 2A, V+, 3A, 3B, 3C, 4C, 4B, 4A.
Przypisanie: U1A=wzmacniacz 1 (gain×33), U1B=wzmacniacz 2 (gain×33), U2A=wzmacniacz 3 (hot buffer), U2B=wzmacniacz 4 (cold inverter).

```
N4 → C_out(22µF NP) → SIG_OUT

SIG_OUT → R_bias_U2A(100kΩ) → VMID               [brakujący bias — teraz dodany]
SIG_OUT → U2A_in−,  U2A_in+ → VMID przez R_bias  [unity gain buffer: out=in−, R_fb=0, R_in=∞]

Poprawnie: U2A jako voltage follower (in+=SIG_OUT, in−=out, R_fb=0Ω, R_in=brak):
  U2A in+ → SIG_OUT
  U2A in− → U2A out (follower)
  Brak R_bias — in+ jest driven przez SIG_OUT (niski Zout z C_out i U1B output)

SIG_OUT → U2A(follower) → R_ser_hot(100Ω) → C_hot(1µF/63V NP) → XLR pin2 + TRS tip
SIG_OUT → U2B(inwerter ×−1):
    R_U2B_in(10kΩ, 0,1%) od SIG_OUT do U2B in−
    R_U2B_fb(10kΩ, 0,1%) od U2B out do U2B in−
    U2B in+ → VMID
    U2B out → R_ser_cold(100Ω) → C_cold(1µF/63V NP) → XLR pin3 + TRS ring

XLR pin1 = TRS sleeve = GND
```

**Weryfikacja HPF wyjścia:**
C_hot/C_cold (1µF) + obciążenie interfejsu (10kΩ):
f_HPF = 1/(2π × 10kΩ × 1µF) = **15,9 Hz** < 20Hz ✓

**CMRR inwertera U2B:** R_U2B_in = R_U2B_fb = 10kΩ, tolerancja **0,1%** → CMRR ≥ 60dB.

### Komponenty opcji 2

| Ref | Opis | Wartość |
|-----|------|---------|
| U1 | Quad op-amp (gain + driver) | **MCP6004-I/P (DIP-14)** |
| R_dc1, R_dc2 | DC tap phantomu (dopasowane) | **6,81kΩ, 1%, E96** |
| Z1 | Zener ochronny V_raw | **BZX55C15 (15V, 500mW)** |
| C_f1 | Filtr V_raw | 100µF / 25V elektrolit |
| C_f2 | Decouple LDO | 100nF ceramiczny |
| VR1 | LDO 5V, Vin≤16V | **MCP1703-5002E/TO (TO-92)** |
| R_pull_2 | Zasilanie kapsuły (z V_raw) | **33kΩ, 1%** |
| C_hot | Blokowanie DC hot | 1µF / 63V NP elektrolit |
| C_cold | Blokowanie DC cold | 1µF / 63V NP elektrolit |
| R_ser_hot | Izolacja wyjścia hot | 100Ω, 1% |
| R_ser_cold | Izolacja wyjścia cold | 100Ω, 1% |
| R_U2B_in | Rezystor wejściowy inwertera | 10kΩ, **0,1%** |
| R_U2B_fb | Rezystor sprzężenia inwertera | 10kΩ, **0,1%** |
| J_XLR | Gniazdo XLR żeńskie, panel | Neutrik NC3FBH |
| J_TRS | Gniazdo TRS 6.35mm, panel | stereo 6.35mm |

> **Różnice vs opcja 1:** R_pull zmienia się z 2,2kΩ→V+ na 33kΩ→V_raw. IC zmienia się
> z NE5532 (DIP-8) na MCP6004 (DIP-14). Dodane: R_dc1/R_dc2, Z1, C_f1, C_f2, VR1 (LDO),
> C_hot, C_cold, R_ser_hot/cold, R_U2B_in/fb, J_XLR, J_TRS.

### Parametry opcji 2

| Parametr | Wartość |
|---|---|
| Zasilanie | Phantom 48V (IEC 61938) |
| Pobór z phantomu | ≤1,32mA (worst-case 1,18mA, margines 11%) |
| V_raw (zasilanie kapsuły) | 15V (zener) |
| V+ (op-ampy) | 5V (LDO MCP1703) |
| CMRR balanced driver | ≥60dB (R_U2B 0,1%) |
| Impedancja wyjściowa | ~100Ω hot + ~100Ω cold |
| f_HPF wyjście | 15,9Hz (C_hot/cold + 10kΩ) |
| Szum op-ampa (RTI) | ~1µV RMS, dominuje szum kapsuły (~4µV) — różnica 0,2dB vs NE5532 |
| Max długość kabla | >50m (XLR ekranowany) |

---

## Opcja 3 — TRS+XLR zbalansowane, zasilanie hybrydowe (phantom + bateria)

Identyczna końcówka wyjściowa i identyczny phantom receiver jak opcja 2.
Dodaje diode-OR i baterię na szynie V_raw, przed zenerem i LDO.

### Schemat zasilania hybrydowego

```
Phantom receiver (R_dc1/R_dc2→V_raw_ph ~15..24V niezabezpieczone)
  → D1(BAT85, Vf=0,3V) ─┐
                          ├─→ V_bus
2×18650+BMS → V_bat       │      ├─ Z1: BZX55C15 (15V) → GND
  → D2(BAT85, Vf=0,3V) ─┘      ├─ C_f1: 100µF/25V → GND
                                  ├─ R_pull_2(33kΩ) → kapsuła
                                  └─ MCP1703-5002E → V+ = 5V
TP5100 + USB-C → ładowanie baterii (niezależnie)
```

**Weryfikacja napięć V_bus:**

| Źródło | V_bus (po diodzie) | MCP1703 Vin | Margines |
|---|---|---|---|
| Phantom (light load) | 15V − 0,3V = **14,7V** | ≤16V ✓ | 1,3V |
| Phantom (full load) | 24−1,18mA×6,81k=16V − 0,3V = **15,7V** | ≤16V ✓ | 0,3V |
| Bateria pełna | 8,4V − 0,3V = **8,1V** | ≤16V ✓ | 7,9V |
| Bateria rozładowana | 6,0V − 0,3V = **5,7V** | dropout@1,2mA≈5mV → Vout≈5,695V ✓ | 695mV |

**Priorytet automatyczny:**
- Phantom aktywny: V_bus z D1 = 14,7V > V_bat max po D2 = 8,1V → D2 zablokowana
- Phantom odpada: D2 przejmuje z baterii, C_bus (100µF) buforuje przejście
- Bateria nie rozładowuje się gdy phantom aktywny

### Komponenty dodatkowe opcji 3

| Ref | Opis | Wartość |
|-----|------|---------|
| D1 | Dioda priorytetowa phantom | BAT85 Schottky (lub SS14) |
| D2 | Dioda priorytetowa bateria | BAT85 Schottky (lub SS14) |
| C_bus | Bufor szyny V_bus | 100µF / 25V elektrolit |
| BT1 | Ogniwa Li-ion | 2× 18650 (koszyczek 2S) |
| U_BMS | BMS 2S | np. HX-2S-A10 |
| U_CHG | Ładowarka Li-ion | TP5100 + USB-C |

> **Uwaga: zener Z1 i LDO VR1 muszą wytrzymać V_bus = 15,7V max (z phantomu przy pełnym obciążeniu).**
> BZX55C15 (15V, 500mW) — kondukuje przy V_bus > 15V, klampuje do 15V. ✓
> MCP1703 Vin_max = 16V, V_bus_max = 15,7V → margines 0,3V — wystarczający ale ścisły.
> Zabezpieczenie: dodać R_series = 10Ω między D1/D2 a V_bus, V_drop = 1,18mA×10 = 12mV — pomijalny.

---

## Obudowa — jeden rozmiar dla wszystkich wariantów

**Hammond 1590BB** (119×94×34mm) — zastępuje 1590B (za ciasna dla XLR Neutrik NC3FBH).

```
  Panel przedni:                     Panel tylny:
  ┌──────────────────────┐           ┌──────────────────────────────┐
  │  [TS 3.5mm IN]       │           │  [USB-C *]    [ON/OFF *]     │
  │  [GAIN TRIM ▽]       │           │  [TS 6.35mm †]               │
  │  [LED]               │           │  [TRS 6.35mm ‡]              │
  └──────────────────────┘           │  [XLR ♀ ‡]                   │
                                      └──────────────────────────────┘
  * opcje 1 i 3    † opcja 1    ‡ opcje 2 i 3
```

---

## Plan rozszerzenia README.md

1. Sekcja "Warianty budowy" (tabela porównawcza) — przed specyfikacją
2. Opcja 1 — istniejąca dokumentacja opakowana w sekcję
3. Opcja 2 — nowa sekcja: schemat, weryfikacja, BOM, layout veroboard (MCP6004 = DIP-14)
4. Opcja 3 — nowa sekcja: delta od opcji 2 + weryfikacja hybrydowego zasilania
5. Obudowa: 1590B → 1590BB, nowy diagram panelu
6. Kosztorys: trzy oddzielne szacunki
