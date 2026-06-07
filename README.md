# Elektroniczny stetoskop DIY — przetwornik dźwięków serca

Projekt akcesorium do magisterki **cHiFi-GAN dla dźwięków serca**.  
Cel: nagrywanie próbek treningowych / inferencyjnych klasy normal / murmur / extrastole.

## Specyfikacja docelowa

| Parametr | Wartość |
|---|---|
| Kapsułka | Panasonic WM-61A (elektret, −42 dBV/Pa, szum ~28 dB SPL) |
| Wzmocnienie | ~40 dB (×100) |
| Pasmo użyteczne | ~16 Hz – 5 kHz |
| Zasilanie | Bateria 9V PP3 (żywotność ~60h przy prądzie 8 mA) |
| Wyjście | TS 6.35mm mono (niebalansowane) → interfejs USB audio |
| Wejście kapsułki | TRS 3.5mm (kabel stetoskop ↔ preamp box) |
| Obudowa | Metalowa (ekranowanie EMI) |

## Architektura (modułowa)

```
[Głowica stetoskopu]            [Preamp box]                [Interfejs USB]
  WM-61A kapsułka               ┌──────────────────────┐
  wklejona w dzwonek    TRS     │ Gniazdo TRS 3.5mm in │
  ─────── kabel 1.5m ──────────>│ NE5532 preamp 40 dB  │─── TS 6.35mm ──> wejście line/mic
  T: sygnał                     │ Bateria 9V PP3        │
  R: +V zasilanie kapsułki      │ Gniazdo TS 6.35mm out│
  S: GND                        └──────────────────────┘
```

**Zasada zasilania kapsułki przez kabel:**  
Preamp box zasila WM-61A przez żyłę Ring kabla TRS (rezystor pullup 2.2 kΩ do +9V).  
Sygnał wraca żyłą Tip. Głowica stetoskopu nie zawiera żadnej elektroniki.

## Kosztorys

| Komponent | Źródło | Koszt |
|---|---|---|
| Stetoskop (używany) | Allegro | 20–35 zł |
| Kapsułka Panasonic WM-61A | TME.eu | 5–8 zł |
| NE5532N (DIP-8) | TME / Botland | 2 zł |
| Obudowa metalowa (Hammond 1590A lub equiv.) | TME / Allegro | 15–30 zł |
| Gniazdo TRS 3.5mm (panel mount) | Botland / Allegro | 4–6 zł |
| Gniazdo TS 6.35mm (panel mount) | Botland / Allegro | 4–6 zł |
| Wyłącznik ON/OFF | Botland | 3–5 zł |
| Koszyczek baterii 9V + bateria | Allegro / sklep elektr. | 7–10 zł |
| Rezystory + kondensatory (assorted) | TME | 5–8 zł |
| Płytka prototypowa veroboard | Botland / Allegro | 5–8 zł |
| Kabel TRS 3.5mm (1.5m, do stetoskopu) | Allegro | 6–10 zł |
| Wtyczka TRS 3.5mm (na kabel głowicy) | Allegro | 2–3 zł |
| **Razem** | | **~78–131 zł** |

> Kabel wyjściowy TS 6.35mm → interfejs pominięty (zakładamy posiadany).  
> Wysyłka TME darmowa od ~100 zł.

## Decyzje projektowe

### Dlaczego bateria 9V zamiast phantom 48V?
Phantom 48V wymaga regulatora napięcia i ogranicza wybór wzmacniacza operacyjnego (NE5532 pobiera ~8 mA — na granicy normy IEC 61938 dla phantom). Bateria 9V eliminuje całą tę złożoność. Żywotność baterii alkalicznej PP3 (~500 mAh / 8 mA) = ~62 godziny nagrywania.

### Dlaczego wyjście niebalansowane TS zamiast XLR?
Przy odległości stetoskop–interfejs ≤ 2 m zakłócenia na kablu TS są pomijalne. XLR wymaga elektronicznego balansowania (dodatkowa połówka NE5532 jako inverter fazowy) i droższego złącza — zbędna komplikacja.

### Dlaczego WM-61A zamiast Primo EM-172?
WM-61A (Panasonic): szum ~28 dB SPL, dostępna od ręki w TME (~5–8 zł).  
EM-172 (Primo): szum ~14 dB SPL, trudna do znalezienia w Polsce, ~35–50 zł.  
Dla dźwięków serca (SPL ~60–80 dB przy stetoskopie) różnica SNR jest nieistotna.

### Dlaczego obudowa metalowa?
Elektret jest czuły na zakłócenia elektromagnetyczne (EMI od laptopa, zasilacza, WiFi). Metalowa obudowa uziemiona do GND układu eliminuje ten problem.

### Połączenie głowica ↔ preamp przez TRS 3.5mm
Modularne podejście — głowica stetoskopu to tylko kapsułka + kabel + wtyczka (zero elektroniki w głowicy). Możliwość wymiany kapsułki lub stetoskopu bez ingerencji w preamp box.

## Następne kroki

- [ ] Schemat elektryczny preamp box (NE5532, zasilanie, filtrowanie)
- [ ] Schemat montażu kapsułki w głowicy stetoskopu
- [ ] Lista zakupów z konkretnymi symbolami TME
- [ ] Schemat płytki prototypowej (layout veroboard)
- [ ] Testy SNR po złożeniu — porównanie z danymi datasetu (mediana SNR normal = −2.7 dB)

## Powiązane projekty

- [magisterka-hifigan-demo](https://github.com/PapaModule/magisterka-hifigan-demo) — strona demo cHiFi-GAN
- [chifigan-heart-sounds](https://github.com/PapaModule/chifigan-heart-sounds) — kod treningu
