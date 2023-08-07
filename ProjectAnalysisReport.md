# Izveštaj analize projekta

## O projektu

- Projekat nad kojim će biti izvršena analiza se zove _Predizolovane cevi_ i može se pronaći na sledećoj adresi:
  https://gitlab.com/matf-bg-ac-rs/course-rs/projects-2020-2021/01-predizolovane-cevi.

- Alati za verifikaciu softvera će biti primenjeni na main granom i commit-om sa hešom 6b9d9e25160f07ca71a70ba23178d0f140d952d6

- Ovaj projekat predstavlja alat koji bi trebalo da pomogne inženjerima koji se bave projektovanjima cevovoda, tako što bi im olakšao razna stručna izračunavanja. Naime, trenutno se sva izračunavanja vrše nad excel tabelama, što može biti veoma zamorno i neprijatno za rad

- Projekat je implementiran uz pomoć **C++** i **Qt** okruženja.

## Valgrind - Memcheck

- Prvi alat koji je upotrebljen za analizu ovog projekta je _Memcheck_. Memcheck se koristi za otkrivanje grešaka u porgramu koje su vezane za memoriju.

- Ukoliko bi se analiza projekta pokrenula nad originalnom verzijom Memcheck bi previše pažnje posvetio Qt funkcijama, što nije cilj ovih analiza, a i što bi dosta otežalo otkrivanje grešaka koje potiču iz koda originalnog projekta. Iz tog razloga, main funkcija je izmenjen na sledeći način.

```
#include "gui/mainwindow.h"
#include "headers/EnergyLoss.hpp"
#include "headers/PressureLoss.hpp"
#include "headers/Tcompesation.hpp"

#include <QApplication>

#define FLOWTEMP 130
#define RETURNTEMP 70
#define NEWTEMP 72
#define INSTALLTEMP 10
#define SOILDENSITY 1.9
#define INTERNALSOILFRACTIONANGLE 32.5
#define DESIGNWATERPRESSURE 2.5
#define TYPE 1
#define SERIES 1
#define DN 25
#define COVERDEPTH 1.0
#define NUMBER 1
#define ROUTELENGTH 200
#define BRANCHLENGTH 50
#define WETNESS dry
#define CREDITINTEREST 10
#define INFLATIONRATE 2.5
#define PRICEREISEDUETOENERGY 4
#define USAGETIME 30
#define PRODUCTCOSTS 0.04
#define TIMEFACTOR 365
#define Q 10000

int main(int argc, char *argv[])
{
//    QApplication app(argc, argv);
//    MainWindow w;
//    w.show();
//    return a.exec();

    EnergyLoss* el = new EnergyLoss(FLOWTEMP, RETURNTEMP, INSTALLTEMP, SOILDENSITY, INTERNALSOILFRACTIONANGLE, DESIGNWATERPRESSURE,
               TYPE, SERIES, DN, COVERDEPTH, NUMBER, ROUTELENGTH, WETNESS, CREDITINTEREST, INFLATIONRATE,
               PRICEREISEDUETOENERGY, USAGETIME, PRODUCTCOSTS, TIMEFACTOR);
    el->setReturnTemp(NEWTEMP);
    el->i_z();

    PressureLoss* pl = new PressureLoss(FLOWTEMP, RETURNTEMP, INSTALLTEMP, SOILDENSITY, INTERNALSOILFRACTIONANGLE,
                                       DESIGNWATERPRESSURE, TYPE, SERIES, DN, COVERDEPTH, ROUTELENGTH, Q);
    pl->setReturnTemp(NEWTEMP);
    pl->R();

    Tcompesation* tc = new Tcompesation(FLOWTEMP, RETURNTEMP, INSTALLTEMP, SOILDENSITY, INTERNALSOILFRACTIONANGLE,
                                        DESIGNWATERPRESSURE, TYPE, SERIES, DN, COVERDEPTH, ROUTELENGTH, BRANCHLENGTH,
                                        BRANCHLENGTH, TYPE, SERIES, DN, COVERDEPTH);
    tc->setReturnTemp(NEWTEMP);
    tc->UnrestrictedExpansion();

    delete  el; delete pl; delete tc;

//    app.exit();
    return 0;
}
```

- Ovako izmenjena main funkcija omogućava testiranje nekoliko najvećih klasa i njihovih funkcija u programu, a to su _EnergyLoss_, _PressureLoss_ i _Tcompesation_. Navedene klase nisu dominantne u odnosu na preostale klase iz projekta, ali je odlučeno da se testira njihov rad jer se mogu smatrati reprezentativnim. Tačnije, ni jedna od klasa u projektu nije dominantna, i sve klase su veoma jednostavne, bave se jednostavnim izračunavanjima po već unapred poznatim formulama, stoga je odlučeno da se testiraju ove tri klase kao prosečni predstavnici svih klasa u projektu. Takodje, definisan je veći broj konstanti pre main funkcije, jer nije bilo drugog načina za kreiranje instanci pomenutih klasa, i u originalnom projektu objekti se instanciraju upotrebom velikog broja magičnih konstanti za koje ne postoji nikakvo objašnjenje, što će kasnije u daljoj analizi biti svakako detaljnije obradjeno

- Bitno je napomenuti su u kompajleru prosledjene opcije **-g -O0** u okviru _Makefile_-a kako bi se kod preveo u debug mode-u, i bez optimizacija i kako bi pronajdene greske ukazivala na linije u originalnom kodu, a ne nekom optimizovanom kodu.

- Nakon bildovanja projekta u okviru Qt okruženja pokrenuta je analiza upotrebe memomorije u projektu upotrebom sledeće komande:

```
valgrind --leak-check=full --track-origins=yes --log-file=valgrind_memcheck.txt  ./main
```

- Izlaz rada alata je preusmeren u fajl **valgrind_memcheck.txt**, a korišćene su i opcije _--leak-check=full_ i _--track-origins=yes_ kako bi se dobili deteljniji opisi

- Pomenuti fajl se može pronaći [ovde](/valgrind/memcheck/valgrind_memcheck.txt)

- Na prvi pogled, pregledom sumiranog rada alata, može se reći da ne postoje nikakvi propusti u radu sa memorijom, ne postoje definitivno, indirektno ili mogluće izgubljeni blokovi. Mada može se rećida je ovo očekivano,jer kao što je već pomenuto, reč je o izizetno jednostavnih klasama koje se bave samo izračunavanjima na osnovu već poznatih vrednosti.

```
==28911== HEAP SUMMARY:
==28911== in use at exit: 18,612 bytes in 6 blocks
==28911== total heap usage: 1,037 allocs, 1,031 frees, 273,811 bytes allocated
==28911==
==28911== LEAK SUMMARY:
==28911== definitely lost: 0 bytes in 0 blocks
==28911== indirectly lost: 0 bytes in 0 blocks
==28911== possibly lost: 0 bytes in 0 blocks
==28911== still reachable: 18,612 bytes in 6 blocks
==28911== suppressed: 0 bytes in 0 blocks
==28911== Reachable blocks (those to which a pointer was found) are not shown.
==28911== To see them, rerun with: --leak-check=full --show-leak-kinds=all
==28911==
==28911== For lists of detected and suppressed errors, rerun with: -s
==28911== ERROR SUMMARY: 8 errors from 8 contexts (suppressed: 0 from 0)
```

- Medjutim, alat ukazuje na veliki broj propusta vezano za upotrebu neinicijlizovanih vrednosti, što može predstavljati problem. Što u kombinaciji sa velikim brojem magičnih konstanti donosi dosta zabune u projekat.
