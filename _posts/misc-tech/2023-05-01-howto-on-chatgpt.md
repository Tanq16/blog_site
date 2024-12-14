---
title: Unleashing ChatGPT - A Guide for Professional Use
date: 2023-05-01 00:57:00 -0600
categories: [Computers and Security]
tags: [chatgpt,programming,productivity]
image:
  path: /assets/img/covers/gpt-cover.jpeg
  alt: ChatGPT Artwork
---

## ChatGPT Primer

OpenAI developed a state-of-the-art language model based on the GPT architecture that uses deep learning to generate human-like responses to natural language input. It ingests text input and processes it in multiple steps to refine the understanding of the input. It then uses autoregression to generate a response one token at a time, using the previous output to generate the next token. The model has been trained over a massive dataset of websites, documents, and books up to the end of 2021.

## Understanding It Better

ChatGPT is very powerful, as we'll soon see, but it is important to recognize its limitations. It's important to note that ChatGPT is not Artificial General Intelligence (AGI) and should not be thought of as such; instead, it's an AI model aimed at text generation. This means that it cannot generate pure conscience-based logic but rather is a text-generation model. That's why there'll always be limitations in what it can come up with.

The quality of the output is highly dependent on the input it receives. Because it generates output from the corpus it is trained on, it may generate biased or incorrect responses. As such, it shouldn't be considered a "source of truth". It can come up with grammatically correct text or syntactically correct code, but all of that is a testament to how good the model is, not to how "sentient" it is. ***Spoiler alert: it is not sentient***, it's a generator, and you can pick up on that if you use it and experiment with it enough.

However, ChatGPT can significantly increase productivity and provide useful insights for several professional jobs when used smartly. Let's see how!

## Example Use Cases

Following here is a set of use cases that I've performed with the help of ChatGPT and should be enough to showcase exactly how it can fit into professional workflows.

### Ideation and Language Generation

As established before, ChatGPT has been trained on a huge amount of text, so it's quite easy for it to generate example language for things like blog posts, emails, topic titles, etc. Try these out &rarr;

> Write me a sample letter of recommendation for a bachelor's student for his outstanding performance on academic projects and examinations. Also, mention his excellent research work on 3 papers published in famous journals on artificial intelligence. End it with recommending the student for further studies commending his high aptitude for research and problem-solving.
{: .prompt-info }

> Build me a step-by-step plan for programming a Chrome extension that keeps track of the time spent on each tab and reports when I spend more than 1 hour on a tab.
{: .prompt-info }

> I am writing a blog post on how to get started in binary exploitation. I will go through the basic concepts and introduce advanced topics such as ASLR and heap exploitation. Suggest me some good titles for this post.
{: .prompt-info }

