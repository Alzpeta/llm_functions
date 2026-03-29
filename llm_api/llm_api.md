# Velké jazykové modely - aplikační programové rozhraní

## Klient-server komunikace

Internetová komunikace funguje jako rozhovor mezi dvěma stranami: klientem a serverem.
Klient je ten, kdo posílá požadavek (například webový prohlížeč nebo mobilní aplikace), zatímco server 
na tento požadavek odpovídá (například vrácením webové stránky nebo dat z databáze).

Celý proces funguje na principu „dotaz → odpověď“. Klient nejprve odešle request a server reaguje až na základě tohoto požadavku. Základním jazykem této komunikace je protokol HTTP (nebo HTTPS), který dnes přenáší nejen hypertext, ale i další formáty dat, například JSON

<p>
  <img src="./clientxserver.png" style="width:400px; margin-left:15%;">
</p>

## API

API (Application Programming Interface) je rozhraní, přes které spolu komunikují dvě aplikace. Definuje pravidla, jak mají vypadat požadavky (requesty) a odpovědi (response).

API určuje:
  - kam se požadavky posílají (např. URL adresa),
  - jakým způsobem (např. HTTP metody),
  - v jakém formátu (např. JSON), 
  - jak bude vypadat odpověď.

Díky API server ví, co očekávat a jak na požadavek reagovat. Bez této definice by mezi aplikacemi nebyla možná spolehlivá komunikace.

<p>
  <img src="./api.png" style="width:400px; margin-left:15%;">
</p>

### JSON

Data mezi klientem a serverem se přenášejí v různých formátech, nejčastěji ve formátu JSON (JavaScript Object Notation).
JSON je strukturovaný formát, který umožňuje snadné čtení i zpracování dat jak pro člověka, tak pro program.

Používá se například pro přenos:

- uživatelských dat,
- výsledků z databází,
- konfigurací nebo stavů aplikace.

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "p_1023",
        "name": "Wireless Mouse",
        "price": {
          "amount": 499,
          "currency": "CZK"
        },
        "in_stock": true
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 2,
      "total_items": 25,
      "total_pages": 13
    }
  },
  "meta": {
    "request_id": "abc123xyz",
    "timestamp": "2026-03-29T12:00:00Z"
  }
}
```


## API Klíč

Existují dva základní typy API:

#### Veřejné API
Je dostupné bez autentizace. Typickým příkladem je načtení webové stránky – například zadání URL do prohlížeče.

#### Autentizované API
Vyžaduje ověření identity. Používá se v případech, kdy:

 - pracuje se s osobními daty,
 - používají se placené služby,
 - přistupuje se k chráněným zdrojům.

Autentizace se často řeší pomocí API klíče, což je unikátní řetězec znaků. Slouží k identifikaci aplikace, omezení přístupu, účtování a zabezpečení. API klíč není heslo, ale neměl by být veřejně sdílen.

```text
Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
## UI vs API
UI (User Interface) je rozhraní, přes které komunikuje uživatel s aplikací – například tlačítka, formuláře nebo obrazovky.

API je naopak rozhraní mezi aplikacemi.

Proces vypadá takto:

- Uživatel klikne na tlačítko (UI)
- Aplikace vytvoří request (klient)
- Request odešle přes API
- Server vrátí odpověď
- UI zobrazí výsledek

UI je tedy jen „obal“, který využívá API na pozadí.

## API a LLM
Aplikace využívající LLM mohou:

 1. být využívány k volání externích API (např. pro vyhledávání na webu),
 2. poskytovat vlastní API pro integraci do jiných aplikací.

Pro použití LLM API je potřeba znát:

- endpoint (kam jsou požadavky posílány),
- formát dat,
- strukturu odpovědi,
- API klíč.

<p>
  <img src="./llmapi.png" style="width:400px; margin-left:15%;">
</p>

## Přístup k LLM API z vlastní aplikace

### Obecný příklad - pseudokód
```json
POST https://api.llm-provider.com/generate

HEADERS:
  Authorization: Bearer API_KEY
  Content-Type: application/json

BODY:
  {
    "model": "nejaky-model-v1",

    "input": "
      INSTRUKCE:
      Jsi asistent pro zákaznický support.
      Rozhoduj pouze podle poskytnutého kontextu.
      Nikdy nevymýšlej informace.

      DATA (KONTEXT):
      - order_status: shipped
      - delivery_eta: 2 days
      - refund_policy: allowed_after_7_days

      DOTAZ:
      Objednávka mi nedorazila, chci vrátit peníze.

      VÝSTUP:
      Vrať JSON ve formátu:
      {
        category: string,
        action: string|null,
        reply: string
      }
    "
  }
```
----
### Role

- **system**  
  Nastavuje chování modelu (styl, pravidla, role).

- **user**  
  Vstup od uživatele (dotaz / instrukce).

- **assistant**  
  Předchozí odpovědi modelu (historie pro kontext).

| Role        | ChatGPT (OpenAI)                          | Claude (Anthropic)                     | Gemini (Google)                          | Mistral                                 | Grok                                    |
|-------------|-------------------------------------------|----------------------------------------|------------------------------------------|------------------------------------------|------------------------------------------|
| system      | {"role": "system", "content": "..."}     | "system": "..."                        | "systemInstruction": {"parts":[{"text":"..."}]} | {"role": "system", "content": "..."}     | {"role": "system", "content": "..."}     |
| user        | {"role": "user", "content": "..."}       | {"role": "user", "content": "..."}     | {"role": "user", "parts":[{"text":"..."}]}     | {"role": "user", "content": "..."}       | {"role": "user", "content": "..."}       |
| assistant   | {"role": "assistant", "content": "..."}  | {"role": "assistant", "content": "..."}| {"role": "model", "parts":[{"text":"..."}]}    | {"role": "assistant", "content": "..."}  | {"role": "assistant", "content": "..."}  |

