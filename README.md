# Career Conversations - AI Personal Assistant

Interaktywny chatbot AI, który działa jako wirtualny asystent kariery, reprezentujący Szymona Maciejewskiego na stronie internetowej. Aplikacja wykorzystuje GPT-4 do odpowiadania na pytania dotyczące kariery, doświadczenia i umiejętności, bazując na profilu LinkedIn oraz dodatkowym podsumowaniu.

## 🎯 Funkcjonalności

- **Inteligentna rozmowa**: Bot odpowiada na pytania w imieniu właściciela, wykorzystując jego profil LinkedIn i dane biograficzne
- **Automatyczne zapisywanie kontaktów**: Rejestruje dane kontaktowe zainteresowanych osób
- **Śledzenie nieznanych pytań**: Zapisuje pytania, na które bot nie potrafił odpowiedzieć
- **Powiadomienia push**: Natychmiastowe powiadomienia przez Pushover o nowych kontaktach i pytaniach
- **Przyjazny interfejs**: Nowoczesny interfejs webowy zbudowany na Gradio
- **Context-aware**: Bot wykorzystuje pełny kontekst rozmowy do generowania spersonalizowanych odpowiedzi

## 📋 Wymagania

### Oprogramowanie
- Python 3.8 lub nowszy
- Konto OpenAI z aktywnym API Key
- Konto Pushover (opcjonalne, do powiadomień)

### Zależności Python
Wszystkie wymagane pakiety znajdują się w pliku `requirements.txt`:
```
requests
python-dotenv
gradio
pypdf
openai
openai-agents
```

## 🚀 Instalacja

### 1. Sklonuj repozytorium
```bash
git clone https://github.com/szymonMCS/career-chatbot.git
cd career-chatbot
```

### 2. Utwórz wirtualne środowisko
**Windows:**
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

**Linux/Mac:**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Zainstaluj zależności
```bash
pip install -r requirements.txt
```

### 4. Przygotuj pliki z danymi osobowymi

Utwórz katalog `me/` w głównym katalogu projektu:
```bash
mkdir me
```

Umieść w nim dwa pliki:
- **`profile.pdf`** - Twój profil LinkedIn wyeksportowany do PDF
- **`summary.txt`** - Krótkie podsumowanie Twojej kariery i osobowości (kodowanie UTF-8)

### 5. Konfiguracja zmiennych środowiskowych

Utwórz plik `.env` w głównym katalogu projektu:
```env
OPENAI_API_KEY=twój_klucz_api_openai
PUSHOVER_TOKEN=twój_token_pushover
PUSHOVER_USER=twój_user_pushover
```

