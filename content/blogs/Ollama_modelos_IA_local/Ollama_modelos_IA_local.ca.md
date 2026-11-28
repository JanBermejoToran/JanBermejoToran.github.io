+++
date = '2026-10-29T14:54:27+01:00'
draft = false
title = 'IA en local (1/3): fonaments per entendre els models'
summary = 'Què és Ollama, com funciona un LLM (tokens, paràmetres, quantització, formats), local vs. núvol i quin maquinari necessites per executar models de IA al teu propi ordinador.'
tags = ["LLM", "local", "ollama", "IA", "fonaments", "maquinari"]
+++

![Portada del post d'IA](/images/ollama/ollama_modelos_ia_local_imagenes/ollama_portada.svg)

Al post d'avui explicaré com executar models d'IA al nostre propi ordinador, sense dependre de ChatGPT ni de cap web o API, fent servir **Ollama**. És una guia pensada tant per als que volen provar-ho a casa com per a qui estigui valorant muntar-ho en un servidor i substituir ChatGPT o Claude per passar-se definitivament a la IA local.

Aquest és el primer d'una sèrie de tres posts:

1. **Fonaments** (aquest): Explicaré què és un model, què signifiquen els números i una explicació bàsica del tema del maquinari.
2. **Instal·lació i ús**: instal·lar Ollama, crear els teus models i fer servir l'API.
3. **Seguretat**: com protegir la teva IA local i quins riscos té.

Aquí no parlaré encara de com instal·lar res ni de com fer-la servir: primer anem a entendre què estem fent.

El post és sobretot teoria, però enmig hi ha una petita **🛠 pràctica** per veure amb els teus propis ulls com es mesuren els tokens i què és la quantització. Si encara no tens Ollama instal·lat, pots llegir el post sense fer-la i tornar-hi després de la instal·lació que explico al post 2.

---

## Índex

1. [Què és Ollama?](#què-és-ollama)
2. [Fonaments: com funcionen els LLM](#fonaments-com-funcionen-els-llm)
3. [Local vs. núvol](#local-vs-núvol)
4. [Maquinari](#maquinari-què-necessites)
5. [Conclusió](#conclusió)

---

## Què és Ollama?

Ollama és una eina de programari de codi obert que permet descarregar, gestionar i executar models oberts d'IA. Serveix per gestionar i executar models d'IA locals de manera senzilla.

Tot funciona a la nostra màquina, amb la qual cosa:

- No depenem d'internet un cop descarregat el model.
- Les nostres converses no surten del nostre ordinador.
- No hi ha límits d'ús ni subscripcions.

Per sota, Ollama fa servir el motor **llama.cpp** i s'encarrega de la part més pesada del procés: descarregar el model, repartir-lo entre GPU i CPU, mantenir-lo en memòria i exposar una API local.

![Logo llama.cpp](/images/ollama/ollama_modelos_ia_local_imagenes/llama_cpp_logo.png)

---

## Fonaments: com funcionen els LLM

### Tokens i finestra de context

Un LLM (*Large Language Model*) no llegeix lletres ni paraules, llegeix **tokens**: en resum, són trossos de text, que poden ser una paraula sencera, part d'una paraula o un simple signe de puntuació. Més o menys, en anglès un token són unes 0,75 paraules; en castellà i en català en surten una mica més per paraula, perquè els tokenitzadors s'entrenen sobretot amb anglès.

La **finestra de context** és el màxim de tokens que el model pot tenir "a la vista" alhora: la teva pregunta, l'historial de la conversa i la seva resposta. Si ens passem, oblida el més antic. I com explicaré al pròxim post, un context més gran consumeix més memòria.

### Què són els paràmetres?

Els paràmetres són els valors numèrics interns, vaja, els pesos de la xarxa neuronal, que el model ajusta durant el seu entrenament i on guarda tot el que ha après. Quan veus un model de "8B" vol dir que té 8 mil milions de paràmetres. Ull, la B ve de l'anglès *billion*, que equival a mil milions, no a un bilió.

Com més paràmetres, normalment millors i més coherents són les respostes, i el model pot fer tasques més complexes. Un model molt petit, per exemple de 0.5B, s'inventa més de la meitat de les coses que diu, sembla que estigui boig.

La contrapartida és que més paràmetres significa més consum de memòria i més lentitud.

### I la quantització?

Imagina una foto en màxima qualitat: pesa un munt. Si la deses com a JPG comprimit, ocupa molt menys i a simple vista gairebé no notes la diferència. Amb els models d'IA passa el mateix, i això s'anomena **quantització**.

Un model guarda tot el que sap en milers de milions de números (els paràmetres). Originalment cada número es guarda amb molta precisió, normalment 16 bits, i això fa que el model ocupi moltíssim. La quantització guarda aquests números amb menys precisió, per exemple 4 bits en lloc de 16. El resultat és un model força més petit a canvi d'una pèrdua de qualitat que, en la majoria de casos, gairebé no es nota.

Els models que descarrega Ollama per defecte ja vénen quantitzats a 4 bits. Per això un model de 8B cap en uns 5 GB i no en 16.

Aquests són els nivells més habituals:

| Quantització | Bits aprox. | Mida d'un 8B | Quan fer-la servir? |
|--------------|-------------|--------------|---------------------|
| Q4_K_M       | ~4,5        | ~5 GB        | L'opció per defecte: gairebé tota la qualitat ocupant poc |
| Q5_K_M       | ~5,5        | ~5,7 GB      | Una mica més de qualitat, si et sobra memòria |
| Q8_0         | ~8,5        | ~8,5 GB      | Gairebé idèntic a l'original; si necessites la màxima precisió |
| FP16         | 16          | ~16 GB       | Sense quantitzar; només per a recerca o ajust fi |

Les mides són orientatives. Es pot baixar de Q4 (Q3, Q2) per estalviar encara més memòria, però aquí sí que es nota, el model comença a respondre bastant pitjor.

**Quina escullo?** Si no saps quina, Q4_K_M. Si et sobra memòria i vols afinar, prova Q5 o Q8.

### Formats de model: GGUF i safetensors

Un model és, al final, un arxiu (o diversos) al teu disc, i pot venir en diferents formats. Els dos que et trobaràs:

- **GGUF**: un únic arxiu que ho conté tot (els pesos, la quantització i les dades del model). Està pensat per a llama.cpp i, per tant, per a Ollama. Si faràs servir IA en local, serà el que més veuràs.
- **safetensors**: el format que fan servir Hugging Face, PyTorch i servidors com vLLM. És segur, perquè no executa codi en carregar-se, i sol venir sense quantitzar o amb altres tipus de quantització (AWQ, GPTQ).

#### Definicions de Google

GGUF (GGML Unified Format) és un format d'arxiu binari dissenyat per emmagatzemar i executar models grans de llenguatge (LLM) en local.

![Definició GGUF](/images/ollama/ollama_modelos_ia_local_imagenes/definición_GGUF.png)

Safetensors és un format d'arxiu segur i ràpid per emmagatzemar els pesos (tensors) de models d'aprenentatge automàtic sense permetre la inserció de codi executable. Desenvolupat per Hugging Face, funciona com una alternativa més segura i ràpida als formats basats en pickle de Python (com .bin, .pt o .ckpt).

![Definició Safetensors](/images/ollama/ollama_modelos_ia_local_imagenes/definición_safetensors.png)

### Models densos i MoE

Hi ha dues maneres de construir un model, i canvia molt com es comporta:

- **Dens**: tots els paràmetres treballen en cada paraula que genera. És el més habitual.
- **MoE** (*Mixture of Experts*, barreja d'experts): el model està dividit en "experts" i, per a cada paraula, només s'activen uns quants. És com una empresa on, per a cada consulta, només es crida els especialistes necessaris i no tota la plantilla.

Per exemple, un MoE de 30B amb 3B actius necessita memòria per als 30B, però genera text a una velocitat semblant a la d'un model de 3B.


### Tipus de model

No tots els models serveixen per al mateix. Aquesta és una guia ràpida:

| Tipus | Per a què serveix? | Exemples de famílies |
|-------|--------------------|----------------------|
| Generalistes (instruct/chat) | Conversar, redactar i resumir | Llama, Qwen, Gemma, Mistral |
| De raonament | "Pensen" pas a pas abans de contestar; millors en lògica i matemàtiques, però més lents | DeepSeek-R1, Qwen3, gpt-oss |
| De codi | Completar, explicar i revisar codi | Qwen2.5-Coder |
| Multimodals | Entenen imatges a més de text | Llama 3.2 Vision, Gemma 3, LLaVA |
| D'embeddings | No escriuen text: converteixen text en números per cercar per significat (la base del RAG) | nomic-embed-text, bge-m3 |

Això canvia molt ràpid, així que mira sempre la [biblioteca d'Ollama](https://ollama.com/library) per veure el més recent.

### Al·lucinacions i límits reals d'un model local

Un LLM prediu la següent paraula més probable; no consulta una base de dades de fets. Per això **al·lucina**: genera respostes que sonen convincents però són falses. En un model petit passa més.

A més, un model local de 3B-8B no iguala els models més grans del núvol en raonament complex, i té un coneixement congelat a la seva data d'entrenament.

Com reduir el problema: baixar la `temperature`, donar-li documents fiables com a context (RAG), demanar-li que digui "no ho sé" quan no tingui informació i verificar sempre el que sigui important.


Per exemple, si descarrego el model més petit que he pogut trobar, que serà aquest de 135M de paràmetres, quan li pregunto qui és el president d'Espanya em contesta amb un nom aleatori d'algú que ni tan sols existeix a la vida real.

```bash
ollama run smollm:135m
```

![Al·lucinació en preguntar el nom del president d'Espanya](/images/ollama/ollama_modelos_ia_local_imagenes/preuba_alucinaciones.png)


### 🛠 Pràctica: tokens i quantització

**a) Veure tokens i velocitat.** Amb `--verbose`, Ollama mostra quants tokens ha processat i generat:

```bash
ollama run llama3.2:3b --verbose "Explícame qué es un puerto en una línea"
```

![Veure dades amb verbose](/images/ollama/ollama_modelos_ia_local_imagenes/Prueba_tokens_ollama.png)

Fixa't en aquests camps, que Ollama divideix en dues fases:

- **`prompt eval`** és la fase de **lectura**: el model llegeix la teva pregunta. `prompt eval count` són els tokens de la teva pregunta, en el meu cas 35.
- **`eval`** és la fase d'**escriptura**: el model genera la resposta. `eval count` són els tokens que ha escrit, que han estat 321, i `eval rate` és la seva velocitat real, en tokens per segon, 47 tokens per segon a la captura.
- **`load duration`** és el que triga a carregar el model a la memòria. Només és notable la primera vegada.
- **`total duration`** és la suma de tot.

De tots, el que més m'interessa ara és l'`eval rate`, que són els **tokens per segon** als quals escriu el model, i el que farem servir per comparar models més endavant.

Veuràs que el `prompt eval rate` surt molt baix. És normal en la primera execució, perquè el model encara s'està col·locant a la memòria. Si repeteixes l'ordre, ja carregat, la lectura serà molt més ràpida.

**b) Comparar quantitzacions.** Descarrega dues versions del mateix model, una en Q4 i una altra en Q8 (comprova els noms exactes dels tags a la pàgina del model a la [biblioteca d'Ollama](https://ollama.com/library)):

```bash
ollama pull llama3.2:3b-instruct-q4_K_M
ollama pull llama3.2:3b-instruct-q8_0
ollama list
```

Amb `ollama list` veuràs quant ocupa cadascuna. Fes la mateixa pregunta a les dues amb `--verbose` i compara mida, velocitat i qualitat de la resposta.

---

## Local vs. núvol

| Aspecte | Local (Ollama) | Núvol (API de pagament) |
|---------|----------------|-------------------------|
| Privacitat | Les dades no surten de la teva màquina | Les dades viatgen al proveïdor |
| Cost | Maquinari inicial, després gairebé gratis | Pagament per ús o subscripció |
| Latència | Sense xarxa, depèn del teu maquinari | Depèn de la xarxa i de la càrrega del servei |
| Disponibilitat | Funciona sense internet | Depèn del proveïdor |
| Control | Tries model, versió i paràmetres | El proveïdor pot canviar o retirar models |
| Qualitat màxima | Limitada pel teu maquinari | Accés als models més potents |

**Quan compensa el local:** dades sensibles, ús intensiu i continuat, necessitat de treballar sense connexió, control de versions del model, o aprenentatge.
**Quan no:** si necessites la màxima qualitat de raonament, tens un ús molt esporàdic o no tens el maquinari adequat.

### Ollama davant d'altres eines

| Eina | Enfocament | Ideal per a |
|------|------------|-------------|
| **Ollama** | CLI + API local, molt senzill | Començar ràpid, ús personal i prototips |
| **LM Studio** | Aplicació amb interfície gràfica | Qui prefereix finestres al terminal |
| **llama.cpp** | Motor de baix nivell, màxim control | Ajust fi del rendiment i maquinari poc comú |
| **vLLM** | Servidor d'alt rendiment (processa moltes peticions alhora) | Producció amb diversos usuaris i GPU potents |

---

## Maquinari: què necessites

El que més importa és la memòria de la gràfica, la **VRAM**. Si el model hi cap sencer, va ràpid. Si no hi cap, una part es reparteix a la RAM del sistema i a la CPU, i tot es torna bastant més lent. Funciona, però molt més a poc a poc.

### Quin model escollir segons el nostre maquinari?

Regla orientativa per a models quantitzats a 4 bits:

| Paràmetres | VRAM aproximada | RAM recomanada |
|------------|-----------------|----------------|
| 1B - 3B    | 2 - 3 GB        | 8 GB           |
| 7B - 8B    | 5 - 6 GB        | 16 GB          |
| 13B - 14B  | 9 - 10 GB       | 32 GB          |

Això és perquè el model vagi ràpid, però si vols pots executar models més grans a la teva màquina, l'únic inconvenient és que tindràs pocs tokens per segon.

Els valors són aproximats: el context, quant text recorda el model, també consumeix memòria. Al pròxim post veiem com calcular-ho amb precisió.

### Per tipus d'equip

| Equip | Què esperar |
|-------|-------------|
| Portàtil sense gràfica dedicada (8-16 GB RAM) | Models d'1B-3B; utilitzable però lent |
| Portàtil amb GPU de 4-8 GB | 3B-4B fluids; 7B-8B amb offloading parcial |
| Sobretaula amb GPU de 12-16 GB | 7B-14B còmodes |
| Sobretaula amb GPU de 24 GB | Fins a ~30B quantitzats |
| Mac amb Apple Silicon | La memòria és unificada: la RAM del Mac fa de VRAM, així que 32 GB o més permeten models grans |
| Servidor amb diverses GPU o molta RAM | Models de 70B o més |

### El meu cas

Un portàtil amb una NVIDIA GeForce RTX 2050 de 4 GB de VRAM. Un model de 8B no hi cap sencer a la gràfica, així que una part va a la RAM. Per a models ràpids amb una gràfica semblant, l'ideal és un de 3B o 4B, com `llama3.2:3b` o `qwen2.5:3b`. El cost són respostes una mica pitjors.

Encara que jo personalment faig servir un servidor dedicat amb una gràfica millor i RAM suficient per poder suportar models amb molts més paràmetres, això dependrà del maquinari, per això aquests posts els faré al meu portàtil i no en un servidor.

---

## Conclusió

Un model d'IA és, al final, un arxiu enorme de números, i tot el que hem vist, paràmetres, quantització, context, es resumeix en una idea: **quanta memòria tens decideix quin model pots fer servir amb soltesa**. Amb 4 GB de VRAM, com en el meu cas, el més raonable és un 3B: un 8B ja no hi cap sencer a la gràfica i una part passa a la RAM, que és més lenta.

Al pròxim post passem a la pràctica: instal·lem Ollama, creem els nostres propis models, fem servir l'API, muntem una interfície web i veiem com portar-ho a una empresa.

**Següent:** [IA en local (2/3): instal·lació i ús (properament)]()