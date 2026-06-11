# Elektroniczny stetoskop DIY — przetwornik dźwięków serca

Projekt akcesorium do magisterki **cHiFi-GAN dla dźwięków serca**.  
Cel: nagrywanie próbek treningowych / inferencyjnych klasy normal / murmur / extrastole.

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

## Opcja 1 — TS 6.35mm, zasilanie bateryjne 2×18650

## Specyfikacja docelowa

| Parametr | Wartość |
|---|---|
| Kapsułka | Panasonic WM-61A (elektret, −42 dBV/Pa, szum ~28 dB SPL) |
| Wzmocnienie | **60 dB (×1089)**, 2 stopnie kaskadowe ×33 (30 dB każdy) + trymer wejściowy |
| Pasmo użyteczne | ~12 Hz – 5 kHz (filtr górnoprzepustowy 22 µF zachowuje tony S3/S4) |
| Zasilanie | **2× ogniwo Li-ion 18650 (2S, 8.4V→6.0V)**, BMS 2S, ładowanie TP5100 + USB-C |
| Żywotność baterii | ~300 h pracy ciągłej / ~3.8 h ładowania |
| Wyjście | TS 6.35mm mono (niebalansowane) → interfejs USB audio |
| Wejście kapsułki | **TS 3.5mm** (kabel głowica stetoskopu ↔ preamp box) |
| Obudowa | Metalowa Hammond 1590BB (ekranowanie EMI, mieści baterie) |

## Architektura (modułowa)

```
[Głowica stetoskopu]                [Preamp box]                     [Interfejs USB]
  WM-61A kapsułka         TS        ┌────────────────────────────┐
  wklejona w dzwonku    3.5mm       │ Gniazdo TS 3.5mm in        │
  ──────── kabel 1.5m ─────────────>│ RV1 (gain trim, na wejściu)│
  Tip: sygnał + zasilanie kapsuły   │ → U1A (×33) → U1B (×33)    │── TS 6.35mm ──> wejście line
  Sleeve: GND                       │   = 60 dB całkowite         │
                                    │ 2×18650 (2S) + BMS + TP5100 │
                                    │ Gniazdo USB-C (ładowanie)   │
                                    │ Gniazdo TS 6.35mm out       │
                                    └────────────────────────────┘
```

**Zasada zasilania kapsułki przez kabel:**  
Kapsuła WM-61A ma tylko 2 wyprowadzenia (sygnał+zasilanie / GND), dlatego kabel to **TS** (nie TRS).  
Preamp box zasila kapsułę przez żyłę Tip (rezystor pull-up R_pull = 2.2 kΩ do V+); tą samą żyłą wraca sygnał audio (sprzęgnięty pojemnościowo C_in). Sleeve = GND. Głowica stetoskopu nie zawiera żadnej elektroniki.

## Łańcuch wzmacniający — schemat elektryczny (zweryfikowany)

```
Kapsuła → RV1(10kΩ, dzielnik napięcia, GAIN TRIM na panelu) → C_in(22µF/16V) → R_in(909Ω) → U1A (inwert. ×33, 30 dB)
        → C_inter(22µF/16V) → R_in(909Ω) → U1B (inwert. ×33, 30 dB) → C_out(22µF/16V) → R_out(100Ω) ⊥ R_bleed(100kΩ→GND) → TS 6.35mm
```

Zasilanie jednonapięciowe (single-supply) z wirtualną masą VMID = V+/2 (dzielnik R_VMID + bypass C_VMID).

