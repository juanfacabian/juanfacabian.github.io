---
title: "When and How to Ask: Dynamic Preference Elicitation Strategies for Conversational Recommender Systems"
permalink: /blog/when-and-how-to-ask/
layout: default
---

<p class="blog-post-meta">Apr 25, 2026 | Feng Xia</p>

This post is a research note for our SIGIR 2026 paper on conversational recommender systems (CRSs). The paper asks a very practical question: if a recommender can talk to users, when should it ask follow-up questions, and what kind of questions should it ask?

My short answer is simple: **there is no single best elicitation strategy for the whole conversation**. Early in the dialogue, users usually need help exploring the space, so attribute-based questions work better. Later, once preferences become clearer, item-based questions become more effective. The paper turns that intuition into a dataset, a modeling framework, and an empirical study.

<figure class="paper-figure">
  <img src="{{ '/assets/images/papers/when-how-ask-stage.png' | relative_url }}" alt="User choices between attribute-based and item-based elicitation strategies across dialogue stages">
  <figcaption>
    A central finding from the paper: user preference for elicitation strategies changes with dialogue stage. Attribute-based prompts dominate early; item-based prompts become stronger later.
  </figcaption>
</figure>

## About

Conversational recommender systems differ from traditional recommenders because they can actively gather preferences during the interaction. That sounds powerful, but it introduces a policy question that many systems still handle in a fairly static way. Some systems mostly ask attribute questions, such as genre or actor preferences. Others rely more on item-level comparisons. In practice, both are useful, but not at the same time and not for the same reason.

This paper studies preference elicitation as a **stage-aware decision problem**. Instead of assuming one fixed strategy, we ask whether a CRS should switch strategy as the conversation moves from vague intent to refined choice.

## The Core Question

The paper is organized around three research questions:

1. Is a single preference elicitation strategy sufficient across dialogue stages?
2. Do different elicitation strategies work better at different stages?
3. Does explicitly modeling strategy selection improve conversational recommendation?

The overall answer is yes to the second and third questions, and effectively no to the first.

## What We Found in the User Study

We first ran a preliminary user study with **328 participants**. Participants read conversational recommendation scenarios and selected their preferred system responses. Because each response represented a different elicitation strategy, this gives us direct evidence about what kinds of questions users prefer in different settings.

Two findings mattered most.

First, a static strategy is not enough. For attribute-based elicitation, only **21%** of participants considered it sufficient across all situations; **53%** said it worked in most situations, and **25%** said it only worked in a few cases. Item-based elicitation showed the same pattern: **25%** judged it sufficient across all situations, **59%** said it worked in most situations, and **16%** said it only worked in a few cases.

Second, strategy preference changes with the stage of the conversation. In the early stage, when users are still exploring what they want, attribute-based questions are preferred. As the dialogue progresses and preferences become more concrete, item-based elicitation steadily becomes more attractive and eventually overtakes attribute-based questioning.

That shift is the conceptual heart of the paper: **good elicitation moves from abstract preference exploration to concrete preference refinement**.

## A Concrete Example

The paper also gives a nice intuition for why stage matters. Suppose a user starts with:

> I'm looking for a movie tonight.

At that point, asking "What kind of movies do you usually enjoy?" is more helpful than asking the user to choose among specific titles. But once the user says they want a light comedy and shows interest in something like *What We Do in the Shadows*, the system has enough context to ask a more concrete question, such as comparing a small set of candidate items.

<figure class="paper-figure paper-figure--narrow">
  <img src="{{ '/assets/images/papers/when-how-ask-flow.png' | relative_url }}" alt="Annotation workflow for determining whether preference elicitation is needed and which strategy to use">
  <figcaption>
    Part of the annotation workflow behind InPE: first decide whether a turn needs elicitation, then choose a strategy, generate strategy-specific candidates, and keep the update only if it improves on the original response.
  </figcaption>
</figure>

That transition is exactly what the paper tries to model, and InPE gives us the annotation structure to do it in a systematic way.

## InPE: A Dataset for Strategy-Aware CRS

To study this properly, we built **InPE** (InSPIRED Preference Elicitation), an annotated and augmented version of the INSPIRED dataset. INSPIRED already contains 999 multi-turn movie recommendation dialogues, but it does not provide strategy-level labels. We enrich it with annotations for:

- whether a turn needs preference elicitation at all;
- which elicitation strategy is preferred;
- whether a strategy-aware rewrite is better than the original system response.

The final dataset used in experiments contains **6,719 turn-level samples**. Annotators label elicitation turns as **attribute-based**, **item-based**, or **hybrid**. The distribution is interesting on its own: **hybrid** strategies account for about **47.03%** of elicitation cases, followed by **item-based** at **28.85%** and **attribute-based** at **24.12%**. That tells us effective elicitation is often not purely one thing or the other.

The human preference comparisons are also strong. When evaluators compared strategy-aware rewrites against original responses, they preferred the strategy-aware versions in **89.34% to 95.5%** of cases depending on the strategy category.

## COPE: Modeling When and How to Ask

On top of InPE, the paper proposes **COPE**, which stands for **COnversational Preference Elicitation via Mixture of Experts**.

The key idea is to treat conversational recommendation as a structured decision problem rather than a generic text-generation task. COPE uses a shared frozen LLM backbone together with task-specialized experts and a **two-stage router**:

1. a task-level router decides whether the system should elicit preferences, recommend items, or continue general interaction;
2. if elicitation is needed, a strategy-level router chooses among attribute-based, item-based, and hybrid elicitation.

This matters because "should I ask now?" and "what kind of question should I ask?" are different decisions. COPE models them explicitly instead of hoping a single monolithic generator will implicitly discover both.

## Main Results

The offline and pairwise results are quite encouraging.

On recommendation quality, COPE achieves the best recall across all reported cutoffs on InPE, including:

- **Recall@1 = 0.144**
- **Recall@10 = 0.314**
- **Recall@50 = 0.442**

The paper notes that the strongest baseline, ReFICR, still falls short, especially at deeper cutoffs. That is important because it suggests the gain is not just from using an LLM, but from explicitly modeling proactive strategy selection.

On pairwise response evaluation, COPE reaches a **win rate of 0.604**, again outperforming the baselines. This tells us the benefit is not only in retrieval metrics; users also prefer the resulting responses.

One especially useful analysis in the paper is the routing intervention result. If the system is forced to use the correct expert and strategy labels, **Recall@10 rises to 0.342**. In other words, the experts themselves are strong, and one of the remaining bottlenecks is routing accuracy. That gives a clear direction for future work.

## Why I Think This Paper Matters

What I like most about this work is that it makes a subtle but important point concrete: conversational recommendation is not just about ranking items better. It is also about **interacting better**.

If a system asks attribute questions for too long, it can feel repetitive and slow. If it jumps to concrete item comparisons too early, users may not yet have enough context to answer well. A good CRS needs to manage that transition.

This paper contributes three things that make that possible:

- evidence that strategy choice is stage-dependent;
- a benchmark dataset for learning these choices;
- a modeling framework that treats elicitation as explicit decision making.

For me, the most memorable takeaway is not just "attribute early, item later." It is the broader idea that **elicitation policy should evolve with the conversation**, and that we can model that evolution directly.

## Closing Note

If you are interested in proactive conversational systems, I think this is a useful lens: recommendation quality and interaction strategy should be studied together, not separately.

The InPE dataset is available here:

[https://github.com/juanfacabian/InPE](https://github.com/juanfacabian/InPE)

