---
title: "AI Unleashed: Running Generative Models Locally. Introduction"
seoTitle: "AI Unleashed: Local Generative Models Guide"
seoDescription: "Optimize AI on local consumer hardware, safely experiment with real data, and boost productivity using open-source language models"
datePublished: Wed Jan 03 2024 15:38:36 GMT+0000 (Coordinated Universal Time)
cuid: clqxy0tty000009jq8idyhfm5
slug: ai-unleashed-running-generative-models-locally-introduction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704289583478/3de01037-530d-4643-9c17-e5e7a6447977.png
tags: ai, opensource, beginner, best-practices, llm, generative-ai

---

## Introduction

This article is the first in the series about running a generative AI model locally on consumer-grade hardware to give you a safe place to experiment and understand your use cases. I will try to answer the question of how to start experimenting with your real data safely.

Gartner states[, "In theory, at least, this (Generative AI) will increase worker productivity"](https://www.gartner.com/en/topics/generative-ai). Another Gartner recommendation is to "Start Inside ... with ... Off-the-shelf products". But it's a theory.

In this series, I'll offer practical steps for implementing Gartner's recommendations for regulated (internally and externally) environments. The series promotes the usage of open-source large language models (LLMs) that are the heart of Generative AI (GenAI) systems.

> **Note**. *If you don't have any restriction to process your real world data on major Public Cloud providers, like* [*AWS Bedrock*](https://aws.amazon.com/bedrock/)*,* [*Azure AI*](https://azure.microsoft.com/en-us/solutions/ai) *or* [*Google Vertex AI*](https://cloud.google.com/vertex-ai)*. Go for the Public Cloud GenAI offerings!!!*

## Key Definitions

**Generative AI:** This subfield of artificial intelligence uses models and algorithms to generate content. It can create anything from written text to images or music, by learning patterns from existing data and producing new content that mimics it.

**Large Language Models (LLMs):** These are AI models trained on a vast amount of text data. They can generate human-like text by predicting the probability of a word given the previous words used in the text. Examples include OpenAI's [GPT-3](https://openai.com/blog/gpt-3-apps) and Google's [Gemini](https://deepmind.google/technologies/gemini/).

**AI Agents:** These are systems or software that can perform tasks or make decisions autonomously. They use Generative AI to "understand" their environment and perform actions to achieve specific goals.

**CUDA:** This stands for Compute Unified Device Architecture. It's a parallel computing platform and application programming interface model created by Nvidia. It allows software developers to use a CUDA-enabled graphics processing unit for general-purpose processing.

**Chatbot:** A chatbot is an AI-powered software designed to interact with humans in their natural languages. These interactions can occur in both text and voice formats.

**Machine Customers:** This is a term coined by Gartner to describe AI systems that can autonomously perform tasks or make purchasing decisions on behalf of human users or other systems.

**Public Cloud GenAI offerings:** These are Generative AI models or services offered by public cloud providers like Google, Amazon, and Microsoft. They provide pre-trained models and services which developers can use to integrate AI capabilities into their applications.

## Steam Revolution is here

This article has a steam engine on its cover because steam and latte electric engines changed the world. The same is happening with AI, specifically with AI Agents.

According to [LangChain](https://python.langchain.com/docs/modules/agents/concepts): "The core idea of (AI) agents is to use a language model to choose a sequence of actions. In chains, a sequence of actions is hardcoded (in code). In agents, a language model is used as a reasoning engine to determine which actions to take and in which order."

So GenAI becomes the "brain", an orchestrator of tools you may use to achieve a given goal or react to the situation. This agent can work 24/7, process enormous amounts of data and be "objective". One type of such agent [Gartner call "**Machine Customers**".](https://www.gartner.com/en/articles/machine-customers-will-decide-who-gets-their-trillion-dollar-business-is-it-you)

## Application Covered by the Series

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704293765792/1ad0f45d-f82f-46a2-9716-af13233475cf.png align="center")

This series will have two big blocks:

1. Setting up and running your "brain" = LLM locally
    
    * With CPU only
        
    * With CPU and Nvidia GPU, you need GPUs that has [CUDA](https://developer.nvidia.com/cuda-toolkit) support
        
2. *Configuring* apps that use "brain" to deliver value:
    
    * [ChatGPT](https://chat.openai.com/) like [chatbot](https://github.com/ollama-webui/ollama-webui)
        
    * [GitHub Copilot](https://github.com/features/copilot) like [VS Code AI Assistant](https://continue.dev/)
        
    * [AWS Knowledge Bases](https://aws.amazon.com/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/) like super-powered search on private docs
        
    * [AI Agents](https://python.langchain.com/docs/modules/agents/) - Demo on how "brain" can use external APIs
        

> **Note**. Diagram shows that Public Cloud providers offer you LLM APIs, so you don't need to worry about hardware and other supportive software.

## Manage your expectations

Technology is here, but hardware is still evolving. Following my tutorials on setting up GenAI locally, you soon feel that some models are "dummer" than ChatGPT 4 (state-of-the-art, closed model). They are slower, because you may not have the latest and greatest CUDA-enabled GPU.

BUT, local LLMs technology is good enough for you to start experimenting with GenAI. Remember GenAI is a company-wide initiative, not only an IT initiative!

## Conclusion

Setting up a local Generative AI model can be a game-changer, providing an avenue to explore, experiment, and build expertise in its use cases. While the technology is available, remember that hardware is still in a state of evolution. Despite some models being slower and less sophisticated than state-of-the-art models, leveraging these tools locally offers a valuable opportunity to experiment and identify the best use cases for Generative AI. This is not just an IT initiative, but a company-wide effort that can revolutionize productivity and efficiency. As we move forward, the fusion of AI with our daily tools and tasks will become increasingly integral to our work and lives.