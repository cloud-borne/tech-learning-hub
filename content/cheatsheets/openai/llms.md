---
title: Large Language Models
linktitle: 🔠 Large Language Models
type: book
date: "2024-06-17T00:00:00+01:00"
tags:
  - AI
  - ChatGPT
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 10
---

<!--more-->
## Overview

###### Artificial Intelligence

AI is when machines learn, think, and do things that usually need human intelligence. It's like having a robot assistant that can help you!
Simply put AI is a computer program that can learn, think, and make decisions like a **human** 🧠. Using data and algorithms to solve problems and improve over time using feedback loops.➿

###### Generative AI

Generative AI refers to Artificial Intelligence that can generate ```new content or data``` that resembles **human-generated** examples.

###### Large Language Models 

Large language models (sometimes referred to as **GPT** models like ```GPT-4```) are a type of ```Generative AI``` models that are trained on massive amounts of text data. They can generate text, translate languages, write different kinds of creative content, and answer your questions in an informative way.
 
**LLMs** are linked to AI in several ways:

* They are trained using Al techniques, such as deep learning.
* They are used to perform AI tasks, such as natural languagem processing (```NLP```).
* They are often used in conjunction with other Al systems, such as chatbots and virtual assistants. 

LLMs are a rapidly developing area of AI research, and they are being used to power a wide range of new and innovative applications. As LLMs continue to improve, they are likely to play an even greater role in the future of AI.

## ChatGPT Vs Gemini

[ChatGPT](https://openai.com/chatgpt/) and [Gemini]() are both large language models (LLMs) trained to generate human-quality text. They have access to a massive amount of text data, which allows them to generate text that is both coherent and informative.

However, there are some key differences between the two models. ChatGPT is a generative **pre-trained transformer** model, while Gemini is a **factual** language model. This means that ```ChatGPT``` is better at generating **creative** text formats, like poems, code, scripts, musical pieces, email, letters, etc., while ```Gemini``` is better at providing summaries of **factual** topics or creating different kinds of creative text formats.

| **Feature**   | **ChatGPT**                                             | **Gemini**                                                                                 |
|---------------|-------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| Type of model | Generative pre\-trained transformer                         | Factual language model                                                                     |
| Strengths     | Generating creative text formats                            | Providing summaries of factual topics or creating different kinds of creative text formats |
| Weaknesses    | Can sometimes generate inaccurate or misleading information | May not be as creative as ChatGPT |

## Custom GPTs

You can also create ```custom``` versions of ChatGPT that combine instructions, extra knowledge, and any combination of skills.
 
* GPTs let you **customise** ChatGPT for a specific purpose
* The best GPTs will be invented by the community
* GPTs have been built \"with privacy and safety in mind\"
* Developers can connect GPTs to the real world
* Enterprise customers can deploy internal-only GPTs

## GPT Plugins

ChatGPT ```Plugins``` are extensions designed to enhance the capabilities of LLMs. Plugins allow ChatGPT to integrate with various external services and data sources, thereby extending its functionality beyond core conversational abilities.

Though not a perfect analogy, plugins can be like “eyes:eye: and ears:ear:”  for language models, giving them access to information that is too **recent**, too **personal**, or too **specific** to be included in the training data. In response to a user’s explicit request, plugins can also enable language models to perform safe, constrained actions on their behalf, increasing the usefulness of the system overall.

 The first plugins were created by [Expedia](https://www.expedia.com/), [Instacart](https://www.instacart.com/), [KAYAK](https://www.kayak.com/) and companies like such.

| **Feature**              | **Custom GPTs**                                                               | **GPT Plugins**                                                                             |
|--------------------------|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| Creation Process         | No\-code, built within ChatGPT using instructions and examples                | Code\-based, built outside ChatGPT using OpenAI API                                         |
| Technical Skill Required | Minimal                                                                       | Programming knowledge and API understanding<br>                                             |
| Focus                    | Specific niche domains or skills                                              | Broad and diverse capabilities, including external<br>integrations                          |
| Strengths                | Easy to build, quick prototyping, highly specialized<br>                      | Powerful and versatile, more control over functionality,<br>access to external resources    |
| Weaknesses               | Limited functionality, less control over code, no external<br>data/API access | Time\-consuming development, technical expertise<br>required, compatibility issues possible |
| Examples                 | Code GPT for Python, Poetry GPT, Customer Service<br>Assistant GPT            | ChatGPT Translate Plugin, Dall\-E 3 integration, Financial<br>Analysis Plugin               |
| Development Time         | Minutes to hours<br>                                                          | Hours to days \(depending on complexity\)<br>                                               |
| Cost                     | Free within ChatGPT limitations                                               | Varies depending on plugin and hosting                                                      |
| Suitable for             | Non technical users wanting quicn customizaton, niche                         | Power users seeking extensive functionality, complex                                        |

{{% callout note %}}

Ultimately, the choice between custom GPTs and plugins depends on your needs and technical skills. 
* 👉 If you want a ```quick``` and ```easy``` way to focus ChatGPT on a specific area, **custom GPTs** are a good option. 
* 👉 If you need more ```power``` and ```flexibility``` for complex tasks, learning to build **plugins** gives you greater control and possibilities.

{{% /callout %}}

Learn more: 
* [📖 API Reference](https://platform.openai.com/docs/api-reference/models)
* [🧠 Awesome ChatGPT Prompts](https://prompts.chat/)
* [🥳 PartyRock](https://partyrock.aws)
* [🔌 ChatGPT Plugins](https://openai.com/index/chatgpt-plugins/)