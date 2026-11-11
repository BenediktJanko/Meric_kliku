# Osnova

**Název práce:** Měřič kliků
**Autor:** Benedikt Janko

1. Úvod a cíl práce
2. Teoretická část
   1. Principy měření vzdálenosti (ultrazvuk, optická odrazová čidla)
   2. M5Stack Core2 a mikrokontrolér ESP32
3. Návrh řešení
   1. Požadavky počítání kliků (horní a dolní poloha)
   2. Blokové schéma a zapojení
   3. Výběr součástek
4. Experimentální srovnání senzorů
   1. Metodika (rozsah 5–60 cm, počet měření, latence, vliv materiálu)
   2. Varianta A: tři senzory (HC-SR04, MH-Sensor-Series, holé optické čidlo)
   3. Varianta B: dva senzory (HC-SR04, MH-Sensor-Series)
5. Realizace
   1. Vývojové prostředí a správa projektu na GitHubu
   2. Firmware: kalibrace a počítání kliků
   3. Akustická a zobrazovací odezva
   4. Uchycení na tělo / umístění čidla (CAD, 3D tisk)
   5. Problémy při realizaci a jejich řešení
6. Testování
7. Závěr a zhodnocení
8. Seznam zdrojů a přílohy
