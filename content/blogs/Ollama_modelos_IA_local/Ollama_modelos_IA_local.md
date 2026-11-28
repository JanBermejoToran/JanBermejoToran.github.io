+++
date = '2026-10-29T14:54:27+01:00'
draft = false
title = 'Local AI (1/3): fundamentals to understand the models'
summary = 'What Ollama is, how an LLM works (tokens, parameters, quantization, formats), local vs. cloud and what hardware you need to run AI models on your own computer.'
tags = ["LLM", "local", "ollama", "AI", "fundamentals", "hardware"]
+++

![AI post cover](/images/ollama/ollama_modelos_ia_local_imagenes/ollama_portada.svg)

In today's post I'm going to explain how to run AI models on our own computer, without depending on ChatGPT or any website or API, using **Ollama**. It's a guide meant both for people who want to try it at home and for those considering setting it up on a server to replace ChatGPT or Claude and move over to local AI for good.

This is the first in a series of three posts:

1. **Fundamentals** (this one): I'll explain what a model is, what the numbers mean, and give a basic overview of the hardware side.
2. **Installation and use**: installing Ollama, creating your own models and using the API.
3. **Security**: how to protect your local AI and what risks it carries.

Here I'm not going to cover how to install anything yet, or how to use it: first let's understand what we're doing.

The post is mostly theory, but in the middle there's a small **🛠 practice** section so you can see for yourself how tokens are measured and what quantization is. If you don't have Ollama installed yet, you can read the post without doing it and come back to it after the installation that I explain in post 2.

---

## Index