> **Krytyczne dla poprawnego działania — kolejność RV1 i C_in:**  
> C_in MUSI siedzieć PO suwaku RV1 (między suwakiem a R_in), nie przed potencjometrem. Przy odwrotnej  
> kolejności (C_in przed RV1) powstaje ścieżka DC z węzła wirtualnej masy przez R_in i dolną część  
> potencjometru do GND — wymagałoby to napięcia DC na wyjściu U1A rzędu kilkunastu–stu woltów (przy  
> zasilaniu 6–8,4V!), więc op-amp wpadłby w saturację przy KAŻDYM ustawieniu trymera i układ w ogóle  
> by nie wzmacniał sygnału. Przy poprawnej kolejności C_in blokuje DC, jedyną drogą do węzła wirtualnej  
> masy pozostaje R_fb — op-amp ustala punkt pracy dokładnie na VMID. Zweryfikowano, że ta zmiana nie  
> wymaga modyfikacji BOM (te same 3× 22µF) i nie pogarsza żadnego wcześniej zweryfikowanego parametru  
> (HPF w najgorszym przypadku nadal daje f_c≈7,96Hz/stopień → −1,3dB @ 20Hz; obciążenie kapsuły  
> R_pull‖RV1=1803Ω; wzmocnienie i headroom bez zmian).

### Wyniki weryfikacji elektrycznej

| Kontrola | Wynik |
|---|---|
| Wzmocnienie całkowite | ×1089 = **60.7 dB** ✓ (zgodne z wymaganiem wynikającym z analizy SPL→V) |
| Filtr górnoprzepustowy (HPF) | C = 22 µF → tłumienie na 20 Hz tylko **−1.3 dB** (S3/S4 zachowane; przy 10 µF byłoby −4.9 dB — niedopuszczalne) |
| Ryzyko clippingu | RV1 na **wejściu** (przed wzmocnieniem) — sygnał nigdy nie przekracza marginesu swing wewnątrz U1A/U1B, niezależnie od stanu baterii i poziomu SPL |
| Poziomy wyjściowe | 173 mV @ 60 dB SPL · 307 mV @ 65 dB SPL · 971 mV @ 75 dB SPL — sensowne poziomy liniowe |
| Zapas napięciowy NE5532 | 3.5–6.3× margines względem sygnału w całym zakresie napięcia baterii (8.4V → 6.0V odcięcie BMS) |
| Pasmo / stabilność | 303 kHz/stopień — 60× zapas nad wymaganym 5 kHz |

**Dwie dodatkowe poprawki znalezione przy analizie polaryzacji DC kondensatorów sprzęgających:**
1. **C_in, C_inter, C_out → kondensatory dwubiegunowe (NP/bipolar), nie standardowe elektrolityczne.**  
   Napięcie DC po obu stronach C_in zależy od pozycji trymera RV1 (suwak może być DC powyżej lub  
   poniżej VMID) — znak napięcia na C_in mógłby się odwrócić w zależności od ustawienia. Standardowy  
   (polaryzowany) elektrolit groziłby pracą z odwrotną polaryzacją (zwiększony upływ, degradacja).  
   Rozwiązanie: kondensatory NP eliminują ten problem i jednocześnie usuwają ryzyko pomyłki przy  
   montażu (3 identyczne wartości 22µF w torze — łatwo pomylić orientację).
2. **Dodano R_bleed = 100 kΩ z węzła wyjściowego (za C_out) do GND.**  
   Bez niego DC po "zimnej" stronie C_out nie miałby zdefiniowanej ścieżki do masy — ryzyko trzasku  
   ("pop") przy podłączaniu/odłączaniu kabla do interfejsu. R_bleed definiuje DC=0V, pobiera prąd ≈0  
   (C_out blokuje DC w stanie ustalonym) — nie wpływa na sygnał ani poziom wyjściowy.

## Kosztorys (orientacyjny, do potwierdzenia cen na TME/Botland)

