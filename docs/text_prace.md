# Text práce (rozpracovaná verze k 11. 11. 2026)

## 1. Úvod a cíl práce

Cílem práce je navrhnout a sestavit zařízení, které automaticky počítá kliky. Zařízení je postaveno na vývojové desce M5Stack Core2 (mikrokontrolér ESP32) a ultrazvukovém dálkoměru HC-SR04. Čidlo leží na zemi pod cvičícím a měří vzdálenost mezi podlahou a tělem cvičícího. V horní poloze kliku je tato vzdálenost největší, v dolní nejmenší.

Cvičící před cvičením zařízení zkalibruje: v horní a v dolní poloze stiskne tlačítko a zařízení si změřené vzdálenosti zapamatuje. Při cvičení zařízení sleduje vzdálenost, počítá platné kliky a zobrazuje jejich počet na displeji. Klik, který nedosáhl dostatečné hloubky, se nezapočítá a zařízení na to upozorní zvukovým signálem.

Součástí práce je experimentální srovnání tří senzorů vzdálenosti (jednoho ultrazvukového a dvou optických), jehož cílem je vybrat senzor s nejstabilnějšími a nejpoužitelnějšími daty pro tuto úlohu. Práce klade důraz na praktickou část (hardware, firmware, měření). Teoretická část tvoří nejvýše třetinu rozsahu.

Původní plán počítal s obecným měřením vzdálenosti s akustickou signalizací překročení limitu. Během práce se zadání upřesnilo na počítadlo kliků. Měření vzdálenosti a akustická odezva zůstávají jeho základními částmi.


## 3. Návrh řešení

### 3.1 Princip počítání kliků

HC-SR04 leží na podlaze a měří vzdálenost k tělu cvičícího nad sebou. Cvičící má na hrudi plastovou desku připevněnou pomocí pásu. Čidlo (HC-SRO4) na zemi míří na plastovou desku a druhý senzor (MH-Sensor) míří ze strany na cvičího.

Zařízení pracuje takto:

1. **Kalibrace horní polohy.** Po stisku tlačítka zařízení třikrát pípne a při třetím pípnutí změří vzdálenost (průměr z 8 měření). Cvičící je v horní poloze (ruce natažené).
2. **Přesun dolů.** Následuje pauza 3 s, během níž cvičící sestoupí do dolní polohy a zůstane v ní.
3. **Kalibrace dolní polohy.** Zařízení znovu třikrát pípne a při třetím pípnutí změří dolní polohu.
4. **Výpočet rozsahu.** Kolem obou poloh se nastaví tolerance ±7 cm, aby cvičící nemusel klikovat pokaždé do úplně stejné polohy. Je-li rozdíl poloh malý, tolerance se zmenší na třetinu rozdílu. Rozdíl menší než 10 cm je odmítnut jako chyba kalibrace.
5. **Počítání.** Klik začíná opuštěním horní zóny. Do počtu se přičte, pokud cvičící dosáhl dolní zóny a vrátil se zpět do horní. Pokud se vrátí nahoru, aniž dosáhl dolní zóny, zazní dlouhé nízké pípnutí a klik se nezapočítá.
6. **Nová kalibrace.** Stiskem tlačítka kdykoli během cvičení se spustí nová kalibrace a počet se vynuluje.

Každé měření se filtruje mediánem ze tří dílčích měření, aby se potlačily náhodné špičky. Hystereze 2 cm brání tomu, aby drobné chvění způsobilo falešné spuštění kliku.

### 3.2 Zapojení

| Součást | Signál | Pin Core2 |
|---|---|---|
| HC-SR04 | Vcc | 5 V |
| HC-SR04 | Gnd | G (GND) |
| HC-SR04 | Trig | GPIO 26 |
| HC-SR04 | Echo | GPIO 35 |
| Externí tlačítko | kalibrace | GPIO 17 (druhý vývod na GND, vnitřní pull-up) |
| MH-Sensor-Series (jen při srovnání) | AO | GPIO 36 |
| MH-Sensor-Series (jen při srovnání) | VCC / GND | 3V3 / G |

Čidlo je připojeno Dupont vodiči k pinové liště na spodní straně Core2, nikoli přes konektor Grove (důvod viz kapitola 5.3, problém č. 7).


## 4. Experimentální srovnání senzorů

### 4.1 Uspořádání a použité senzory

Srovnání proběhlo postupně ve třech variantách, vždy s menším počtem senzorů. HC-SR04 ležel na zemi pod cvičícím a ostatní senzory byly těsně vedle něj, aby měřily stejným směrem.

