# Email Classifier Agent 🤖

AI-powered email classification system using Reinforcement Learning concepts,
Claude API, Flask backend, and a dark industrial UI.

---

## File Structure

```
email_classifier/
│
├── app.py                    # Flask app — routes, SSE stream
│
├── agents/
│   ├── __init__.py
│   ├── classifier.py         # Claude API agent
│   └── environment.py        # RL environment (state, step, reset)
│
├── data/
│   ├── __init__.py
│   ├── emails.py             # Sample emails + ground truth
│   └── prompts.py            # Prompt templates + reward tables
│
├── utils/
│   ├── __init__.py
│   └── reward.py             # Reward calculation (RL concept)
│
├── templates/
│   └── index.html            # Jinja2 HTML template
│
├── static/
│   ├── css/style.css         # Full dark UI stylesheet
│   └── js/app.js             # SSE client + UI logic
│
├── .env.example              # Environment variables template
├── requirements.txt          # Python dependencies
└── README.md
```

---

## Setup

### 1. Clone / enter the project folder
```bash
cd email_classifier
```

### 2. Create virtual environment
```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Set your API key
```bash
cp .env.example .env
# Open .env and replace:  ANTHROPIC_API_KEY=your_api_key_here
```

### 5. Run the app
```bash
python app.py
```

### 6. Open browser
```
http://localhost:5000
```

---

## How It Works

### RL Concepts Used

| Concept              | Implementation                          |
|----------------------|-----------------------------------------|
| State                | `EmailEnvironment.get_state()`          |
| Observation Space    | Current email (subject + body)          |
| Action Space         | `{"label": "spam/work/personal"}`       |
| Reward Function      | `utils/reward.py` — correct = +1/-0.2  |
| Step Function        | `environment.step(action, level)`       |
| Episode              | All 5 emails processed → `done=True`   |
| Agent                | `EmailClassifierAgent` using Claude     |

### Difficulty Levels

| Level  | Prompt Output                        | Max Reward |
|--------|--------------------------------------|------------|
| Easy   | `{"label": "..."}`                   | +1.0       |
| Medium | `{"label": "...", "reason": "..."}`  | +0.7       |
| Hard   | `{category, priority, action, reason}` | +1.0     |

### Reward Design

**Easy**
- Correct category → +1.0
- Wrong category   → −0.2

**Medium**
- Correct category → +0.7
- Wrong category   → −0.2

**Hard (split reward)**
- Correct category → +0.5
- Correct priority → +0.3
- Correct action   → +0.2
- Wrong category   → −0.2

---

## API Endpoints

| Method | Route              | Description                      |
|--------|--------------------|----------------------------------|
| GET    | `/`                | Main UI                          |
| POST   | `/api/reset`       | Reset environment                |
| GET    | `/api/state`       | Current environment state (JSON) |
| GET    | `/api/rewards/<level>` | Reward table for level       |
| GET    | `/api/run?level=<level>` | SSE stream — run agent     |

---

## Agent Loop (Python)

```python
from agents.environment import EmailEnvironment
from agents.classifier  import EmailClassifierAgent

env   = EmailEnvironment()
agent = EmailClassifierAgent(api_key="...")

obs = env.reset()
while not env.done:
    action = agent.classify(obs, level="hard")
    result = env.step(action, level="hard")
    obs    = result["observation"]
    print(f"reward: {result['reward']} | total: {result['info']['total_reward']}")
```
