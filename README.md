# DuckLZ4: wrapper LZ4 dla Ducklinga

Zespół: Pixel of Software (Marcin Kowalczyk). Temat: P02 DuckBridge, biblioteka LZ4 (poziom 1). Skład: 1 osoba.

Moduł `lz4` (`app/lz4.dk`) udostępnia w Ducklingu kompresję danych aplikacji przez bibliotekę C LZ4: zasobów, lokalnego cache i zapisów stanu. API przyjmuje i zwraca `List[u8]` (albo `str`).

* Wynik jest zawsze **kopią** należącą do wywołującego, a nie wskaźnikiem do pamięci biblioteki.
* Rozmiary są sprawdzane względem typu `int` z C, zanim trafią do biblioteki.
* Bufory są zwalniane przed powrotem z każdej funkcji.

Całe minimum karty LZ4 działa i jest sprawdzone testami. Do tego są dwa przykłady (podstawowa kompresja i praktyczny pakowacz plików) oraz rozszerzenia:
* rozróżnienie „za mały limit” od „uszkodzone dane”, wraz z potrzebnym rozmiarem;
* własny format pliku DKZ4, który przechowuje rozmiar i sumę kontrolną.

## Uruchomienie

Wymagane są:
* Python 3;
* Duckling (`duckc` w `PATH`, instalacja według tutoriala);
* biblioteka LZ4.

| System | Instalacja LZ4 |
| --- | --- |
| Ubuntu | `sudo apt install liblz4-1` (nagłówki `liblz4-dev` nie są potrzebne) |
| macOS | `brew install lz4` |

Z katalogu projektu:

```bash
python3 build.py --target all --output build      # aplikacja, przykład i testy (wariant natywny)
python3 run.py --build build --target tests       # testy: PASS/FAIL, kod = liczba błędów
python3 run.py --build build --target przyklad    # przykład podstawowy
printf 'spakuj\napp/lz4.dk\nbuild/lz4.dk.dkz4\n' | python3 run.py --build build --workdir .
printf 'rozpakuj\nbuild/lz4.dk.dkz4\nbuild/odtworzony.dk\n' | python3 run.py --build build --workdir .
```

Pakowacz czyta trzy wiersze ze standardowego wejścia:
1. polecenie: `spakuj` albo `rozpakuj`;
2. plik wejściowy;
3. plik wyjściowy.

Po spakowaniu sam rozpakowuje wynik i porównuje go z oryginałem. Kody zakończenia:

| Kod | Znaczenie |
| --- | --- |
| 0 | sukces |
| 2 | złe parametry |
| 3 | błąd pliku wejściowego |
| 4 | błąd zapisu |
| 5 | uszkodzone dane |

`build.py` szuka biblioteki w tej kolejności:
1. ścieżka z opcji `--lz4 /sciezka/liblz4.so.1`;
2. `pkg-config liblz4`;
3. typowe miejsca: `/usr/lib/x86_64-linux-gnu/liblz4.so.1` oraz `/opt/homebrew/lib/liblz4.dylib`;
4. `ctypes.util.find_library`.

Wybraną ścieżkę zapisuje w `build/budowa.json`. `build.py` i `run.py` powstały z szablonu konkursu: doszła obsługa biblioteki i cel `przyklad`, a został tylko wariant natywny.

## Funkcje i demonstracja

| Funkcja z minimum karty | Sposób uruchomienia | Oczekiwany rezultat |
| --- | --- | --- |
| Kompresja bloku z wcześniej obliczonym rozmiarem bufora | przykład | `Dane: 1841 B, granica wyniku (LZ4_compressBound): 1864 B, blok: 68 B` |
| Dekompresja z podanym limitem; dane identyczne bajt po bajcie | przykład | `Dekompresja z limitem 1841: OK, 1841 B, identyczne bajt po bajcie: true` |
| Błędne wywołanie odróżnione od uszkodzonych danych | przykład | limit −1: `BLEDNE_PARAMETRY`; blok z dopasowaniem przed początkiem danych: `USZKODZONE_DANE` |
| Niewystarczający limit | przykład | `ZA_MALY_LIMIT - wynik ma 1841 bajtow, a limit to 100 (potrzebny rozmiar: 1841)` |
| Wynik to kopia; rozmiar sprawdzony względem `int` z C | `tests` | `miesciSieWInt`, `granicaKompresji`, odrzucenie limitu 2147483648 |
| Puste dane, bajt NUL, błędny blok | przykład i `tests` | puste dane: blok 1 B, odtworzone 0 B; NUL w środku danych bez zmian; pusty, ucięty i błędny blok: `USZKODZONE_DANE` |
| Format zapisu plikowego i przechowywanie rozmiaru | pakowacz `spakuj` / `rozpakuj` | `app/lz4.dk`: 15253 B → 7201 B; rozpakowanie daje identyczny plik (`cmp`) |

