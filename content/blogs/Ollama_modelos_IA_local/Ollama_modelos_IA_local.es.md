+++
date = '2026-10-29T14:54:27+01:00'
draft = false
title = 'IA en local (1/3): fundamentos para entender los modelos'
summary = 'Qué es Ollama, cómo funciona un LLM (tokens, parámetros, cuantización, formatos), local vs. nube y qué hardware necesitas para correr modelos de IA en tu propio ordenador.'
tags = ["LLM", "local", "ollama", "IA", "fundamentos", "hardware"]
+++

![Portada del post de IA](/images/ollama/ollama_modelos_ia_local_imagenes/ollama_portada.svg)

En el post de hoy voy a explicar cómo correr modelos de IA en nuestro propio ordenador, sin depender de ChatGPT ni de ninguna web o API, usando **Ollama**. Es una guía pensada tanto para los que quieren probarlo en casa como para quien esté valorando montarlo en un servidor y reemplazar ChatGPT o Claude para pasarse definitivamente a IA local.

Este es el primero de una serie de tres posts:

1. **Fundamentos** (este): Explicaré qué es un modelo, qué significan los números y una explicación básica del tema hardware.
2. **Instalación y uso**: instalar Ollama, crear tus modelos y usar la API.
3. **Seguridad**: cómo proteger tu IA local y qué riesgos tiene.

Aquí no voy a hablar de cómo instalar nada todavía ni cómo usarla: primero vamos a entender qué estamos haciendo.

El post es sobre todo teoría, pero en medio hay una pequeña **🛠 práctica** para ver con tus propios ojos cómo se miden los tokens y qué es la cuantización. Si aún no tienes Ollama instalado, puedes leer el post sin hacerla y volver a ella después de la instalación que explico en el post 2.

---

## Índice

