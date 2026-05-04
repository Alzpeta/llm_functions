# Evaluace LLM
- Evaluace = posouzení kvality výstupů jazykového modelu
- Cíl:
  - objektivně měřit, jak dobře model funguje

### Co hodnotíme
- správnost (pravdivost)
- relevance (odpovídá na otázku)
- srozumitelnost (dává smysl)
- užitečnost
- bezpečnost (halucinace, škodlivý obsah)

- Nejde jen o true/false. Často neexistuje jedna správná odpověď. Hodnotí se více dimenzí zároveň


## Automatická evaluace

### Princip
- Porovnání výstupu modelu s referencí
- Výpočet skóre bez zásahu člověka

Používá se:
- při tréninku (nepřímo – přes loss)
- při ladění modelu
- v produkci pro monitoring a A/B testy

---

### Cross-entropy (trénink)

Cross-entropy je loss funkce, kterou model minimalizuje během tréninku.

- model dává pravděpodobnosti jednotlivým tokenům
- porovnává se s tím, co je správně (ground truth)
- penalizuje se nízká pravděpodobnost správného tokenu

Platí:
- vysoká pravděpodobnost správného tokenu → nízký loss
- nízká pravděpodobnost správného tokenu → vysoký loss

model se učí minimalizovat tuto chybu

---

### Perplexity (PPL)

Perplexity je metrika odvozená z cross-entropy.


- Měří nejistotu modelu při predikci
- Nízká PPL = model si je jistý
- Vysoká PPL = model tápe

$$
PPL = \exp\left(-\frac{1}{N} \sum_{i=1}^{N} \log P(w_i)\right)
$$

#### Příklad

Věta: „kočka snědla myš“

Pravděpodobnosti správných tokenů:
- P(kočka) = 0.5  
- P(snědla) = 0.25  
- P(myš) = 0.2  

Výpočet:
- log(0.5) ≈ -0.69  
- log(0.25) ≈ -1.39  
- log(0.2) ≈ -1.61  

Součet = -3.69  
Průměr = -1.23  
PPL = e^{1.23} ≈ 3.4  

=> Model se chová, jako by vybíral z ~3–4 možností na token

#### Nevýhody
- Nechápe význam

---

### BLEU (Bilingual Evaluation Understudy)
Porovnání n-gramů (slovních posloupností)

#### Princip

- Text se rozdělí na n-gramy (posloupnosti tokenů)
- Porovnává se, kolik n-gramů z výstupu modelu se objevuje v referenci
- Počítá se **precision** (přesnost) pro různé délky n-gramů (typicky 1–4)

$$
BLEU = BP \cdot \exp\left( \sum_{n=1}^{4} w_n \log p_n \right)
$$

kde:
- \(p_n\) = precision pro n-gramy délky n
- \(w_n\) = váhy (typicky \(1/4\) pro každý n)
- \(BP\) = brevity penalty (penalizace krátkých vět)

#### Brevity Penalty (BP)
BP řeší problém, že model by mohl generovat krátké věty s vysokou precision.

$$
BP =
\begin{cases}
1 & \text{pokud } c > r \\
e^{(1 - r/c)} & \text{pokud } c \leq r
\end{cases}
$$

kde:
- \(c\) = délka generované věty
- \(r\) = délka referenční věty


Použití:
  - strojový překlad
  - generování textů

Vlastnosti:
  - precision-oriented
  - používá 1–4 gram
  - potřeba referenční odpovědi
  - geometrický průměr

Mechanismy:
  - clipping (zabraňuje opakování)
  - brevity penalty (trestá krátké odpovědi)

1.0 = perfektní shoda, 0.7+ velmi dobré, 0.3 - 0.5 použitelné, <2 špatné

#### Nevýhody
- Nechápe význam
- Penalizuje parafráze
- Jiná struktura věty je problém
- Nevhodné pro open-ended úlohy 
- Závisí na referencích

---

### ROUGE (Recall-Oriented Understudy for Gisting Evaluation)

Počítá **recall** - pokrytí

Použití: sumarizace - kolik relevantního obsahu z reference se objevilo ve výstupu.

##### Princip

- text se rozdělí na n-gramy (tokeny)
- porovnává se, kolik n-gramů z reference se objevuje ve výstupu
- důraz je na **pokrytí informací**, ne na přesnost