1. **HC-SR04**: ultrazvukový dálkoměr (doba letu pulzu). Udávaný rozsah cca 2–400 cm. Napájení 5 V, výstup Echo 5 V.
2. **MH-Sensor-Series**: modul s infračerveným odrazovým čidlem s potenciometrem pro nastavení prahu, digitálním výstupem DO a analogovým výstupem AO.
3. **Holé optické čidlo**: malá destička s optickým čidlem bez zesilovače, komparátoru a potenciometru.

### 4.2 Varianta A: tři senzory

Nejdříve byly zkoušeny všechny tři senzory současně.

Holé optické čidlo bylo vyřazeno, protože mi došlo, že je zbytečné. Bohatě stačí dvě čidla.


Experiment pokračoval se dvěma senzory.

### 4.3 Varianta B: dva senzory (HC-SR04 a MH-Sensor-Series)

Ve druhé fázi se porovnávalo ultrazvukové čidlo s modulem MH-Sensor-Series. Měřilo se na vzdálenostech **[DOPLŇ: např. 5, 10, 15, 20, 30, 40, 50, 60 cm]**, na každé **[DOPLŇ: počet]** měření.

| Vzdálenost [cm] | HC-SR04: průměr [cm] | MH-Sensor: průměr (ADC) |
|---|---|---|
| 5 | 5,3 | 3650 |
| 10 | 10,1 | 2900 |
| 20 | 20,0 | 1550 |
| 30 | 29,8 | 880 |
| 40 | 41,6 | 380 |


Pozorované problémy MH-Sensor-Series:

- Optické odrazové čidlo tohoto typu má obvykle dosah jen několik až 15 cm , zatímco při kliku je třeba pokrýt rozsah desítek centimetrů.
- Odražené množství světla závisí na barvě a povrchu cíle a na okolním osvětlení.
- Digitální výstup DO dává jen informaci „blíž / dál než práh", ne vzdálenost, a pro počítání kliků s tolerancí poloh nestačí.

Výhody HC-SR04: výstup přímo ve vzdálenosti cm, dostatečný rozsah pro celý pohyb při kliku. Naměřil jsem nejvyšší naměřenou délku 45,5 cm. Jeho slabinou je neschopnost měřit velmi malé vzdálenosti (spolehlivě od cca 3 cm) a citlivost na odrazivost povrchu a úhel dopadu. Což ale pro projekt vůbec nevadí!


### 4.4 Varianta C: jeden senzor (HC-SR04)

Použití pouze HC-SR04 senzoru zní možná jako dobrý nápad. *Doplním až budu mít hotový pás a vyzkouším jestli by to bylo možné*

## 5. Realizace

### 5.1 Použitý hardware a software

**Hardware:** vývojová deska M5Stack Core (ESP32, displej, reproduktor), ultrazvukový senzor HC-SR04, optický modul MH-Sensor-Series, holé optické čidlo, externí tlačítko, propojovací Dupont vodiče, kabely Grove.

**Software:** Arduino IDE 2.3.10, jazyk C++, balíček desek M5Stack, knihovna M5Unified (včetně závislosti M5GFX), Git a GitHub.

### 5.2 Založení projektu a volba vývojového prostředí

Byl založen repozitář na GitHubu se souborem `README.md`, souborem `.gitignore` (vylučuje dočasné soubory překladu a soubory s citlivými údaji, např. `secrets.h`) a se strukturou složek `firmware/`, `config/`, `cad/` a `docs/`.

Původní plán počítal s VS Code a rozšířením PlatformIO. Z praktických důvodů jsem zvolil **Arduino IDE** a jazyk C++, které jsou jednodušší na zprovoznění. Zvolený jazyk i platforma (C++ pro M5Stack) zůstávají stejné. Jazyk C++ jsem zvolil, protože mi nejvíce sedí a byla to pro mě nejjednodušší varianta.

### 5.3 Průběh realizace a problémy

Tato kapitola popisuje chronologicky, co se při realizaci nepovedlo, proč, a jak se to vyřešilo. Problémy jsou rozdělené na vývojové prostředí, hardware a firmware.

#### A) Vývojové prostředí

**Problém 1: `Missing FQBN (Fully Qualified Board Name)`.**
Při prvním nahrání testovacího programu se objevila chyba o chybějícím jméně desky. Program byl správně, ale v Arduino IDE nebyla vybrána žádná deska, takže překladač nevěděl, pro jaký čip má program přeložit. Řešení: výběr desky v menu Tools → Board.

