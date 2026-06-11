# Elektroniczny stetoskop DIY — przetwornik dźwięków serca

Projekt akcesorium do magisterki **cHiFi-GAN dla dźwięków serca**.  
Cel: nagrywanie próbek treningowych / inferencyjnych klasy normal / murmur / extrastole.

## Warianty budowy

Projekt oferuje trzy warianty budowy współdzielące tę samą kapsułkę WM-61A i rdzeń wzmacniający (65 dB, stopień 1 ×33 + stopień 2 ×55). Różnią się wyjściem i źródłem zasilania.

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
| Koszt orientacyjny | ~153–249 zł | ~129–207 zł | ~196–320 zł |

Opcja 1 opisana jest w pełni poniżej. Opcje 2 i 3 dodane są jako oddzielne sekcje na końcu dokumentu.

---

## Opcja 1 — TS 6.35mm, zasilanie bateryjne 2×18650

## Specyfikacja docelowa

| Parametr | Wartość |
|---|---|
| Kapsułka | Panasonic WM-61A (elektret, −42 dBV/Pa, szum ~28 dB SPL) |
| Wzmocnienie | **65 dB (×1817)** stały, stopień 1: ×33/30,4dB + stopień 2: ×55/34,8dB; bez trymera |
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
  ──────── kabel 1.5m ─────────────>│ U1A(×33)→U1B(×55) = 65dB   │── TS 6.35mm ──> wejście line
  Tip: sygnał + zasilanie kapsuły   │   = 65 dB całkowite         │
  Sleeve: GND                       │ gain stały, trim w interfejsie│
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
Kapsuła → C_in(22µF NP) → R_in1(909Ω) → U1A (inwert. ×33, 30,4 dB)
        → C_inter(22µF NP) → R_in2(909Ω) → U1B (inwert. ×55, 34,8 dB)
        → C_out(22µF NP) → R_out(100Ω) ⊥ R_bleed(100kΩ→GND) → TS 6.35mm