----

### Ukázka parametrů

- **temperature**  
  Řídí náhodnost výstupu (0 = přesné, vyšší = kreativní).

- **top_p**  
  Omezuje výběr tokenů podle pravděpodobnosti (nižší = konzervativní).

- **max_tokens**  
  Maximální délka odpovědi.

- **stop**  
  Sekvence, při které se generování ukončí.

- **stream**  
  Odpověď přichází postupně (streamování).

| Parametr              | Popis                                      | ChatGPT (OpenAI) | Claude (Anthropic) | Gemini (Google)      | Mistral | Grok |
|----------------------|-------------------------------------------|------------------|--------------------|----------------------|---------|------|
| temperature          | Řídí náhodnost výstupu                    | temperature      | temperature        | temperature          | temperature | temperature |
| top_p                | Nucleus sampling (výběr pravděpodobností) | top_p            | top_p              | topP                 | top_p   | top_p |
| max_tokens           | Maximální délka odpovědi                  | max_tokens       | max_tokens         | maxOutputTokens      | max_tokens | max_tokens |
| stop                 | Sekvence, kde se generování zastaví       | stop             | stop_sequences     | stopSequences        | stop    | stop |
| stream               | Postupné streamování odpovědi             | stream           | stream             | stream               | stream  | stream |

### Streaming

Streaming znamená, že odpověď nepřijde najednou, ale postupně po částech (tokenech).

Výhody:

- rychlejší pocit odezvy,
- uživatel vidí odpověď průběžně.

Nevýhody:

- složitější zpracování,
- nutnost detekovat konec streamu.

Bez streamingu uživatel čeká na kompletní odpověď, což může působit pomaleji.

### Ukázky dotazů na API pro různé modely

#### ChatGPT (OpenAI)
```python
import requests

res = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers={
        "Authorization": "Bearer YOUR_OPENAI_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "model": "gpt-5.3",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "What is a black hole?"},
            {"role": "assistant", "content": "A region of spacetime with strong gravity."},
            {"role": "user", "content": "Explain briefly."}
        ]
    }
)

print(res.json())
```
#### Claude (Anthropic)
```python
import requests

res = requests.post(
    "https://api.anthropic.com/v1/messages",
    headers={
        "x-api-key": "YOUR_ANTHROPIC_API_KEY",
        "anthropic-version": "2023-06-01",
        "content-type": "application/json"
    },
    json={
        "model": "claude-3-opus-20240229",
        "system": "You are a helpful assistant.",
        "messages": [
            {"role": "user", "content": "What is a black hole?"},
            {"role": "assistant", "content": "A region of spacetime with strong gravity."},
            {"role": "user", "content": "Explain briefly."}
        ]
    }
)

print(res.json())
```
#### Claude (Anthropic)
```python
import requests

res = requests.post(
    "https://api.anthropic.com/v1/messages",
    headers={
        "x-api-key": "YOUR_ANTHROPIC_API_KEY",
        "anthropic-version": "2023-06-01",
        "content-type": "application/json"
    },
    json={
        "model": "claude-3-opus-20240229",
        "system": "You are a helpful assistant.",
        "messages": [
            {"role": "user", "content": "What is a black hole?"},
            {"role": "assistant", "content": "A region of spacetime with strong gravity."},
            {"role": "user", "content": "Explain briefly."}
        ]
    }
)

print(res.json())
```
#### Gemini (Google)
```python
import requests

res = requests.post(
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent?key=YOUR_GOOGLE_API_KEY",
    json={
        "contents": [
            {
                "role": "user",
                "parts": [{"text": "What is a black hole?"}]
            },
            {
                "role": "model",
                "parts": [{"text": "A region of spacetime with strong gravity."}]
            },
            {
                "role": "user",
                "parts": [{"text": "Explain briefly."}]
            }
        ],
        "systemInstruction": {
            "parts": [{"text": "You are a helpful assistant."}]
        }
    }
)

print(res.json())
```
#### Grok (xAI)
```python
import requests

res = requests.post(
    "https://api.x.ai/v1/chat/completions",
    headers={
        "Authorization": "Bearer YOUR_XAI_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "model": "grok-1",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "What is a black hole?"},
            {"role": "assistant", "content": "A region of spacetime with strong gravity."},
            {"role": "user", "content": "Explain briefly."}
        ]
    }
)

print(res.json())
```
#### Mistral
```python
import requests

res = requests.post(
    "https://api.mistral.ai/v1/chat/completions",
    headers={
        "Authorization": "Bearer YOUR_MISTRAL_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "model": "mistral-large-latest",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "What is a black hole?"},
            {"role": "assistant", "content": "A region of spacetime with strong gravity."},
            {"role": "user", "content": "Explain briefly."}
        ]
    }
)

print(res.json())
```
### Pricing + cost engineering

Platí se za počet tokenů (input + output)

```text
cena = (input_tokeny x input_price) + (output_tokeny x output_price)
```

Optimalizace:

- zkracovat kontext
- omezit max_tokens
- používat vhodný model

[pricing modelů](https://nicolalazzari.ai/articles/ai-api-pricing-comparison-2026)

[kalkulačka ceny]( https://llmpricingcalculator.com/)




### Bezpečnost

Důležité oblasti:

- ochrana API klíče
- validace vstupu
- validace výstupu
- ochrana proti prompt injection
#### Prompt injection
uživatel se snaží obejít instrukce modelu

Řešení:

- jasný system prompt
- omezení kontextu
- validace odpovědi