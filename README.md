# React Quiz Master

## Opis

Interaktywna aplikacja do quizów zbudowana w React i TypeScript. Użytkownicy mogą rozwiązywać quizy pobierane z API, sprawdzać swoje wyniki i tworzyć nowe quizy, dostarczając dane w formacie JSON.

Aplikacja frontendowa jest zaprojektowana do współpracy z backendem FastAPI działającym na `http://localhost:8000`.

## Funkcje

- **Przeglądaj quizy**: Zobacz listę dostępnych quizów.
- **Rozwiąż quiz**: Rozpocznij quiz z losową kolejnością pytań i odpowiedzi.
- **Natychmiastowa informacja zwrotna**: Otrzymuj informację o poprawności odpowiedzi i wyjaśnienia.
- **Zobacz wyniki**: Sprawdź swój wynik końcowy i procentową poprawność.
- **Twórz nowe quizy**: Dodawaj nowe quizy za pomocą wbudowanego edytora JSON.

## Wymagania

### Backend
- Python 3.8+
- `pip`
- Działający serwer API na `http://localhost:8000` z odpowiednimi endpointami. Zależności to zazwyczaj:
  - `uvicorn`
  - `fastapi`
  - `sqlmodel`

### Frontend
- Nowoczesna przeglądarka internetowa.
- Serwer HTTP do serwowania plików statycznych (np. `python -m http.server`).

## Uruchomienie

### 1. Uruchomienie Backendu

Backend musi działać na `http://localhost:8000`.

1.  **Sklonuj repozytorium** (jeśli dotyczy) i przejdź do folderu z backendem.

2.  **Utwórz i aktywuj wirtualne środowisko** (zalecane):
    ```bash
    python -m venv venv
    source venv/bin/activate  # Na Windows: venv\Scripts\activate
    ```

3.  **Zainstaluj zależności**:
    Utwórz plik `requirements.txt`:
    ```txt
    fastapi
    uvicorn[standard]
    sqlmodel
    ```
    A następnie zainstaluj:
    ```bash
    pip install -r requirements.txt
    ```

4.  **Uruchom serwer API**:
    Mając plik `main.py` z kodu, uruchom serwer:
    ```bash
    uvicorn main:app --reload --port 8000
    ```
    Serwer powinien być teraz dostępny pod adresem `http://localhost:8000`.

### Ważna konfiguracja Backendu: CORS

Jeśli w konsoli przeglądarki napotkasz błąd `CORS policy`, oznacza to, że serwer backendowy musi jawnie zezwolić na żądania z Twojej aplikacji frontendowej.

Aby to naprawić, **zmodyfikuj swój plik `main.py`**, dodając do niego `CORSMiddleware`. Poniżej znajduje się kompletny, poprawiony kod dla `main.py`:

```python
# main.py
from http.client import HTTPException

from fastapi import FastAPI, Depends
# KROK 1: Zaimportuj CORSMiddleware
from fastapi.middleware.cors import CORSMiddleware
from sqlmodel import Session, select
from typing import List

# Załóżmy, że te importy istnieją w Twoim projekcie
from database import engine, create_db_and_tables
from models import QuestionsSet, QuestionsSetCreate, QuestionsSetRead

# Inicjalizacja aplikacji FastAPI
app = FastAPI(title="Quiz API")

# KROK 2: Dodaj konfigurację CORS Middleware
# Lista originów, które mogą wysyłać żądania do Twojego API
origins = [
    "http://localhost",
    "http://localhost:8080",  # Adres serwera frontendu
    # Możesz dodać inne adresy, jeśli jest taka potrzeba
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],  # Zezwalaj na wszystkie metody (GET, POST, etc.)
    allow_headers=["*"],  # Zezwalaj na wszystkie nagłówki
)

@app.on_event("startup")
def on_startup():
    create_db_and_tables()

def get_session():
    with Session(engine) as session:
        yield session

@app.post("/quizes/", response_model=QuestionsSetRead)
def create_quiz(
        quiz: QuestionsSetCreate,
        session: Session = Depends(get_session)
):
    db_quiz = QuestionsSet.model_validate(quiz)
    session.add(db_quiz)
    session.commit()
    session.refresh(db_quiz)
    return db_quiz

@app.get("/quizes/", response_model=List[QuestionsSetRead])
def read_quizes(session: Session = Depends(get_session)):
    quizes = session.exec(select(QuestionsSet)).all()
    return quizes

@app.get("/quizes/{id}", response_model=QuestionsSetRead)
def quiz_by_id(id: int, session: Session = Depends(get_session)):
    quiz = session.get(QuestionsSet, id)
    if not quiz:
        raise HTTPException(status_code=404, detail="Quiz not found")
    return quiz

```
Po wprowadzeniu tych zmian i ponownym uruchomieniu serwera `uvicorn`, błąd CORS powinien zniknąć.

### 2. Uruchomienie Frontendu

Cały kod frontendu znajduje się w jednym pliku: `index.html`. Nie jest wymagany żaden proces budowania.

1.  Umieść plik `index.html` w wybranym folderze.

2.  **Uruchom lokalny serwer HTTP** w tym folderze. Najprościej jest użyć wbudowanego serwera Pythona:
    ```bash
    # Dla Pythona 3.x
    python3 -m http.server 8080
    ```

3.  Otwórz przeglądarkę i przejdź pod adres wskazany przez serwer (np. `http://localhost:8080`). Aplikacja powinna się załadować.

## Użytkowanie Aplikacji

### Rozwiązywanie Quizu
1.  Po otwarciu aplikacji zobaczysz listę dostępnych quizów.
2.  Kliknij na wybrany quiz, aby go rozpocząć.
3.  Odpowiadaj na pytania, wybierając jedną z opcji i klikając "Submit".
4.  Po każdej odpowiedzi zobaczysz, czy była poprawna, oraz wyjaśnienie.
5.  Kliknij "Next Question", aby przejść dalej.
6.  Po ukończeniu quizu zobaczysz swój wynik.

### Tworzenie Nowego Quizu
1.  Na stronie głównej znajdź sekcję "Create a New Quiz" i rozwiń ją.
2.  W polu tekstowym znajduje się przykładowa struktura JSON dla quizu.
3.  Zmodyfikuj lub wklej własny JSON zgodnie z formatem. JSON musi zawierać:
    - `name` (string): Tytuł quizu.
    - `questions` (array of objects): Lista pytań.
      Każdy obiekt pytania musi zawierać:
      - `question` (string): Treść pytania.
      - `options` (array of strings): Lista możliwych odpowiedzi.
      - `correct_answer` (string): Dokładna treść poprawnej odpowiedzi.
      - `explanation` (string): Wyjaśnienie.
4.  Kliknij przycisk "Create Quiz".
5.  Jeśli JSON jest poprawny, quiz zostanie utworzony, a lista quizów odświeżona.
