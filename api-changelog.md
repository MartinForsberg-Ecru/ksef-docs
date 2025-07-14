## Zmiany w API 2.0

Wersja 2.0.0 RC2
- **Nowe endpointy do zarządzania sesjami uwierzytelniania**  
  Umożliwiają przeglądanie oraz unieważnianie aktywnych sesji uwierzytelniających.  
  [Zarządzanie sesjami uwierzytelniania](uwierzytelnianie.md#zarządzanie-sesjami-uwierzytelniania)

- **Nowy endpoint do pobierania listy sesji wysyłki faktur**\
  `/sessions` – umożliwia pobranie metadanych dla sesji wysyłkowych (interaktywnych i wsadowych), z możliwością filtrowania m.in. po statusie, dacie zamknięcia i typie sesji.\
  [Pobieranie listy sesji](wysyłka-faktur/sesja-sprawdzenie-stanu-i-pobranie-upo.md#1-pobranie-listy-sesji)
  

- **Zmiana w listowaniu uprawnień**  
  `/permissions/query/authorizations/grants` – dodano typ zapytania (queryType) w filtrowaniu [uprawnień podmiotowych](uprawnienia.md#pobranie-listy-uprawnień-podmiotowych-do-obsługi-faktur).

- **Obsługa nowej wersji schematu faktury FA(3)**  
  W ramach otwierania sesji interaktywnej i wsadowej możliwy jest wybór schemy FA(3).

-  **Dodanie pola invoiceFileName w odpowiedzi dla sesji wsadowej**\
  `/sessions/{referenceNumber}/invoices` – dodano pole invoiceFileName zawierające nazwę pliku faktury. Pole występuje tylko dla sesji wsadowych.
   [Pobranie informacji na temat przesłanych faktur](wysyłka-faktur/sesja-sprawdzenie-stanu-i-pobranie-upo.md#3-pobranie-informacji-na-temat-przesłanych-faktur)