**Gdzie uzyskać klucze:**
- **OPENAI_API_KEY**: [OpenAI Platform](https://platform.openai.com/api-keys)
- **PUSHOVER_TOKEN** i **PUSHOVER_USER**: [Pushover](https://pushover.net/)

## 💻 Użycie

### Uruchomienie lokalne
```bash
python app.py
```

Aplikacja uruchomi się domyślnie na `http://127.0.0.1:7860`

### Deployment na Hugging Face Spaces
```bash
gradio deploy
```

Postępuj zgodnie z instrukcjami interaktywnymi, aby wdrożyć aplikację na Hugging Face Spaces.

## 📁 Struktura projektu

```
career-chatbot/
├── app.py                  # Główny plik aplikacji
├── requirements.txt        # Zależności Python
├── .env                    # Zmienne środowiskowe (nie commitować!)
├── .gitignore             # Pliki ignorowane przez git
├── README.md              # Ten plik
└── me/                    # Dane osobowe (nie commitować!)
    ├── profile.pdf        # Profil LinkedIn w PDF
    └── summary.txt        # Podsumowanie kariery
```

## 🔧 Jak to działa

### Architektura aplikacji

1. **Inicjalizacja (`Me.__init__`)**:
   - Wczytuje profil LinkedIn z PDF
   - Wczytuje podsumowanie z pliku tekstowego
   - Inicjalizuje klienta OpenAI

2. **System Prompt (`system_prompt()`)**:
   - Generuje prompt systemowy dla GPT-4
   - Zawiera pełny kontekst: profil LinkedIn + podsumowanie
   - Instruuje model, aby działał jako właściciel profilu

3. **Obsługa rozmowy (`chat()`)**:
   - Przetwarza wiadomości użytkownika
   - Wywołuje API OpenAI z pełnym kontekstem
   - Obsługuje wywołania narzędzi (tools)

4. **Narzędzia AI (Tools)**:
   - **`record_user_details`**: Zapisuje email i dane kontaktowe
   - **`record_unknown_question`**: Rejestruje nieznane pytania

5. **Powiadomienia (`push()`)**:
   - Wysyła powiadomienia przez Pushover API
   - Informuje o nowych kontaktach i nieznanych pytaniach

### Schemat przepływu danych

```
Użytkownik → Gradio UI → chat() → OpenAI API → GPT-4 
                                        ↓
                                   Tool Calls
                                        ↓
                            record_user_details / 
                            record_unknown_question
                                        ↓
                                  Pushover API
```

## 🎨 Personalizacja

### Zmiana danych osobowych

W `app.py`, w klasie `Me.__init__()`:
```python
self.name = "Twoje Imię i Nazwisko"
```

### Modyfikacja promptu systemowego

Edytuj metodę `system_prompt()`, aby dostosować zachowanie bota:
```python
def system_prompt(self):
    system_prompt = f"You are acting as {self.name}..."
    # Dodaj własne instrukcje
    return system_prompt
```

### Zmiana modelu AI

W metodzie `chat()`:
```python
response = self.openai.chat.completions.create(
    model="gpt-4",  # lub "gpt-4-turbo", "gpt-3.5-turbo"
    messages=messages,
    tools=tools
)
```

### Dodanie nowych narzędzi (Tools)

1. Zdefiniuj funkcję narzędzia:
```python
def new_tool(parameter):
    # logika narzędzia
    return {"result": "ok"}
```

2. Dodaj definicję JSON dla OpenAI:
```python
new_tool_json = {
    "name": "new_tool",
    "description": "Opis narzędzia",
    "parameters": {
        "type": "object",
        "properties": {
            "parameter": {
                "type": "string",
                "description": "Opis parametru"
            }
        },
        "required": ["parameter"]
    }
}
```

3. Dodaj do listy `tools`:
```python
tools = [
    {"type": "function", "function": record_user_details_json},
    {"type": "function", "function": record_unknown_question_json},
    {"type": "function", "function": new_tool_json}
]
```

## 🔒 Bezpieczeństwo

### ⚠️ KRYTYCZNE - Nigdy nie commituj wrażliwych danych!

Upewnij się, że `.gitignore` zawiera:
```gitignore
# Zmienne środowiskowe
.env
.env.local

# Wirtualne środowisko
venv/
.venv/
env/

# Dane osobowe
me/profile.pdf
me/summary.txt

# Cache i tymczasowe pliki
__pycache__/
*.pyc
*.pyo
.ipynb_checkpoints/
.DS_Store
```

### Dobre praktyki:
- ✅ Regularnie rotuj klucze API
- ✅ Używaj zmiennych środowiskowych dla sekretów
- ✅ Ograniczaj uprawnienia kluczy API do minimum
- ✅ Monitoruj użycie API OpenAI
- ✅ Włącz powiadomienia o nietypowej aktywności

## 🐛 Rozwiązywanie problemów

### Bot nie uruchamia się

**Problem**: `ModuleNotFoundError`
```bash
# Sprawdź, czy środowisko jest aktywne
pip list

# Zainstaluj ponownie zależności
pip install -r requirements.txt
```

**Problem**: `FileNotFoundError: me/profile.pdf`
```bash
# Sprawdź strukturę katalogów
ls me/
# Lub na Windows:
dir me\

# Upewnij się, że pliki istnieją
```

### Błędy API OpenAI

**Problem**: `AuthenticationError`
- Sprawdź poprawność klucza API w `.env`
- Upewnij się, że plik `.env` jest w głównym katalogu
- Zweryfikuj, czy klucz nie wygasł

**Problem**: `RateLimitError`
- Sprawdź limity swojego konta OpenAI
- Rozważ upgrade planu
- Dodaj obsługę retry w kodzie

**Problem**: Model nie jest dostępny
```python
# Użyj dostępnego modelu
model="gpt-4o-mini"  # tańszy i szybszy
# lub
model="gpt-3.5-turbo"  # najtańszy
```

### Powiadomienia Pushover nie działają

**Problem**: Brak powiadomień
- Sprawdź poprawność tokenów w `.env`
- Zweryfikuj, czy aplikacja Pushover jest aktywna
- Sprawdź logi aplikacji: `print()` w funkcji `push()`

### Problemy z PDF

**Problem**: `PdfReadError`
- Sprawdź, czy PDF nie jest zaszyfrowany
- Upewnij się, że plik nie jest uszkodzony
- Spróbuj wyeksportować LinkedIn ponownie

## 📊 Monitoring i analityka

### Logowanie wydarzeń
Dodaj logowanie do pliku:
```python
import logging

logging.basicConfig(
    filename='chatbot.log',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def chat(self, message, history):
    logging.info(f"User message: {message}")
    # ... reszta kodu
```

### Śledzenie kosztów API
```python
def chat(self, message, history):
    response = self.openai.chat.completions.create(...)
    
    # Loguj użycie tokenów
    print(f"Tokens used: {response.usage.total_tokens}")
    print(f"Estimated cost: ${response.usage.total_tokens * 0.00001:.4f}")
```

## 🚢 Deployment

### Heroku
```bash
# Utwórz Procfile
echo "web: python app.py" > Procfile

# Deploy
heroku create career-chatbot
git push heroku main
```

### Docker
```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["python", "app.py"]
```

### Hugging Face Spaces
Już skonfigurowane! Użyj:
```bash
gradio deploy
```

## 🤝 Wkład w projekt

Pomysły na ulepszenia? Pull requesty są mile widziane!

1. Fork projektu
2. Utwórz branch funkcjonalności (`git checkout -b feature/amazing-feature`)
3. Commit zmian (`git commit -m 'Add amazing feature'`)
4. Push do brancha (`git push origin feature/amazing-feature`)
5. Otwórz Pull Request

## 📝 Licencja

Ten projekt jest dostarczany "jak jest", bez żadnych gwarancji.

## 👤 Autor

**Szymon Maciejewski**
- LinkedIn: [szymonmaciejewski365](https://www.linkedin.com/in/szymonmaciejewski365)
- Email: szymon8m@wp.pl
- GitHub: [@szymonMCS](https://github.com/szymonMCS)

## 🙏 Podziękowania

- OpenAI za GPT-4 API
- Gradio za framework interfejsu
- Pushover za system powiadomień
- Społeczność open source

---

**💡 Wskazówka**: Pamiętaj o dostosowaniu zawartości plików `me/profile.pdf` i `me/summary.txt` oraz zmiennych w `.env` do swoich potrzeb przed uruchomieniem aplikacji!

**🔗 Live Demo**: [https://huggingface.co/spaces/szymonMCS/career_conversations](https://huggingface.co/spaces/szymonMCS/career_conversations)