| Komponent | Źródło | Koszt |
|---|---|---|
| Stetoskop (używany) | Allegro | 20–35 zł |
| Kapsułka Panasonic WM-61A | TME.eu | 5–8 zł |
| 1× NE5532N (DIP-8 + podstawka) | TME / Botland | 2–3 zł |
| Obudowa metalowa Hammond 1590BB lub equiv. | TME / Allegro | 25–45 zł |
| Gniazdo TS 3.5mm (panel mount, mono!) | Botland / Allegro | 4–6 zł |
| Gniazdo TS 6.35mm (panel mount) | Botland / Allegro | 4–6 zł |
| Potencjometr 10 kΩ + pokrętło (gain trim, panel) | TME / Botland | 6–10 zł |
| 2× ogniwo 18650 (markowe, np. Samsung/Molicel) | sklep elektroniczny | 30–50 zł |
| Koszyczek 2×18650 + moduł BMS 2S 8.4V | Botland / AliExpress | 15–25 zł |
| Moduł ładowania TP5100 (2S) + gniazdo USB-C | Botland / AliExpress | 12–20 zł |
| Wyłącznik ON/OFF | Botland | 3–5 zł |
| Rezystory + kondensatory (assorted, w tym 3× 22µF/16V dwubiegunowe NP) | TME | 10–15 zł |
| Płytka prototypowa veroboard | Botland / Allegro | 5–8 zł |
| Kabel TS 3.5mm (1.5m, ekranowany, do stetoskopu) | Allegro | 6–10 zł |
| Wtyczka TS 3.5mm (na kabel głowicy) | Allegro | 2–3 zł |
| **Razem** | | **~145–250 zł** |

> Wzrost kosztu względem pierwotnej wersji (9V) wynika z przejścia na zasilanie 2×18650 + ładowanie USB-C  
> (żywotność ~300h zamiast ~60h, ładowalne zamiast wymiany baterii) oraz większej obudowy.  
> Kabel wyjściowy TS 6.35mm → interfejs pominięty (zakładamy posiadany). Wysyłka TME darmowa od ~100 zł.

## Decyzje projektowe

### Dlaczego 2×18650 (Li-ion, ładowane USB-C) zamiast baterii 9V PP3?
Pierwotnie wybrano baterię 9V (eliminacja złożoności phantom power). Po dalszej analizie żywotności  
(~60h dla 9V PP3 vs potencjalnie ~300h dla 2×18650 w konfiguracji 2S) i preferencji użytkownika  
zdecydowano o przejściu na ogniwa Li-ion 18650: 2 ogniwa w serii (2S, 8.4V pełne → 6.0V odcięcie BMS),  
moduł BMS 2S (ochrona nad/pod-napięciowa), ładowanie przez moduł TP5100 z gniazda USB-C.  
NE5532 zweryfikowano jako stabilny i bezpieczny w całym zakresie napięcia rozładowania (margines 3.5–6.3×).

### Dlaczego Opcja 1 ma wyjście niebalansowane TS zamiast XLR / phantom 48V?
Opcja 1 celuje w prostotę budowy i autonomię (~300 h). Przy odległości stetoskop–interfejs ≤ 2–5 m zakłócenia na kablu TS są pomijalne. NE5532 (DIP-8) pobiera ~8 mA — na granicy normy phantom IEC 61938 (~10 mA per urządzenie). Zasilanie bateryjne eliminuje zależność od interfejsu z phantom power.

Opcja 2 i 3 zapewniają wyjście zbalansowane XLR+TRS z CMRR ≥60 dB i pracą na kablach >50 m — opisane w dedykowanych sekcjach.

### Dlaczego WM-61A zamiast Primo EM-172?
WM-61A (Panasonic): szum ~28 dB SPL, dostępna od ręki w TME (~5–8 zł).  
EM-172 (Primo): szum ~14 dB SPL, trudna do znalezienia w Polsce, ~35–50 zł.  
Dla dźwięków serca (SPL ~60–80 dB przy stetoskopie) różnica SNR jest nieistotna.

### Dlaczego obudowa metalowa?
Elektret jest czuły na zakłócenia elektromagnetyczne (EMI od laptopa, zasilacza, WiFi). Metalowa  
obudowa uziemiona do GND układu (w jednym punkcie — star ground) eliminuje ten problem.

### Dlaczego TS 3.5mm zamiast TRS do połączenia głowica ↔ preamp?
WM-61A ma tylko 2 wyprowadzenia (łączone sygnał+zasilanie oraz GND) — trzeci styk TRS byłby  
nadmiarowy. Tip kabla TS niesie jednocześnie zasilanie DC (przez R_pull = 2.2kΩ) i sygnał audio  
(sprzężony pojemnościowo przez C_in), Sleeve = GND. Modularne podejście zachowane: głowica = tylko  
kapsuła + kabel + wtyczka, zero elektroniki — można wymienić kapsułę lub stetoskop bez ingerencji w preamp.

