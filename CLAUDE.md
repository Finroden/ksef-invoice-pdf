# ksef-invoice-pdf — wizualizator e-faktur KSeF

Zamienia ustrukturyzowaną fakturę z Krajowego Systemu e-Faktur (XML, struktura logiczna FA(3))
na czytelny PDF. Ma prostą stronę: wrzucasz plik XML, dostajesz PDF albo zrozumiały opis błędów.

Projekt portfolio. Pokazuje pracę z cudzym, złożonym, oficjalnym schematem XSD i aktualnym
polskim tematem. Jakość kodu, historia Gita i README są tak samo ważne jak działanie.

## Status

Start projektu: jest dokumentacja, nie ma jeszcze kodu. Pierwsze zadania:
1. Pobranie oficjalnego XSD FA(3) z zależnościami i walidacja offline przykładowej faktury.
2. Szkielet Maven + CLI: XML → XSLT → XSL-FO → PDF dla prostej faktury VAT.
3. Podsumowanie VAT według stawek, wiele stawek.
4. Faktura korygująca (stan przed / po), waluta obca.
5. Strona HTML do wgrywania pliku.

Aktualizuj tę sekcję w PR, który kończy etap.

## Stack

Ten sam rdzeń co w `correspondence-engine`, żeby oba projekty dało się porównać:

- **Java 21 + Maven**, **Saxon-HE** (XSLT 3.0), **Apache FOP** (PDF), **XSpec** (testy XSLT)
- Strona: statyczny HTML/CSS + mały serwer HTTP z JDK (`com.sun.net.httpserver`), bez frameworka
- **GitHub Actions**: CI

Przy dodawaniu zależności sprawdź na Maven Central aktualną stabilną wersję (nie zgaduj
z pamięci), a wybór zapisz w `docs/DECISIONS.md`. Każda nowa zależność wymaga mojej zgody.

## Komendy

Uzupełnij, gdy powstanie `pom.xml`. Docelowo:

```bash
mvn verify                                  # walidacja przykładów + XSpec + testy
java -jar target/<jar> render faktura.xml   # XML → PDF z linii poleceń
java -jar target/<jar> serve                # strona na http://localhost:8080
```

## Planowana struktura

```
schemas/fa3/       oficjalny XSD FA(3) i schematy, które importuje (kopie lokalne) + catalog.xml
templates/         XSLT → XSL-FO: układ faktury, tabela pozycji, podsumowanie VAT, korekta
config/            fop.xconf (fonty z polskimi znakami)
fonts/             fonty TTF z licencją
samples/valid/     przykładowe faktury, które muszą się wyrenderować
samples/invalid/   przykładowe błędne pliki + oczekiwany komunikat
examples/          wygenerowane PDF-y i zrzuty do README (commitowane celowo)
web/               strona HTML/CSS
src/               serwer, walidacja, tłumaczenie błędów XSD na komunikaty
docs/              DECISIONS.md, mapa pól FA(3) używanych w szablonie
```

## Zasady domeny

**Schemat i źródła**
- FA(3) obowiązuje od 1 lutego 2026 r. XSD bierzesz wyłącznie ze źródeł Ministerstwa Finansów
  (strona KSeF na podatki.gov.pl). Nie odtwarzaj go z pamięci, nie zgaduj przestrzeni nazw
  ani nazw pól.
- Do `docs/DECISIONS.md` zapisz: skąd pobrano XSD, datę pobrania i wersję.
- Schemat importuje inne schematy z adresów `crd.gov.pl`. Trzymaj ich kopie w repo i używaj
  katalogu XML, żeby walidacja i testy działały bez internetu.
- Znaczenie pól (np. pola podsumowania VAT, rodzaj faktury, kurs waluty, stan przed korektą)
  sprawdzasz w broszurze informacyjnej MF do FA(3) i zapisujesz w `docs/` mapę pól, których
  używa szablon. Przy niepewności pytasz, zamiast zgadywać.
- Wymagania dotyczące wizualizacji faktury poza KSeF (np. kody QR do weryfikacji) potwierdź
  w aktualnej dokumentacji MF, zanim je zaimplementujesz.
- Arkusz stylów jest pisany od zera. Nie kopiuj gotowych wizualizacji (MF ani innych projektów).

**Dane**
- Wyłącznie faktury przykładowe z fikcyjnymi firmami. NIP-y z poprawną sumą kontrolną, ale
  wygenerowane. Nigdy prawdziwych faktur ani danych kontrahentów.

**Bezpieczeństwo strony** (przyjmuje pliki od obcych osób)
- Parser XML z wyłączonym DTD i encjami zewnętrznymi (ochrona przed XXE).
- Limit rozmiaru pliku i czasu przetwarzania.
- Pliki przetwarzasz w pamięci i niczego nie zapisujesz ani nie logujesz z ich treści.

**Komunikaty błędów**
- Błąd walidacji XSD tłumaczysz na komunikat dla człowieka po polsku: które pole,
  w której linii, co jest nie tak. Surowy komunikat parsera tylko w szczegółach.
- Każdy plik w `samples/invalid/` ma test sprawdzający treść komunikatu.

**PDF**
- Polskie znaki wymagają osadzonych fontów w `fop.xconf` (Helvetica w FOP ich nie ma).
- Kwoty `1 234,56` z twardą spacją i walutą, daty `dd.MM.yyyy`.
- Długie tabele pozycji łamią się między stronami z powtarzanym nagłówkiem, numeracja „strona X z Y”.

## Konwencje kodu

- Nazwy w kodzie po angielsku, z wyjątkiem nazw pól FA(3), które zostają jak w schemacie.
  Komentarze, README, issues, commity i PR po polsku.
- Każdy nowy fragment szablonu = test XSpec. Każdy typ faktury = przykład w `samples/valid/`
  i PDF w `examples/`.
- Komentarze wyjaśniają *dlaczego*, nie *co*.

## Workflow (Git)

1. Gałąź domyślna: `main`. Nie pracuj na `main`. Gałąź na zadanie: `feat/…`, `fix/…`,
   `test/…`, `docs/…`, `chore/…`.
2. Każde zadanie wychodzi od issue (wymaganie biznesowe + kryteria akceptacji).
   Etykiety: `feature`, `bug`, `template`, `data`.
3. Przed kodem: krótki plan i czekaj na moje OK.
4. Commity w Conventional Commits z zakresem, po polsku, np. `feat(korekta): pozycje przed i po korekcie`.
5. Pull Request według szablonu, z `Closes #N` i zrzutem PDF przy zmianach w szablonie.
6. Merge, tagi i release robię ja. Ty przygotowujesz wpis w `CHANGELOG.md` („Unreleased”).

## Definicja „gotowe”

- `mvn verify` przechodzi: wszystkie `samples/valid/` są zgodne z XSD i renderują się,
  wszystkie `samples/invalid/` dają oczekiwane komunikaty,
- zmieniony PDF otwarty i obejrzany (polskie znaki, sumy, łamanie stron),
- przy zmianach w stronie: sprawdzone wgranie poprawnego i błędnego pliku,
- w odpowiedzi wypisane, czego nie dało się sprawdzić.

## Granice

- Nie dodawaj zależności bez pytania.
- Nie edytuj plików w `schemas/fa3/`. Aktualizacja schematu to osobne zadanie z nowym źródłem.
- Żadnych prawdziwych faktur ani danych firm.
- Nigdy: `git push --force`, przepisywanie historii, push na `main`, tworzenie tagów i release'ów.
