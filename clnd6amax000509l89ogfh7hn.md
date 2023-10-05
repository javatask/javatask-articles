---
title: "A Beginner's Guide to AWS Bedrock: Exploring Amazon's AI Generation Service"
seoDescription: "Master AWS Bedrock Generative AI for code summarization with Anthropic's Claude 2 model using Python and boto3, ensuring privacy and flexible pricing"
datePublished: Thu Oct 05 2023 12:47:53 GMT+0000 (Coordinated Universal Time)
cuid: clnd6amax000509l89ogfh7hn
slug: a-beginners-guide-to-aws-bedrock-exploring-amazons-ai-generation-service
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696509178336/cf920f69-e049-4a46-ab41-8c97d79071c9.png
tags: aws, python, summarization, generative-ai, langchain

---

# Introduction

In this blog post, we will discuss how to use Amazon's AWS Bedrock Generative AI Service for code summarization with the help of Anthropic's Claude 2 model. We will walk through a practical example using Python and the boto3 library to demonstrate the process of summarizing a set of documents.

Link to the repository: [generative-ai/cloud-bedrock3.ipynb at main · javatask/generative-ai (](https://github.com/javatask/generative-ai/blob/main/cloud-bedrock3.ipynb)[github.com](http://github.com)[)](https://github.com/javatask/generative-ai/blob/main/cloud-bedrock3.ipynb)

# Incentive or Why not GPT-4?

Using AWS Bedrock offers several advantages, including:

* allowing your company to cover billing expenses through AWS billing ;)
    
* ensuring privacy and data protection
    
* providing access to a wide range of pre-trained AI models in the gallery
    
* and offering flexible pricing depending on your need
    

I hope that more and more specialized models like Text-to-SQL will appear in the gallery.

# Theory

Please reference to Langchain's excellent documentation on using AI for summarization and much more [Summarization | 🦜️🔗 Langchain](https://python.langchain.com/docs/use_cases/summarization)

# Setting Up the Environment

To set up the AWS session, I used AWS CLI v2 and SSO to log in and obtain the necessary credentials for accessing AWS services.

Before diving into the code, let's ensure we have the necessary packages installed. You will need to have boto3, langchain, and PyPDF2 installed. You can install them using pip:

```bash
pip install boto3 langchain pypdf
```

# The Code

## Bedrock-runtime

Now let's import the required modules and create an instance of the Bedrock class with the "anthropic.claude-v2" model:

> !!!**Note**!!! It's important to check the list of `model_kwargs` because they are different for different `model_id`

```bash
import boto3
from langchain.llms.bedrock import Bedrock
boto3_bedrock = boto3.client("bedrock-runtime", 'us-east-1')
llm = Bedrock(
    model_id="anthropic.claude-v2",
    model_kwargs={
        "temperature": 0,
        "top_k": 250,
        "top_p": 1,
    },
    client=boto3_bedrock,
)
```

## Defining the Chains

Next, we will define the Map and Reduce templates for our LLM chains:

```bash
from langchain.chains import ReduceDocumentsChain, StuffDocumentsChain, MapReduceDocumentsChain, LLMChain
from langchain.prompts import PromptTemplate
from langchain.document_loaders.pdf import PyPDFLoader

map_template = """The following is a set of documents
{docs}
Based on this list of docs, please identify the main themes 
Helpful Answer:"""
map_prompt = PromptTemplate.from_template(map_template)
map_chain = LLMChain(llm=llm, prompt=map_prompt)

reduce_template = """
The following is set of summaries:
{doc_summaries}
Take these and distill it into a final, consolidated summary of the main themes. """
reduce_prompt = PromptTemplate.from_template(reduce_template)
reduce_chain = LLMChain(llm=llm, prompt=reduce_prompt)
```

## Creating the MapReduce Chain

Now, we will create a MapReduce chain that combines and iteratively reduces the mapped documents using the defined Map and Reduce chains:

```bash
combine_documents_chain = StuffDocumentsChain(
    llm_chain=reduce_chain, document_variable_name="doc_summaries"
)

reduce_documents_chain = ReduceDocumentsChain(
    combine_documents_chain=combine_documents_chain,
    collapse_documents_chain=combine_documents_chain,
    token_max=8000,
    verbose=True
)

map_reduce_chain = MapReduceDocumentsChain(
    llm_chain=map_chain,
    reduce_documents_chain=reduce_documents_chain,
    document_variable_name="docs",
    return_intermediate_steps=False,
    verbose=True
)
```

## Loading and Summarizing Documents

Finally, we will load a single PDF document using PyPDFLoader and run the MapReduce chain to obtain the summarized output:

```bash
loader = PyPDFLoader("./cloud/aws-caf-ebook.pdf")
split_docs = loader.load_and_split()

result = map_reduce_chain.run(split_docs)
print(result)
```

# Conclusion

In this blog post, we explore how to use Amazon's AWS Bedrock Generative AI Service for code summarization with Anthropic's Claude 2 model. We walk through a practical example using Python and the boto3 library to summarize a set of documents. We discuss the benefits of using AWS Bedrock and provide a step-by-step guide on setting up the environment, defining the LLM chains, creating the MapReduce chain, and loading and summarizing documents.