### Dlaczego 60 dB wzmocnienia (a nie 40 dB jak pierwotnie założono)?
Analiza łańcucha SPL→napięcie pokazała, że przy 40 dB (×100) typowy dźwięk serca (65 dB SPL z  
uwzględnieniem wzmocnienia akustycznego dzwonka stetoskopu) dawałby tylko ~28 mV — zbyt cicho dla  
wejścia LINE (~775 mV) i jednocześnie zbyt głośno dla wejścia MIC (~2.5 mV). 60 dB (×1089), uzyskane  
przez kaskadę obu połówek NE5532 jako wzmacniaczy odwracających (30 dB każdy), daje poziomy  
173–971 mV w realistycznym zakresie SPL — pasujące do wejścia LINE.

### Dlaczego trymer wzmocnienia (RV1) na WEJŚCIU, a nie na wyjściu?
Pierwotnie planowano potencjometr na wyjściu układu. Weryfikacja wykazała, że przy głośnych  
dźwiękach (80 dB SPL) i niskim stanie baterii (VMID obniżone do odcięcia BMS) wewnętrzny sygnał  
na wyjściu U1B osiągałby 1721 mV RMS — powyżej maksymalnego marginesu swing 1061 mV — czyli  
**clipping wewnątrz wzmacniacza, zanim potencjometr zdążyłby go stłumić**. Przeniesienie RV1 przed  
C_in i U1A (na wejście) tłumi sygnał *przed* wzmocnieniem — całkowicie eliminuje ryzyko clippingu  
niezależnie od stanu baterii i głośności, kosztem nieistotnego pogorszenia szumu (dominuje  
self-noise kapsuły, nie szum op-ampu).

### Dlaczego filtr górnoprzepustowy z C = 22 µF (a nie 10 µF)?
Przy C_in = C_inter = 10 µF i R_eff = 909 Ω pojedynczy stopień ma f_c = 17.5 Hz, ale dwa kaskadowe  
stopnie HPF dają łączny punkt −3dB ok. 27.1 Hz — co oznacza tłumienie aż −4.9 dB na 20 Hz, czyli  
dokładnie w paśmie tonu **S4 (20–30 Hz)**. Z dokumentacji macierzystego projektu wynika, że  
filtrowanie dolnoprzepustowe w tym paśmie już wcześniej powodowało problemy z datasetem.  
Zwiększenie do 22 µF obniża łączny punkt −3dB do ~12.3 Hz, ograniczając tłumienie na 20 Hz do  
zaledwie −1.3 dB — zachowuje zarówno S4 (20–30 Hz), jak i S3 (25–50 Hz).

## Montaż — głowica stetoskopu (kapsuła WM-61A)

1. Usunąć oryginalną membranę/przewód akustyczny ze stetoskopu, zachowując dzwonek jako rezonator  
   (jego kształt nadaje charakterystyczne wzmocnienie tonów niskich — istotne dla S3/S4).
2. Wywiercić mały otwór (Ø ~3-4mm) w ściance dzwonka na kabel — z dala od osi akustycznej membrany.
3. Wkleić kapsułę WM-61A stroną z otworem dźwiękowym w stronę membrany/pacjenta, blisko centrum  
   dzwonka, używając elastycznego silikonu akustycznego (np. uszczelniacz sanitarny neutralny) —  
   zapobiega przenoszeniu drgań mechanicznych obudowy na kapsułę (mikrofonia).
4. Lutowanie: ekranowany kabel sygnałowy (oplot = GND), możliwie krótkie nieekranowane odcinki  
   przy samej kapsule (podatność na EMI).
5. Uszczelnić otwór kablowy — zapobiega przeciekom akustycznym i kondensacji wilgoci.
6. Test szczelności: lekkie stuknięcie w membranę przy podłączonym wzmacniaczu — brak "świstów"  
   wskazuje na szczelne wklejenie.

