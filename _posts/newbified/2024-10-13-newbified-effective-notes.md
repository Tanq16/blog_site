---
title: Effective Note-Taking
date: 2024-10-13 06:21:00 -0500
categories: [Computers Newbified]
tags: [markdown,note-taking,productivity,obsidian]
image:
  path: /assets/img/covers/markdown-cover.jpeg
  alt: Markdown Artwork
---

## Intent

I'm not the best at taking notes, and I've been pursuing a goal for improving my note-taking ever since I started my Masters degree. During that time, I discovered [Notion](https://www.notion.so/), which gave me a lot of control over my notes, inspired me to take more notes and build a "dashboard" of sorts, popularized a lot on YouTube. I also discovered the concept of [Building a Second Brain](https://www.buildingasecondbrain.com/) by **Tiago Forte** and one of his organization methods called [PARA](https://fortelabs.com/blog/para/) (Projects, Areas, Research, Archives). This kept me excited for a time, and I took decent notes for my Masters. Soon after, I started working and felt like my note-taking took a nosedive. Nothing was working as I wanted it to, and I was confused not only about what notes to take but also about finding things I had already taken notes on.

## More Research

While Notion was also adding weird (to me) subscription tiering that limited the number of blocks and storage I could use, I wanted to shift to something I might stick with for a longer term. So, I ended up trying all of these &rarr;

- [Craft](https://www.craft.do/) (Apple only)
- [Obsidian](https://obsidian.md/)
- [Trilium](https://trilium.rocks/)
- [VS Code](https://code.visualstudio.com/) (local files and some plugins)
- [LogSeq](https://logseq.com/)
- even Google Docs, but it was way too slow for my use case

This was also a time I was setting up a home lab, so I was in a mood to have complete control over my data and use open-source solutions wherever feasible. By trying out several applications and platforms this way, I found that three major things make up the art of note-taking &rarr;

- A method for note-taking, which includes a per-note format and an organization method
- Application or software gimmicks
- Notes preservation, sync, and migration-ability

### Note-Taking Methods & Organization

There are a LOT of methodologies for note-taking. Of course, people create their methodologies all the time. But, the most famous ones are these &rarr;

- [Zettlekasten](https://zettelkasten.de/overview/) &rarr; Atomic notes with attributes and connections
- [Outline](https://www.goodnotes.com/blog/outline-note-taking-method) &rarr; Indented bullet lists detailing main points
- [Cornell](https://lsc.cornell.edu/how-to-study/taking-notes/cornell-note-taking-system/) &rarr; Strict note format aimed at improving core understanding

Each of these has a loose or strict organization mechanism. Generally speaking, how the notes are represented also depends on the software or application being used. Typically, it is also tied to how the notes are written - markdown-compatible (Markdown supported rich text), Markdown (pure Markdown and/or latex), or something else (rich text, plain, handwritten). However, there will always be a loose concept of files, folders, subfolders, and tags.

> There is another class of managed note-storing/processing applications that automatically handle organization, recall, and ingestion of information. Such applications, however, are more challenging to use in computer science-related fields. If they work for someone, great, but I won't include them in this blog.
{: .prompt-info }

### Gimmicks

Gimmicks provided by various applications are their USPs; all applications have some. Example &rarr;

- ***Craft Calendar Notes*** &rarr; Craft allows daily notes to be accessible through a calendar UI. Plugins with other applications may also allow something similar.
- ***Obsidian & LogSeq Graph*** &rarr; These two applications represent notes as graphs, which look very cool and represent connections within notes. However, the utility is limited to very particular use cases.
- ***Notion AI*** &rarr; Allowing LLMs to generate, summarize texts, and answer things is a new offering in various applications.
- ***Notes Publish*** &rarr; Applications like Craft, Obsidian, and Notion allow you to publish notes online with just a few clicks, making sharing very easy.
- ***Craft UI & PDFs*** &rarr; Applications, especially Craft, allow you to generate extremely good-looking PDF documents for sharing or marketing content.

### Takeaways

After a lot of deliberation, migration, trial, and error, I arrived at the following parameters that are crucial to me for a note-taking system &rarr;

- ***No Blocks*** &rarr; The note-taking system should have its basic unit be a file containing text rather than a block represented in the filesystem as a database entry or a JSON field. Block-based systems are harder to be compatible universally and don't allow granular control over the knowledge base.
- ***Pure Markdown Syntax*** &rarr; Given the basic unit is a file, it makes sense for the note-taking system to understand and use pure Markdown because custom syntax with Markdown support cannot be easily translated universally. Additionally, computer science and cybersecurity notes are best suited as Markdown artifacts.
- ***Flat File Storage*** &rarr; A flat file structure of the knowledge base not only allows granular control but also makes it easy to integrate custom scripts and code that don't necessarily depend on the note-taking system itself. An example is using custom AI modules (take a look at my thoughts on gimmicks below).
- ***Easy Backups*** &rarr; Typically, if a note-taking system respects all the points stated above, it will automatically be easy to backup. Support within an application or software is excellent, but a note-taking system should allow using any backup method the user chooses.
- ***Version Control (Low Priority)*** &rarr; Version control is always great, whether it involves rolling changes back or just seeing how something has changed over time. It isn't a crucial part of a note-taking system but can be occasionally useful.

Contrary to popular belief, here are my thoughts on some of the gimmicks &rarr;

- ***AI Features*** &rarr; Inbuilt AI features can be helpful for specific use cases, but I prefer this to be a separate functionality that is not dependent on the note-taking application. That would allow the implementation of self-written code and projects like Ollama to integrate custom LLMs for specific use cases, such as using Ollama and a custom RAG implementation for the knowledge base.
- ***Online Publish*** &rarr; This is only useful if the user wishes to publish many notes rapidly without any modification. Otherwise, using a hosted blog built via a static site generator (for example, the blog you're reading this article on) sounds like a better option to me.
- ***Note Graph*** &rarr; Realistically, graphs are only helpful if you have a large number of connected notes and there isn't a set organization structure of the knowledge base. They can be helpful in searching for information, although I still find them limiting. In my opinion, the "cool factor" outweighs the usability. Also, it's far less helpful for the setup I arrived at (as you'll see in the next section).

## Current Setup

### Software & Philosophy

> ***Philosophy*** &rarr; Notes on a given topic should be easy to find, and any single note should not confuse the user with multiple topics contained within, i.e., they should have atomic context. Additionally, the system should act as a one-stop solution for everything the user wants to write and store without limiting the addition of other software or applications. The system should allow full control over the data without limiting migration to other note-taking software or applications. Lastly, it should inspire and make it easy to take notes, irrespective of whether they end up as published content or knowledge material.
{: .prompt-info }

Based on this philosophy and all the research and experiments I conducted, my current setup uses ***Obsidian*** as the backbone. I don't conform to a particular note-taking methodology; however, my methodology is derived from and is an amalgamation of several famous ones mentioned earlier. I also follow a custom organization structure. Obsidian best suits me because of the flat file structure, pure Markdown support, inline-Latex support, and Mermaid diagram support.

### Sync

Though Obsidian offers Obsidian Sync as an add-on subscription to allow seamless note syncing, I prefer to do the sync manually. I used to do this using iCloud but later shifted to using a GitHub repository to accommodate sync between my personal devices and work machine.

- ***GitHub Backend*** &rarr; The Obsidian Vault I use is a collection of simple files and folders that are version-controlled through GitHub.
- ***Obsidian Git Plugin*** &rarr; I use a GitHub PAT for syncing the repository. Although I can manually sync to GitHub via CLI, I manage this through the Obsidian Git plugin. The plugin also has a port for iOS and Android that allows syncing on mobile devices.

Another reason for using GitHub is that it renders Markdown natively, including Mermaid diagrams. Therefore, even if I plan to switch from Obsidian to another Markdown and flat file structure-based application, the migration can be seamless, and the knowledge base is equally readable on GitHub without relying on the application of the note-taking system. Basically, GitHub is a decoupling agent for the note-taking system and the note-writing application.

### Note Format

After much trial and error, I arrived at a specific note format. Broadly speaking, notes can be of two types &rarr;

- ***General Note*** &rarr; These are notes I'd take while doing research for learning cybersecurity, studying for work, exploring new TTPs for work, etc.
- ***Personal Note*** &rarr; These are notes that either relate to storing useful personal information or essay-like documents. For example, learning about the USA's tax system, collecting quotes, course material, and even this blog post would all fall under the personal note category.

The Personal Note type does not have a fixed format. The idea is to keep this free-form with an arbitrary number of headings and structures as long as the atomicity is satisfied, i.e., each note talks about a singular topic only (this topic could contain several sub-topics).

The General Note, however, is special - it follows a set format and strictly deals with a singular topic only. Subtopics are intended to be kept to a minimum. These are also the ones that are most methodically organized under specific file paths. The format for these notes is strict because they are what I refer back to the most. This particular format helps make revisiting, updating, understanding, and sharing the note much easier.

> Additionally, Obsidian supports Markdown callouts that help increase readability. While Obsidian supports several callout tags, I only use the `TIP`, `WARNING`, and `INFO` callout tags to keep things consistent with GitHub.
{: .prompt-tip }

The format is represented as follows &rarr;

```
[resources](A bullet list of resources at the top of the page without any heading)

[heading](The atomic topic of concern)
[sub-headings](Sub-headings should be avoided, but if needed a couple will follow in a typical fashion)

[helpers](Change the name appropriately to store helping artifacts like scripts, graphs, stray images, examples, etc.)
```

Sticking to this format helps solidify how the note-taking system works.

> Each file is named `TYPE - topic` where `TYPE` is `Concept` or `Process`. I use `Methodology` instead of `Process`, but the idea is that a concept describes information about something. In contrast, a process describes how the information in the note can be used to perform something specific.
{: .prompt-tip }

### Organization

Since Obsidian uses a flat file structure, the organization is totally up to the user. My Organization is as follows &rarr;

```
_Archives
_Attachments
0 - Personal
1 - Work
2 - Cloud Security
3 - Application Security
4 - Network Security
5 - Coding and General
6 - Courses and Learning
7 - Homelab Collection
Tracker.md
```

In the above list, I use the `Tracker.md` file as an inbox for tasks, links, plans, etc. However, the important thing is that no information that can potentially increase the size of the file should be added to it. It's just temporary information.

> I use the numbers 0, 1, 2, etc., before the folder names to order the display order based on what I want it to look like. Also, the \_Attachments folder is meant to store ALL attachments irrespective of the note to which they're added.
{: .prompt-tip }

By convention, one should only go as deep as five levels in the hierarchy. Two sub-levels inside a folder should be sufficient.

## FIN

And that's it! That's the note-taking system I developed based on experimentation and research. It works for me, but what I want most from this blog post is to impart the following message &rarr; _A note-taking system is as efficient as you make it. But what's essential is taking notes rather than getting hung up on following or perfecting a system_.