**Problém 2: dlouhý výpis chyb při překladu.**
Po výběru desky přišel výpis desítek chyb (`error: variable or field 'pinMode' declared void`, `'digitalPinToGPIONumber' is not a type`, `'pinMode' is not a member of 'm5gfx'`, `'frequency' was not declared in this scope`). Chyby nebyly v mém kódu, ale v propojení knihovny M5Unified s deskou. Cesta ke zdrojovým souborům v chybové hlášce (`packages\arduino\hardware\esp32\2.0.18-arduino.5`) ukázala, že se překládalo pro balíček **Arduino ESP32 Boards**, nikoli pro balíček **M5Stack**. Balíček od Arduina definuje makra pro přemapování pinů (`io_pin_remap.h`), která se střetávají s deklaracemi uvnitř knihovny M5GFX/M5Unified. Řešení: instalace balíčku desek M5Stack a výběr desky M5Stack Core2.

**Problém 3: selhání instalace balíčku M5Stack.**
Instalace balíčku skončila chybou `Failed to install platform: 'M5Stack:3.3.10' … There is not enough space on the disk`. Balíček obsahuje velké sady překladačů a nástrojů (i pro čipy, které Core2 nepotřebuje), a na disku C: nezbylo dost místa. Řešení: uvolnění místa (Vyčištění disku, vysypání koše, smazání dočasných souborů Arduina a nepotřebných souborů), poté nová instalace. Po uvolnění zbývaly cca 4 GB, což na překlad stačilo.

**Problém 4: stejná chyba překladu se vrátila.**
Dokud selhala instalace z problému 3, balíček M5Stack v systému nebyl a v nabídce zůstávala jen nevhodná deska z problému 2. Zopakovala se tedy stejná chyba. Po úspěšné instalaci se překlad povedl (zabral cca 7 % programové paměti).

**Problém 5: `Could not open COM3, the port is busy or doesn't exist`.**
Překlad už procházel, ale nahrání do desky selhalo, protože počítač neviděl Core2 na portu COM3. Musel jsem tedy hledat nový kabel, který by už přenášel data a nebyl pouze na nabíjení. Pro jistotu jsem to zapojil i do jiného portu USB, kdyby to bylo tím. Po vyřešení se na displeji objevil text „Ahoj!" a vývojové prostředí bylo ověřeno!

Testovací program:

```cpp
#include <M5Unified.h>

void setup() {
  auto cfg = M5.config();
  M5.begin(cfg);
  M5.Display.setTextSize(3);
  M5.Display.println("Ahoj!");
}

void loop() {
}
```

#### B) Hardware a zapojení

**Problém 6: kabely.**
Dodané kabely neumožnily zapojit čidlo podle plánu. Čidlo HC-SR04 bylo dodáno s kabelem, který má na jedné straně Dupont zásuvky a na druhé konektor, jenž **nezapadl do portu A (Grove)** na boku Core2. Dupont vodiče do portu A také nelze zasunout, protože port přijímá pouze konektor Grove. Zapojení čidel se tím zpozdilo o několik dní. Navíc nebylo možné zapojit ani optické čidlo (piny VCC, GND, DO, AO) přes Grove.

**Problém 7: změna způsobu zapojení.**
Místo portu A bylo čidlo připojeno Dupont vodiči přímo k pinové liště na spodní straně Core2. Tím odpadlo omezení Grove, ale zapojení bylo nutné sestavit podle popisků pinů přímo na krytu zařízení.

**Problém 8: chybné určení pinů v první verzi.**
První verze zapojení počítala s portem A, kde jsou signály na GPIO 32 a 33. Určení, který vodič (podle barvy) je Trig a který Echo, bylo jen odhadem a v průběhu se samo změnilo (nejprve žlutý = 33 a bílý = 32, později obráceně). Barvy vodičů nejsou spolehlivý pramen informace, protože jsou dané výrobcem kabelu, ne standardem. Po přechodu na pinovou lištu jsem piny určil podle jejich popisků a zapojení bylo ověřeno fotografiemi. Konečné zapojení: Trig → GPIO 26, Echo → GPIO 35. První verze kódu (piny 32 a 33) proto byla zahozena.

**Problém 9: konflikt pinů při přidání optického čidla.**
Pro analogový výstup AO optického modulu byl nejdřív navržen GPIO 35. Ten ale mezitím převzal výstup Echo z HC-SR04. Řešení: AO se přesunul na GPIO 36, který je také pouze vstupní a podporuje analogové čtení, napájení z 3V3.

**Problém 10: napětí na výstupu Echo.**
HC-SR04 se napájí z 5 V a jeho Echo vrací signál o úrovni 5 V, zatímco piny ESP32 snášejí 3,3 V. Čidlo bylo zatím připojeno bez děliče napětí, což může při dlouhodobém provozu poškodit mikrokontrolér. Řešení: doplnit dělič 1 kΩ + 2 kΩ (viz kapitola 3.2).

#### C) Firmware

Firmware prošel několika verzemi.