## Montaż — preamp box (Hammond 1590BB)

```
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

Zasady krytyczne dla szumów:
- **Baterie/BMS/ładowarka fizycznie odseparowane** od veroboard z NE5532 (przegroda lub odstęp >2cm) —  
  TP5100 i przełączanie generują zakłócenia impulsowe.
- **Gniazdo wejściowe jak najbliżej C_in/U1A** — minimalizacja długości najbardziej podatnej na szum  
  ścieżki (przed wzmocnieniem ×1089 każdy piko-wolt zakłóceń też zostanie wzmocniony).
- **Jeden punkt masy (star ground)** — wszystkie GND zbiegają się przy U1, brak pętli masy.
- **Obudowa uziemiona do GND sygnałowego w jednym punkcie** — ekranowanie EMI bez pętli masy.
- **GAIN TRIM (RV1) na panelu przednim** — łatwy dostęp do regulacji poziomu podczas nagrywania.

## Layout płytki veroboard (paski miedzi)

Konstrukcja: płytka prototypowa **veroboard/stripboard** (równoległe paski miedzi), montaż punkt-punkt
zgodnie z poniższym netlistem. Wybrana zamiast perfboardu (łatwiejsze prowadzenie szyn zasilania
wzdłuż pasków) i zamiast PCB (brak potrzeby zamawiania, jednostkowy prototyp).

### Netlist (połączenia punkt-punkt)

Węzły nazwane zgodnie ze schematem: `CAP` = węzeł kapsuły/R_pull, `N1`=wejście U1A,
`N2`=wyjście U1A / wejście C_inter, `N3`=wejście U1B, `N4`=wyjście U1B / wejście C_out,
`OUT`=węzeł wyjściowy (po C_out, do gniazda), `VMID`=wirtualna masa (V+/2).

| # | Połączenie | Uwagi |
|---|---|---|
| 1 | TS 3.5mm IN (tip) → CAP | sygnał z głowicy kapsuły |
| 2 | TS 3.5mm IN (sleeve) → GND | masa sygnałowa |
| 3 | CAP → R_pull (2,2 kΩ) → V+ | zasilanie kapsuły FET (patrz specyfikacja WM-61A) |
| 4 | CAP → RV1 terminal górny | DC-sprzężenie bezpośrednie — **bez C_in przed potencjometrem** |
| 5 | RV1 terminal dolny → GND | dzielnik napięcia (3-terminalowy, stały ładunek 10 kΩ) |
| 6 | RV1 suwak → C_in (+) | AC-sprzężenie PO trymerze |
| 7 | C_in (−) → R_in (909 Ω) → N1 | wejście stopnia U1A |
| 8 | N1 → R_fb (30,1 kΩ) → N2 | sprzężenie zwrotne stopnia 1 |
| 9 | U1A pin3 (IN+) → VMID | bias nieodwracającego wejścia |
| 10 | U1A pin1 (OUT) = N2 | wyjście stopnia 1 |
| 11 | N2 → C_inter (+) | sprzęgacz międzystopniowy |
| 12 | C_inter (−) → R_in (909 Ω) → N3 | wejście stopnia U1B |
| 13 | N3 → R_fb (30,1 kΩ) → N4 | sprzężenie zwrotne stopnia 2 |
| 14 | U1B pin5 (IN+) → VMID | bias nieodwracającego wejścia |
| 15 | U1B pin7 (OUT) = N4 | wyjście stopnia 2 |
| 16 | N4 → C_out (+) | sprzęgacz wyjściowy |
| 17 | C_out (−) → OUT | węzeł wyjściowy |
| 18 | OUT → R_bleed (100 kΩ) → GND | definiuje DC=0V na wyjściu (anty-pop) |
| 19 | OUT → R_out (100 Ω) → TS 6.35mm (tip) | izolacja wyjścia od pojemności kabla |
| 20 | TS 6.35mm (sleeve) → GND | masa sygnałowa wyjścia |
| 21 | VMID = R_VMID/R_VMID dzielnik z V+ → C_VMID → GND | generacja wirtualnej masy |
| 22 | U1 pin8 (V+) → szyna zasilania, C_decouple (100 nF) → GND blisko pinu | odsprzęganie zasilania |
| 23 | U1 pin4 (V−/GND) → **punkt star-ground** | wszystkie GND zbiegają tutaj |
| 24 | Obudowa → star-ground (jeden punkt) | ekranowanie EMI bez pętli masy |

### Montaż NE5532 (DIP-8) na paskach równoległych

Krytyczny punkt: nóżki 1↔8, 2↔7, 3↔6, 4↔5 leżą na tych samych paskach miedzi (IC „okracza"
4 kolumny A–D), więc bez przecięć każda para zostałaby zwarta:

```
        kolumna:   A    B    C    D    D    C    B    A
                   │    │    │    │    │    │    │    │
   pasek miedzi →──┼────┼────┼────┼─ ╳ ─┼────┼────┼────┼──
                   │    │    │    │    │    │    │    │
        DIP-8:    pin1 pin2 pin3 pin4 pin5 pin6 pin7 pin8
                  N2   N1  VMID GND  VMID  N3   N4   V+
                  OUT  IN- IN+  ⊥*  IN+  IN- OUT
                  (A)  (B)  (C) (D)  (D)  (C)  (B)  (A)

   ╳ = WYMAGANE PRZECIĘCIE paska między rzędem górnym (piny 1-4)
       a dolnym (piny 5-8) — w KAŻDEJ z 4 kolumn A, B, C, D.
   * pin4 = punkt star-ground całego układu
