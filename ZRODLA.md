# Autorstwo i źródła

## Wkład autorów

| Autor | Wykonane elementy | Lokalizacja w projekcie |
| --- | --- | --- |
| Marcin Kowalczyk (Pixel of Software) | wybór biblioteki, założenia, przegląd i akceptacja kodu, publikacja | całość |
| Asystent AI Claude (Anthropic) | wrapper, przykłady, testy, skrypty budowania, dokumentacja | `app/`, `przyklady/`, `tests/`, `build.py`, `run.py`, `README.md` |

## Wykorzystane materiały

| Materiał | Autor i źródło | Wersja lub data | Licencja | Sposób wykorzystania |
| --- | --- | --- | --- | --- |
| LZ4 | Yann Collet i współtwórcy, https://github.com/lz4/lz4 | 1.10.0 (odniesienie z karty: 1.9.4) | BSD-2-Clause dla `lib/` (https://github.com/lz4/lz4/blob/v1.9.4/lib/LICENSE) | biblioteka łączona dynamicznie, nie jest dołączona; deklaracje funkcji i stała `LZ4_MAX_INPUT_SIZE` przepisane ręcznie z `lz4.h` |
| `build.py`, `run.py`, `LICENSE` | Fundacja Duckling, szablon projektu z materiałów na start DuckQuest | 2026 | MIT (plik `LICENSE`) | skrypty rozbudowane o dołączanie LZ4 i cel `przyklad`; licencja z dopisanym autorem |
| Duckling (kompilator i biblioteka standardowa) | ducktype-org/duckling | 0.3-alpha, commit 83882805a | licencja Ducklinga | zależność budowania |

Suma kontrolna FNV-1a (32 bity) to publicznie opisany algorytm (Fowler, Noll, Vo); kod w Ducklingu napisano samodzielnie. Dane w testach i przykładach powstają w programach (tekst, wzorce, pseudolosowe bajty).

## Narzędzia AI

Kod wrappera, aplikacji, przykładu i testów, zmiany skryptów budowania oraz dokumentację przygotowano z pomocą asystenta AI Claude (Anthropic). Autor ustalił zakres i wybór biblioteki, przejrzał wyniki, sprawdził działanie (budowa, 30 testów, przykład, pakowacz) i odpowiada za oddaną wersję.

## Licencja własnego projektu

MIT, treść w pliku `LICENSE` (szablon Fundacji Duckling z dopisaną linią `Copyright (c) 2026 Pixel of Software (Marcin Kowalczyk)`). Licencja LZ4 (BSD-2-Clause) dotyczy samej biblioteki, instalowanej osobno.
