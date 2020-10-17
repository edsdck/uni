Koliokvumas
=====

## 1. Kas yra programinės įrangos testavimas, koks jo tikslas(-ai)? Kokį požiūrį turi turėti testuotojas, testuojantis programinę įrangą?

Tai yra rizikos valdymo strategija, naudojama verifikuoti, ar programinė įranga ir  jos veikimas bei dokumentavimas atitinka specifikuotus reikalavimus.

Testuojama gali būti šiais tikslais:
* Užtikrinimui, kad programa veikia be klaidų (angl. bugs) arba klaidų radimui
* Įvertinti programinės įrangos kokybę, atitiktį standartams ir politikos ar teisės aktams.

**Reikalingas požiūris:**
> Pabandyk ir pažiūrėk, ar veikia -> išsiaiškink išsamiai visa, kas įtakoja sėkmingą veikimą ir kas įtakoja neveikimą.

Ir visada turi siekti aukščiausios kokybės. 

## 2. Testavimo gyvavimo ciklas, jo etapai

* Reikalavimų analizė
* Testavimo strategijos parinkimas
* Testų projektavimas
* Testavimo aplinkos parengimas
* Testavimo atlikimas, įskaitant gautų ir laukiamų rezultatų palyginimą
* Defektų registravimas bei jiems pašalinti reikalingų veiksmų numatymas
* Ataskaitos parengimas

## 3. Apibūdinti derinimą

Debugging is the process of finding and resolving bugs within computer programs, software, or systems.

Debugging tactics can involve interactive debugging, control flow analysis, unit testing, integration testing, log file analysis, monitoring at the application or system level, memory dumps, and profiling.

## 4. Apibrėžti klaidą, defektą, trikį

**Klaida** – programų autoriaus apsirikimas, klaidingas supratimas, tai yra žmogaus veiksmas, kurio pasekoje gaunamas klaidingas rezultatas (a mistake in coding is called error).

**Defektas** reiškia, kad yra klaida arba trikis (error found by tester is called defect).

**Trikis** (ang. fault/bug) programinės įrangos „negebėjimas“ atlikti to, kas numatyta reikalavimų specifikacijoje.

## 5. Kas yra testavimo aprėptis/padengimas?  Kokie yra kodo padengimo kriterijai?

Kodo padengimas/aprėptis – matas, parodantis, kiek (kuri dalis) programinės įrangos kodo buvo vykdoma testuojant. Jis padeda įsitikinti, kad vykdomi testai iš tikro patikrina įmanomus atvejus. 

“Padengimo vienetu” gali būti bet koks programinės įrangos vienetas, kurį galime suskaičiuoti ir ištestuoti:

```Coverage = Number of coverage items exercised / Total number of coverage items *100%.```

100% padengimas nereiškia, kad aplikacija pilnai ištestuota, bet kuo didesnis padengimas, tuo didesnė tikimybė, kad palikta mažiau defektų, kadangi tuo didesnė dalis ištestuota.

**Kodo padengimo kriterijai:**

* Function coverage – has each function (or subroutine) in the program been called?
* Statement coverage – has each statement in the program been executed?
* Edge coverage – has every edge in the Control flow graph been executed?
* Branch coverage – has each branch (also called DD-path) of each control structure (such as in if and case statements) been executed? For example, given an if statement, have both the true and false branches been executed? This is a subset of edge coverage.
* Condition coverage (or predicate coverage) – has each Boolean sub-expression evaluated both to true and false?

## 6. Kaip vertinamas programinės įrangos testavimo aprėpties išbaigtumas?

* **Struktūrinis padengimas** atsako į klausimą,  ar kiekvienas produkto elementas buvo vykdomas ir testuojamas; priemonės: kodo teiginių, sprendimų, sąlygų ir sprendimų aprėpties tikrinimas
* **Architektūrinis padengimas** atsako į klausimą,  ar faktiniai  loginių kontrolių ir duomenų sąryšiai buvo ištestuoti; priemonės: kelių testavimas, loginių kontrolių sąryšiai
* **Reikalavimų padengimas** atsako į klausimą,  ar ištestuota atitiktis visiems reikalavimams normaliomis sąlygomis, taip pat ar ištestuota atitiktis visiems reikalavimams netikėtomis ir nenormaliomis sąlygomis. Priemonės: atitikties reikalavimų specifikacijos reikalavimams, projektinės dokumentacijos reikalavimams tikrinimas; padengimo silpnumo/tvirtumo tikrinimas

