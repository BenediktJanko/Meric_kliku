# Text práce (rozpracovaná verze k 7. 11. 2026)

## 1. Úvod a cíl práce

Cílem práce je navrhnout a sestavit zařízení nošené na těle, které měří vzdálenost k okolnímu objektu a při překročení nastaveného limitu upozorní uživatele zvukovým signálem. Zařízení je postaveno na vývojové desce M5Stack Core2 (mikrokontrolér ESP32). Součástí práce je laboratorní srovnání ultrazvukového a optického čidla vzdálenosti z hlediska přesnosti, latence a vlivu oděvu na vzdálenostech 5–60 cm. Na základě srovnání bude vybráno optimální čidlo a vyrobeno uchycení na tělo.

Práce klade důraz na praktickou část (hardware, firmware, měření). Teoretická část tvoří nejvýše třetinu rozsahu.

## 2. Použitý hardware a software

### 2.1 Hardware

- Vývojová deska **M5Stack Core2** (ESP32, displej, reproduktor).
- Ultrazvukové čidlo **HC-SR04**, dodané s kabelem zakončeným konektorem typu Grove na jedné straně a Dupont konektory na druhé.
- Optický modul s potenciometrem (označení na desce „MH-Sensor-Series", piny VCC, GND, DO, AO). Podle vzhledu jde pravděpodobně o odrazové čidlo typu TCRT5000. **[DOPLŇ: ověřit označení čidla]**
- Malá destička s optickým čidlem (bez dalších součástek). **[DOPLŇ: ověřit typ]**
- Druhý M5Stack a propojovací kabely. **[DOPLŇ: model druhého M5Stacku]**

### 2.2 Software

- **Arduino IDE** 2.x, programovací jazyk C++. **[DOPLŇ: přesná verze z Help → About]**
- Balíček desek **M5Stack** (verze 3.3.10 se nepodařilo nainstalovat, viz kapitola 3.3) **[DOPLŇ: nakonec nainstalovaná verze]**.
- Knihovna **M5Unified** (včetně závislosti M5GFX).
- Správa verzí: **Git** a **GitHub**.

## 3. Dosavadní postup realizace

### 3.1 Založení projektu a repozitáře

Byl založen repozitář na GitHubu se souborem `README.md`, souborem `.gitignore` (vylučuje dočasné soubory překladu a soubory s citlivými údaji, např. `secrets.h`) a se strukturou složek `firmware/`, `config/`, `cad/` a `docs/`. Repozitář byl sdílen s vedoucím práce **[DOPLŇ: potvrdit, že je sdílený]**.

### 3.2 Volba vývojového prostředí

Původní plán počítal s VS Code a rozšířením PlatformIO. Z praktických důvodů jsem se rozhodl pro **Arduino IDE** a jazyk C++, které jsou jednodušší na zprovoznění. Zadání (C++ pro M5Stack) tím zůstává splněno.

### 3.3 Zprovoznění vývojového prostředí a vzniklé problémy

Zprovoznění desky Core2 vyžadovalo několik kroků a při něm vznikly následující problémy:

| # | Problém (chybová hláška) | Příčina | Řešení |
|---|---|---|---|
| 1 | `Missing FQBN (Fully Qualified Board Name)` | V Arduino IDE nebyla vybrána žádná deska. | Výběr desky v menu Tools → Board. |
| 2 | Dlouhý výpis chyb při překladu (`pinMode declared void`, `digitalPinToGPIONumber is not a type`, `'pinMode' is not a member of 'm5gfx'`) | Překlad probíhal pro desku z balíčku Arduino ESP32 Boards. Jeho definice pinů je nekompatibilní s knihovnou M5Unified. | Instalace balíčku M5Stack a výběr desky M5Stack. |
| 3 | `Failed to install platform: 'M5Stack:3.3.10' … There is not enough space on the disk` | Na disku C: došlo volné místo. Balíček obsahuje velké sady překladačů. | Uvolnění místa na disku (vyčištění disku, mazání souborů a dočasných souborů Arduina) a opakovaná instalace. Po uvolnění zbývaly cca 4 GB. |
| 4 | Stejná chyba překladu jako v bodě 2 se opakovala | Balíček M5Stack se kvůli bodu 3 nenainstaloval, zůstala vybraná nevhodná deska. | Opakovaná instalace balíčku, poté výběr desky M5Stack Core2. |
| 5 | `Could not open COM3, the port is busy or doesn't exist` (překlad již prošel, nahrání selhalo) | Počítač neviděl desku na portu COM3. | **[DOPLŇ: co přesně pomohlo, např. jiný USB kabel / jiný port / výběr jiného COM portu / ovladač]** |

Po vyřešení těchto problémů se podařilo nahrát do desky první testovací program, který vypíše text „Ahoj!" na displej. Tím bylo ověřeno funkční vývojové prostředí.

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

### 3.4 Přípravy na zapojení čidel

Při pokusu o zapojení ultrazvukového čidla HC-SR04 se ukázalo, že dodané propojovací kabely nejsou vhodné (konektor Grove na straně desky Core2 a Dupont konektory na straně čidla neodpovídají zapojení potřebnému pro měření). **[DOPLŇ: přesně popsat, co je na kabelech špatně]** Zapojení čidel proto bude provedeno až po obstarání vhodných kabelů.

Při návrhu zapojení byla zjištěna dvě rizika, která je nutné ošetřit:

- Výstup Echo čidla HC-SR04 má úroveň 5 V, zatímco piny ESP32 snášejí 3,3 V. Pro bezpečné měření je třeba použít dělič napětí (např. rezistory 1 kΩ a 2 kΩ).
- Optická čidla typu TCRT5000 mají podle obecných údajů dosah jen několik centimetrů a nelineární výstup. Pro rozsah 5–60 cm mohou být nevhodná. Ověření proběhne laboratorním měřením.

## 4. Další postup

1. Obstarat vhodné propojovací kabely a případně rezistory pro dělič napětí.
2. Zprovoznit měření vzdálenosti ultrazvukovým čidlem a vypisovat hodnoty na displej.
3. Zprovoznit optické čidlo.
4. Provést laboratorní srovnání na vzdálenostech 5, 10, 15, 20, 30, 40, 50 a 60 cm (přesnost, latence, vliv oděvu), výsledky uložit do složky `docs/`.
5. Přidat zvukovou odezvu z reproduktoru Core2 při překročení limitu vzdálenosti.
6. Navrhnout uchycení na tělo.