```

> **To najczęstszy błąd montażu DIP na stripboardzie.** Cztery przecięcia (po jednym na
> kolumnę, między rzędami pinów) są obowiązkowe — bez nich każda para pinów 1-8, 2-7,
> 3-6, 4-5 zostanie zwarta przez wspólny pasek. **Zweryfikować miernikiem ciągłości
> (każda para pinów = rozwarcie) przed włożeniem układu w podstawkę.**

### Plan rozmieszczenia (strefy)

```
  ┌─────────────┬───────────────────────┬─────────────┐
  │  STREFA     │   STREFA WZMOCNIENIA  │   STREFA    │
  │  WEJŚCIOWA  │   (IC + pasywne       │  WYJŚCIOWA  │
  │             │    stopni 1 i 2)      │             │
  │ • TS IN     │                       │ • R_out     │
  │ • R_pull    │   ┌───────────────┐   │ • R_bleed   │
  │ • RV1       │   │  U1 (DIP-8)   │   │ • C_out     │
  │ • C_in      │   │  + R_in/R_fb  │   │ • TS OUT    │
  │ • R_in      │   │  + C_inter    │   │             │
  │             │   │  ★ star-GND   │   │             │
  │             │   │   (pin 4)     │   │             │
  ├─────────────┴───┴───────┬───────┴───┴─────────────┤
  │     STREFA ZASILANIA / VMID (wzdłuż dolnej krawędzi)│
  │  R_VMID×2, C_VMID, C_decouple, BMS/bateria → V+    │
  └─────────────────────────────────────────────────────┘

  Sygnał płynie LEWO → PRAWO.  Masa (★) promieniście od pin4 U1 (star ground).