1. [What is Ollama?](#what-is-ollama)
2. [Fundamentals: how LLMs work](#fundamentals-how-llms-work)
3. [Local vs. cloud](#local-vs-cloud)
4. [Hardware](#hardware-what-you-need)
5. [Conclusion](#conclusion)

---

## What is Ollama?

Ollama is an open source software tool that lets you download, manage and run open AI models. It's used to manage and run local AI models in a simple way.

Everything runs on our own machine, which means:

- We don't depend on the internet once the model is downloaded.
- Our conversations don't leave our computer.
- There are no usage limits or subscriptions.

Under the hood, Ollama uses the **llama.cpp** engine and takes care of the heaviest part of the process: downloading the model, splitting it between GPU and CPU, keeping it in memory and exposing a local API.

![llama.cpp logo](/images/ollama/ollama_modelos_ia_local_imagenes/llama_cpp_logo.png)

---

## Fundamentals: how LLMs work

### Tokens and context window

An LLM (*Large Language Model*) doesn't read letters or words, it reads **tokens**: in a nutshell, chunks of text, which can be a whole word, part of a word or just a punctuation mark. Roughly, in English a token is about 0.75 words; in Spanish you get somewhat more tokens per word, because tokenizers are trained mostly on English.

The **context window** is the maximum number of tokens the model can have "in view" at once: your question, the conversation history and its answer. If we go over it, the model forgets the oldest content. And as I'll explain in the next post, a larger context uses more memory.

### What are parameters?

Parameters are the internal numerical values, that is, the weights of the neural network, which the model adjusts during training and where it stores everything it has learned. When you see a model called "8B" it means it has 8 billion parameters. Careful: the B comes from the English word *billion*, which means a thousand million, not a trillion.

The more parameters, the better and more coherent the answers usually are, and the model can handle more complex tasks. A very small model, for example 0.5B, makes up more than half of what it says; it sounds like it has lost its mind.

The trade-off is that more parameters means more memory use and less speed.

### And what about quantization?

Picture a photo at maximum quality: it weighs a ton. If you save it as a compressed JPG, it takes up far less space and at a glance you barely notice the difference. The same thing happens with AI models, and it's called **quantization**.

A model stores everything it knows in billions of numbers (the parameters). Originally each number is stored with a lot of precision, normally 16 bits, and that makes the model huge. Quantization stores those numbers with less precision, for example 4 bits instead of 16. The result is a much smaller model in exchange for a loss of quality that, in most cases, you barely notice.

The models Ollama downloads by default are already quantized to 4 bits. That's why an 8B model fits in about 5 GB and not 16.

These are the most common levels:

| Quantization | Approx. bits | Size of an 8B | When to use it |
|--------------|--------------|---------------|----------------|
| Q4_K_M       | ~4.5         | ~5 GB         | The default option: almost all the quality while taking up little space |
| Q5_K_M       | ~5.5         | ~5.7 GB       | A bit more quality, if you have memory to spare |
| Q8_0         | ~8.5         | ~8.5 GB       | Almost identical to the original; if you need maximum precision |
| FP16         | 16           | ~16 GB        | Not quantized; only for research or fine-tuning |

Sizes are approximate. You can go below Q4 (Q3, Q2) to save even more memory, but there you do notice it: the model starts answering noticeably worse.

**Which one should I pick?** If you don't know, Q4_K_M. If you have memory to spare and want to fine-tune, try Q5 or Q8.

### Model formats: GGUF and safetensors

A model is, in the end, a file (or several) on your disk, and it can come in different formats. The two you'll run into:

- **GGUF**: a single file that contains everything (the weights, the quantization and the model's metadata). It's designed for llama.cpp and therefore for Ollama. If you're going to use AI locally, this is what you'll see the most.
- **safetensors**: the format used by Hugging Face, PyTorch and servers like vLLM. It's safe, because it doesn't execute code when loaded, and it usually comes unquantized or with other types of quantization (AWQ, GPTQ).

#### Google definitions

GGUF (GGML Unified Format) is a binary file format designed to store and run large language models (LLMs) locally.

![GGUF definition](/images/ollama/ollama_modelos_ia_local_imagenes/definición_GGUF.png)

Safetensors is a safe and fast file format for storing the weights (tensors) of machine learning models without allowing executable code to be embedded. Developed by Hugging Face, it works as a safer and faster alternative to Python pickle-based formats (such as .bin, .pt or .ckpt).

![Safetensors definition](/images/ollama/ollama_modelos_ia_local_imagenes/definición_safetensors.png)

### Dense models and MoE

There are two ways to build a model, and it changes a lot how it behaves:

- **Dense**: all the parameters work on every word it generates. This is the most common kind.
- **MoE** (*Mixture of Experts*): the model is split into "experts" and, for each word, only a few of them are activated. It's like a company where, for each query, only the necessary specialists are called in and not the whole staff.

For example, a 30B MoE with 3B active needs memory for all 30B, but generates text at a speed similar to that of a 3B model.


### Types of models

Not all models are good for the same things. Here's a quick guide:

| Type | What is it for? | Example families |
|------|-----------------|------------------|
| General-purpose (instruct/chat) | Chatting, writing and summarizing | Llama, Qwen, Gemma, Mistral |
| Reasoning | They "think" step by step before answering; better at logic and maths, but slower | DeepSeek-R1, Qwen3, gpt-oss |
| Code | Completing, explaining and reviewing code | Qwen2.5-Coder |
| Multimodal | They understand images as well as text | Llama 3.2 Vision, Gemma 3, LLaVA |
| Embeddings | They don't write text: they turn text into numbers to search by meaning (the basis of RAG) | nomic-embed-text, bge-m3 |

This changes very fast, so always check the [Ollama library](https://ollama.com/library) for the latest.

### Hallucinations and the real limits of a local model

An LLM predicts the most likely next word; it doesn't look things up in a database of facts. That's why it **hallucinates**: it produces answers that sound convincing but are false. It happens more with a small model.

Also, a local 3B-8B model doesn't match the biggest cloud models in complex reasoning, and its knowledge is frozen at its training date.

How to reduce the problem: lower the `temperature`, give it reliable documents as context (RAG), ask it to say "I don't know" when it has no information, and always verify what matters.


For example, if I download the smallest model I could find, this 135M-parameter one, when I ask it who the president of Spain is it answers with a random name of someone who doesn't even exist in real life.

```bash
ollama run smollm:135m
```

![Hallucination when asking for the name of the president of Spain](/images/ollama/ollama_modelos_ia_local_imagenes/preuba_alucinaciones.png)


### 🛠 Practice: tokens and quantization

**a) See tokens and speed.** With `--verbose`, Ollama shows how many tokens it has processed and generated:

```bash
ollama run llama3.2:3b --verbose "Explícame qué es un puerto en una línea"
```

![Viewing data with verbose](/images/ollama/ollama_modelos_ia_local_imagenes/Prueba_tokens_ollama.png)

Look at these fields, which Ollama splits into two phases:

- **`prompt eval`** is the **reading** phase: the model reads your question. `prompt eval count` is the number of tokens in your question, 35 in my case.
- **`eval`** is the **writing** phase: the model generates the answer. `eval count` is the number of tokens it wrote, which was 321, and `eval rate` is its real speed, in tokens per second, 47 tokens per second in the screenshot.
- **`load duration`** is the time it takes to load the model into memory. It's only noticeable the first time.
- **`total duration`** is the sum of everything.

Of all of them, the one I care about most right now is `eval rate`, which is the **tokens per second** at which the model writes, and what we'll use to compare models later on.

You'll see that `prompt eval rate` comes out very low. That's normal on the first run, because the model is still settling into memory. If you repeat the command, once it's already loaded, reading will be much faster.

**b) Compare quantizations.** Download two versions of the same model, one in Q4 and one in Q8 (check the exact tag names on the model's page in the [Ollama library](https://ollama.com/library)):

```bash
ollama pull llama3.2:3b-instruct-q4_K_M
ollama pull llama3.2:3b-instruct-q8_0
ollama list
```

With `ollama list` you'll see how much space each one takes. Ask both the same question with `--verbose` and compare size, speed and answer quality.

---

## Local vs. cloud

| Aspect | Local (Ollama) | Cloud (paid API) |
|--------|----------------|------------------|
| Privacy | Data doesn't leave your machine | Data travels to the provider |
| Cost | Upfront hardware, then almost free | Pay per use or subscription |
| Latency | No network, depends on your hardware | Depends on the network and the service load |
| Availability | Works without internet | Depends on the provider |
| Control | You choose model, version and parameters | The provider can change or withdraw models |
| Maximum quality | Limited by your hardware | Access to the most powerful models |

**When local pays off:** sensitive data, heavy and continuous use, the need to work offline, control over model versions, or learning.
**When it doesn't:** if you need the best possible reasoning quality, you only use it occasionally or you don't have suitable hardware.

### Ollama compared with other tools

| Tool | Approach | Ideal for |
|------|----------|-----------|
| **Ollama** | CLI + local API, very simple | Getting started quickly, personal use and prototypes |
| **LM Studio** | Application with a graphical interface | People who prefer windows to a terminal |
| **llama.cpp** | Low-level engine, maximum control | Fine-tuning performance and unusual hardware |
| **vLLM** | High-performance server (handles many requests at once) | Production with several users and powerful GPUs |

---

## Hardware: what you need

What matters most is the graphics card memory, the **VRAM**. If the model fits entirely in VRAM, it runs fast. If it doesn't fit, part of it is spread across the system RAM and the CPU, and everything becomes quite a bit slower. It works, but much more slowly.

### Which model should we choose for our hardware?

Rule of thumb for models quantized to 4 bits:

| Parameters | Approx. VRAM | Recommended RAM |
|------------|--------------|-----------------|
| 1B - 3B    | 2 - 3 GB     | 8 GB            |
| 7B - 8B    | 5 - 6 GB     | 16 GB           |
| 13B - 14B  | 9 - 10 GB    | 32 GB           |

This is so the model runs fast, but if you want you can run larger models on your machine; the only downside is that you'll get few tokens per second.

The values are approximate: the context, how much text the model remembers, also uses memory. In the next post we'll see how to calculate it precisely.

### By type of machine

| Machine | What to expect |
|---------|----------------|
| Laptop without a dedicated graphics card (8-16 GB RAM) | 1B-3B models; usable but slow |
| Laptop with a 4-8 GB GPU | 3B-4B run smoothly; 7B-8B with partial offloading |
| Desktop with a 12-16 GB GPU | 7B-14B comfortably |
| Desktop with a 24 GB GPU | Up to ~30B quantized |
| Mac with Apple Silicon | Memory is unified: the Mac's RAM acts as VRAM, so 32 GB or more allows large models |
| Server with several GPUs or lots of RAM | Models of 70B or more |

### My setup

A laptop with an NVIDIA GeForce RTX 2050 with 4 GB of VRAM. An 8B model doesn't fit entirely in the graphics card, so part of it goes to RAM. For fast models with a similar graphics card, the ideal is a 3B or 4B one, such as `llama3.2:3b` or `qwen2.5:3b`. The cost is somewhat worse answers.

Although I personally use a dedicated server with a better graphics card and enough RAM to support models with quite a few more parameters, that will depend on your hardware, which is why I'm going to write these posts on my laptop and not on a server.

---

## Conclusion

An AI model is, in the end, an enormous file of numbers, and everything we've seen, parameters, quantization, context, boils down to one idea: **how much memory you have decides which model you can use comfortably**. With 4 GB of VRAM, as in my case, a 3B is the sensible choice: an 8B no longer fits entirely in the graphics card and part of it moves to RAM, which is slower.

In the next post we move on to practice: we install Ollama, create our own models, use the API, set up a web interface and see how to take it to a company.

**Next:** [Local AI (2/3): installation and use (coming soon)]()