## 7. Web aplikacijų testavimas (įskaitant Web aplikacijų patikimumo ir saugos testavimą) – aprašyti, kaip testuojamos web aplikacijos.

1. Functionality testing
2. Usability testing
3. Interface testing
4. Compatibility testing
5. Performance testing
6. Security testing

Detailed explanation [here](https://stackify.com/web-application-testing-a-6-step-guide/)

## 8. Kaip testuojamos web aplikacijos kliento pusėje? “Bypass” testavimas – apibūdinti.

*too lazy rn*

## 9. Kaip testuojamos web aplikacijos serverio pusėje?

## 10. Testavimo aplinka.

Tai tarsi „smėlio dėžė“, kurioje galima tyrinėti ir sužinoti kaip veikia programinė įranga, nesibaiminant sugadinti production aplinkos (realios aplinkos);

*or*

Tai įdiegta ir atitinkamai sukonfigūruota programinės, techninės ir tinklo įrangos sąranka, skirta testavimui, tai yra testavimo atvejų vykdymui. Konfigūruojama pagal poreikį, atitinkantį testuojamos aplikacijos testavimo tikslus.

## 11. Statiniai testavimo atvejų kūrimo būdai. Apibūdinti kiekvieną.

* Walk through: „perėjimas pažingsniui“ nėra formalus peržiūros procesas; jį atlieka autorius, kuris komandos nariams teikia dokumentacijos paaiškinimus, paaiškina savų minčių eigą tam, kad gauti grįžtamąjį ryšį ir pasiekti bendro supratimo bei sutarimo.
* Informal Review: neformali peržiūra, kurios tikslas – patobulinti testuojamo produkto kokybę diskutuojant; dažniausiai nedokumentuojama. Iš pradžių gali neformaliai peržiūrėti du ar keli komandos nariai, vėliau – daugiau narių, organizuojant susirinkimus.
* Technical Review: atlieka techninis ekspertas, mažiau formali už inspektavimą. Tai „a peer review without any participation from management“ tipas.  (atlieka techninis ekspertas;  Peer review – testus peržiūri kiti komandos nariai (ne autorius),
* Audit: nepriklausomas su produktu susijusių dokumemtų nagrinėjimas, vykdomas išorinio agento.  Auditas padeda suintereuotiems asmenims įgyti didesnį pasitikėjimą sukurtu produktu,  įvertinti atitiktį standartams ir pan.
* Inspektavimas (angl. inspection): tai formalios peržiūros tipas, vykdomas apmokytų specialistų. Peržiūros metu dokumentai ir produktai nagrinėjami, juos peržiūrint ir identifikuojant trūkumus. Trūkumai dokumentuojami, sukuriamas jų sąrašas, taip pat su jais susijusių spręstinų problemų sąrašas (formali dokumentų ir/arba produkto peržiūra, nustatant defektus, kurie taip pat dokumentuojami).
* Code metrics is a set of software measures that provide developers better insight into the code they are developing. By taking advantage of code metrics, developers can understand which types and/or methods should be reworked or more thoroughly tested. Development teams can identify potential risks, understand the current state of a project, and track progress during software development. Code Metrics is an important measure that let us understand the complexity and maintainability of the code.

## 12. Dinaminiai testavimo atvejų kūrimo būdai. Apibūdinti kiekvieną.

1. Specification-Based / Black Box Test Design Technique - tai specifikacijomis paremtas testavimo atvejų projektavimo būdas, paremtas „juodosios dėžės“ principu: naudojamas sistemos ir jos elgsenos „išorinis“ aprašymas: techninė specifikacija, projektinė dokumentacija,  naudotojo reikalavimai ir kt. Testuotojas gali nenagrinėti programinės įrangos kodo ir nežinoti jos vidinės struktūros. Šis būdas taiko tokius metodus:
   * ribinių reikšmių analizės
   * testavimą, paremtą sprendimų lentelėmis
   * testavimą, paremtą baigtiniu skaičiumi būsenų perėjimų - State Transition Testing
   * testavimą, paremtą dalinimu į ekvivalentines klasases - Equivalence Class Partitioning
   * panaudos atvejų testavimą - Use Case Testing
2. Structure-based / White Box Test Design Technique – tai „baltosios dėžės“ principu paremtas būdas, kurį taikant, reikia žinoti vidinę programinės įrangos struktūrą.
   * sąlygų aprėpties - Condition Coverage
   * sprendimų aprėpties - Decision Coverage
   * teiginių/sakinių aprėpties - Statement Coverage
   * daugelio sąlygų aprėpties - Multiple Condition Coverage
3. Patirtimi paremtas testavimo atvejų kūrimo būdas /Experience-Based Test Design Technique: būdas nereikalauja žinoti nei vidinės programinės įrangos loginės struktūros, nei išorinių reikalavimų.
   * tiriamasis testavimas - Exploratory Testing
   * trikių atakų - Fault Attack, injection
## 13. Integracinis testavimas. Integracinio testavimo metodai/būdai (apibūdinti).

Integracinis testavimas pradedamas, kai individualūs moduliai apjungiami į grupę.  Tipiškai programinės įrangos produktas apima daugelį komponentų, kuriuos kuria įvairūs programuotojai. Integracinis testas tikrina, ar apjungti komponentai komunikuoja tarpusavy. Taip pat įvertinama kiekvieno komponento įtaka bendram produkto modeliui. Tai sistemos visumos ar jos integruotos dalies testavimas, tikrinant jos funkcionavimą, tame tarpe ir paremtą dviejų sistemų ar komponentų sąveika.

Būdai:

* Big Bang - This is an approach to Integration Testing where all or most of the units are combined together and tested at one go.
* Top Down - This is an approach to Integration Testing where top-level units are tested first and lower level units are tested step by step after that.
* Bottom Up - This is an approach to Integration Testing where bottom level units are tested first and upper-level units step by step after that.
* Sandwich/ Hybrid - This is an approach to Integration Testing which is a combination of Top Down and Bottom Up approaches. 

## 14. Priėmimo testavimas, jo rūšys. Apibūdinti.

Acceptance testing (ISTQB): formal testing with respect to user needs, requirements, and business processes conducted to determine whether or not a system satisfies the acceptance criteria and to enable the user, customers or other authorized entity to determine whether or not to accept the system.

Priėmimo testų tipai:
* atitikties (teisės aktams, įstatymams) testavimas,
* operacinis-gamybinis (atitiktis operaciniams reikalavimams) testavimas.

Alfa testavimą atlieka kūrėjai ir testuotojai;

Beta testavimas atliekamas pas klientą/užsakovą (angl. before delivery), atlieka naudotojai.

Gama testavimas atliekamas pas klientą, kai programinės įrangos einamoji versija parengta išleidimui.

## 15. Mutavimo testavimas. Apibūdinti.

Mutavimo testavimas – tai struktūrinis testavimas,  kuris remiasi „baltosios dėžės“ principu: jei pakeiti kodą, o testuojamas atvejis baigiasi sėkmingu testavimu, vadinasi, yra defektai programoje.

Mutavimo testavimo metu nežymiai modifikuojama programinė įranga. Mutavusi programinės įrangos versija vadinama mutantu („paveldimai pakitęs organizmas“). Testai aptinka ir atmeta mutantus (angl. killing the mutant). Mutantai kuriami apibrėžtų mutavimo operatorių pagrindu. Mutavimo operatoriai imituoja programavimo klaidas (pvz., nurodomas klaidingas kintamojo pavadinimas) arba priverstinai sąlygoja naujų testų sukūrimą (pvz., dalybai iš nulio). Mutavimo testavimo tikslas – aptikti silpnas vietas testiniuose duomenyse arba programiniame kode, ypač tame, kuris retai naudojamas,  taip sukuriant efektyvesnius testus.

## 16. “Chaos monkey” testavimas. Apibūdinti.

„Chaos monkey“  vadovaujasi nuostata  „geriausias būdas išvengti esminių trikių – nuolatos patirti trikius“ (angl. the best way to avoid major failures is to fail constantly).

Testuojant stengiamasi imituoti trikius tuo metu, kai jie gali būti operatyviai valdomi, tai yra kai jiems galima būti pasiruošus, nelaukiant, kol ištiks reali nuostolius nešanti katastrofa. Testavimo metu tikrinama, ar gerai valdomi/sprendžiami kilę incidentai/problemos.

## 17. Kokybės užtikrinimas; kokybės kontrolė; testavimas. Apibūdinti. Kokie skirtumai tarp šių trijų veiklų?

|Kokybės užtikrinimas| Kokybės kontrolė| Testavimas|
|---|---|---|
|Veiklų kompleksas, apimantis visus technologinius aspektus visuose programinio produkto kūrimo, diegimo ir eksploatavimo inicijavimo etapuose, ir vykdomas, siekiant užtikrinti tam tikrą programinio produkto kokybės lygį | Kontroliavimas, kad kuriamas programinis produktas atitiktų nustatytus reikalavimus | Procesas, apimantis testavimo atvejų kūrimą ir vykdymą, defektų aptikimą ir pan. | 
| Fokusuojasi į procesus ir priemones, o ne testavimo atlikimą | Koncentruojasi į testavimo atlikimą, vykdant programą; naudoja patvirtintus procesus ir priemones;  tikslas - nustatyti defektus. | Fokusuojasi į testavimo atlikimą |
| Orientuotas į procesą | Orientuotas į produktą | Orientuotas į produktą | 
| Preventinio pobūdžio, pateikia prevencines priemones | Koreguojantis procesas | Koreguojantis procesas |
| Programinės įrangos gyvavimo ciklo procesų poaibis | Kokybės užtikrinimo procesų poaibis (kokybės kontrolė – dalis kokybės užtikrinimo) | Kokybės kontrolės procesų poaibis(testavimas – dalis kokybės kontrolės) |

## 18. Testavimo scenarijai ir testavimo atvejai. Apibūdinti.

* Testavimams atlikti rengiami testavimo atvejai – aibės veiksmų, atliekamų, siekiant patikrinti programinės įrangos savybes arba funkcionalumą.
* Testavimo scenarijus – tai testavimo veiklos struktūros aprašymas. Jis apima vaidmenų sąrašą bei vaidmenų atliekamas testavimo veiklas ( pvz., vaidmenys - buhalteris, kasininkas; testavimo veiklas – sukurti prekių realizavimo  dukumentus);
* Testavimo atvejis atsako į klausimą, kaip testuoti, o testavimo scenarijus – kas turi būti testuojama.

## 19. Apibūdinti test-driven testavimą.

* Testavimu paremtas kūrimas (angl. Test driven development) – evoliucinis produkto kūrimas, kombinuojantis „test-first“ kūrimą (prieš rašant kodą, kuriami testai, kuriuos sėkmingai turi „praeiti“ kodas, išgaunamas iš testo refaktorizavimo/restruktūrizavimo būdu (išlaikant programos elgesio semantiką, pvz., vertinant „juodosios dėžės“ principu paremto tikrinimo požiūriu).
* Testavimu paremto kūrimo tiesioginis tikslas - specifikavimas, o ne validavimas. Tai būdas mąstyti per reikalavimų prizmę, prieš rašant funkcinį kodą. Tai svarbi AGILE tiek reikalavimų, tiek programinės įrangos projektavimo priemonė. Egzistuoja požiūris, kad tai ir programavimo priemonė, kurios tikslas – parašyti „švarų“ veikiantį kodą.


