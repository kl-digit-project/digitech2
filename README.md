# Intelligens Redőnyvezérlő – Projekt Összefoglaló (Tinkercad demonstráció) 

### Projekt leírása:
A projekt lényege, hogy egy intelligens fényérzékelő rendszer – luxmérővel és szögérzékeléssel – automatikusan állítja a redőnyt az ideális szobai fényviszonyok érdekében, időzítés és manuális igények figyelembevételével.

### Funkciók:
-	**Időzítés**: pl. reggel 7:00 fel, napnyugta után le. 
-	**Szobában mért fényerő**: százalékosan állítható, hogy hogyan reagáljon a fényviszonyokra. Pl: 30%-os fényerőre van állítva: ha kevesebb mint 30% fentebb húzza a redőnyt, ha több akkor lentebb. 
-	**A napsütés beesési szöge**: Érzékelők segítségével a nap besütési szögéhez állítva tudjuk a redőny magasságát szabályozni, így sosem süt például a monitorunkba a napfény.
-	**Manuális vezérlés**: Telefonon keresztüli vagy egyéb távirányítóval való vezérlés is lehetséges.

**A funkciók mindegyike kombinálható!**

### Szükséges alkatrészek:


Az alkatrészek piacának jelenlegi szereplői közül választottam a számomra leghatékonyabbat ár és tudás szerint.


* **Mikrovezérlő**: ESP32 DevKit V1
  + A többi számomra hasznossal összehasonlítva, mint például az Arduino UNO R4 WiFi, Arduino Nano, ESP8266 NodeMCU, Arduino Mega 2560, olcsóbb, gyorsabb, van Wi‑Fi, sok az analóg bemenete. 2
* **Fényérzékelés**: BH1750 szenzor
  + MAX44009 és a BH1750 között gondolkodtam, és igazából nem kell a BH1750-nél jobb ehhez a projekthez nekem, szóval az ára miatt is elég a BH1750. Ha esetleg piaci célra kiterjeszteném, akkor ott szóba jöhet egy még pontosabb/érzékenyebb mérőeszköz vagy ha a BH1750 nem találom megfelelőnek.
*	**Besütési szög érzékelése**: kétfajta opció létezik: szenzoros vagy szoftveres, én a szenzort választom ebben az esetben.
  + LDR eltérő szögben. Olcsó, egyszerű, és a beesési szög logikához bőven elég információt ad.
* **Redőnymotor vezérlés**: 2 csatornás relé modul
  + Olcsó, egyszerű, pont erre való – fel/le vezérléshez tökéletes.
*	**Időzítés**: az ESP32 eléri a pontos internet időt.
  + Ez bőven elég.

### Összegezve: 

| Kategória| Ajánlott alkatrész | Indok |
|-----|---------|--------|
| Mikrovezérlő | ESP32 DevKit V1 | olcsó, gyors, wifi, sok analóg bemenet |
|Lux mérés | BH1750 | pontos, stabil fényerő adatok |
|Beesési szög | 4 db LDR eltérő szögben | egyszerű és olcsó megoldás |
|Motorvezérlés | 2 csatornás relémodul | egyenesen redőnymotorhoz való |
|Időzítés | Internet NTP | nincs extra modul, pontos |


### Tinkercad-beli megvalósítás
Az eredeti rendszer ESP32-re készült volna, mivel az ESP32 képes WiFi-re, pontos internetidő (NTP) lekérésére, BH1750 luxszenzor kezelésére, valamint komplex nappozíciós számításokra. A Tinkercad azonban nem támogat ESP32-t, BH1750-et, napállás-szimulációt vagy valódi időt, ezért egy leegyszerűsített Arduino UNO alapú demonstráció készült.

### A Tinkercad-es rendszer működése
A szimuláció 4 darab LDR segítségével érzékeli a fényerőt és annak irányát. A rendszer reakciói:
- A szervó pozíciója arányosan a fényerőhöz igazodik.
- Manuális vezérlés fel/le gombokkal (D2, D3).
- Automata mód külön gombbal ki/bekapcsolható (D4).
- LED-ek jelzik a mozgásirányt (D6 = fel, D7 = le) és az automata mód állapotát (D5).
- Alapból az automata funkció ki van kapcsolva.
- Induláskor (áram alá helyezéskor) a szervó nem mozdul meg.

