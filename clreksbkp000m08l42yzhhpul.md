---
title: "AI Unleashed: Assembling Local Engines. CPU Only"
seoTitle: "AI Unleashed: Assembling Local Engines. CPU Only"
seoDescription: "Master local AI: Run open-source LLMs on Windows 10/11 with CPU-only setup using Ollama, enabling private, powerful AI applications"
datePublished: Mon Jan 15 2024 07:00:09 GMT+0000 (Coordinated Universal Time)
cuid: clreksbkp000m08l42yzhhpul
slug: ai-unleashed-assembling-local-engines-cpu-only
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705092409372/7efe0f09-797d-4c0d-86da-21fc66012792.png
tags: tutorial, privacy, ai-tools, generative-ai

---

## Introduction

In this article, we aim to empower individuals who face limitations in using publicly hosted Large Language Models (LLMs) by guiding them through the process of running open-source LLMs locally. This guide focuses on Windows 10/11 PCs and CPU-only use cases using [Ollama](https://ollama.ai/) \- a platform that offers a variety of open-source LLMs. By the end of this guide, you will be able to run LLMs locally, thereby making the most out of AI advancements in private setups. Let's get started!

The article focuses on Windows 10/11 PCs because if you are a Mac or Linux user - you are already halfway there. This part is all about CPU-only use cases. A separate article will cover CPU+GPU-powered cases.

This article is part of [On-Prem Generative AI](https://blog.javatask.dev/series/gen-ai-on-prem) series.

> Note 1. There is alternative product called [GPT4All](https://gpt4all.io/index.html) \- which has native Windows installer, so there is no need to install WSL2 and work with Linux Bash.
> 
> Note 2. Also [GPT4All](https://gpt4all.io/index.html) has build in ChatBot UI. There is no need to install other software
> 
> Note3. My incentive to use Ollama is to show more complex use cases. Ultimately, I give you, my reader, the ability to develop locally E2E GenAI-powered products. Thus Linux is the way ;)

## Prerequisites

You need:

1. Admin access to your Windows 10 or 11
    
2. At least 16 GB of RAM
    
3. Basic knowledge of [Bash](https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/)
    

Let's go!

## Guide

As the world of AI continues to expand, having tools like local LLMs at your fingertips is becoming increasingly valuable. This guide will walk tech enthusiasts and professionals wanting to run LLMs locally on a Windows 10 or 11 machine through the process. We'll be leveraging the power of Ubuntu 22.04 on Windows Subsystem for Linux 2 (WSL2) and installing Ollama, a framework for running large language models like [Meta Llama2](https://huggingface.co/meta-llama/Llama-2-7b).

### Step 1: Install Ubuntu 22.04 on Windows WSL2

The first step involves setting up Ubuntu 22.04 on Windows 10 or 11 using WSL2. WSL2 is a lightweight virtual machine that allows you to run a Linux kernel alongside your Windows OS. Follow the comprehensive guide on [Ubuntu's official tutorial](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview) to get Ubuntu up and running on your system.

> Note. !!!! SUPER importent step !!!

Create file called `.wslconfig` in C:\\Users\\USER\_NAME folder, with content:

```ini
[wsl2]
memory=12GB
```

This file controls the amount of RAM accessible by the WSL2 engine. I recommend to allocate ~80% of available RAM to your WSL2 Ubuntu instance. LLMs are RAM-hungry.

> Note. !!!! SUPER importent step!!!

### Step 2: Installing Ollama on Ubuntu 22.04 in WSL2

Once you have Ubuntu installed, the next step is to install [Ollama](https://github.com/jmorganca/ollama). Ollama is a framework designed to run large language models that power generative AI applications. To install Ollama, use the following command in your Ubuntu terminal:

```bash
curl https://ollama.ai/install.sh | sh
```

For detailed instructions and troubleshooting, visit the [Ollama GitHub page](https://github.com/jmorganca/ollama).

### Step 3: Download the Model and use IT

It's not over yet - you need to download your model. Open the Ubuntu terminal and run the command to fetch [`codellama (ollama.ai)`](https://ollama.ai/library/codellama) model:

```bash
ollama pull codellama:7b
```

Wait...

After a successful download, let's test the model.

### Step 4: Test model

Open the Ubuntu terminal and run the command to test `codellama` model:

> Note. Example is taken from Ollama blog - [How to prompt Code Llama · Ollama Blog](https://ollama.ai/blog/how-to-prompt-code-llama)

```bash
ollama run codellama:7b '
Where is the bug in this code?

def fib(n):
    if n <= 0:
        return n
    else:
        return fib(n-1) + fib(n-2)
'
```

Response is:

````bash
The bug in this code is that it does not handle the case where `n` is 1 or 2. The base case of the
recursion is `if n <= 0: return n`, but this will never be reached when `n` is 1 or 2 because `fib(1)` and
`fib(2)` are both positive numbers and will not satisfy the condition `n <= 0`.

To fix this bug, we can add a new base case that handles the cases where `n` is 1 or 2:
```
def fib(n): ...
````

Congratulations, you did it!

## Model Garden

Ollama offers a lot of LLMs for you - look what it offers - [library (ollama.ai)](https://ollama.ai/library)!

## Conclusion

In this guide, we have explored the process of setting up a local Generative AI engine on a Windows 10/11 PC using CPU-only setup. By leveraging Ubuntu 22.04 on Windows Subsystem for Linux 2 (WSL2) and the Ollama framework, we've demonstrated how to run large language models locally. This allows for more complex use cases and the potential to develop end-to-end AI-powered products. The possibilities are vast, with many models available in the Ollama library. As AI advances, having these tools at your disposal will prove increasingly valuable.