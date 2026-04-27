# Agentní systémy a AI workflows

## AI workflows
Sekvence předem definovaných kroků, kde AI modely transformují vstup na výstup.  
Deterministický proces řízený orchestrátorem (kód, logika, diagram).  

- pevně daný tok kroků (lineární nebo větvený)
- využití AI (např. klasifikace, extrakce)
- napojení na externí služby (API, databáze)
- sdílená paměť pro mezivýsledky
- opakovatelnost a kontrola procesu  

**Příklad:** extrakce dat z faktury  
PDF → OCR → LLM extrakce → validace → DB  
## AI agent
Systém, který samostatně plánuje a rozhoduje, jak dosáhnout cíle.  

- nedostává postup, jen cíl
- iteruje (loop)
- používá nástroje
- reaguje na prostředí  

**Typický cyklus:**
1. zadání úkolu
2. plánování
3. volání nástrojů
4. vyhodnocení
5. iterace  

ReAct pattern = reasoning + acting

Dobrý když
- Neznáme přesný postup řešení
- Kroky se můžou měnit
- Je potřeba rozhodovat na základě kontextu
- Automatizace s flexibilitou

Nevýhody:
- méně kontrolovatelný
- dražší (více volání LLM)
- hůře debugovatelný  
## Agentní systémy
Celý systém kolem agenta:  

- agent (logika)
- orchestrátor
- tools
- memory
- loop  

Agent = „mozek“  
Agentní systém = kompletní infrastruktura  

## Multiagentní systémy
Více agentů spolupracuje nebo soutěží na řešení problému.  

### Kdy použít
- rozdělitelný problém → MAS
- sekvenční problém → raději single agent  

### Nevýhody
- vyšší náklady
- složitá koordinace  

### Architektury
- **Independent** – paralelní agenti bez komunikace  
- **Decentralized** – agenti komunikují bez hierarchie  
- **Centralized** – hlavní agent deleguje úkoly  
- **Hierarchical** - agentni jsou uspořádány do hierarchické struktury
- **Hybrid** – kombinace přístupů  

### Komunikace
- message passing (přímé zprávy)
- shared memory (blackboard)
- nepřímá komunikace přes prostředí  

Standard: FIPA-ACL  

```json
(query-if
   :sender AgentA
   :receiver AgentB
   :content "(and(= (weather Prague) rain)(> (temperature Prague) 15))"
   :language SL
)
```

```json
(inform
   :sender AgentB
   :receiver AgentA
   :content "(= (temperature Prague) 20)"
   :language SL
)
```

## MCP
Model Context Protocol – standard komunikace mezi agentem a nástroji.  

- sjednocuje přístup k externím systémům
- odděluje „co agent chce“ od „jak se to provede“ 
- 
```json
{
  "name": "get_user",
  "description": "Get user by ID",
  "input_schema": {
    "type": "object",
    "properties": {
      "user_id": { "type": "string" }
    },
    "required": ["user_id"]
  }
}
```

```json
{
  "name": "get_user",
  "arguments": {
    "user_id": "123"
  }
}
```

https://modelcontextprotocol.io/

### Komponenty
- MCP client (v agentovi)
- MCP server (správa nástrojů)
- backend (API, DB, služby)  

### Funkce serveru
- poskytování nástrojů
- dodávání kontextu
- vykonávání akcí  

### Flow
1. user zadá úkol
2. LLM vybere tool:
3. orchestrátor volá MCP
4. MCP volá backend
5. výsledek zpět do LLM  

Výhody:
- abstrakce
- zaměnitelnost backendů
- plug & play nástroje  

## Tvorba AI workflows a agentích systémů

### Workflow nástroje
- n8n – node-based automatizace https://n8n.io/
- Make – pokročilé workflow (lepší pro neprogramátory) https://www.make.com/
- Zapier – jednoduché, lineární  https://zapier.com/

### Frameworky

**LangChain**
- workflow + agenti
- práce s memory a tools
- SDK  

```python
from langchain.chat_models import ChatOpenAI
from langchain.agents import initialize_agent, Tool

llm = ChatOpenAI()

def get_weather(city):
    return f"Počasí v {city}: 18°C"

tools = [
    Tool(name="weather", func=get_weather, description="Získá počasí")
]

agent = initialize_agent(tools, llm)

agent.run("Jaké je počasí v Praze?")
```

**LangGraph**
- explicitní orchestrace
- vhodné pro komplexní systémy  

```python
from langgraph.graph import StateGraph
from langchain.chat_models import ChatOpenAI
import json

llm = ChatOpenAI()

def get_weather(city):
    return f"Počasí v {city}: 18°C"

class State(dict): pass

def think(s):
    res = llm.invoke(f"""
User: {s['input']}
If weather → {{"action":"weather","city":"Praha"}}
Else → {{"action":"final","answer":"..."}}
""")
    return json.loads(res.content)

def act(s):
    if s["action"] == "weather":
        return {"answer": get_weather(s["city"])}

g = StateGraph(State)
g.add_node("think", think)
g.add_node("act", act)
g.set_entry_point("think")
g.add_edge("think","act")
g.add_edge("act","__end__")

app = g.compile()
print(app.invoke({"input":"Jaké je počasí v Praze?"}))
```

**AutoGen**
- multiagentní komunikace
- experimentální 
- volná konverzace, žádné pevné kroky

```python
from autogen import AssistantAgent, UserProxyAgent

assistant = AssistantAgent(
    name="assistant",
    llm_config={"model": "gpt-4"},
    system_message="You write Python code"
)

user = UserProxyAgent(
    name="user",
    code_execution_config={"work_dir": "./"}
)

user.initiate_chat(
    assistant,
    message="Napiš Python funkci na faktoriál a spusť ji pro 5"
)
```

**CrewAI**
- role-based agenti (team)
- automatická orchestrace  
- pevné tasky

```python
researcher = Agent(
    role="Researcher",
    goal="Najít relevantní informace",
    backstory="Jsi expert na vyhledávání informací"
)
writer = Agent(role="Writer", goal="Napsat článek")
task1 = Task(description="Najdi info o AI", agent=researcher)
task2 = Task(description="Napiš článek", agent=writer)
```

**Semantic Kernel**
- enterprise řešení
- vyšší stabilita a kontrola  

**OpenClaw**

https://github.com/openclaw/openclaw


## [Více](https://alzpeta.github.io/AIBR_Prednasky/agentni_systemy/agenticka-ai-frameworky.html)

## [Workflow terminologie](https://alzpeta.github.io/AIBR_Prednasky/agentni_systemy/workflow_terminology.html)

