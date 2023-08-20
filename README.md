# 2023_Analysis_predizolovane-cevi

## Analiza projekta Predizolovane cevi

U ovom projektu je vršena analiza projekta _Predizolovane cevi_ koji se može pronaći na adresi: https://gitlab.com/matf-bg-ac-rs/course-rs/projects-2020-2021/01-predizolovane-cevi. Primena alata biće izvršena nad main granom, i commit-om čiji je heš: 6b9d9e25160f07ca71a70ba23178d0f140d952d6.

## Opis projekta

Predizolovane cevi predstavlju alat koji bi trebalo da pomogne inženjerima koji se bave projektovanjima cevovoda, tako što bi im olakšao razna stručna izračunavanja. Naime, trenutno se sva izračunavanja vrše nad excel tabelama, što može biti veoma zamorno i neprijatno za rad.

## Korišćeni alati:

```
- Valgrind - memcheck
- Valgrind - massif
- ClangTidy
- ClangFormat
```

## Zaključak:

Projekat Predizolovane cevi je po obimu mali, a i po kompleksnosti jednostavan. Ne postoji nikakva dinamika u toku rada programa, što dovodi do toga da ne postoji veći broj instanciranih objekata u toku izvršavanja, pa očekivano nakon upotrebe alata koji ispituju upotrebu memorije, se dolazi do zaključka da tu ne postoje problemi. Medjutim, tokom prvog pregledanja projekta uočeno je dosta nepravilnosti u samom pisanju koda, u smislu stila pisanja, organizacije koja i razumljivosti koda, što je bio razlog za donošenje odluke da se iskoriste ClangTidy i ClangFormat alati. U detaljnijim izveštajima se može videti da je i rezltat rada ovih alata, da sam kod nije napisan na najbolji način.
