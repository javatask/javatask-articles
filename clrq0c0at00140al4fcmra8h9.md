---
title: "AI Unleashed: Local Chat Bot"
seoTitle: "AI Chatbot: Unleashed & Localized"
seoDescription: "Master AI chatbot setup on Ubuntu 22.04, WSL2 with Ollama framework using our guide. Enhance AI experience with ChatGPT-like chatbot"
datePublished: Tue Jan 23 2024 07:00:49 GMT+0000 (Coordinated Universal Time)
cuid: clrq0c0at00140al4fcmra8h9
slug: ai-unleashed-local-chat-bot
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705919225333/d28b8497-7f08-4a31-aecf-7a1ba05cd7fb.png
tags: privacy, chatbot, llm, generative-ai, chatgpt

---

## Introduction

In this article, I aim to empower individuals who face limitations in using publicly hosted Large Language Models (LLMs) by showing them how to run local Generative AI chatbot. The guide is part of the [AI Unleashed series](https://blog.javatask.dev/series/ai-unleashed).

In previous parts, I introduced the [overall architecture](https://blog.javatask.dev/ai-unleashed-running-generative-models-locally-introduction) and a guide for [running local LLMs](https://blog.javatask.dev/ai-unleashed-assembling-local-engines-cpu-only). This article uses [Ubunut 22.04 on WSL2](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/#1-overview) and [Ollama](https://ollama.ai/) as prerequisites to configure the local chatbot.

It's best to start the article with the result:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704211386364/ff288d75-f138-40f5-86e9-8f2dcb838405.gif align="center")

Let's get started!

## Chatbot installation guide

As the world of AI continues to expand, having tools like ChatGPT at your fingertips is becoming increasingly valuable. This guide will walk tech enthusiasts and professionals wanting to run a ChatGPT-like chatbot locally on a Windows 10 or 11 machine through the process. We'll be leveraging the power of Ubuntu 22.04 on Windows Subsystem for Linux 2 (WSL2) and installing Ollama, a framework for running large language models like [Meta Llama2](https://huggingface.co/meta-llama/Llama-2-7b).

I will use the [Ollama Web UI](https://github.com/ollama-webui/ollama-webui) project with an [MIT License](https://github.com/ollama-webui/ollama-webui/blob/main/LICENSE) in this article. I highly encourage you to support [Timothy](https://jryng.com/) - this great project's author!

I intentionally use a non-container approach to deploy the chatbot in this guide. I chose this due to dependencies on external Ollama, simplicity in supporting GPU-enabled workloads, and the straightforward way of upgrading your local chatbot. Containers are not always the way to go ;)

### Step 1: **Installing NodeJS on Ubuntu**

Installing Node.js 20 can be done similarly:

1. **Open Terminal**
    
2. **Install Node Package Manager**: [GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions](https://github.com/nvm-sh/nvm):
    
    ```bash
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
    ```
    
    > Note. If you don'y have `curl` command install it via `sudo apt update && sudo apt install curl`
    
3. **Install Node.js**: Use the following command to install Node.js:
    
    ```bash
    nvm install 20
    ```
    
4. **Verify Installation**: After installation, check the installed version of Node.js:
    
    ```bash
    node --version
    ```
    

### Step 2: Installing Ollama-Web

Ollama-web provides a user interface to interact with the Ollama model garden. Follow these steps to launch the web interface:

1. Clone the Ollama-Webui repository:
    
    ```bash
    git clone https://github.com/ollama-webui/ollama-webui.git
    cd ollama-webui/
    ```
    
2. Copy the required `.env` file:
    
    ```bash
    cp -RPp example.env .env
    ```
    
3. Build the frontend:
    
    ```bash
    npm i
    npm run build
    ```
    
4. Serve the frontend with the backend:
    
    ```bash
    cd ./backend
    pip install -r requirements.txt -U
    sh start.sh
    ```
    

For more information, refer to the [Ollama-web GitHub page](https://github.com/ollama-webui/ollama-webui).

### Step 3: Download the Model and use IT

Open your browser \`[http://localhost:8080/](http://localhost:8080/)\`. During the first load, you will be asked to create a user. This first user will have SuperAdmin access.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704212616603/fc55d566-5849-4dc9-8c6e-e151437bff3a.png align="center")

It's not over yet - you need to download your model. Click on the gear icon, insert `llama2:7b` a model name into the Pull model input and click the green button. Wait...

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704212896134/81fcc1a7-01ec-41aa-a429-ce8fda03c206.png align="center")

After successful download your model questions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704212988305/4f7f353c-4f42-479c-ad98-b504624c8321.gif align="center")

## Model Garden

Ollama offers a lot of LLMs for you - look what it offers - [library (ollama.ai)](https://ollama.ai/library)!

## Conclusion

In this tutorial, we've set up a local AI chatbot using Ubuntu 22.04 on the Windows Subsystem for Linux 2 (WSL2) and the Ollama framework. We've covered the installation of NodeJS on Ubuntu, the setup of Ollama-Web, and how to download and utilize the AI model. This guide is intended to empower those who might encounter restrictions when using publicly hosted Large Language Models by giving them the tools to run their local Generative AI chatbot.