**Verze 1: měření vzdálenosti (první pokus).**
Základní program četl vzdálenost přes Trig (GPIO 32) a Echo (GPIO 33), vypisoval ji na displej a na sériovou linku. Nikdy se však nespustil na skutečném zapojení, protože kabely nevyhovovaly (problém 6) a piny byly určeny chybně (problém 8). Verze byla nahrazena.

**Verze 2: optické čidlo.**
Program četl hodnotu `analogRead()` na pinu AO. Původně na GPIO 35, po konfliktu (problém 9) na GPIO 36.

**Verze 3: počítadlo kliků (aktuální).**
Kód je uložen v `firmware/03_pushup_counter/03_pushup_counter.ino`. Skládá se z těchto částí:

- **Měření vzdálenosti:** 10µs pulz na Trig, měření délky odpovědi na Echo funkcí `pulseIn()` s časovým limitem 30 ms. Vzdálenost = doba × 0,0343 / 2 [cm]. Při vypršení limitu (žádné echo) funkce vrací −1.
- **Filtrace:** `readFiltered()` vrací medián ze tří měření (potlačení náhodných špiček), `measureAvg()` průměr z osmi takových měření pro kalibraci. Pokud se nepodaří aspoň polovina měření, kalibrace selže.
- **Kalibrace:** řízená externím tlačítkem na GPIO 17 (`INPUT_PULLUP`, odrušení zákmitů 50 ms). Odpočet třemi pípnutími, měření při třetím.
- **Stavový automat počítání:** proměnné `inRep` (probíhá klik) a `reachedBottom` (dosažena dolní zóna). Klik začíná při poklesu pod `topLimit − HYST_CM`, počítá se při návratu nad `topLimit`, pokud mezitím `d ≤ botLimit`. Jinak zazní neplatný signál.
- **Odezva:** displej zobrazuje počet kliků, meze a aktuální vzdálenost. Reproduktor signalizuje odpočet, hotovou kalibraci, platný a neplatný klik.

Zjištěné chyby ve firmwaru:

| # | Problém | Příčina | Řešení |
|---|---|---|---|
| F1 | První verze nešla spustit s reálným čidlem | Chybné určení pinů (problém 8) | Přepsáno na Trig = 26, Echo = 35 podle popisků na krytu |
| F2 | Konflikt pinů při přidání optického čidla | GPIO 35 už používal Echo | AO přesunut na GPIO 36 |
| F3 | Kalibrace může selhat | Rozdíl poloh menší než 10 cm, nebo méně než polovina měření úspěšná | Chybové hlášení, signál a možnost kalibraci zopakovat |

Očekávané slabiny kódu:

| # | Slabina | Důsledek | Možné řešení |
|---|---|---|---|
| S1 | **Mrtvá zóna HC-SR04.** Čidlo spolehlivě měří od cca 3–5 cm. Když je tělo v dolní poloze příliš blízko čidla, echo se ztratí a funkce vrací −1. | Program takovou hodnotu přeskočí (`if (d < 0) return;`), počítání se může zastavit. | Čidlo umístit tak, aby i v dolní poloze byla vzdálenost aspoň několik cm, případně chybějící echo vyhodnotit jako „velmi blízko". |
| S2 | **Blokující funkce.** `pulseIn()` a `delay()` při pípání a v kalibraci blokují smyčku. Jedno měření s mediánem trvá desítky ms, neplatné pípnutí 500 ms. | Při rychlém cvičení může zařízení vynechat část pohybu. | Pípání bez `delay()`, měření ve vlastním časovači. |
| S3 | **Cíl nemusí být celé tělo.** Ultrazvuk měří vzdálenost k nejbližšímu odrazivému předmětu v kuželu cca 15°. Může to být hruď, ruka nebo brada. | Naměřená vzdálenost nemusí odpovídat výšce těla. | Ověřit měřením, případně upravit umístění nebo ke čidlu přidat odraznou desku. |
| S4 | **Vliv oděvu.** Měkká látka pohlcuje ultrazvuk. | Slabé nebo žádné echo. | Změřit vliv různých oděvů (kapitola 4). |
| S5 | **Pevná tolerance** ±7 cm. | Pro různě hluboké kliky nemusí být vhodná. | Konstanty `TOL_CM` a `HYST_CM` ladit na reálných datech. |
| S6 | **Hardcoded hodnoty** (hlasitost, frekvence pípnutí, prodleva) | Obtížná úprava | Přesunout do složky `config/`. |

### 5.4 Uchycení a umístění čidla

HC-SR04 je umístěn na zemi pod cvičícím a míří svisle nahoru. *Držák budu teprve dělat. V CADU vytvořím a vytisknu podložku a držák na senzory*

## 6. Testování

*Až dodělám zbytek*

