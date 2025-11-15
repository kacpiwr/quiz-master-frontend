# React Quiz Master

## Description

An interactive quiz application built with React and TypeScript. Users can take quizzes fetched from an API, check their results, and create new quizzes by providing data in JSON format.

The frontend application is designed to work with a FastAPI backend running on `http://localhost:8000`.

## Features

- **Browse Quizzes**: View a list of available quizzes.
- **Take a Quiz**: Start a quiz with randomized question and answer order.
- **Instant Feedback**: Get immediate feedback on your answers with explanations.
- **View Results**: Check your final score and accuracy percentage.
- **Create New Quizzes**: Add new quizzes using the built-in JSON editor.

## Requirements

### Backend
- Python 3.8+
- `pip`
- Running API server at `http://localhost:8000` with appropriate endpoints. Dependencies typically include:
  - `uvicorn`
  - `fastapi`
  - `sqlmodel`

### Frontend
- Modern web browser
- HTTP server to serve static files (e.g., `python -m http.server`)

## Setup and Running

### 1. Backend Setup

The backend must be running on `http://localhost:8000`.

1. **Clone the repository** (if applicable) and navigate to the backend directory.

2. **Create and activate a virtual environment** (recommended):
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```

3. **Install dependencies**:
    Create a `requirements.txt` file:
    ```txt
    fastapi
    uvicorn[standard]
    sqlmodel
    ```
    Then install the requirements:
    ```bash
    pip install -r requirements.txt
    ```

4. **Run the API server**:
    With the `main.py` file in place, start the server:
    ```bash
    uvicorn main:app --reload --port 8000
    ```
    The server should now be available at `http://localhost:8000`.

### Important Backend Configuration: CORS

If you encounter a `CORS policy` error in the browser console, it means the backend server needs to explicitly allow requests from your frontend application.

To fix this, **modify your `main.py` file** by adding `CORSMiddleware`. Below is the complete, corrected code for `main.py`:

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
# List of origins that are allowed to make requests to your API
origins = [
    "http://localhost",
    "http://localhost:8080",  # Frontend server address
    # You can add other addresses if needed
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],  # Allow all methods (GET, POST, etc.)
    allow_headers=["*"],  # Allow all headers
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
After making these changes and restarting the `uvicorn` server, the CORS error should be resolved.

### 2. Frontend Setup

The entire frontend code is contained in a single file: `index.html`. No build process is required.

1.  Place the `index.html` file in your chosen directory.

2.  **Start a local HTTP server** in that directory. The easiest way is to use Python's built-in server:
    ```bash
    # For Python 3.x
    python3 -m http.server 8080
    ```

3.  Open your browser and navigate to the server address (e.g., `http://localhost:8080`). The application should load.

## Using the Application

### Taking a Quiz
1.  When you open the application, you'll see a list of available quizzes.
2.  Click on a quiz to start it.
3.  Answer the questions by selecting an option and clicking "Submit".
4.  After each answer, you'll see if it was correct along with an explanation.
5.  Click "Next Question" to proceed.
6.  After completing the quiz, you'll see your final score.

### Creating a New Quiz
1.  On the home page, find the "Create a New Quiz" section and expand it.
2.  The text area contains a sample JSON structure for a quiz.
3.  Modify or paste your own JSON following this format. The JSON must include:
    - `name` (string): Quiz title.
    - `questions` (array of objects): List of questions.
      Each question object must contain:
      - `question` (string): The question text.
      - `options` (array of strings): List of possible answers.
      - `correct_answer` (string): The exact text of the correct answer.
      - `explanation` (string): Explanation for the answer.
4.  Click the "Create Quiz" button.
5.  If the JSON is valid, the quiz will be created and the quiz list will refresh.

---

## Polish Version (Polska Wersja)

[Polish documentation is available here](README_PL.md)
