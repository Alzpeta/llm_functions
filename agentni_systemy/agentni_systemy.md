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

# stav
class State(dict):
    pass

def think(state):
    question = state["input"]
    # simulace LLM
    return {"action": "weather", "city": "Praha"}

def act(state):
    if state["action"] == "weather":
        return {"result": f"Počasí v {state['city']}: 18°C"}

def should_continue(state):
    return "result" not in state

# graph
builder = StateGraph(State)

builder.add_node("think", think)
builder.add_node("act", act)

builder.set_entry_point("think")

builder.add_edge("think", "act")
builder.add_conditional_edges(
    "act",
    should_continue,
    {
        True: "think",
        False: "__end__"
    }
)

graph = builder.compile()

graph.invoke({"input": "Jaké je počasí v Praze?"})
```

**AutoGen**
- multiagentní komunikace
- experimentální  

**CrewAI**
- role-based agenti (team)
- automatická orchestrace  

**Semantic Kernel**
- enterprise řešení
- vyšší stabilita a kontrola  

**OpenClaw**

https://github.com/openclaw/openclaw


## [Více](https://alzpeta.github.io/AIBR_Prednasky/agenti_systemy/agenticka-ai-frameworky.html)