```

Zasilanie jednonapięciowe (single-supply) z wirtualną masą VMID = V+/2 (dzielnik R_VMID + bypass C_VMID).

> **Regulacja poziomu nagrania:** wyłącznie przez interfejs audio (gain na wejściu LINE).
> Brak trymera w preampie — gain stały 65 dB (kapsuła przy ciele bezpośrednio, clip nierealny).

### Wyniki weryfikacji elektrycznej

| Kontrola | Wynik |
|---|---|
| Wzmocnienie całkowite | ×1817 = **65,2 dB** ✓ (stopień 1: ×33/30,4dB + stopień 2: ×55/34,8dB; +5dB dla słabych S3/S4 i szmerów gr.I-II w zaszumionym datasecie) |
| Filtr górnoprzepustowy (HPF) | C = 22 µF → tłumienie na 20 Hz tylko **−1.3 dB** (S3/S4 zachowane; przy 10 µF byłoby −4.9 dB — niedopuszczalne) |
| Ryzyko clippingu | Brak trymera — gain stały 65 dB. Clip przy >78 dB SPL przy kapsule — nierealny (kapsuła siedzi w dzwonku stetoskopu przy ciele; SPL ograniczony do ~50–75 dB) |
| Poziomy wyjściowe | 29 mV @ 40 dB SPL · 91 mV @ 50 dB SPL · 289 mV @ 60 dB SPL · 1,62 V @ 75 dB SPL — zakres dla zaszumionych nagrań cHiFi-GAN |
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
| 2× ogniwo 18650 (Samsung/Molicel) | 30–50 zł |
| Koszyczek 2×18650 + BMS 2S | 15–25 zł |
| Moduł TP5100 (2S) + gniazdo USB-C | 12–20 zł |
| Przełącznik ON/OFF | 3–5 zł |
| Rezystory, kondensatory (BOM Opt1) | 10–15 zł |
| Płytka veroboard | 5–8 zł |
| Kabel TS 3.5mm głowicy (1,5m) + wtyczka | 8–13 zł |
| **Razem** | **~153–249 zł** |

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
| Rezystory 0,1% (R_dc×2, R_U2B×2) | 8–12 zł |
| Rezystory 1%, kondensatory (BOM Opt2) | 12–18 zł |
| Kondensatory 10µF/63V NP (C_hot, C_cold) | 4–6 zł |
| Płytka veroboard (większa — DIP-14) | 6–10 zł |
| Kabel TS 3.5mm głowicy (1,5m) + wtyczka | 8–13 zł |
| **Razem** | **~129–207 zł** |

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

### Dlaczego 65 dB wzmocnienia (a nie 60 dB jak poprzednio)?
Pierwotna analiza zakładała SPL ~60–75 dB (typowe S1/S2). Dataset cHiFi-GAN okazał się jednak silnie zaszumiony i wymaga nagrywania słabych tonów S3/S4 oraz szmerów klas I-II (SPL ~40–55 dB).

Przy 60 dB: V_out ≈ 17–55 mV przy 40–50 dB SPL → SNR ≈14 dB na szumie kapsułki WM-61A (~4 µV × 1089 = 4 mV floor). Przy 65 dB: V_out ≈ 54–173 mV → SNR ≈20 dB — użyteczna różnica dla algorytmu ML.

Granica 70 dB wykluczona: przy 75 dB SPL z suwnikiem na górze V_out przekracza margines swing op-ampa. 65 dB to optymalny punkt: maksymalna czułość dla słabych dźwięków, trim redukuje gdy S1/S2 są głośne.

### Dlaczego brak trymera gain (RV1)?
Kapsuła WM-61A siedzi bezpośrednio w dzwonku stetoskopu przyłożonym do ciała pacjenta. SPL przy kapsule ograniczony do typowo 50–75 dB — klip (78 dB SPL) jest nierealny. Trymer był rozważany w rev6 i rev7, ale każda pozycja niosła wady: na wejściu amplifikował szum ścieraka ×1817; między stopniami — ×33 i wprowadzał zmienność HPF (−1,5 do −2,4 dB @ 20 Hz zależnie od suwaka); na wyjściu — ryzyko clipu wewnątrz op-ampa.

Usunięcie RV1 eliminuje wszystkie te problemy. Regulacja poziomu nagrania odbywa się przez gain wejściowy interfejsu audio — co jest standardową praktyką dla mikrofonów o stałym wzmocnieniu.

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
  ścieżki (przed stopniem 2 (×55) każdy piko-wolt zakłóceń też zostanie wzmocniony).
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
| 4 | CAP → C_in (+) | AC-sprzężenie wejście kapsuły |
| 5 | C_in (−) → R_in1 (909Ω) → N1 | wejście U1A |
| 6 | N1 → R_fb1 (30,1kΩ) → N2 | sprzężenie zwrotne U1A |
| 7 | U1A pin3 (IN+) → VMID | bias nieodwracający |
| 8 | U1A pin1 (OUT) = N2 | wyjście stopnia 1 |
| 9 | N2 → C_inter (+) | sprzęgacz międzystopniowy |
| 10 | C_inter (−) → R_in2 (909Ω) → N3 | wejście U1B |
| 11 | N3 → R_fb2 (49,9kΩ) → N4 | sprzężenie zwrotne U1B (gain ×55) |
| 12 | U1B pin5 (IN+) → VMID | bias nieodwracającego wejścia |
| 13 | U1B pin7 (OUT) = N4 | wyjście stopnia 2 |
| 14 | N4 → C_out (+) | sprzęgacz wyjściowy |
| 15 | C_out (−) → OUT | węzeł wyjściowy |
| 16 | OUT → R_bleed (100 kΩ) → GND | definiuje DC=0V na wyjściu (anty-pop) |
| 17 | OUT → R_out (100 Ω) → TS 6.35mm (tip) | izolacja wyjścia od pojemności kabla |
| 18 | TS 6.35mm (sleeve) → GND | masa sygnałowa wyjścia |
| 19 | VMID = R_VMID/R_VMID dzielnik z V+ → C_VMID → GND | generacja wirtualnej masy |
| 20 | U1 pin8 (V+) → szyna zasilania, C_decouple (100 nF) → GND blisko pinu | odsprzęganie zasilania |
| 21 | U1 pin4 (V−/GND) → **punkt star-ground** | wszystkie GND zbiegają tutaj |
| 22 | Obudowa → star-ground (jeden punkt) | ekranowanie EMI bez pętli masy |

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
| C_in, C_inter, C_out | Kondensator sprzęgający | 22 µF / 16 V, **elektrolit. DWUBIEGUNOWY (NP/bipolar)**, THT | 3 |
| — | Kondensatory odsprzęgające zasilanie | 100 nF ceram. + 100 µF elektrolit | po 2 |
| R_in | Rezystor wejściowy stopnia (ustala HPF, zgodny z R_eff=909Ω z analizy filtra) | **909 Ω**, E96, 1%, 0.25W | 2 |
| R_fb1 | Sprzężenie stopnia 1 (U1A, gain=×33) | **30,1 kΩ**, E96, 1%, 0.25W | 1 |
| R_fb2 | Sprzężenie stopnia 2 (U1B, gain=×55) | **49,9 kΩ**, E96, 1%, 0.25W | 1 |
| R_pull | Rezystor zasilania kapsuły | 2.2 kΩ, 1% | 1 |
| R_VMID | Dzielnik wirtualnej masy | 2× równe (np. 100 kΩ), 1% | 2 |
| C_VMID | Bypass wirtualnej masy | 10 µF elektrolit. | 1 |
| R_out | Rezystor wyjściowy | 100 Ω, 1% | 1 |
| R_bleed | Rezystor definiujący DC=0V na wyjściu (zapobiega "pop" przy podłączaniu kabla) | 100 kΩ, 1% | 1 |

**Dobór R_in/R_fb — uzasadnienie:**
R_in = 909 Ω z C_in/C_inter = 22µF → HPF f_c = 7,96 Hz/stopień, łączny −3dB ≈ 11,3 Hz, −1,4 dB @ 20 Hz (stały, niezależny od czegokolwiek). Zachowuje S3/S4.
- **R_fb1 = 30,1 kΩ** → U1A: gain = 33,1× = 30,4 dB
- **R_fb2 = 49,9 kΩ** → U1B: gain = 54,9× = 34,8 dB
- Łącznie: ×1817 = **65,2 dB** (stały, bez trymera)

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

### Wspólne (wszystkie warianty)
- [x] Architektura modułowa (głowica TS 3.5mm + preamp box)
- [x] Rdzeń wzmacniający 65 dB (stopień 1 ×33 + stopień 2 ×55, NE5532 / MCP6004)
- [x] Filtr HPF dobrany pod tony S3/S4 (C=22µF, −1,3 dB @ 20Hz)
- [x] Trymer wzmocnienia RV1 między U1A a U1B (redukcja szumu ścieraka o 34,8 dB)
- [x] Wartości R_in=909Ω, R_fb1=30,1kΩ/R_fb2=49,9kΩ E96 1% (gain ×33/×55, 65,2 dB łącznie)
- [x] C_VMID = 10µF NP w rdzeniu wspólnym (bypass VMID, fc=0,07Hz)
- [x] Obudowa Hammond 1590BB — jeden rozmiar dla wszystkich wariantów
- [x] Spec wariantów (rev 6): `docs/superpowers/specs/2026-06-08-balanced-output-variants-design.md`

### Opcja 1 — TS 6.35mm, bateria 2×18650
- [x] Schemat elektryczny (NE5532N DIP-8, zweryfikowany)
- [x] BOM z kosztorysem (~159–259 zł)
- [x] Layout veroboard (netlist 22 połączenia, 4 przecięcia DIP-8, kolejność montażu)
- [ ] Zakup komponentów (weryfikacja cen TME/Botland)
- [ ] Budowa i test: SNR vs dataset mediana (normal = −2,7 dB)

### Opcja 2 — XLR+TRS, phantom 48V
- [x] Spec phantom receiver (V_th=48V, zener 9V, I_avail=5,72mA, margines 79%)
- [x] Spec balanced driver (U2A follower + U2B inwerter ×-1, CMRR ≥60dB)
- [x] Weryfikacja budżetu prądowego (1,21mA / 5,72mA worst-case)
- [x] BOM (MCP6004-I/P, MCP1703, BZX55C9V1, C_hot/cold 63V, R 0,1%)
- [x] Layout veroboard (netlist 38 połączeń, 7 przecięć DIP-14, kolejność montażu)
- [ ] Zakup komponentów
- [ ] Budowa i test (V_raw≈9V, V+≈5V, VMID≈2,5V przy uruchomieniu; pomiar CMRR)

### Opcja 3 — XLR+TRS, hybryda phantom+bateria
- [x] Spec zasilania hybrydowego (diode-OR BAT85, V_bus 8,7V/8,1V/5,7V)
- [x] Weryfikacja marginesu diode-OR (0,6V; BAT85 I_r <1µA w temp. roboczej)
- [x] BOM delta (D1/D2 BAT85, C_bus, 2×18650, BMS 2S, TP5100)
- [ ] Zakup komponentów
- [ ] Budowa i test (auto-przełączanie phantom→bateria przy odłączeniu XLR)

---

## Opcja 2 — XLR+TRS zbalansowane, zasilanie wyłącznie phantom 48V

### Architektura

```
[Interfejs audio]         [Preamp box — Opcja 2]                    [Interfejs audio]
  Phantom 48V       XLR   ┌──────────────────────────────────────┐
  R_ph=6,81kΩ ──────────>│ R_dc1/R_dc2 (6,81kΩ 0,1%) → V_raw   │
                           │ Z1(BZX55C9V1 9V) + C_f1 + LDO 5V    │