**Demonstracja:**
1. Uruchom przykład. Pokazuje on kompresję tekstu z polskimi literami i bajtem NUL, odtworzenie z dokładnym limitem, trzy rodzaje błędów i puste dane.
2. Spakuj i rozpakuj dowolny plik pakowaczem; porównaj oryginał z wynikiem przez `cmp`.
3. Spróbuj rozpakować plik, który nie jest w formacie DKZ4. Dostaniesz kod 5 i komunikat `to nie jest plik DKZ4 (zly znacznik)`.

## API modułu `lz4`

| Funkcja | Zwraca | Błąd (`rodzaj`) |
| --- | --- | --- |
| `kompresuj(dane: ref List[u8]) -> Wynik` | `dane`: surowy blok LZ4 (kopia) | `BLEDNE_PARAMETRY`: dane ponad 2 113 929 216 B; `BRAK_PAMIECI` |
| `kompresujTekst(s: str) -> Wynik` | jak wyżej, dla bajtów tekstu | jak wyżej |
| `dekompresuj(blok: ref List[u8], limit: i64) -> Wynik` | `dane`: odtworzone bajty (kopia) | `BLEDNE_PARAMETRY` (limit lub blok poza 0..INT_MAX), `ZA_MALY_LIMIT` (+ `potrzebnyRozmiar`), `USZKODZONE_DANE`, `BRAK_PAMIECI` |
| `granicaKompresji(rozmiar: i64) -> i64` | największy możliwy rozmiar bloku (`LZ4_compressBound`) | −1 dla rozmiaru ujemnego lub ponad `LZ4_MAX_INPUT_SIZE` |
| `miesciSieWInt(n: i64) -> bool` | czy 0 ≤ n ≤ 2 147 483 647 | - |
| `zakoduj(dane) -> Wynik`, `odkoduj(kontener) -> Wynik` | format DKZ4 w pamięci | `BLAD_PLIKU` (nagłówek), `USZKODZONE_DANE` (blok, rozmiar, suma) |
| `spakujDoPliku(sciezka, dane) -> Wynik`, `rozpakujZPliku(sciezka) -> Wynik` | zapis i odczyt pliku DKZ4 | jak wyżej + `BLAD_PLIKU` przy błędach pliku |
| `czytajBajty(sciezka)`, `zapiszBajty(sciezka, dane)` | odczyt i zapis surowego pliku | `BLAD_PLIKU` |
| `bajtyTekstu(s)`, `tekstZBajtow(dane)`, `sumaKontrolna(dane)`, `wersjaBiblioteki()`, `nazwaRodzaju(rodzaj)` | pomocnicze | - |

`Wynik` ma pola:
* `ok` - czy operacja się udała;
* `rodzaj` - jedna ze stałych `OK`, `BLEDNE_PARAMETRY`, `ZA_MALY_LIMIT`, `USZKODZONE_DANE`, `BLAD_PLIKU`, `BRAK_PAMIECI`;
* `dane` - wynik, czyli kopia należąca do wywołującego;
* `blad` - opis po polsku;
* `potrzebnyRozmiar` - rzeczywisty rozmiar wyniku przy `ZA_MALY_LIMIT`.

Żadna funkcja nie przerywa programu.

## Środowisko i zależności

| Element | Wartość |
| --- | --- |
| Kompilator | `duckc 0.3-alpha (83882805a 2026-11-20)`, `commit-hash: 83882805a` |
| Uruchomienie | natywnie |
| System operacyjny | macOS 27.2, Apple Silicon (arm64) |
| Zależności | LZ4 1.10.0 (Homebrew, `/opt/homebrew/lib/liblz4.dylib`), Python 3 |

Wersja odniesienia z karty to LZ4 1.9.4. Użyte funkcje (`LZ4_compressBound`, `LZ4_compress_default`, `LZ4_decompress_safe`, `LZ4_versionString`) i stała `LZ4_MAX_INPUT_SIZE` mają w 1.10.0 te same deklaracje. Projekt sprawdziłem na 1.10.0. Na Linuksie x86_64 (Ubuntu 24.04, `liblz4.so.1` 1.9.4) go nie sprawdzałem.

## Struktura i działanie

| Plik | Odpowiedzialność |
| --- | --- |
| `app/lz4.dk` | wrapper: deklaracje C, kontrola rozmiarów, bufory, rodzaje błędów, format DKZ4, odczyt i zapis plików |
| `app/app.dk` | praktyczne zastosowanie: pakowacz plików DKZ4 ze sprawdzeniem wyniku |
| `przyklady/podstawowy.dk` | przykład podstawowy: kompresja, dekompresja, kontrolowane błędy, puste dane |
| `tests/testy.dk` | testy zwykłych danych, granic, błędów i formatu |

**Deklaracje C** są ręczne, według `lz4.h`:

