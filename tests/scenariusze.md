# Scenariusze sprawdzenia

Próby z 2026-11-21 na macOS 27.2 (Apple Silicon), `duckc 0.3-alpha (83882805a 2026-11-20)`, LZ4 1.10.0 (Homebrew), wariant natywny, po `python3 build.py --target all --output build`.

| Scenariusz | Dane lub czynności | Oczekiwany wynik | Komenda lub procedura | Otrzymany wynik |
| --- | --- | --- | --- | --- |
| Zwykłe działanie: kompresja i dekompresja | tekst z polskimi literami ×40 z bajtem NUL (1841 B) | blok mniejszy niż dane, odtworzenie identyczne | `run.py --target przyklad` | blok 68 B (granica 1864 B), identyczne - OK |
| Duże i trudne dane | 1 MB powtarzalnych, 100 KB losowych, 70 000 × NUL | obieg bez zmian; losowe: blok większy niż dane, w granicy | `run.py --target tests` | zgodny - OK |
| Puste dane i bajt NUL | 0 B, jeden NUL, NUL i 255 w środku | blok 1 B dla pustych; obieg bez zmian | testy, przykład | zgodny - OK |
| Niewystarczający limit | blok 1000 B z limitem 999 i 0; przykład z limitem 100 | `ZA_MALY_LIMIT`, potrzebny rozmiar 1000 / 1841, brak danych | testy, przykład | zgodny - OK |
| Niepoprawne wywołanie | limit −1 i 2147483648 | `BLEDNE_PARAMETRY` (nie uszkodzone dane) | testy, przykład | zgodny - OK |
| Niepoprawne dane | pusty blok, ucięty blok, dopasowanie przed początkiem, literał dłuższy niż blok | `USZKODZONE_DANE` | testy, przykład | zgodny - OK |
| Granice rozmiarów | `LZ4_compressBound` dla 0, 255, 1000, `LZ4_MAX_INPUT_SIZE` i +1, −5; `miesciSieWInt` | 16, 272, 1019, >wejście, −1, −1; INT_MAX tak, INT_MAX+1 nie | testy | zgodny - OK |
| Format DKZ4 | nagłówek, fałszywy rozmiar, zły znacznik, obcięty plik, zła suma, zapis/odczyt, brak pliku | wykrycie każdego uszkodzenia, obieg bez zmian | testy | zgodny - OK |
| Praktyczne zastosowanie | spakowanie i rozpakowanie `app/lz4.dk` | 15253 B → 7201 B; rozpakowany plik identyczny | pakowacz `spakuj` i `rozpakuj`, `cmp` | zgodny, kod 0 - OK |
| Pakowacz: złe dane | rozpakowanie zwykłego pliku; nieznane polecenie | kod 5 „zly znacznik”; kod 2 | pakowacz | zgodny - OK |
| Testy | 30 sprawdzeń | `Bledy: 0`, kod 0 | `python3 run.py --build build --target tests` | `Bledy: 0` - OK |
| Odtworzenie w nowym katalogu | świeży klon repozytorium | budowa, przykład, pakowacz i testy zgodne z README | `git clone`, komendy z README | macOS: OK; Linux x86_64 (Ubuntu 24.04, liblz4 1.9.4): nie sprawdzono |