[Głowica]         TS      │ MCP6004 quad DIP-14:                  │
  WM-61A     3.5mm────>   │  U1A×33 → U1B×55 (65 dB rdzeń)       │──XLR pin2/TRS tip──>
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
| R_fb1 | Sprzężenie zwrotne U1A (gain=×33) | 30,1kΩ, E96, 1% | 1 |
| R_fb2 | Sprzężenie zwrotne U1B (gain=×55) | **49,9kΩ**, E96, 1% | 1 |
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

#### Netlist Opcji 2 — 38 połączeń

Węzły: `CAP`=kapsuła, `N1`–`N4`=wewnętrzne, `SIG_OUT`=za C_out, `VMID`=V+/2, `V_raw`=9V po zenerze, `V+`=5V z LDO, `HOT`/`COLD`=XLR+TRS wyjście.

| # | Połączenie | Uwagi |
|---|---|---|
| 1 | TS 3.5mm IN (tip) → CAP | sygnał z głowicy |
| 2 | TS 3.5mm IN (sleeve) → GND | masa sygnałowa |
| 3 | CAP → R_pull_2 (22kΩ) → V_raw | zasilanie kapsuły FET |
| 4 | CAP → C_in (+) | AC-sprzężenie wejście kapsuły |
| 5 | C_in (−) → R_in1 (909Ω) → N1 | wejście U1A |
| 6 | N1 → R_fb1 (30,1kΩ) → N2 | sprzężenie zwrotne U1A |
| 7 | U1A IN+ (pin3) → VMID | bias nieodwracający |
| 8 | U1A OUT (pin1) = N2 | wyjście stopnia 1 |
| 9 | N2 → C_inter (+) | sprzęgacz międzystopniowy |
| 10 | C_inter (−) → R_in2 (909Ω) → N3 | wejście U1B |
| 11 | N3 → R_fb2 (49,9kΩ) → N4 | sprzężenie zwrotne U1B (gain ×55) |
| 12 | U1B IN+ (pin5) → VMID | bias nieodwracający |
| 13 | U1B OUT (pin7) = N4 | wyjście stopnia 2 |
| 14 | N4 → C_out (+) | sprzęgacz wyjściowy |
| 15 | C_out (−) = SIG_OUT | wyjście rdzenia |
| 16 | SIG_OUT → R_bleed (100kΩ) → GND | anty-pop |
| 17 | SIG_OUT → U2A IN+ (pin10) | wejście voltage follower HOT |
| 18 | U2A IN- (pin9) = U2A OUT (pin8) | sprzężenie ujemne follower |
| 19 | U2A OUT (pin8) → R_ser_hot (100Ω) → C_hot (+) | HOT przed DC-blokiem |
| 20 | C_hot (−) → XLR pin2 + TRS tip | wyjście HOT |
| 21 | SIG_OUT → R_U2B_in (10kΩ, 0,1%) → U2B IN- (pin13) | wejście inwertera COLD |
| 22 | U2B OUT (pin14) → R_U2B_fb (10kΩ, 0,1%) → U2B IN- (pin13) | sprzężenie inwertera |
| 23 | U2B IN+ (pin12) → VMID | ref inwertera ×-1 |
| 24 | U2B OUT (pin14) → R_ser_cold (100Ω) → C_cold (+) | COLD przed DC-blokiem |
| 25 | C_cold (−) → XLR pin3 + TRS ring | wyjście COLD |
| 26 | XLR pin1 = TRS sleeve = GND | masa sygnałowa wyjścia |
| 27 | XLR pin2 → R_dc1 (6,81kΩ, 0,1%) → V_raw | phantom DC tap HOT |
| 28 | XLR pin3 → R_dc2 (6,81kΩ, 0,1%) → V_raw | phantom DC tap COLD |
| 29 | V_raw → Z1 (BZX55C9V1, 9V, 500mW) → GND | ochrona Vs_max WM-61A |
| 30 | V_raw → C_f1 (100µF/25V) → GND | bulk filter |
| 31 | V_raw → MCP1703 Vin | wejście LDO |
| 32 | MCP1703 Vout = V+ | 5V dla MCP6004 i VMID |
| 33 | V+ → C_f2 (100nF) → GND | decouple LDO |
| 34 | VMID = dzielnik R_VMID(470kΩ) + R_VMID(470kΩ) z V+ → GND | wirtualna masa |
| 35 | VMID → C_VMID (10µF NP / 25V) → GND | bypass VMID, fc=0,07Hz |
| 36 | U1 pin4 (VDD) → V+; C_decouple (100nF) → GND blisko pinu | odsprzęganie zasilania |
| 37 | U1 pin11 (VSS) → GND — **punkt star-ground** | wszystkie GND zbiegają tu |
| 38 | Obudowa → star-ground (jeden punkt) | ekranowanie EMI |

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

## Powiązane projekty

- [magisterka-hifigan-demo](https://github.com/PapaModule/magisterka-hifigan-demo) — strona demo cHiFi-GAN
- [chifigan-heart-sounds](https://github.com/PapaModule/chifigan-heart-sounds) — kod treningu
