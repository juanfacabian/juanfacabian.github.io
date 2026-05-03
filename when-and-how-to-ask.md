---
title: "When and How to Ask: Dynamic Preference Elicitation Strategies for Conversational Recommender Systems"
permalink: /blog/when-and-how-to-ask/
layout: default
---

<p class="blog-meta">4.25, 2026 &nbsp;&nbsp; Feng Xia</p>

Conversational recommender systems are becoming increasingly common in domains such as movies, music, travel, and e-commerce. Instead of only relying on historical clicks or ratings, these systems can talk to users, ask questions, clarify needs, and gradually refine recommendations through multi-turn conversations.

But this raises a simple yet important question:

**When should a recommender system ask the user for more preferences, and how should it ask?**

This question is the focus of our SIGIR 2026 paper:

> **When and How to Ask: Dynamic Preference Elicitation Strategies for Conversational Recommender Systems**  
> Feng Xia, Shuo Zhang, Xi Wang  
> SIGIR 2026

In this work, we study preference elicitation in conversational recommender systems from a stage-aware perspective. Our main finding is that there is no single best way to ask users about their preferences. Instead, the best elicitation strategy depends on where the conversation is and how clearly the user has already expressed their needs.

## Background: Preference Elicitation in CRS

A conversational recommender system, or CRS, aims to recommend items through natural language interaction. Compared with traditional recommender systems, a CRS has one major advantage: it can actively collect user preferences during the conversation.

For example, if a user says:

> I want to watch a movie tonight.

The system may respond in different ways:

> What kind of movies do you usually enjoy?

or:

> Would you prefer something like *Jojo Rabbit* or *What We Do in the Shadows*?

These two responses represent different preference elicitation strategies.

The first is **attribute-based elicitation**. It asks about item attributes, such as genre, actor, director, mood, price range, or location.

The second is **item-based elicitation**. It asks users to compare or choose among concrete items.

Most existing conversational recommender systems tend to rely on a relatively fixed strategy. They may keep asking attribute questions, or they may repeatedly present items for comparison. However, real conversations are dynamic. A user may begin with vague needs and gradually form more concrete preferences. This suggests that the system should also change how it asks questions over time.

## Core Question

The paper investigates three research questions:

1. **Is a single preference elicitation strategy sufficient across different dialogue stages?**

2. **Do different elicitation strategies work better at different stages of a conversation?**

3. **Can explicitly modeling elicitation strategies improve conversational recommendation performance?**

Our answer is: yes, strategy matters; and more importantly, strategy should change dynamically.

## A Preliminary User Study

We first conducted a user study with **328 participants**. Participants were shown conversational recommendation scenarios and asked to choose which system response they preferred.

The results show that a static strategy is not enough.

For attribute-based elicitation, only **21%** of participants thought it was sufficient across all situations. For item-based elicitation, only **25%** thought it was sufficient across all situations. Most users preferred different strategies in different contexts.

More interestingly, we observed a clear stage-dependent pattern.

In the early stage of a conversation, users usually have vague needs. At this point, attribute-based questions are more useful because they help narrow down the search space:

> What kind of movies do you like?

> Do you prefer something light-hearted or serious?

Later in the conversation, once the user has already provided some preferences, item-based elicitation becomes more effective:

> Between these two movies, which one sounds closer to what you want?

> Would you prefer this option or something similar to that one?

This reveals a natural transition:

**from abstract preferences to concrete choices.**

## The InPE Dataset

To support this line of research, we introduce **InPE**, short for **InSPIRED Preference Elicitation**.

InPE is built on top of the existing INSPIRED conversational recommendation dataset. The original dataset contains multi-turn recommendation dialogues, but it does not provide fine-grained annotations for preference elicitation strategies. We enrich it with new labels that indicate:

- whether a dialogue turn requires preference elicitation;
- which strategy should be used;
- whether a strategy-aware response improves the original system response.

The annotation process combines LLM-based filtering with human verification. We first use an LLM to identify candidate turns that may require preference elicitation. Human annotators then verify these turns and label the preferred strategy.

In total, the dataset contains **6,719 turn-level samples**. Among them:

- **46.3%** are general interaction turns;
- **19.8%** are preference elicitation turns;
- **33.9%** are recommendation turns.

For preference elicitation turns, the annotated strategies include:

