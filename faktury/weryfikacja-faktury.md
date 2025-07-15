# Weryfikacja faktury

Faktura przesyłana do systemu KSeF podlega szeregowi kontroli technicznych i semantycznych. Weryfikacja obejmuje następujące kryteria:

## Zgodność ze schematem XSD
Faktura musi być przygotowana w formacie XML, kodowana w UTF-8 oraz zgodna z aktualnym schematem opublikowanym przez Ministerstwo Finansów.

## Unikalność faktury
- System wykrywa duplikaty na podstawie trzech pól:
  1. NIP sprzedawcy (`Podmiot1:NIP`)
  2. Rodzaj faktury (`RodzajFaktury`)
  3. Numer faktury (`P_2`)
- W przypadku duplikatu zwracany jest kod błędu 440 („Duplikat faktury”).

## Rozmiar pliku
- Maksymalny rozmiar faktury bez załączników: **1 MB**
- Maksymalny rozmiar faktury z załącznikami: **3 MB**

## Poprawne szyfrowanie
- Faktura powinna być zaszyfrowana algorytmem AES-256-CBC (klucz symetryczny 256 bit, IV 128 bit, PKCS#7).
- Klucz symetryczny szyfrowany algorytmem RSAES-OAEP (SHA-256/MGF1).

## Zgodność metadanych faktury w sesji interaktywnej
- Obliczenie i weryfikacja skrótu faktury wraz z rozmiarem pliku.
- Obliczenie i weryfikacja skrótu zaszyfrowanej faktury wraz z rozmiarem pliku