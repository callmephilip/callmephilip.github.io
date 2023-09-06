---
title: "How Smol AI Developer Works"
date: 2023-09-06T12:30:54+01:00
draft: false
---

[Smol AI/developer](https://github.com/smol-ai/developer) is an open source code generation tool by mighty [swyx](https://github.com/swyxio) that got a lot of people excited. You can see some demos of its usage [here](https://www.youtube.com/watch?v=DzRoYc2UGKI) and [here](https://www.youtube.com/watch?v=zsxyqz6SYp8). In this post we'll take a look under the hood to see how it works.

After reading this you should be able to recreate something similar potentially better tailored to your needs and your stack. I assume you have some basic experience with LLMs and Python programming language.

We know that developer takes a `prompt` (spec) from the user and spits out a fully scaffolded app based on it. Let's take a [sample prompt from the repo](https://github.com/smol-ai/developer/blob/main/prompt.md) and use it in this walkthrough:

```md
A Chrome Manifest V3 extension that reads the current page, and offers a popup UI
that has the page title+content and a textarea for a prompt (with a default value
we specify).

When the user hits submit, it sends the page title+content to the Anthropic Claude API
along with the up to date prompt to summarize it.

The user can modify that prompt and re-send the prompt+content to get another summary
view of the content.

... [[[skipped a bunch of stuff here for brevity, please refer to the link above to see the whole prompt]]]
```

## High level understanding

Let's start by taking a look at the [list of dependencies](https://github.com/smol-ai/developer/blob/main/pyproject.toml) for the project to get an idea of what we are gonna be dealing with here:

```toml
[tool.poetry.dependencies]
python = ">=3.8"
openai = "^0.27.8"
openai-function-call = "^0.0.5"
tenacity = "^8.2.2"
agent-protocol = "^0.2.3"
```

No major surprises here, OpenAI is involved and we will be likely using some version of OpenAI's GPT LLM. Let's get a general view of the flow of the program. Here I am desconstructing [main.py](https://github.com/smol-ai/developer/blob/main/smol_dev/main.py) and pulling relevant information out into a tiny snippet we will be using for our analysis. Here's what we've got:

```python
shared_deps = plan(prompt, _, model=model)
file_paths = specify_file_paths(prompt, shared_deps, model=model)

for file_path in file_paths:
    code = generate_code(prompt, shared_deps, file_path, stream_handler, model=model)
    write_file(file_path, code)
```

This is it in the nutshell. We first `plan` what we will be generating based on the `prompt` (i.e. spec) we get from the user. We then figure out which files need to be generated to satisfy the requirements and proceed by generating code for every file in the list and saving it to our hard drive.

`Plan` -> `Specify file paths` -> `Generate code and save it`.

Let's now take a look at each step in more detail.

## Plan

It's all about the prompts with LLMs, [they say](https://time.com/6272103/ai-prompt-engineer-job/). Turns out the key ingredients for smol-ai/developer are found in the [prompts submodule](https://github.com/smol-ai/developer/blob/main/smol_dev/prompts.py).

In this section we are looking inside the planner.

> Please note, just like with the pseudocode above, we are stripping some boilerplate needed to glue things together below.

```python
import openai

SMOL_DEV_SYSTEM_PROMPT = """
You are a top tier AI developer who is trying to write a program that
will generate code for the user based on their intent. Do not leave any todos,
fully implement every feature requested.

When writing code, add comments to explain what you intend to do and
why it aligns with the program plan and specific instructions from the original prompt.
"""

def plan(prompt: str, model):
    return openai.ChatCompletion.create(
        model=model,
        temperature=0.7,
        messages=[
            {
                "role": "system",
                "content": f"""{SMOL_DEV_SYSTEM_PROMPT}

                In response to the user's prompt, write a plan using GitHub Markdown
                syntax. Begin with a YAML description of the new files that will
                be created.

                In this plan, please name and briefly describe the structure of code
                that will be generated, including, for each file we are generating,
                what variables they export, data schemas, id names of every DOM
                elements that javascript functions will use, message names,
                and function names.

                Respond only with plans following the above schema.
                """,
            },
            {
                "role": "user",
                "content": f""" the app prompt is: {prompt} """,
            },
        ],
    )
```

Okay, this looks slightly meatier than the high level flow snippet. Let's talk about inputs first.

`Plan` requires original user's prompt and ID of the OpenAI GPT model to use (this currently defaults to `gpt-4-0613` for smol-ai/developer). There is another prompt involved here as well - `SMOL_DEV_SYSTEM_PROMPT`. Notice that it is used in the `system` context (via `"role": "system"`). This sets LLM's behavior (personality?) that GPT will follow throughout the session. Notice how this general scenario description is augmented with a more specific set of instructions for the planning phase.

We ask for:

- YAML as output
- outline of the project structure (sounds useful for `specify_file_paths`)
- brief description of the code that will be needed for each file (surely handy for `generate_code` step)

Armed with this data, we invoke OpenAI [Chat Completion API](https://platform.openai.com/docs/guides/gpt/chat-completions-api), passing original user's prompt along.

> Meteorology aficionados among you might be wondering about the temperature parameter. In the context of GPT, temperature (which takes values in the range of `[0, 1]`) configures how consistent the outputs of the model are. Remember, these models are probabilistic so there is always room for some variations. Temperature of `0`, gives you mostly deterministic results; higher values make GPT more creative. `0.7` is pretty high which gives `smol-ai/developer` room for creativity.

At the end of the plan phase, we get our project outline from OpenAI GPT. Here's [sample output](/plan.md) for the plan phase

## Specify file paths

Moving on to the second stage of the flow. Let's take a look at `specify_file_paths` function:

```python
import openai
from openai_function_call import openai_function

@openai_function
def file_paths(files_to_edit: List[str]) -> List[str]:
    """
    Construct a list of strings.
    """
    return files_to_edit


def specify_file_paths(prompt: str, plan: str, model: str):
    completion = openai.ChatCompletion.create(
        model=model,
        temperature=0.7,
        functions=[file_paths.openai_schema],
        function_call={"name": "file_paths"},
        messages=[
            {
                "role": "system",
                "content": f"""{SMOL_DEV_SYSTEM_PROMPT}
                  Given the prompt and the plan, return a list of strings
                  corresponding to the new files that will be generated.
                """,
            },
            {
                "role": "user",
                "content": f""" I want a: {prompt} """,
            },
            {
                "role": "user",
                "content": f""" The plan we have agreed on is: {plan} """,
            },
        ],
    )
    result = file_paths.from_response(completion)
    return result
```

We are gonna start with inputs again. `specify_file_paths` receives the original user prompt, as well as the output from the plan phase plus GPT model to use for generation.

> Note that we are reusing general system prompt we saw earlier in the planning phase:

```python
SMOL_DEV_SYSTEM_PROMPT = """
You are a top tier AI developer who is trying to write a program that will
generate code for the user based on their intent. Do not leave any todos,
fully implement every feature requested.

When writing code, add comments to explain what you intend to do and why it
aligns with the program plan and specific instructions from the original prompt.
"""
```

We also remind GPT of the original user prompt and the plan it generated in the previous phase. The key part of this prompt is hidden within `system` role subprompt:

```md
Given the prompt and the plan, return a list of strings corresponding to the new files
that will be generated.
```

We are restricting GPT to a very specific output and notice how we pass additional parameters to the `ChatCompletion` API via `functions` and `function_call`. This functionality is supported via [Function calling](https://platform.openai.com/docs/guides/gpt/function-calling) and it assures that the model outputs information in the format that is compatible with the function signature.

> Please note, GPT won't call the function for us, we still do this explicitly here:

```python
result = file_paths.from_response(completion)
```

Putting all this together, we get a list of file paths back and we are ready to move to the final stage - actual code generation. The output of this phase looks as follows:

```json
["manifest.json", "background.js", "popup.html", "popup.js"]
```

## Generate code

Let's look at the code.

> As always, we remove some boilerplate to keep things a bit more digestable and focus on the core functionality:

````python
def generate_code(prompt: str, plan: str, current_file: str, model: str):
    return openai.ChatCompletion.acreate(
        model=model,
        temperature=0.7,
        messages=[
            {
                "role": "system",
                "content": f"""{SMOL_DEV_SYSTEM_PROMPT}
                  In response to the user's prompt, Please name and briefly describe
                  the structure of the app we will generate, including, for each file
                  we are generating, what variables they export, data schemas,
                  id names of every DOM elements that javascript functions will use,
                  message names, and function names.

                  We have broken up the program into per-file generation. Now your job
                  is to generate only the code for the file: {current_file}

                  only write valid code for the given filepath and file type,
                  and return only the code. do not add any other explanation, only
                  return valid code for that file type.
                  """,
            },
            {
                "role": "user",
                "content": f""" the plan we have agreed on is: {plan} """,
            },
            {
                "role": "user",
                "content": f""" the app prompt is: {prompt} """,
            },
            {
                "role": "user",
                "content": f"""
                  Make sure to have consistent filenames if you reference other files
                  we are also generating.

                  Remember that you must obey 3 things:
                   - you are generating code for the file {current_file}
                   - do not stray from the names of the files and the plan we have decided on
                   - MOST IMPORTANT OF ALL - every line of code you generate must be valid code.
                   Do not include code fences in your response, for example

                  Bad response (because it contains the code fence):

                    ```javascript
                       console.log("hello world")
                    ```

                   Good response (because it only contains the code):

                     console.log("hello world")

                   Begin generating the code now.
                """,
            },
        ],
    )
````

There is a lot of prompting involved here. We are gonna start by looking at the inputs first. No surprises here, we get

- `prompt`
- `plan`
- `current_file`
- `model`

`current_file` points to the file name and comes from the list of file names that is the output from the previous phase. As a reminder, here's how `generate_code` gets invoked:

```python
for file_path in file_paths:
    code = generate_code(prompt, shared_deps, file_path, stream_handler, model)
```

Now let's move to prompting. System prompt makes an appearance again, as well as the reminders of the generated `plan` and original user `prompt`. The key prompt for this stage is as follows:

````python
"""
Make sure to have consistent filenames if you reference other files we are also generating.

Remember that you must obey 3 things:
    - you are generating code for the file {current_file}
    - do not stray from the names of the files and the plan we have decided on
    - MOST IMPORTANT OF ALL - every line of code you generate must be valid code. Do not include code fences in your response, for example

Bad response (because it contains the code fence):

"```javascript
console.log("hello world")
```"

Good response (because it only contains the code):
console.log("hello world")

Begin generating the code now.
"""
````

## Conclusion

This is it, believe it or not. 3 distinct function calls (`generate_code` does get invoked for every file but we can consider it as 1 distinct invocation). Promise of [Karpathy's Software 2.0](https://karpathy.medium.com/software-2-0-a64152b37c35) where instead of operating using mostly deterministic chunks of code, we instead rely on probabilistic models and use their outputs and inputs into themselves.

This stuff is mostly vanila OpenAPI GPT where creativity/secret sauce is concentrated in the prompting phase. When customizing this for your specific use cases, this is where you will spend the bulk of your energy, learning to negotiate with the model to produce better results.
