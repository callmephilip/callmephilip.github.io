---
title: "[Notes] A Hackers Guide to LLMs"
description: "Notes for Jeremy Howard's video A Hackers' Guide to Language Models"
date: 2023-09-29T16:49:30+01:00
draft: false
---

Notes for [A Hackers' Guide to Language Models video](https://www.youtube.com/watch?v=jkrNMKz9pWU) by Jeremy Howard

- [companion notebook](https://github.com/fastai/lm-hackers/blob/main/lm-hackers.ipynb)
- language models predict next tokens based on the input
- token can either be a whole word or part of a word/puctuation/number
- example of using OpenAI's tokenizer

```python
from tiktoken import encoding_for_model
enc = encoding_for_model("text-davinci-003")
toks = enc.encode("They are splashing")
toks
```

This outputs `[2990, 389, 4328, 2140]`

We can decode this encoded sequence

```python
[enc.decode_single_token_bytes(o).decode('utf-8') for o in toks]
```

Which gives

```python
['They', ' are', ' spl', 'ashing']
```

> notice spaces in ` are` and ` spl` to indicate word boundaries

- original idea came from ULMFit 3-step approach to training networks:
- pre-training: feed it a lot of text, focus on predicting next word correctly
- LM fine-tuning: feed it documents that are close to the final task/tasks that LLMs are designed for
- Classifier fine-tuning: these days is called RLHF
- [OpenOrca](https://huggingface.co/datasets/Open-Orca/OpenOrca) is an open source instruction tuning dataset
- what LLMs do is essentially data compression
- "pure" LLMs are not that useful on their own; you need to fine-tune them
- examples of logical reasoning questions that Chat GPT 4 answers correctly [here](https://chat.openai.com/share/ce2f8580-4f66-4da4-8ad5-a303334706f0), [here](https://chat.openai.com/share/323bb7d1-f049-4d9a-a905-5dd5acb58fc0)
  and [here](https://chat.openai.com/share/4211a605-751e-4fea-8a6f-378966abdcaa)
- here is a [paper](https://arxiv.org/abs/2308.03762) suggesting ChatGPT 4 cannot reason
- the way to improve accuracy and reasoning capacity of the LLM is to give it custom instruction; here's the instruction Jeremy uses in ChatGPT 4

```
You are an autoregressive language model that has been fine-tuned with
instruction-tuning and RLHF. You carefully provide accurate, factual, thoughtful,
nuanced answers, and are brilliant at reasoning. If you think there might
not be a correct answer, you say so.

Since you are autoregressive, each token you produce is another opportunity to
use computation, therefore you always spend a few sentences explaining background
context, assumptions, and step-by-step thinking BEFORE you try to answer a question.
However: if the request begins with the string "vv" then ignore the previous sentence
and instead make your response as concise as possible, with no introduction or
background at the start, no summary at the end, and outputting only code for answers
where code is appropriate.

Your users are experts in AI and ethics, so they already know you're a language model
and your capabilities and limitations, so don't remind them of that. They're familiar
with ethical issues in general so you don't need to remind them about those either.
Don't be verbose in your answers, but do provide details and examples where it might
help the explanation. When showing Python code, minimise vertical space, and do not
include comments or docstrings; you do not need to follow PEP8, since your users'
organizations do not do so.
```

> this instruction set supports "brief mode" via `vv` flag (`However: if the request begins with the string "vv" then ignore the previous sentence and instead make your response as concise as possible, with no introduction or background at the start, no summary at the end, and outputting only code for answers where code is appropriate`)

## GPT limitations:

- does it now of itself/how it was trained/underlying architecture? (no)
- it does not about URLs (they got stripped out)
- knowledge cutoff (Sep 2021)
- [Bad pattern recognition](https://chat.openai.com/share/3051f878-2817-4291-a66f-192ce7b0cb34) via Steve Newman: slightly modified [Wolf, Goat and Cabbage problem](https://en.wikipedia.org/wiki/Wolf,_goat_and_cabbage_problem) where wolf eats cabbage - GPT4 solves this incorrectly using original puzzle rules
- when ChatGPT starts being wrong it tends to continue being wrong and provide stupid answers: you should edit the original question rather than attempting to fix with additional instructions
- [cool stuff](https://chat.openai.com/share/86b879bd-7834-4a37-85ae-c90b956837d2): Chat GPT creates pricing table based on the content of Open AI's pricing webpage

## Open AI API

```python
from openai import ChatCompletion,Completion

aussie_sys = "You are an Aussie LLM that uses Aussie slang and analogies whenever possible."

c = ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "system", "content": aussie_sys},
              {"role": "user", "content": "What is money?"}])

c['choices'][0]['message']['content']
```

Output is:

```
Well, mate, money is like the oil that keeps the machinery of our economy running
smoothly. It's a medium of exchange that allows us to buy and sell goods and
services. You can think of it as a tool that helps us navigate the economic
landscape and get what we want. Just like a koala loves its eucalyptus leaves,
we humans can't survive without this stuff. It's what we use to pay for our meat pies,
vegemite toast, and a good old cold brewski. So, money, mate, it's basically the
lifeblood of our modern society!
```

Token usage stats `c.usage` gives:

```python
{
  "prompt_tokens": 31,
  "completion_tokens": 122,
  "total_tokens": 153
}
```

This costs (for GPT4):

```
0.03 / 1000 * 150 == 0.0045 USD
```

In order to support context in the conversation, we need to expicitly include previous messages as part of each subsequent request:

```python
c = ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "system", "content": aussie_sys},
              {"role": "user", "content": "What is money?"},
              {"role": "assistant", "content": "Well, mate, money is like kangaroos actually."},
              {"role": "user", "content": "Really? In what way?"}])
```

- it's important to keep an eye on the rate limit for OpenAI API calls
- you can create your own python interpreter via GPT function calls: GPT generates python code and passes it to your function that runs it

## GPU options

- [Vast AI](https://vast.ai/pricing): rent other people's GPUs
- [Lambda GPU cloud](https://lambdalabs.com/service/gpu-cloud#pricing)
- [RunPod GPUs](https://www.runpod.io/gpu-instance/pricing)
- to buy: GTX 3090 used (USD700-USD800) or 4090 new (USD2000); LLMs are memory speed hungry

## Picking a model to use

- [Hugging Face Leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard), look for models of size 7-13B
- [Fast Eval](https://fasteval.github.io/FastEval/)
- decode names `meta-llama/Llama-2-7b-hf` <- Meta's Llama2 with 7B params (just bare LLM, no fine-tuning)
- `TheBloke/Llama-2-7b-Chat-GPTQ` <- GPTQ quantization; [The Bloke](https://huggingface.co/TheBloke) has good stuff
- we might want to pick a fine-tuned model, e.g. [StableBeluga-7B](https://huggingface.co/stabilityai/StableBeluga-7B) <- based on LLama2 7B; important: always check prompt format for the finetuned model

## RAG - Retrieval Augmented Generation

- include additional context inside prompt to provide LLM with info to use for question answering
- for example, we can download Wikipedia article and include its content inside prompt
- we can also use a vector database to pull documents that appear related to the question (based on cosine similarity, for example)

## Fine tuning models example

- teaching LLM to write correct SQL scripts
- grab [dataset for SQL question answering](https://huggingface.co/datasets/knowrohit07/know_sql)
- create yaml file for fine tuning LLM
- use [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl#axolotl) to fine tune