#### ROUGE-N (n-gramy)
$$
ROUGE\text{-}N = \frac{\text{počet sdílených n-gramů}}{\text{počet n-gramů v referenci}}
$$

Příklad

Reference:  
„kočka snědla k večeři myš“

Model 1:  
„kočka snědla myš“  
→ většina důležitých slov zachována → **vysoký ROUGE**

Model 2:  
„kočka jedla“  
→ chybí klíčové informace → **nízký ROUGE**

#### ROUGE-L (longest common subsequence)
Měří nejdelší společnou posloupnost tokenů ve správném pořadí

#### Rozdíl BLEU vs ROUGE
- BLEU → „nevymýšlej nic navíc“
- ROUGE → „nezapomeň nic důležitého“

#### Problémy ROUGE

- nechápe význam, jen porovnává slova
- penalizuje parafráze
- synonyma jsou problém
- jiná formulace je problém
- zvýhodňuje delší texty (více překryvů)
- přestože chceme krátké shrnutí

---

### Exact Match (EM)

Exact Match je nejjednodušší evaluační metrika, která kontroluje, zda je odpověď modelu přesně stejná jako referenční odpověď.

Vrací:
- 1 → pokud jsou odpovědi identické
- 0 → pokud se liší (byť jen minimálně)

#### Nevýhody
- Příliš přísné
- Ignoruje význam

#### Příklad

Reference:  
„Alžběta II.“

Model:  
„královna Alžběta 2“

EM = 0

#### Použití

- vhodné pro jednoduché úlohy:
  - question answering (např. SQuAD)
  - krátké faktické odpovědi (jméno, datum, číslo)

---

### F1 score
- Kombinace precision + recall
- Měří částečnou shodu
- Lepší než EM

$$
\text{precision} = \frac{\text{počet sdílených tokenů}}{\text{počet tokenů ve výstupu}}
$$

$$
\text{recall} = \frac{\text{počet sdílených tokenů}}{\text{počet tokenů v referenci}}
$$

$$
F1 = 2 \cdot \frac{\text{precision} \cdot \text{recall}}{\text{precision} + \text{recall}}
$$

#### Příklad

Reference:  
„Alžběta II.“

Model:  
„královna Alžběta 2“

Tokeny (zjednodušeně):
- reference: [Alžběta, II]
- model: [královna, Alžběta, 2]

Sdílený token:
- „Alžběta“

Výpočet:
- precision = 1 / 3
- recall = 1 / 2

$$
F1 = 2 \cdot \frac{(1/3) \cdot (1/2)}{(1/3 + 1/2)} \approx 0.4
$$

není to 0 jako u EM → zachytí částečnou shodu

---

## Embedding-based metriky

Embedding-based metriky hodnotí podobnost textů na základě **významu**, ne pouze shody slov. Text (slova nebo celé věty) se převádí do vektorového prostoru (embeddingů) a následně se měří jejich podobnost.

Nejčastěji se používá cosine similarity:
- 1 → velmi podobný význam
- 0 → nesouvisející
- -1 → opačný význam (v praxi spíš 0–1)

---

### Princip

- referenční věta a výstup modelu se převedou na embeddingy
- tyto embeddingy se porovnají pomocí cosine similarity
- čím vyšší podobnost, tím lepší skóre

---

### BERTScore

BERTScore je embedding-based metrika, která využívá model jako

- získá embedding pro každý token
- pro každý token hledá nejpodobnější token ve druhé větě
- počítá precision, recall a F1 v embedding prostoru

Výhoda:
- zachytí synonyma a parafráze

Nevýhoda:
- pomalejší, závislá na modelu

---

### Sentence embeddings

Místo tokenů se převede celá věta na jeden vektor.

- porovnává se jedna věta vs druhá
- výpočet je rychlý a jednoduchý

Nevýhody:
- ztrácí detail
- méně přesné než token-level metody

---

#### Výhody

- chápe význam textu
- robustní vůči parafrázím
- zvládá synonyma

---

#### Nevýhody

- neřeší pravdivost (faktickou správnost)
- závisí na použitém modelu (bias)
- výpočetně náročnější
- někdy příliš tolerantní

---
## Human evaluace

Human evaluace znamená hodnocení výstupů modelu skutečnými lidmi. Posuzuje se zejména:
- správnost
- srozumitelnost
- relevance
- užitečnost

