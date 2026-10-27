# ksef-invoice-pdf

Wizualizator e-faktur z Krajowego Systemu e-Faktur: zamienia fakturę w XML (struktura FA(3))
na czytelny PDF. Ma też stronę, na którą wgrywasz plik i dostajesz PDF albo listę błędów.

> **Status:** w budowie. Postęp widać w [Issues](../../issues) i [Pull Requests](../../pulls).

## Zakres

- [ ] Walidacja XML względem oficjalnego schematu FA(3) (offline, z lokalną kopią schematów)
- [ ] XSLT → XSL-FO → PDF: sprzedawca, nabywca, pozycje, podsumowanie VAT według stawek
- [ ] Faktura korygująca (stan przed i po korekcie)
- [ ] Wiele stawek VAT, faktura w walucie obcej
- [ ] Strona HTML/CSS: wgraj XML → pobierz PDF
- [ ] Czytelne komunikaty błędów walidacji po polsku
- [ ] Testy na zestawie przykładowych faktur (poprawnych i błędnych)

## Technologie

Java 21 · Maven · XSD · XSLT 3.0 (Saxon-HE) · XSL-FO (Apache FOP) · XSpec · HTML/CSS · GitHub Actions

## Uruchomienie

Instrukcja pojawi się razem z pierwszą działającą wersją.

## Dane

Repozytorium zawiera wyłącznie przykładowe faktury z fikcyjnymi firmami.