| Funkcja C | Rola |
| --- | --- |
| `int LZ4_compressBound(int)` | granica rozmiaru wyniku |
| `int LZ4_compress_default(const char*, char*, int, int)` | kompresja |
| `int LZ4_decompress_safe(const char*, char*, int, int)` | dekompresja |
| `const char* LZ4_versionString(void)` | wersja biblioteki |

Do plików wrapper używa `fopen`, `fread`, `fwrite` i `fclose`.

**Kompresja:**
1. Sprawdzenie, czy rozmiar danych nie przekracza `LZ4_MAX_INPUT_SIZE`; ta granica jest mniejsza niż `INT_MAX`.
2. Obliczenie `LZ4_compressBound`, czyli największego możliwego wyniku.
3. Przydział bufora o tym rozmiarze.
4. Kompresja i skopiowanie dokładnie tylu bajtów, ile zwróciła biblioteka.

**Dekompresja:** limit musi mieścić się w `int` (0..2 147 483 647). Surowy blok nie zapisuje pierwotnego rozmiaru, dlatego limit jest obowiązkowy. `LZ4_decompress_safe` nigdy nie pisze poza bufor, ale zarówno za mały bufor, jak i uszkodzony blok zgłasza tą samą liczbą ujemną. Wrapper rozróżnia te przypadki drugą próbą z większym buforem: 255 × rozmiar bloku + 64, bo LZ4 rozwija dane najwyżej około 255 razy, a górna granica tej próby to 256 MiB.
* Gdy druga próba się uda, wynik to `ZA_MALY_LIMIT`, a `potrzebnyRozmiar` podaje rzeczywisty rozmiar.
* Gdy się nie uda, wynik to `USZKODZONE_DANE`.

Pusty blok (0 bajtów) jest uszkodzony: pusty wynik kompresji to blok jednobajtowy.

**Kto zwalnia pamięć:**
* Bufory C (`malloc`) przydziela i zwalnia wrapper wewnątrz każdego wywołania, także przy błędzie.
* Zwrócone `List[u8]` należą do wywołującego i nie zależą od pamięci biblioteki.
* `malloc` zawsze dostaje co najmniej 1 bajt, bo `malloc(0)` może zwrócić NULL.

**Format pliku DKZ4.** Wszystkie liczby są zapisane w kolejności little-endian:

| Bajty | Zawartość |
| --- | --- |
| 0-3 | znacznik `DKZ4` |
| 4 | wersja formatu: 1 |
| 5-8 | pierwotny rozmiar (u32) |
| 9-12 | rozmiar bloku (u32) |
| 13-16 | suma kontrolna FNV-1a pierwotnych danych (u32) |
| 17- | surowy blok LZ4 |

Przy odczycie wrapper sprawdza:
* znacznik i wersję;
* zgodność rozmiaru bloku z długością pliku;
* czy dane mieszczą się w zapisanym rozmiarze;
* rozmiar odtworzonych danych;
* sumę kontrolną.

Surowy blok LZ4 nie ma sumy kontrolnej, więc niektóre uszkodzenia mogłyby dać inne dane bez błędu. Suma FNV-1a w DKZ4 je wykrywa. Format ramki LZ4 (`LZ4F_*`) nie jest użyty, bo minimum dotyczy surowego bloku.

**Ograniczenia:**
* jeden blok na plik, najwyżej ok. 2 GiB danych;
* całe dane są w pamięci;
* brak strumieniowania i słownika;
* wariant DVM nie jest obsługiwany.

## Sprawdzenie projektu

```bash
python3 run.py --build build --target tests      # 30 sprawdzeń
python3 run.py --build build --target przyklad   # kod 0 = wszystkie oczekiwania przykładu spełnione
```

Testy sprawdzają:
* obieg kompresja → dekompresja bajt po bajcie dla: pustych danych, jednego bajtu NUL, NUL i 255 w środku danych, tekstu UTF-8, 1 MB danych powtarzalnych (blok poniżej 10 KB), 100 KB danych losowych (blok większy niż dane, ale w granicy) i 70 000 bajtów NUL;
* limit równy rozmiarowi, o 1 za mały i 0;
* limit ujemny i ponad `int`;
* granice `LZ4_compressBound` i `LZ4_MAX_INPUT_SIZE`;
* pusty, ucięty i błędny blok oraz literał dłuższy niż blok;
* format DKZ4: nagłówek, fałszywy rozmiar, zły znacznik, obcięty plik, zła suma, zapis i odczyt pliku, brakujący plik.

Scenariusze są w [tests/scenariusze.md](tests/scenariusze.md).

## Autorstwo i materiały

Kod i dokumentacja: Pixel of Software (Marcin Kowalczyk), z pomocą asystenta AI. Szczegóły są w [ZRODLA.md](ZRODLA.md).

Licencja własnego kodu: MIT (plik `LICENSE`). Biblioteka LZ4 (`lib/`) ma licencję BSD-2-Clause i nie jest dołączona do repozytorium; wrapper łączy się z nią dynamicznie.