Na rozdíl od automatických metrik dokáže člověk hodnotit význam, kontext i kvalitu odpovědi.

[evaluace pokus](evaluace_test.md)

---

### Základní metody

#### Rating
- hodnotitel dává celkové skóre (např. 1–5)
- jednoduché, ale subjektivní

#### Pairwise comparison
- porovnání odpovědí (např A vs B)
- vybírá se lepší
- konzistentnější než rating

#### Rubric-based evaluace
- hodnocení podle předem daných kritérií
- např.:
  - správnost
  - srozumitelnost
  - úplnost
- strukturovanější a méně subjektivní

#### Task-specific evaluace
- hodnocení podle konkrétního use-case 


### Nevýhody

- drahé
- pomalé
- subjektivní (různé preference hodnotitelů)


### Výhody

- chápe význam a kontext
- hodnotí pravdivost a užitečnost
- považováno za „zlatý standard“

---

## LLM as a Judge

### Princip
- Jiný LLM hodnotí odpovědi

### Výhody
- škálovatelné
- koreluje s human evaluací

### Problémy
- bias (preferuje vlastní styl)
- position bias
- length bias
- nekonzistence

### Řešení
- více průchodů
- randomizace pořadí
- lepší prompt
- kalibrace s lidmi

## Shrnutí
- Ideální je kombinace:
  - automatické metriky
  - human evaluace
  - LLM-as-a-judge
---

### Benchmarky

Benchmarky jsou standardizované testy pro porovnání modelů. Obsahují:
- dataset úloh
- referenční odpovědi
- metriky pro vyhodnocení

Používají se k tomu, aby všechny modely řešily stejné úlohy a daly se mezi sebou objektivně porovnat.

---

#### Příklady benchmarků

- GLUE  
  - jeden z prvních velkých benchmarků  
  - obsahuje různé NLP úlohy:
    - sentiment analysis
    - textual entailment
    - podobnost vět  

- SuperGLUE  
  - těžší verze GLUE  
  - náročnější úlohy na reasoning  

- MMLU  
  - multiple choice otázky  
  - pokrývá desítky oborů  
  - testuje znalosti a reasoning  

- BIG-bench  
  - stovky různých úloh  
  - logické hádanky, kreativní úlohy  
  - velmi heterogenní → výsledky se hůř interpretují  

- HELM (Holistic Evaluation of Language Models)  
  - komplexní benchmark  
  - nehodnotí jen výkon, ale i:
    - bias
    - fairness
    - robustnost  
  - snaží se o realističtější evaluaci  

- SQuAD  
  - benchmark pro question answering  
  - používá metriky jako Exact Match a F1  

---

#### Problémy benchmarků

- data leakage  
  - model se mohl benchmark „naučit“ během tréninku  
  - výsledky pak nejsou férové  

- overfitting na benchmarky  
  - model je optimalizovaný na konkrétní test  
  - ale nemusí fungovat v praxi  

- nereprezentují reálný svět  
  - v realitě často není jedna správná odpověď  

- optimalizace na skóre místo kvality  
  - model může „hrát hru na benchmark“  

#### Leaderboardy (žebříčky modelů)

- [BenchLM leaderboard](https://benchlm.ai/?utm_source=chatgpt.com)  
  - porovnává stovky modelů napříč benchmarky (coding, reasoning, knowledge)  

- [Artificial Analysis leaderboard](https://huggingface.co/spaces/ArtificialAnalysis/LLM-Performance-Leaderboard?utm_source=chatgpt.com)  
  - agregované výsledky modelů přes více benchmarků

- [LLM Stats benchmarks](https://llm-stats.com/benchmarks?utm_source=chatgpt.com)  
  - přehled benchmarků a výkonu modelů

- [Open LLM Leaderboard (HuggingFace)](https://huggingface.co/open-llm-leaderboard?utm_source=chatgpt.com)  
  - standardizované porovnání open-source modelů

- [LMArena leaderboard (Chatbot Arena)](https://huggingface.co/spaces/lmarena-ai/arena-leaderboard?utm_source=chatgpt.com)  
  - hodnocení modelů na základě lidských preferencí (A/B testy)


#### Další poznámky

- Humanity’s Last Exam  
  - spíš marketingový benchmark  
  - nepokrývá celý reálný svět  

- existují i oborově specifické benchmarky  
  - zaměřené na konkrétní use-cases
