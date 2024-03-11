# Automatizované kalibrace měřících přistrojů
Kalibrace měření kmitočtů osciloskopem Siglent SDS1102X+ porovnáním s  etalonovým čítačem Agilent 53131A. Generování AC napětí definovaného kmitočtů  bude realizováno programově pomocí generátoru Agilent 33220A

## Popis řešení:
Systém je postaven na stavovém automatu se stavy Init, Measure, Wait, End. Před 
spuštěním programu, je uživatel požádán o zadání portu pro generátor Agilent 33220A a pro 
osciloskop Siglent SDS1102X+. Zároveň je uživatel musí zadat adresu připojeného čítače 
Agilent 53131A, který není připojen přes USB rozhraní, ale přes GPIB rozhraní. 
Po spuštění programu je default stav Init, ve kterém si uživatel může nastavit, jestli chce na 
etalonovém čítači zapnout/vypnout 100kHz filtr, typ vazby (AC/DC) anebo vstupní 
impedanci (1MOhm/50Ohm). Uživatel má následně výběr až 5 kalibračních bodů kmitočtu, 
kde jednotlivé body můžou nabývat frekvencí 50 Hz, 1 kHz, 5 kHz, 10 kHz, 20 kHz, 50 kHz, 
100 kHz. Může si také nastavit výstupní napětí až do hodnoty 3.5V. Pokud vše je v pořádku 
nastaveno, tak se pomocí stisku tlačítka „Confirm initialization“ potvrdí nastavení přístrojů a 
čeká se na stisknutí tlačítka „Start Measuring“, kde po jeho stisku přejde stavový automat na 
stav „Measure“

![ems_1](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/3ffae8b3-a79b-4afe-bd92-de074e95b486)


V tomto stavu se nastaví nejdříve požadovaná frekvence na generátoru pomocí příkazu 
„FREQ:FIXED 5000“ (nastaví se frekvence 5kHz…pro ukázku). Pro zadání tohoto a 
předchozích příkazů je vytvořeno subVI „VISA W/R“.

![ems_2](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/261bafd7-afee-4ed3-8186-ef0956b5dd89)

Poté co je na výstupu generátoru zadaná frekvence, tak začne měřit osciloskop. Ve flat 
sekvenci nejdříve se pomocí příkazu „AUTO_SETUP“, který nastaví na osciloskopu minimální 
počet period pro správný výpočet kmitočtu. Poté počká 500ms a až poté začne vyčítat
naměřenou frekvenci pomocí příkazu „CYMOMETER?“. Výstupem příkazu je string ve tvaru 
„CYMT 1.00000kHz“, tudíž je nutné pro další práci tento výstup upravit. To je uděláno 
v subVI „Extract Numbers with Match Pattern“, kde se mu pošle výsledný string a udělá z něj 
číslo ve formátu double. 

![ems_3](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/2d6c1d92-559b-42f5-88bb-720b2972bdaa)

Současně se provádí měření na čítači Agilent 53131A. K tomu se používá příkaz 
„:MEAS:FREQ?“, který se tentokrát nezapisuje do subVI „VISA W/R“, ale do funkčního bloku 
„GPIB Write“. 

![ems_4](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/fb654d4c-70ab-4b6e-be45-6c85ca96d57f)


Výsledky z obou metod jsou následně indexovány přes for smyčku, která je zde pro uživatele, 
kdy si může nastavit počet opakování měření jednotlivých kalibračních bodů. Po skončení 
všech nastavených opakování měření se vypočítají nejistoty typu A.

![ems_5](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/805e0d75-a72e-4160-a8ef-f60e200003f7)


Tyto nejistoty spolu s průměrnou naměřenou hodnotou se uloží do tabulky. Po skončení 
prvního kalibračního bodu měření se nastaví druhá zvolená frekvence a měření se opakuje a 
znovu uloží hodnoty do tabulky. Tento proces se opakuje, dokud nedojde k projití všech 
kalibračních bodů. Následně jsou tyto naměřené hodnoty zobrazeny jednak v tabulce na 
front panelu, tak také uloženy do textového souboru. Poté se splní podmínka pro přejití do 
stavu „Wait“, kde se počká 1000ms a přejde se znovu do stavu „Init“, kde si uživatel, může 
změnit jednotlivé kalibrační body nebo počet opakování.


## Vývojový diagram 
![ems_6](https://github.com/HonzaAntos/automatizovane_kalibrace_mericich_pristroju/assets/112206462/6fb29d93-0e72-4d01-9cbe-0e39968afe5b)