Now read the responses and see how usable those are! That's the strength of ChatGPT - take in text dumps, contextualize them, and produce meaningful text. For all these queries, it's super easy to use the responses as a base and create something unique and impactful out of it. I used ChatGPT to generate quite a bit of language for my previous blog post, [Streamlining Security-Related Workflows with Docker Containers](https://tanishq.page/blog/posts/docker-for-security/).

### Conceptual Learning

Since ChatGPT is trained on public websites with documentation for several kinds of service providers such as AWS, GCP, Kubernetes, etc., so it's an awesome entry point to getting information about theoretical concepts. Try these &rarr;

> What are admission controllers in Kubernetes? Give me an example and tell me how to deploy it in my cluster.
{: .prompt-info }

> What are the security best practices for exposing an RDS instance to the public? If I want to give direct access to an RDS database to certain developers, what would be the best way to do so?
{: .prompt-info }

> Explain to me what Azure Active Directory is and how can an example startup company use it with an e-commerce coupon service offering.
{: .prompt-info }

***NOTE:*** This is the place where people get tripped up. Do you consider this information to be the source of truth? Absolutely not! There are high chances of the information being correct, but the intent is to get a general sense of what you're looking for. You save time by getting an introduction to a topic right away in a concise manner rather than spending the next 10-15 minutes doing Google-fu. Then after a quick 2-3 minute read of the information, you can start researching the things that matter based on the terms and keywords you get from ChatGPT's result. Overall, this has the potential to cut your research time in almost half, and that's huge!

### Skip Library Documentation

Often, we encounter situations where we need to look up SDK documentation to understand source code or to get to know how to perform a certain action in a given programming language. This is especially true for cybersecurity, where there is a need to understand and analyze source code quickly. There could also be situations where one is working on a project but has to incorporate a different language for a particular segment. Having documentation at your fingertips is very useful in such situations. Since ChatGPT has ingested documentation from the internet, it's easy to generate text related to specific libraries or functions within those libraries. Here's an example &rarr;

> What functions should I use to implement storage of variables in Keychain in Flutter?
{: .prompt-info }

Look at the response, and you can implement that part in Flutter without reading any documentation and looking up examples. That's a really quick way to get information. But once again, this isn't the source of truth! It will produce accurate information almost all the time, but definitely fact-check for super-niche things.

### Programming and Scripting

Scripting is one of the most common activities that a cybersecurity professional needs to perform. Even otherwise, computer science professionals often need to generate definitions of functions to make their job easy. Look at the following examples &rarr;

> Write a function in PHP that implements a rot13 cipher for the provided text, which is obtained from a GET request parameter.
{: .prompt-info }

> Write me a Python script that checks the current IP address and returns a red colored text "APIPA" if the machine has an APIPA address, otherwise tests a lookup to "google.com" and returns "Success" in green if it succeeds or "Failure" in red if it fails.
{: .prompt-info }

Now here is the catch &rarr; ChatGPT will generate code based on numerous examples that it has ingested from a huge corpus of text. Quite often, that code will not execute as expected outright. But we're all smart people and can do minor tinkering to ensure everything is as we want and get a working code at the end. The catch is, instead of spending time writing it ourselves from scratch and trying to read documentation or StackOverflow for stuff that we don't have in our brains, we leverage ChatGPT to generate a starting point for us. Once there, it's super easy to repackage that to obtain the desired end result. This has saved me a ton of time when writing scripts for different things.

### Programming to the Max

We've seen how ChatGPT can help us generate code for scripts or helpful function definitions when we need them. Here's some more advanced stuff, which is very hit-or-miss from a conversation continuity point of view, and there's some wording hacks that we need to employ &rarr;

> Write me IaC in Terraform to deploy a Lambda function with a tag production: true and a lambda execution role with the permissions of s3:ListBuckets, s3:GetObjects, s3:ListObjects on all resources. Additionally, configure an event bridge trigger to invoke the Lambda function using a cron job every day at midnight.
{: .prompt-info }

Literally, this output is insane!! Look at how much time it saves you! Here's another example use case &rarr;

> Write me a step-by-step plan to build a website that presents a form to the user and accepts contact information for the user in that form. With all the information received, the data is stored in an SQLite database, and the user is prompted "Subscribed". I want to build this using Flask and deploy it in a docker container. In the backend, in the same container, I also want to run a cron script that sends an email with a SendGrid API token passed as an environment variable to the container to all the users in the SQLite database. I also want an appropriate Dockerfile for the container. Write me the numbered step-by-step plan, and I will ask you about details for each step by referring to its number.
{: .prompt-info }

Follow this with interactions like &rarr;

> Give me the details of steps 1 and 2 from this plan.
{: .prompt-info }

And you can literally get help implementing this entire project from the ground up! Think of how much time ChatGPT would save for such a project. That is AWESOME if nothing else, but you have to agree it is super helpful.

## Word of Caution

Always keep 2 things in mind &rarr;

>ChatGPT is not the source of truth for any information.
{: .prompt-danger }

What's more, technically, it's outdated, so if you need more up-to-date information, prefer Google Bard or Bing Chat with low creativity factors.

>DO NOT write sensitive information into ChatGPT text prompts.
{: .prompt-danger }

People will always make mistakes, and those mistakes are the primary cause of leaks and compromise. Special note on ensuring that you do not enter any information related to your company's IP (intellectual property) or your clients' IP.

## Outro

Being able to generate code, provide detailed information on specific topics, etc., makes ChatGPT an amazing tool that can help save hours of time and increase productivity. Despite its limitations, ChatGPT is one of the best tech tools that has surfaced recently. It's a valuable tool in the computer science and cybersecurity fields. I hope this post provides insights into how ChatGPT should be used in the intended way to improve your workflow.
