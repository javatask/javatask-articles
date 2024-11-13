---
title: "What C-Level Leaders Need to Know About AI Agents"
seoTitle: "AI Agents: Essential Insights for C-Level Leaders"
seoDescription: "C-Level leaders can use generative AI for productivity and innovation, applying mental models to enhance business processes"
datePublished: Tue Nov 12 2024 17:25:38 GMT+0000 (Coordinated Universal Time)
cuid: cm3eq3yol000r09l21kpy343w
slug: what-c-level-leaders-need-to-know-about-ai-agents
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1731432224134/3e644d88-3741-4bda-bf11-a335a44615f8.jpeg
tags: ai, management, innovation, llm, promptengineering, agent, rag

---

## Introduction

Why should you, as a senior executive or manager, spend precious time on "hyped" generative AI topics? I will offer your business a productivity boost, and then it is up to you to invest time into it.

In this article, I will try to offer mental models of available generative AI technologies. I assume that my reader has already played with claude.ai or chatgpt.com, preferably with paid versions, as free versions have no near "intelligence" in comparison to paid ones. If you, my reader, have no experience with generative AI, I encourage you to read Ethan Mollick's book Co-Intelligence, which inspired this article.

I'll try to make this article as compact as possible to build common ground for discussions about the business value of generative AI approaches. I'll cover "raw" Large Language Models (LLMs), Prompt Engineering, Retrieval-Augmented Generation (RAG), Fine-tuning, and finally, AI Agents. Let's go!

## Story

Imagine a medium-sized bakery constrained by local demographics, hiring new personnel to innovate on multiple fronts. The CEO has many ideas, from moving to renewable energy to new innovative marketing ideas. Everything needs to be done in parallel to keeping lights on for existing businesses. As the bakery's products are delicious good, but may not be attractive to new college graduates, it is hard to find new people to start innovating.