![Intelligens Redőnyvezérlő.png](https://github.com/kl-digit-project/digitech2/blob/main/Intelligens%20Red%C5%91nyvez%C3%A9rl%C5%91.png)

### Főbb kódrészletek és működési logika

Automata mód alapállapota:
autoEnabled = false (ezért bekapcsolás után nem fogja automatikusan állítgatni a szervó pozíciót, hanem csak ha már azt szeretnénk)
Fény → redőny arányos vezérlés:
avgLight = átlagolt LDR érték
blindPos = 100 - avgLight

Mozgás irányának kijelzése LED-del:
setMotionLED(+1) → fel LED
setMotionLED(-1) → le LED
setMotionLED(0) → mindkettő ki
A LED-ek mozgásra és nem gombnyomásra világítanak. Ez egy fontos  technikai különbség és biztonságosabb is.

### Miért butított verzió a Tinkercad változat?
A Tinkercad nem képes:
- ESP32 szimulációra
- BH1750 luxszenzor kezelésre
- NTP alapú időzítésre
- Nap beesési szögének szimulációjára
Ezért a bemutató változat csak az alap logikákat mutatja be.

### Hogyan bővíthető a valódi ESP32 projektre?
ESP32-re áthelyezve a rendszer tudná:
- A nap beesési szögét kiszámolni (napmagasság + azimut)
- Internetidő alapú időzítést (pl. napfelkelte/hanyatlás alapján)
- BH1750 luxmérést precízen
- Webes felületet (redőnyvezérlés telefonról)
- Precíz automata módot (PID-szerű fényerőszabályzás)

### Összefoglalás
Ez a dokumentum és a Tinkercad-es rendszer a működés elvét mutatja be. A valós, teljes funkcionalitás ESP32-vel és valódi érzékelőkkel valósítható meg.

## Kód magyarázata részletesebben

###  Változók és pinek

- **const int LED_AUTO = 5;** — Ez az automata módot jelző LED lába. Ha az automata be van kapcsolva, ez a LED világít.
- **const int BTN_AUTO = 4;** — Ez az automata módot kapcsoló gomb. INPUT_PULLUP miatt nyomáskor LOW szintet ad.
- **const int LDR_PINS[4] = {A0,A1,A2,A3};** — A 4 fényérzékelő bemenetei. Ezek átlagából számoljuk a szoba fényerejét.
- **const int SERVO_PIN = 9;** — A szervó vezérlőlába. Ez mozgatja a redőnyt jelképező motort.

###  Állapotváltozók szerepe

- **int blindPos = 50;** — A redőny pozícióját tárolja 0–100% között. Ez alapján állítjuk a szervót.
- **bool autoEnabled = false;** — Alapból kikapcsolt automata mód. Csak gombnyomásra lehet bekapcsolni.
- **unsigned long bootQuietMs = 5000;** — Az indulási csendes idő. 5 másodpercig nem mozdul a rendszer bekapcsolás után.

###  Gombkezelés működése

- **if(digitalRead(BTN_UP) == LOW)** — A fel gomb megnyomása. INPUT_PULLUP miatt LOW a jel.
- **if(digitalRead(BTN_DOWN) == LOW)** — A le gomb megnyomása.
A gombok manuálisan felülírják az automata módot, így a felhasználó mindig irányíthatja a rendszert.

###  Automata mód váltása

- **if(ar == LOW && !autoLatch){ autoEnabled = !autoEnabled; }** — Egyetlen gombnyomás váltja az automatikát KI/BE.
A sárga LED jelzi a mód állapotát. Debounce biztosítja, hogy egy gombnyomás ne legyen kettőnek érzékelve.

###  Fényerő mérés és célpozíció

- **int raw = analogRead(A0..A3)** — A 4 LDR fényértékének összeadása.
- **avgLight = mapP(raw)** — 0–100 skálára alakítva.
- **target = 100 - avgLight** — Minél világosabb van, annál lejjebb kell húzni a redőnyt.

###  Szervó mozgatása

- **if(target > blindPos) blindPos++;** — Ha világosabb van a kelleténél, lefelé mozdul.
- **if(target < blindPos) blindPos--;** — Ha sötét van, felfelé mozdul.
- **writeServoPos(blindPos);** — A százalékos pozíciót 0–180° közti szögre alakítja és mozgatja a szervót.

###  LED visszajelzés

- **setMotionLED(+1)** — Ha felfelé mozog, a zöld LED világít.
- **setMotionLED(-1)** — Ha lefelé mozog, a piros LED világít.
**LED késleltetés 300ms** — Akkor is látszik az irányjelzés, ha csak egy apró lépést mozdult a motor.


