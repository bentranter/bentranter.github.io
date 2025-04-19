---
author: "Ben Tranter"
title: "RAG Hallucination Scoring Hypothesis"
date: "2025-04-18"
description: "Scoring RAG hallucinations with simple TF-IDF scores"
draft: true
---

Suppose you have an LLM and a dataset to use for retrieval-augmented generation (RAG). When you query your LLM, how do you know if it's hallucinating?

The current state-of-the-art for hallucination detection is to simple ask _another_ LLM if the first one is lying. This works surprisingly well (there are [entire companies built around this](https://docs.galileo.ai/galileo/gen-ai-studio-products/galileo-evaluate/how-to/identify-hallucinations#identify-hallucinations)), but like an LLM, there are two issues:

1. It also hallucinates.
2. It's slow.

So what else can you do?

In the recent [RaDIO paper](https://ojs.aaai.org/index.php/AAAI/article/view/34809/36964), part of their implementation is to use TF-IDF scoring:

> To this end, RaDIO calculates the TF-IDF score (TD) of each word in the text to identify words with high information density enabling the system to prioritize content-rich tokens over less informative ones, thereby better capturing key parts of the information.

In the spirit of, "X is all you need", what if we **only** used TF-IDF? Unlike general hallucination detection, for RAG, we have access to the entire knowledge base. A reply with a disproportionately high TF-IDF score would surely signal a likely hallucination.