The CEO talked to his personnel and saw support for his ideas to innovate, but all people are already 300% loaded, and he heard about this new hyped thing called generative AI that “will steal all jobs”. He starts experimenting by getting a [claude.ai](https://claude.ai/) account and reading through tons of materials promising to help match staff. His daughter just joined a computer science master's program, and the generative AI promise also inspires her. She just passed her [AWS Certified AI Practitioner](https://aws.amazon.com/certification/certified-ai-practitioner/) certification and is keen to use her new knowledge in practice.

During dinner at home, the father (CEO) and daughter start a conversation to decode all these terms and how they can help the business.

## Disclaimer

In this article, I'll use human terms like interns, thinking, reasoning, etc. Current generative AI technology is not alive; it cannot think and reason. I'm using these terms here to simplify the mental model that can be shared within a company.

And I'll repeat one more time: Large Language Models are incapable of thinking. Their behaviour is much more similar to humans and far from traditional computers and programming languages. This is why I use human terms in a technology setting without quotes.

## Who are Interns or Large Language Models (LLMs)?

Starting the conversation, the question is easy - it is an intern. Depending on the education, price, and size of the LLM, you will get better or worse performance. Some intelligent people tend to overthink and do some work much longer, the same with LLMs - bigger LLMs are smarter interns, but they cost more and think longer. Smaller ones are less intelligent but faster as they do not have much knowledge to think through.

LLMs are already helpful; they can help with image generation for your branding, summarizing text, or can be a partner in testing new ideas. Think about them as interns who graduated from an educational institution (school, university, institute), and they have basic knowledge but no practical experience. And here, please notice that LLMs are not capable of learning on their own. Here's the main difference between human interns and LLMs - humans learn, and LLMs are not yet capable.

"Interesting analogy, daughter," reacted Father to the start of the discussion. "I already tried to ask questions about gluten-free bread. I'm discovering this opportunity, but the answer was too primitive. Nevertheless, I used the paid version..."

"Great question, Father," his daughter replied. "Let's talk about conversation."

## And Here Comes Your Context or Prompt Engineering

"Remember, Father? I shared a story about my friend who constantly starts conversations from the middle of the story, not the beginning, not the end. Middle because the beginning is evident to him, and he runs fast to get results and the end of the story. It makes me mad when he jumps in the middle, and I feel like a fool trying to understand the goal.

I'm not telling you that you acted similarly, but for LLMs, you were this person who started from the middle. When you talk to your colleagues, they know so much context about your business - that you're CEO, love bread, have challenges with personnel, and want to improve your product range. But LLMs read every book on the planet; they know computer science, biology, chemistry, etc. They need your context to understand how to help you specifically. And all knowledge is not helping in the beginning, they try to guess."

There are really good online training resources on this topic at [Pluralsight](https://www.pluralsight.com/paths/prompt-engineering-and-generative-ai), but you're a busy person, so simply start by giving personas to your questions and providing more context and details. There are also advanced prompting techniques that force LLMs to plan, research, etc., but they may be too complex for now.

Another exciting offer from major chatbot providers is the ability to set personas and contexts for all conversations. For example, claude.ai offers "projects" that inject your persona and context into conversations.

"Imagine that you asked a newly joined intern the same question about gluten-free bread. Maybe they would give you the same boring answer," she concluded.

"Oh, I understand, so the quality of the answer now depends on the quality of the question, as in real human interaction," the father reflected. "Listen, daughter, so I can help my lawyer and accountant with these prompts. How would I pass my contracts on to the chatbot? Should they copy and paste them into input? I want to review contracts signed in the past. We have new unified types of contracts, and I want to help migrate them to the latest templates. And we have tons of them... with schools, cafes, restaurants, ad-hoc, etc."

## Let's Talk About Tons of Documents or Retrieval-Augmented Generation

Daughter: "Copy-pasting? Hmm, for thousands of documents, maybe it's not a good idea. And maybe your lawyer and accountant want to have analytics for new, old, updated, and similar metrics."

Father: "So your interns can do all of this?!"

Daughter: "Not so fast. You saw that major chatbots have this 'Enterprise' subscription, right?"

Father: "Right, it's enormously expensive, and I saw that I can upload some files. Why should I pay more money?"

Daughter: "Here's the catch - as a human intern can hold only seven things at a time in their head, the same applies to LLMs that cannot consume all your documents simultaneously. And even if you can do this, it may still add more mess as you're not working on one group of contracts but all of them. How many people can work at one time on everything? Not much? You need to focus, just like LLMs input.

Clever techniques were introduced to handle a lot of data. It was called RAG because before a question reaches LLMs, the chatbot does 'googling' for keywords in the initial question on your documents and dynamically adds document data to the query with reference to the initial document. LLMs think you did copy-paste, but it was done on your behalf. So, back to prompt engineering - when you focus on a specific set of contracts, they will only be included in the context to answer specific questions.

As you understand, all your PDF, Word, and scanned copies need to be preprocessed and stored in your private Google, ready to be injected into LLMs questions. And this costs money. The good thing is that now your non-tech people can even do analytics on top of these documents.

Father: "Hmm, interesting. I need to talk to Juli (lawyer). But what about our local tax regulations? I want LLMs to offer me a strategy for benefiting from the majority of economic stimulus currently available."

Daughter: "I think, Dad, it can help, but very little. Here, we still need tax consultants, and maybe they can train the intern on our local tax regulations and specific use cases."

Father: "But you told me that LLM interns cannot learn."

Daughter: "Wait, it's far from simple. It's called fine-tuning, similar to paying for my University."

Father: "Ah, cost... how much?"

## Intern in University or LLM Fine-tuning

Daughter: "As the actor in Le Mans '66 said, not everything money can buy. Maybe fine-tuning may cost you only $10,000, but money is only part of the success. As in university, the main goal is to teach reasoning in a specific field, but LLMs are incapable of reasoning, so what should you do? It would help if you fed them 100,000 and many more pairs of examples, ideally marked as good and bad. For example, you need to say that if a small company is in this situation, a good tax strategy is this, and repeat 100,000 times."

Father: "Oh, I get it. It is not possible at my scale."

Daughter: "Yes, but for some industries with specific terminology and tons of documentation not available to LLM training dataset, it's an ideal way to add persistent knowledge to LLMs."

Father: "OK, so you will tell me that AI Agents are also not for me?"

Daughter: "Mhhh... it's not so simple. AI Agents are for everyone, and it's a good part of the answer."

## Giving Some Freedom to Your Intern or Agentic AI

Daughter: "Before I dive deep into the answer, tell me, father, did you provide your new intern with a new place in the office? Did you buy a PC and a new chair? What tools are available to your intern, if any?"

Father: "Hmm, they have my local documents 'google' and, and, and nothing else. So you want me to give them office programs so they can do what?"

Daughter: "Tell what you want interns to help with; it's a tricky question with humans. Interns often come to a job to figure out what to do and how to be helpful. But remember that computer interns cannot learn; they can follow your instructions.

So in this case, it's good to start gathering or providing AI agents to your employees and look for detailed and prescribed steps for what and how to do it, preferably in the digital world."

Father: "Let me think about it. So I can give it Excel and ask them to calculate how many goods can fit our warehouse, constantly monitor prices, and offer me ideas about where, when, and how much to buy, right?"

Daughter: "Oh, so complex. Preferably, start with primitive and trivial tasks, but let's start with your use case. You must understand that you need a software developer who can use, e.g., AWS Bedrock AI Agents framework to glue everything together. There's also [Computer Use from Anthropic](https://www.anthropic.com/news/3-5-models-and-computer-use), and other frameworks like [autogen](https://microsoft.github.io/autogen/0.2/). Oops, I went too techy. So you need a developer who will combine software, classical code, or API calls to get the current status of the warehouse, current prices for goods, and LLM to run on a schedule based on your precise prompt on when to call The Boss :) and offer good deals on specific positions."

Father: "Interesting and complicated. So I need to think now and maybe share with employees the idea of assistants, with only one requirement - give me detailed simple instructions that these assistants should follow. It will take time for me to adapt."

## Recap or the Beginning of a New Era

Father: "So LLMs are new interns, to which I need always to give detailed context and questions via prompt engineering techniques. If I need to feed many documents or these documents constantly change, I need RAG. Fine-tuning is for big companies, and finally, AI Agents can perform simple actions following my instructions, but I need to invest in a software developer, right daughter?"

Daughter: "Yes, father, and I want to write my thesis on AI Agents to manage our fleet of autonomous buses powered by our solar station located on the farm. Good night, Father."

Father: "And now you want me to sleep? No way! I'll write the first instructions for my first AI Agent. By the way, is coding hard?"

## Conclusion

In conclusion, integrating generative AI technologies into business operations offers significant potential for enhancing productivity and innovation. As demonstrated through the analogy of a bakery, AI can serve as a valuable tool for executives and managers, acting as an intern that requires guidance and context to perform effectively. By understanding and utilizing concepts such as Large Language Models, Prompt Engineering, Retrieval-Augmented Generation, Fine-tuning, and AI Agents, businesses can leverage AI to streamline processes, manage large volumes of data, and automate routine tasks. However, successful implementation requires investment in both technology and human resources, such as software developers, to tailor AI solutions to specific business needs. As AI continues to evolve, it presents new opportunities for businesses to innovate and adapt, marking the beginning of a new era in business operations.