## Zmiany w API 2.0

### Wersja 2.0.0 RC3

- **Dodanie endpointu do pobierania listy metadanych faktur**  
  - `/invoices/query` (mock) zastąpiony przez `/invoices/query/metadata` – produkcyjny endpoint do pobierania metadanych faktur
  - Aktualizacja powiązanych modeli danych.

- **Aktualizacja mockowego endpointu `invoices/async-query` do inicjalizacji zapytania o pobranie faktur**
  - Zaktualizowano powiązane modele danych.

- **OpenAPI**
  - Dodano wymagane uprawnienia (`x-required-permissions`) do chronionych endpointów.
  - Dodano odpowiedzi `403 Forbidden` i `401 Unauthorized` w specyfikacji endpointów.
  - Dodawanie atrybutów ```required``` w odpowiedziach na zapytania o uprawnienia.
  - Aktualizacja opisu ```/api/v2/tokens```

- **Poprawka ContextIdentifier w AuthTokenRequest.xsd**  
  [Przygotowanie dokumentu XML](uwierzytelnianie.md#1-przygotowanie-dokumentu-xml-authtokenrequest)

- **Usunięcie endpointu do anonimowego pobierania faktury ```invoices/download```**  
  - Funkcjonalność dostępna tylko w narzędziu webowym do weryfikacji i pobierania faktur.


### Wersja 2.0.0 RC2
- **Nowe endpointy do zarządzania sesjami uwierzytelniania**  
  Umożliwiają przeglądanie oraz unieważnianie aktywnych sesji uwierzytelniających.  
  [Zarządzanie sesjami uwierzytelniania](auth/sesje.md)

- **Nowy endpoint do pobierania listy sesji wysyłki faktur**\
  `/sessions` – umożliwia pobranie metadanych dla sesji wysyłkowych (interaktywnych i wsadowych), z możliwością filtrowania m.in. po statusie, dacie zamknięcia i typie sesji.\
  [Pobieranie listy sesji](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#1-pobranie-listy-sesji)
  

- **Zmiana w listowaniu uprawnień**  
  `/permissions/query/authorizations/grants` – dodano typ zapytania (queryType) w filtrowaniu [uprawnień podmiotowych](uprawnienia.md#pobranie-listy-uprawnień-podmiotowych-do-obsługi-faktur).

- **Obsługa nowej wersji schematu faktury FA(3)**  
  W ramach otwierania sesji interaktywnej i wsadowej możliwy jest wybór schemy FA(3).

- **Dodanie pola invoiceFileName w odpowiedzi dla sesji wsadowej**\
  `/sessions/{referenceNumber}/invoices` – dodano pole invoiceFileName zawierające nazwę pliku faktury. Pole występuje tylko dla sesji wsadowych.
   [Pobranie informacji na temat przesłanych faktur](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo.md#3-pobranie-informacji-na-temat-przesłanych-faktur)