```

### Kolejność montażu (zalecana)

1. **Wykonać 4 przecięcia w obszarze IC** (kolumny A-D, między rzędami pinów) i
   **zweryfikować miernikiem ciągłości** — każda para 1-8/2-7/3-6/4-5 musi być rozwarta.
2. Zamontować podstawkę DIP-8 (bez układu scalonego).
3. Wlutować rezystory (R_pull, RV1 — przewody na panel, R_in×2, R_fb×2, R_out, R_bleed, R_VMID×2).
4. Wlutować kondensatory ceramiczne (C_decouple, C_VMID).
5. Wlutować kondensatory elektrolityczne **NP/bipolarne** (C_in, C_inter, C_out) — zwrócić uwagę,
   że wszystkie trzy są wizualnie identyczne (22 µF/16 V) — łatwo o pomyłkę w kolejności montażu,
   nie w polaryzacji (NP nie ma polaryzacji, ale można pomylić miejsce w sygnale).
6. Połączenia poza płytką: RV1, gniazda TS 3.5/6.35, przełącznik, BMS/bateria, LED.
7. **Przetestować ciągłość całego netlistu miernikiem PRZED włożeniem układu scalonego**
   — szczególnie pary 1-8/2-7/3-6/4-5 (rozwarcie) i każde połączenie z tabeli netlistu (ciągłość).
8. Pierwsze włączenie: **zmierzyć napięcie DC na pinach 1 i 7** (wyjścia U1A/U1B) —
   oczekiwana wartość ≈ V_MID (np. ~4,2 V przy pełnej baterii 2×18650).
   > Jeśli zamiast tego napięcie jest bliskie szynie zasilania (V+ lub GND) — **STOP**.
   > To dokładnie objaw błędu kolejności C_in/RV1 (patrz sekcja "Łańcuch wzmacniający" wyżej,
   > opisany i naprawiony błąd #3) — oznacza błąd montażu, sprawdzić węzły 4-7 z netlistu.

## Bill of Materials (BOM)

### Elektronika preamp boxa

| Poz. | Element | Wartość/typ | Ilość |
|---|---|---|---|
| U1 | Wzmacniacz operacyjny (dual op-amp — oba wzmacniacze U1A+U1B użyte jako 2 stopnie) | NE5532N (DIP-8, THT) + podstawka | 1 |
| RV1 | Potencjometr GAIN TRIM | 10 kΩ, liniowy, mono, panel mount | 1 |
| C_in, C_inter, C_out | Kondensator sprzęgający | 22 µF / 16 V, **elektrolit. DWUBIEGUNOWY (NP/bipolar)**, THT | 3 |
| — | Kondensatory odsprzęgające zasilanie | 100 nF ceram. + 100 µF elektrolit | po 2 |
| R_in | Rezystor wejściowy stopnia (ustala HPF, zgodny z R_eff=909Ω z analizy filtra) | **909 Ω**, E96, 1%, 0.25W | 2 |
| R_fb | Rezystor sprzężenia zwrotnego (gain = R_fb/R_in = 30,1k/909 = 33,11× = 30,4 dB/stopień) | **30,1 kΩ**, E96, 1%, 0.25W | 2 |
| R_pull | Rezystor zasilania kapsuły | 2.2 kΩ, 1% | 1 |
| R_VMID | Dzielnik wirtualnej masy | 2× równe (np. 100 kΩ), 1% | 2 |
| C_VMID | Bypass wirtualnej masy | 10 µF elektrolit. | 1 |
| R_out | Rezystor wyjściowy | 100 Ω, 1% | 1 |
| R_bleed | Rezystor definiujący DC=0V na wyjściu (zapobiega "pop" przy podłączaniu kabla) | 100 kΩ, 1% | 1 |

**Dobór R_in/R_fb — uzasadnienie:**  
R_in = 909 Ω wynika wprost z wcześniejszej analizy filtra HPF (R_eff = 909Ω, z C=22µF daje f_c≈7,96Hz/stopień,  
łącznie ~12,3Hz, −1,3dB @ 20Hz — zachowuje S3/S4). R_fb = 30,1 kΩ to najbliższa wartość E96 dająca  
gain ≈ 33× (dokładnie 33,11× = 30,4dB/stopień; całkowite wzmocnienie = 33,11² = 1096,5× = **60,80 dB**,  
różnica 0,1dB względem celu — nieistotna, korygowalna trymerem RV1).

Dodatkowe kontrole: moc rozpraszana w R_in/R_fb przy max sygnale to odpowiednio ~1,2µW / ~0,04µW —  
margines >200×/>6000× względem ratingu 0,25W. Szum Johnsona obu rezystorów (274nV / 1574nV RMS w  
paśmie 5kHz) jest porównywalny z szumem własnym NE5532 (354nV) — oba nieistotne wobec self-noise  
kapsuły WM-61A (dominujące źródło szumu układu).

> **Ważne — sposób podłączenia RV1:** trymer MUSI być podłączony jako dzielnik napięcia  
> (3 wyprowadzenia: górne = bezpośrednio z węzła kapsuły/R_pull, dolne = GND, suwak = do C_in →  
> R_in → U1A), NIE jako rezystor szeregowy (2 wyprowadzenia) i NIE z C_in przed potencjometrem  
> (patrz ramka wyżej — to uniemożliwiłoby działanie układu). Tylko przy takim podłączeniu obciążenie  
> AC/DC kapsuły jest stałe — R_pull(2,2kΩ) ‖ pełna rezystancja RV1(10kΩ) = **1803 Ω** niezależnie od  
> ustawienia suwaka, w zalecanym zakresie obciążenia WM-61A. Przy podłączeniu szeregowym obciążenie  
> zmieniałoby się wraz z pozycją trymera (od 643Ω do 1831Ω), schodząc w skrajnym ustawieniu poniżej  
> zalecanego minimum.

### Zasilanie

| Element | Typ | Uwagi |
|---|---|---|
| 2× ogniwo Li-ion 18650 | markowe (Samsung INR18650-30Q, Molicel lub równ.) | nie kupować no-name |
| Koszyczek 2×18650 | z przewodami, montaż serii | dopasować pod obudowę |
| BMS 2S | moduł ochrony Li-ion 2S 8.4V, ~10-20A | **wymagany — bez niego ryzyko pożaru/uszkodzenia ogniw** |
| Moduł ładowania TP5100 | wersja 2S | wejście 5V z USB-C |
| Gniazdo USB-C | ew. z PD trigger (np. CH224K) dla szybszego ładowania | opcjonalnie |
| Przełącznik ON/OFF | dźwigniowy, panel mount | |
| LED + rezystor | wskaźnik zasilania | |

### Złącza, mechanika

| Element | Typ | Uwagi |
|---|---|---|
| Gniazdo wejściowe | TS 3.5mm (mono!), panel mount | **uwaga: TS, nie TRS** |
| Gniazdo wyjściowe | TS 6.35mm, panel mount | Neutrik lub equiv. |
| Obudowa | Hammond 1590BB (119×94×34mm) | metalowa; jeden rozmiar dla wszystkich wariantów |
| Kabel głowicy | TS 3.5mm, 1.5m, ekranowany | |
| Wtyczka kabla głowicy | TS 3.5mm | |
| Płytka prototypowa | veroboard, THT | |
| Kapsuła | Panasonic WM-61A | wklejana w dzwonek stetoskopu |

## Status projektu

- [x] Architektura ogólna (głowica TS + preamp box + zasilanie Li-ion)
- [x] Schemat elektryczny łańcucha wzmacniającego (60dB, 1× NE5532 dual op-amp, zweryfikowany)
- [x] Filtr górnoprzepustowy dobrany pod tony S3/S4 (C=22µF)
- [x] Analiza ryzyka clippingu i umiejscowienie trymera (RV1 na wejściu)
- [x] BOM z kategoriami komponentów i orientacyjnym kosztorysem
- [x] Plan montażu głowicy i preamp boxa
- [x] Dokładne wartości R_fb/R_in dobrane (R_in=909Ω, R_fb=30,1kΩ, E96 1% — gain=33,11×/stopień, 60,80dB całkowite)
- [x] Finalny layout veroboard (netlist 24 połączeń, plan przecięć IC, strefy rozmieszczenia, kolejność montażu)
- [ ] Zakup komponentów, weryfikacja symboli TME/Botland przed zamówieniem
- [ ] Budowa i testy SNR po złożeniu — porównanie z danymi datasetu (mediana SNR normal = −2.7 dB)

## Powiązane projekty

- [magisterka-hifigan-demo](https://github.com/PapaModule/magisterka-hifigan-demo) — strona demo cHiFi-GAN
- [chifigan-heart-sounds](https://github.com/PapaModule/chifigan-heart-sounds) — kod treningu
