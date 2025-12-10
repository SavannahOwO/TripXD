# 🌏 TripXD — AI Trip Planner Backend
**Team: Passerby Fish**

TripXD is an AI-powered travel planning backend system.  
It receives a user's natural-language travel request, parses their intent, plans multi-day routes, estimates budgets, and integrates external travel APIs (POI, transport, hotel, passes, etc.).

This backend is designed to work with large language models (LLMs) such as GPT-OSS-120B running on AMD Instinct MI300X.

---

## 🚀 Features

### 🔍 1. NLP Trip Intent Parsing
- Understands free-form user requests in Chinese/English.
- Extracts destination, dates, budget, interests, and travelers count.
- Uses structured Pydantic models.

### 🗺 2. Multi-Day Route Planning
- Generates rough routes (day-by-day).
- Refines into detailed itineraries.
- Ensures time feasibility and geographic ordering.

### 💰 3. Budget Planning
- Splits budget across days.
- Applies heuristics and LLM-based reasoning.

### 🧭 4. POI, Transport, Accommodation APIs
- Wrapper services allow swapping actual external APIs.
- Designed for expandability and production use.

### 🔄 5. Orchestrator Flow Control
- Manages all agent calls.
- Handles session cache and trip context.
- Ensures consistency and resolves conflicts.

---

## 📁 Project Structure
```bash
trip_planner_backend/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI / entrypoint
│   │
│   ├── orchestrator/              # 🔧 只負責流程控制
│   │   ├── __init__.py
│   │   ├── orchestrator.py        # 主流程：call 各 agent + service
│   │   ├── context_manager.py     # state / session / cache 管理
│   │   └── state_models.py        # Pydantic models：TripState, DayPlan 等
│   │
│   ├── agents/                    # 🤖 所有跟 LLM 有關的邏輯
│   │   ├── __init__.py
│   │   ├── nlp_parser_agent.py    # 解析 user prompt → 結構化需求
│   │   ├── rough_route_agent.py   # 產生多天粗略路線
│   │   ├── budget_planner_agent.py# 依路線+偏好 → 預算分配
│   │   ├── daily_planner_agent.py # 單日細部行程（含工具使用）
│   │   ├── consistency_agent.py   # 最終檢查 / 自動修正
│   │   └── prompts/               # 各 agent 的 prompt
│   │       ├── __init__.py
│   │       ├── nlp_parser_prompt.txt
│   │       ├── rough_route_prompt.txt
│   │       ├── budget_planner_prompt.txt
│   │       ├── daily_planner_prompt.txt
│   │       └── consistency_prompt.txt
│   │
│   ├── services/                  # 🌐 各種外部 API 封裝
│   │   ├── __init__.py
│   │   ├── poi_service.py         # POI API wrapper
│   │   ├── route_service.py       # route_api（估兩點間時間）
│   │   ├── transport_service.py   # transport_api（查班次）
│   │   ├── accommodation_service.py# 住宿 API
│   │   └── pass_service.py        # 交通套票查詢/比價
│   │
│   └── persistence/               # 💾 資料存取
│       ├── __init__.py
│       ├── cache.py               # Redis / in‑memory cache 介面
│       └── db.py                  # 若之後有 Postgres 等

├── tests/
│   ├── __init__.py
│   ├── test_orchestrator.py
│   ├── test_agents.py
│   └── test_services.py
│
├── configs/
│   ├── settings.example.yaml      # 環境設定範本
│   └── logging.conf
│
├── requirements.txt
└── README.md
```
## 🛠 Installation

### 1. Clone the repo
```bash
git clone https://github.com/Passerby-Fish/TripXD.git
cd TripXD
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Setup environment variables
Create a `.env` file:
```env
# ---- LLM / MI300X ----
# 如果只連你自己的 Ollama / vLLM，就用 dummy-key 就好
OPENAI_API_KEY=dummy-key

# 這個是你現在本機的 LLM 端點
vllm_gpt_oss_120b_1="http://210.61.209.139:45014/v1/"
vllm_gpt_oss_120b_2="http://210.61.209.139:45005/v1/"
MI300X_ENDPOINT=http://localhost:11434/v1/

# ---- Google Maps ----
GOOGLE_MAPS_API_KEY=AIzaSyBvmR0Mt3DDL58su-AKtjCJfWET6qEt-bE.

# ---- RapidAPI ----
RAPIDAPI_KEY=557710edb6msh6e2b460b59db2cep16114ajsn062567bdf3d4
```

--- 

# **② Run the Server（如何啟動後端）**

你的 main.py 是 FastAPI → 所以要教使用者怎麼跑。

````markdown
## ▶️ Run the Server

```bash
uvicorn app.main:app --reload
```

API docs:
```
http://localhost:8000/docs
```

--- 

# **③ Example Usage（給使用者示範 API call）**

尤其你們是旅行規劃後端，最好給一個「使用示例」。

````markdown
## 📌 Example: Trip Request API

POST `/plan-trip`

```json
{
  "query": "我想 3/1 到 3/6 去東京玩，兩個人，預算 30000 台幣，想吃美食。"
}
```

Response example:

```json
{
  "destination": "東京",
  "start_date": "2025-03-01",
  "end_date": "2025-03-06",
  "travelers": 2,
  "rough_plan": [...],
  "daily_plan": [...],
  "budget": {...}
}
```

---

# **④ Tech Stack（技術棧）**

````markdown
## 🧰 Tech Stack

- **Python 3.10**
- **FastAPI** — backend framework  
- **Pydantic v2** — data modeling  
- **AMD MI300X + vLLM** — run GPT-OSS-120B  
- **Redis (optional)** — caching trip states  
- **Requests** — external API calls  
- **PyTest** — unit testing  

---

