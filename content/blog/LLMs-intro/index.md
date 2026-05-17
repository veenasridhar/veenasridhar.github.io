---
title: Introduction to Large Language Models
summary: TBA
date: 2026-05-17

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
#image:
#  caption: 'Image credit: [**Unsplash**](https://unsplash.com)'

cover:
  #image: "https://images.unsplash.com/photo-1557682250-33bd709cbe85?q=80&w=2560"
  position:
    x: 50
    y: 40
  overlay:
    enabled: true
    type: "gradient"
    opacity: 0.4
    gradient: "bottom"
  fade:
    enabled: true
    height: "80px"
  icon:
    name: "✨"

authors:
  - me
  - Ted

tags:
  - LLMs
  - 
  - 

content_meta:
  trending: true
---



{{< toc mobile_only=true is_open=true >}}

### What is a LLM?

 Large Language Models (LLM) 

 ### Base LLM vs Instruction Tuned LLM


### Tokens
Tokens can be considered as sequence of characters. In text generation and embedding models, the text is processed in chunks known as tokens. There is not standard number of characters per token but as a rule of thumb, a token can be of approximately 4 characters or 0.75 of a word in English text.

### Prompt Engineering
Role of the LLM. Instruction to the LLM. Examples for LLM. 

There is shift in building with LLMs from prompts to context. It has moved from finding the words to phrase a question to configuring the right context to generate the desired behaviour from the model.

### Context
Context is the content that is part of the LLM's input. Context is always restricted in length.

For eg. Making an AI agent respond to personal emails just like the actual person responding.

Have to provide personal context for the LLM to work as intended. Important concepts to consider: Metadata - Concise - Recenccy - Rules - Feedback from Human

<h3>Parts of Context Engineering</h3>
<ol>
<li>User Input</li>
<li>System Prompt</li>
<li>RAG</li>
<li>Tool Results (MCP)</li>
<li>Memory</li>
<li>Conversation History</li>
<li>Dynamic Examples</li>
<li>Guardrails and schemes</li>
</ol>

<h3>OS Analogy to a LLM System</h3>

LLM Models              <===> CPU
System Prompt           <===> OS kernel
Context Window          <===> RAM
RAGs (Vector DB, docs)  <===> File System
Tool / API Calls (MCP)  <===> System Calls
Agents                  <===> Applications

LLM models are analogous to a CPU as they are the main computational resource of the system. <br>
Context window is similar to RAM as both are limited and temporary in nature. 

<h3> Why is Context Engineering needed? </h3>
Typical Context window of a LLM model ranges from 128,000 tokens (91,000 words) to 2,000,000 tokens (7,142,000 words) depending on the model.<br>
<b>But if the context window is large enough what is the need to engineer the context?</b>
 - Cost - The cost for using the LLM models are billed based on the tokens used. Thus, efficient token usage reduces the cost.
- Context Rot - The model's ability to recall and perform accuractely decreases as the number of tokens in the context increases.
Just like humans who have a limited working memory, LLM models also have a <b> attention budget</b> that they draw on when parsing on large volumes of context.
LLMs which are based on transformer architecture is the reason behind this attention scarcity. The transformer architecture allows every token to attend to every other token in the context creating pairwise relationships. As the context increases the model's ability to capture these pairwise interactions is stretched thin. Also, the attention pattern of the model are based on the training data distributions where shorter sequences are more common. This means the models might struggle to find patterns in large contexts. This can be improved by positional encoding interpolation making the models degrade in a gradient rather than a sharp jump. \ref{2}

#### Context Engineering

Defined as a per step optimization
" The delicate art and science of filling the context window with just the right information for the next step" -Andrej Karpathy

Defined as Systems Thinking
" Building dynamic systems to provide the right info and tool in the right format"
Harrison Chase

Generally, Context engineering is finidng the smallest set of high signal tokens that maximize the likelihood of your desired outcome.

Context engineering allows you to put the information in the sweet spot. Too little tokens with little information causes the model to haullicinate while a large number of tokens causes context rot.

##### 1. Components of a Context and Context Budget

- System Prompt &ensp; &ensp;&ensp;- General Instructions like the roles, rules and constraints.
- Current Query - Input query from user for that exchange.
- Conversation History &ensp;- History of conversation between the user and the assistant.
- Memory & State &ensp; - Persistant facts, Metadata
- Retrieved Knowledge RAG &ensp; - Retrieved result as chunks
- Tool Outputs &ensp; - results from API(MCP) calls or tools.

The order of placing these elements in the context is also important. As, LLMs tend to give more attention to the content in the start and end of the context rather than to the middle. This is known as the Lost in the middle effect. Thus most important parts like System prompts and current query should go either in the start or end of a context.





<h3>References</h3>
[1] [Effective Context Engineering - Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
[2][Positional Encoding Interpolation](https://arxiv.org/pdf/2306.15595)
[3] [Vizuara Context Engineering Bootcamp - Lecture 1, ]