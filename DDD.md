# Domain Driven Design – Moduł Uwierzytelniania

## 1. Cel modułu
Moduł odpowiedzialny jest za proces uwierzytelniania użytkownika w systemie, generowanie tokenów dostępowych (access_token, refresh_token), ich odświeżanie oraz bezpieczne wylogowanie. Celem jest zapewnienie bezpiecznego i skalowalnego sposobu kontroli dostępu do zasobów API.

---

## 2. Część całkowita (ang. *total function / complete process*)
Część całkowita obejmuje pełny przebieg procesu uwierzytelniania od momentu, gdy użytkownik przesyła dane logowania, aż po otrzymanie tokenów uwierzytelniających:

1. Użytkownik przesyła login i hasło.
2. Backend weryfikuje dane w bazie danych.
3. Backend generuje krótkożyjący access_token oraz długotrwały refresh_token
4. Frontend przechowuje tokeny (access_token w pamięci, refresh w HTTP-only cookie).

To jest proces kompletny — zawsze kończy się sukcesem lub błędem

---

## 3. Część częściowa (ang. *partial function / partial process*)
Część częściowa opisuje procesy, które nie są wykonywane zawsze, tylko *gdy zajdzie potrzeba*:

### **3.1. Odświeżanie tokena**
Wykonywane tylko wtedy, gdy access_token utraci ważność.
System sprawdza refresh_token i generuje nowy access_token (opcjonalnie również nowy refresh_token).

### **3.2. Wylogowanie**
Wykonywane tylko na żądanie użytkownika.
Refresh_token jest unieważniany w bazie danych.

---

## 4. Diagram przepływu

```mermaid
sequenceDiagram
    participant U as User (Client)
    participant FE as Frontend
    participant BE as Backend (API)
    participant DB as Database

    Note over U,BE: 1. Logowanie użytkownika

    U->>FE: Wprowadza login + hasło
    FE->>BE: POST /auth/login (credentials)
    BE->>DB: Sprawdzenie użytkownika i hasła
    DB-->>BE: Dane użytkownika poprawne

    Note over BE,FE: 2. Generowanie tokenów

    BE-->>FE: access_token (JWT, krótki) + refresh_token (długi)

    FE->>U: Przechowuje access_token w pamięci, refresh w httpOnly cookie

    Note over FE,BE: 3. Dostęp do zasobów API

    FE->>BE: GET /api/resource (Authorization: Bearer access_token)
    BE-->>FE: Dane z API

    Note over FE,BE: 4. Access token wygasł

    FE->>BE: POST /auth/refresh (refresh_token)
    BE->>DB: Walidacja refresh tokena
    DB-->>BE: Refresh OK
    BE-->>FE: nowy access_token (+ opcjonalnie nowy refresh_token)

    Note over FE,BE: 5. Wylogowanie

    FE->>BE: POST /auth/logout (refresh_token)
    BE->>DB: Unieważnienie refresh tokena
    BE-->>FE: OK (wylogowano)