1. [¿Qué es Ollama?](#qué-es-ollama)
2. [Fundamentos: cómo funcionan los LLM](#fundamentos-cómo-funcionan-los-llm)
3. [Local vs. nube](#local-vs-nube)
4. [Hardware](#hardware-qué-necesitas)
5. [Conclusión](#conclusión)

---

## ¿Qué es Ollama?

Ollama es una herramienta de software open source que permite descargar, gestionar y ejecutar modelos abiertos de IA. Sirve para gestionar y ejecutar modelos de IA locales de forma sencilla.

Todo corre en nuestra máquina, con lo cual:

- No dependemos de internet una vez descargado el modelo.
- Nuestras conversaciones no salen de nuestro ordenador.
- No hay límites de uso ni suscripciones.

Por debajo, Ollama usa el motor **llama.cpp** y se encarga de lo más pesado del proceso: descargar el modelo, repartirlo entre GPU y CPU, mantenerlo en memoria y exponer una API local.

![Logo LLaMAcpp](/images/ollama/ollama_modelos_ia_local_imagenes/llama_cpp_logo.png)

---

## Fundamentos: cómo funcionan los LLM

### Tokens y ventana de contexto

Un LLM (*Large Language Model*) no lee letras ni palabras, lee **tokens**: a resumidas cuentas son trozos de texto, que pueden ser una palabra entera, parte de una palabra o un simple signo de puntuación. Más o menos, en inglés un token son unas 0,75 palabras; en español salen algo más tokens por palabra, porque los tokenizadores se entrenan sobre todo con inglés.

La **ventana de contexto** es el máximo de tokens que el modelo puede tener "a la vista" a la vez: tu pregunta, el historial de la conversación y su respuesta. Si nos pasamos, olvida lo más antiguo. Y como explicaré en el siguiente post, un contexto mayor consume más memoria.

### ¿Qué son los parámetros?

Los parámetros son los valores numéricos internos, vamos, los pesos de la red neuronal, que el modelo ajusta durante su entrenamiento y donde guarda todo lo que ha aprendido. Cuando ves un modelo de "8B" significa que tiene 8 mil millones de parámetros. Ojo, la B viene del inglés *billion*, que equivale a mil millones en español, no a un billón.

Cuantos más parámetros, normalmente mejores y más coherentes son las respuestas, y el modelo puede hacer tareas más complejas. Un modelo muy pequeño, por ejemplo de 0.5B, se inventa más de la mitad de cosas que dice, parece que está loca.

La contrapartida es que más parámetros significa más consumo de memoria y más lentitud.

### ¿Y la cuantización?

Imagina una foto en máxima calidad: pesa un montón. Si la guardas como JPG comprimido, ocupa mucho menos y a simple vista casi no notas la diferencia. Con los modelos de IA pasa lo mismo, y a eso se le llama **cuantización**.

Un modelo guarda todo lo que sabe en miles de millones de números (los parámetros). Originalmente cada número se guarda con mucha precisión, normalmente 16 bits, y eso hace que el modelo ocupe muchísimo. La cuantización guarda esos números con menos precisión, por ejemplo 4 bits en lugar de 16. El resultado es un modelo bastante más pequeño a cambio de una pérdida de calidad que, en la mayoría de casos, apenas se nota.

Los modelos que descarga Ollama por defecto ya vienen cuantizados a 4 bits. Por eso un modelo de 8B cabe en unos 5 GB y no en 16.

Estos son los niveles más habituales:

| Cuantización | Bits aprox. | Tamaño de un 8B | ¿Cuándo usarla? |
|--------------|-------------|-----------------|-----------------|
| Q4_K_M       | ~4,5        | ~5 GB           | La opción por defecto: casi toda la calidad ocupando poco |
| Q5_K_M       | ~5,5        | ~5,7 GB         | Un poco más de calidad, si te sobra memoria |
| Q8_0         | ~8,5        | ~8,5 GB         | Casi idéntico al original; si necesitas la máxima precisión |
| FP16         | 16          | ~16 GB          | Sin cuantizar; solo para investigación o ajuste fino |

Los tamaños son orientativos. Se puede bajar de Q4 (Q3, Q2) para ahorrar aún más memoria, pero ahí sí se nota, el modelo empieza a responder bastante peor.

**¿Cuál elijo?** Si no sabes cuál, Q4_K_M. Si te sobra memoria y quieres afinar, prueba Q5 o Q8.

### Formatos de modelo: GGUF y safetensors

Un modelo es, al final, un archivo (o varios) en tu disco, y puede venir en distintos formatos. Los dos que te vas a encontrar:

- **GGUF**: un único archivo que lo contiene todo (los pesos, la cuantización y los datos del modelo). Está pensado para llama.cpp y, por tanto, para Ollama. Si vas a usar IA en local, será lo que más veas.
- **safetensors**: el formato que usan Hugging Face, PyTorch y servidores como vLLM. Es seguro, porque no ejecuta código al cargarse, y suele venir sin cuantizar o con otros tipos de cuantización (AWQ, GPTQ).

#### Definiciones de Google

GGUF (GGML Unified Format) es un formato de archivo binario diseñado para almacenar y ejecutar modelos grandes de lenguaje (LLM) en local.

![Definición GGUF](/images/ollama/ollama_modelos_ia_local_imagenes/definición_GGUF.png)

Safetensors es un formato de archivo seguro y rápido para almacenar los pesos (tensores) de modelos de aprendizaje automático sin permitir la inserción de código ejecutable. Desarrollado por Hugging Face, funciona como una alternativa más segura y veloz a los formatos basados en pickle de Python (como .bin, .pt o .ckpt).

![Definición Safetensors](/images/ollama/ollama_modelos_ia_local_imagenes/definición_safetensors.png)

### Modelos densos y MoE

Hay dos formas de construir un modelo, y cambia mucho cómo se comporta:

- **Denso**: todos los parámetros trabajan en cada palabra que genera. Es lo más habitual.
- **MoE** (*Mixture of Experts*, mezcla de expertos): el modelo está dividido en "expertos" y, para cada palabra, solo se activan unos pocos. Es como una empresa donde, para cada consulta, solo se llama a los especialistas necesarios y no a toda la plantilla.

Por ejemplo, un MoE de 30B con 3B activos necesita memoria para los 30B, pero genera texto a una velocidad parecida a la de un modelo de 3B.


### Tipos de modelo

No todos los modelos sirven para lo mismo. Esta es una guía rápida:

| Tipo | ¿Para qué sirve? | Ejemplos de familias |
|------|------------------|----------------------|
| Generalistas (instruct/chat) | Conversar, redactar y resumir | Llama, Qwen, Gemma, Mistral |
| De razonamiento | "Piensan" paso a paso antes de contestar; mejores en lógica y matemáticas, pero más lentos | DeepSeek-R1, Qwen3, gpt-oss |
| De código | Completar, explicar y revisar código | Qwen2.5-Coder |
| Multimodales | Entienden imágenes además de texto | Llama 3.2 Vision, Gemma 3, LLaVA |
| De embeddings | No escriben texto: convierten texto en números para buscar por significado (la base del RAG) | nomic-embed-text, bge-m3 |

Esto cambia muy rápido, así que mira siempre la [librería de Ollama](https://ollama.com/library) para ver lo más reciente.

### Alucinaciones y límites reales de un modelo local

Un LLM predice la siguiente palabra más probable; no consulta una base de datos de hechos. Por eso **alucina**: genera respuestas que suenan convincentes pero son falsas. En un modelo pequeño pasa más. 

Además, un modelo local de 3B-8B no iguala a los modelos más grandes de la nube en razonamiento complejo, y tiene un conocimiento congelado en su fecha de entrenamiento.

Cómo reducir el problema: bajar la `temperature`, darle documentos fiables como contexto (RAG), pedirle que diga "no lo sé" cuando no tenga información y verificar siempre lo importante.


Por ejemplo, si descargo el modelo más pequeño que he podido encontrar, será este de 135M de parámetros, el cual cuando le pregunto quién es el presidente de España me contesta con un nombre aleatorio de alguien que ni existe en la vida real.

```bash
ollama run smollm:135m
```

![Alucinación al preguntar el nombre del presidente de España](/images/ollama/ollama_modelos_ia_local_imagenes/preuba_alucinaciones.png)


### 🛠 Práctica: tokens y cuantización

**a) Ver tokens y velocidad.** Con `--verbose`, Ollama muestra cuántos tokens ha procesado y generado:

```bash
ollama run llama3.2:3b --verbose "Explícame qué es un puerto en una línea"
```

![Ver datos con verbose](/images/ollama/ollama_modelos_ia_local_imagenes/Prueba_tokens_ollama.png)

Fíjate en estos campos, que Ollama divide en dos fases:

- **`prompt eval`** es la fase de **lectura**: el modelo lee tu pregunta. `prompt eval count` son los tokens de tu pregunta, en mi caso 35.
- **`eval`** es la fase de **escritura**: el modelo genera la respuesta. `eval count` son los tokens que ha escrito, que han sido 321, y `eval rate` es su velocidad real, en tokens por segundo, 47 tokens por segundo en la captura.
- **`load duration`** es lo que tarda en cargar el modelo en memoria. Solo es notable la primera vez.
- **`total duration`** es la suma de todo.

De todos, el que más me interesa ahora es el `eval rate`, que son los **tokens por segundo** a los que escribe el modelo, y lo que usaremos para comparar modelos más adelante.

Verás que el `prompt eval rate` sale muy bajo. Es normal en la primera ejecución, porque el modelo aún se está colocando en memoria. Si repites el comando, ya cargado, la lectura será mucho más rápida.

**b) Comparar cuantizaciones.** Descarga dos versiones del mismo modelo, una en Q4 y otra en Q8 (comprueba los nombres exactos de los tags en la página del modelo en la [librería de Ollama](https://ollama.com/library)):

```bash
ollama pull llama3.2:3b-instruct-q4_K_M
ollama pull llama3.2:3b-instruct-q8_0
ollama list
```

Con `ollama list` verás cuánto ocupa cada una. Haz la misma pregunta a las dos con `--verbose` y compara tamaño, velocidad y calidad de la respuesta.

---

## Local vs. nube

| Aspecto | Local (Ollama) | Nube (API de pago) |
|---------|----------------|--------------------|
| Privacidad | Los datos no salen de tu máquina | Los datos viajan al proveedor |
| Coste | Hardware inicial, luego casi gratis | Pago por uso o suscripción |
| Latencia | Sin red, depende de tu hardware | Depende de la red y de la carga del servicio |
| Disponibilidad | Funciona sin internet | Depende del proveedor |
| Control | Eliges modelo, versión y parámetros | El proveedor puede cambiar o retirar modelos |
| Calidad máxima | Limitada por tu hardware | Acceso a los modelos más potentes |

**Cuándo compensa local:** datos sensibles, uso intensivo y continuo, necesidad de trabajar sin conexión, control de versiones del modelo, o aprendizaje.
**Cuándo no:** si necesitas la máxima calidad de razonamiento, tienes un uso muy esporádico o no tienes hardware adecuado.

### Ollama frente a otras herramientas

| Herramienta | Enfoque | Ideal para |
|-------------|---------|------------|
| **Ollama** | CLI + API local, muy sencillo | Empezar rápido, uso personal y prototipos |
| **LM Studio** | Aplicación con interfaz gráfica | Quien prefiere ventanas a terminal |
| **llama.cpp** | Motor de bajo nivel, máximo control | Ajuste fino de rendimiento y hardware poco común |
| **vLLM** | Servidor de alto rendimiento (procesa muchas peticiones a la vez) | Producción con varios usuarios y GPU potentes |

---

## Hardware: qué necesitas

Lo que más importa es la memoria de la gráfica, la **VRAM**. Si el modelo cabe entero en la VRAM, va rápido. Si no cabe, parte se reparte en la RAM del sistema y la CPU, y todo se vuelve bastante más lento. Funciona, pero mucho más despacio.

### ¿Qué modelo elegir según nuestro hardware?

Regla orientativa para modelos cuantizados a 4 bits:

| Parámetros | VRAM aproximada | RAM recomendada  |
|------------|-----------------|-------------------------------|
| 1B - 3B    | 2 - 3 GB        | 8 GB                          |
| 7B - 8B    | 5 - 6 GB        | 16 GB                         |
| 13B - 14B  | 9 - 10 GB       | 32 GB                         |

Esto es para que el modelo vaya rápido, pero si quieres puedes correr modelos más grandes en tu máquina, lo único malo es que tendrás pocos tokens por segundo.

Los valores son aproximados: el contexto, cuánto texto recuerda el modelo, también consume memoria. En el siguiente post vemos cómo calcularlo con precisión.

### Por tipo de equipo

| Equipo | Qué esperar |
|--------|-------------|
| Portátil sin gráfica dedicada (8-16 GB RAM) | Modelos de 1B-3B; usable pero lento |
| Portátil con GPU de 4-8 GB | 3B-4B fluidos; 7B-8B con offloading parcial |
| Sobremesa con GPU de 12-16 GB | 7B-14B cómodos |
| Sobremesa con GPU de 24 GB | Hasta ~30B cuantizados |
| Mac con Apple Silicon | La memoria es unificada: la RAM del Mac hace de VRAM, así que 32 GB o más permiten modelos grandes |
| Servidor con varias GPU o mucha RAM | Modelos de 70B o más |

### Mi caso

Un portátil con una NVIDIA GeForce RTX 2050 de 4 GB de VRAM. Un modelo de 8B no cabe entero en la gráfica, así que parte va a la RAM. Para modelos rápidos con una gráfica parecida, lo ideal es uno de 3B o 4B, como `llama3.2:3b` o `qwen2.5:3b`. El coste son respuestas algo peores.

Aunque yo personalmente uso un servidor dedicado con una gráfica mejor y suficiente RAM para poder soportar modelos de bastantes más parámetros, eso dependerá del hardware, por eso estos posts los voy a hacer en mi portátil y no en un servidor.

---

## Conclusión

Un modelo de IA es, al final, un archivo enorme de números, y todo lo que hemos visto, parámetros, cuantización, contexto, se resume en una idea: **cuánta memoria tienes decide qué modelo puedes usar con soltura**. Con 4 GB de VRAM, como en mi caso, lo razonable es un 3B: un 8B ya no cabe entero en la gráfica y parte pasa a la RAM, que es más lenta.

En el siguiente post pasamos a la práctica: instalamos Ollama, creamos nuestros propios modelos, usamos la API, montamos una interfaz web y vemos cómo llevarlo a una empresa.

**Siguiente:** [IA en local (2/3): instalación y uso (próximamente)]()