- **attribute-based**;
- **item-based**;
- **hybrid**, which combines both item and attribute information.

One important observation is that hybrid strategies are very common. In the annotated data, hybrid strategies account for around **47%** of elicitation cases, followed by item-based and attribute-based strategies. This suggests that effective elicitation often requires combining different types of preference cues rather than relying on only one.

## Example: How Strategy Changes During a Conversation

Consider a movie recommendation conversation.

At the beginning, the user may simply say:

> I want to watch a movie.

The system does not know much yet, so an attribute-based question is appropriate:

> What kinds of movies do you usually enjoy?

Suppose the user then says:

> I like something light-hearted, maybe similar to *What We Do in the Shadows*.

At this point, the system has more concrete signals. It can move to an item-based strategy:

> Between *What We Do in the Shadows* and *Jojo Rabbit*, which one would you prefer?

After the user gives more feedback, the system may be ready to make a recommendation:

> I recommend *Hunt for the Wilderpeople*.

This example illustrates the central idea of our work: preference elicitation should not be treated as a fixed behavior. A good CRS should decide when to ask, what to ask, and when to recommend.

## COPE: Conversational Preference Elicitation via Mixture of Experts

Based on these observations, we propose **COPE**, a new architecture for conversational recommendation.

COPE stands for:

**COnversational Preference Elicitation via Mixture of Experts**

The key idea is to explicitly model different conversational behaviors with different experts. Instead of using one model to handle all turns in the same way, COPE contains specialized modules for different actions, such as:

- general conversation;
- preference elicitation;
- recommendation.

For preference elicitation, the model further decides which strategy to use:

- attribute-based elicitation;
- item-based elicitation;
- hybrid elicitation.

COPE uses a hierarchical router. At each dialogue turn, the router first predicts the high-level action: should the system elicit preferences, recommend an item, or simply continue the conversation? If preference elicitation is needed, the router then predicts the most suitable elicitation strategy.

This allows the system to dynamically switch strategies as the conversation evolves.

## Experimental Results

We evaluate COPE on the InPE dataset and compare it with several conversational recommendation baselines, including knowledge-graph-based methods and LLM-based methods.

The results show that COPE achieves the best recommendation performance.

For example, on Recall@10, COPE achieves **0.314**, outperforming the strongest baseline ReFICR, which achieves **0.274**. On Recall@50, COPE reaches **0.442**, compared with **0.396** from ReFICR.

We also evaluate response quality using human-annotated preference pairs. COPE achieves the highest pairwise win rate, **0.604**, outperforming LLaMA, Qwen, and ReFICR.

These results suggest that explicitly modeling preference elicitation strategies is useful not only for asking better questions, but also for improving recommendation quality.

## What We Learned

The paper provides three main takeaways.

First, preference elicitation is dynamic. A single static strategy cannot cover all conversational stages.

Second, different strategies are useful at different stages. Attribute-based questions are more effective early in the conversation, while item-based strategies become more helpful once the user’s preferences become clearer.

Third, strategy-aware modeling improves CRS performance. By explicitly learning when and how to ask, COPE can better support proactive conversational recommendation.

## Why This Matters

A good conversational recommender should not only know what to recommend. It should also know how to interact with users.

Asking too many questions can make the conversation inefficient. Asking the wrong type of question can confuse the user or fail to collect useful information. Recommending too early may lead to poor results, while recommending too late may frustrate the user.

This work shows that preference elicitation should be treated as a strategic decision-making problem. A CRS needs to decide:

**Should I ask now?**  
**What kind of question should I ask?**  
**Should I ask about attributes, items, or both?**  
**Is the conversation ready for recommendation?**

By modeling these decisions explicitly, we can build conversational recommender systems that are more adaptive, proactive, and user-centered.

## Conclusion

In this work, we study when and how conversational recommender systems should ask users for preferences. Through a user study, dataset annotation, and model design, we show that preference elicitation strategies are stage-dependent and context-sensitive.

We introduce **InPE**, a dataset with fine-grained annotations for preference elicitation, and propose **COPE**, a Mixture-of-Experts framework that dynamically selects conversational actions and elicitation strategies.

Our results suggest that the future of conversational recommendation lies not only in better item ranking, but also in better interaction strategy.

The dataset is available at:

[https://github.com/juanfacabian/InPE](https://github.com/juanfacabian/InPE)