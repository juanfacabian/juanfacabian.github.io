---
title: "When and How to Ask: Dynamic Preference Elicitation Strategies for Conversational Recommender Systems"
permalink: /blog/when-and-how-to-ask/
layout: default
published: false
---

<p class="blog-post-meta">Apr 25, 2026 | Feng Xia</p>

This post is a detailed research note for our SIGIR 2026 paper, *When and How to Ask: Dynamic Preference Elicitation Strategies for Conversational Recommender Systems*. I wrote it to explain not only what the paper does, but also why we framed the problem this way, what the dataset actually captures, and what I think the most important takeaways are.

At a high level, the paper argues that conversational recommender systems should not treat preference elicitation as a fixed behavior. A system should decide **whether to ask**, **what kind of question to ask**, and **when to switch from asking to recommending**. The central claim is that these decisions are stage-dependent: when user preferences are still vague, attribute-based questions are more useful; when preferences become clearer, item-based elicitation becomes more effective.

## Why We Worked on This Problem

Conversational recommender systems (CRSs) are attractive because they can do more than infer preferences from logs. They can actively interact with the user, clarify intent, and refine recommendations through dialogue. But that also makes the system policy much more important.

In many existing systems, preference elicitation is still fairly static. A model may keep asking about genres, directors, or actors throughout the interaction. Another system may rely heavily on concrete item comparisons. Both approaches can work, but they are usually applied with too little regard for conversational stage.

This is the gap we wanted to study. In a realistic recommendation dialogue, user preferences evolve. Early turns often contain broad or underspecified goals. Middle turns contain refinement. Later turns involve concrete preference adjustment after seeing candidate items or recommendations. If the user state changes over time, then the elicitation strategy should change too.

That simple observation became the starting point for the paper.

## The Three Questions Behind the Paper

We formulated the work around three research questions:

1. Is a single commonly used preference elicitation strategy sufficient across dialogue stages?
2. Do different elicitation strategies work better at different stages?
3. Does explicitly modeling elicitation strategy improve conversational recommendation performance?

The rest of the paper is essentially one long attempt to answer these three questions with progressively stronger evidence: first with a user study, then with a new annotated dataset, and finally with a strategy-aware model.

## A Short Refresher: What Counts as Preference Elicitation?

The paper mainly discusses two broad families of elicitation strategy.

**Attribute-based elicitation** asks about aspects of items. In a movie setting, that might be genre, tone, pacing, cast, or director. These questions are useful when the system needs to narrow the search space:

> What kind of movies do you usually enjoy?

> Are you looking for something light-hearted or more serious?

**Item-based elicitation** asks the user to react to specific items or small candidate sets:

> Which of these two movies sounds closer to what you want?

> Would you prefer something like *Jojo Rabbit* or something closer to *What We Do in the Shadows*?

The paper also includes **hybrid** strategies, where both attribute cues and item cues are used together.

One of the paper's main points is that these are not interchangeable tools. They have different strengths, and those strengths show up at different points in the conversation.

<figure class="paper-figure">
  <img src="{{ '/assets/images/papers/when-how-ask-stage.png' | relative_url }}" alt="User choices between attribute-based and item-based elicitation strategies across dialogue stages">
  <figcaption>
    In the user study, preference for attribute-based elicitation is stronger in the early stages of a conversation, while item-based elicitation becomes more preferred as user intent becomes more concrete.
  </figcaption>
</figure>

## The Preliminary User Study

To answer the first two research questions, we began with a preliminary user study involving **328 participants**. Participants were recruited through CloudResearch and completed an online survey hosted on Qualtrics. They were shown designed conversational recommendation scenarios and asked to choose preferred system responses. Since each response represented a particular elicitation strategy, these choices give direct evidence about which strategies users prefer under different conditions.

### RQ1: Is a Single Strategy Enough?

The answer was basically no.

For attribute-based elicitation:

- **21%** of participants felt it was sufficient across all situations
- **53%** felt it worked in most situations
- **25%** felt it only worked in a few cases

For item-based elicitation:

- **25%** felt it was sufficient across all situations
- **59%** felt it worked in most situations
- **16%** felt it only worked in a few cases

So neither strategy was viewed as universally sufficient. That is a useful result by itself, because it pushes against the idea that we can just pick one policy family and apply it everywhere.

### RQ2: Does Strategy Preference Change Across Dialogue Stages?

Here the answer was yes, very clearly.

We designed **eight dialogue scenarios** that span three broad stages:

- early-stage preference exploration
- middle-stage preference refinement
- later-stage readjustment after preferences become more concrete

The pattern was consistent. In the early stage, users preferred attribute-based elicitation. As the dialogue moved toward more concrete preferences, item-based elicitation rose steadily and eventually overtook attribute-based questioning.

This is the conceptual core of the paper: **preference elicitation is not static; it evolves with the dialogue**.

## Why This Matters in Practice

It helps to ground this in an example.

Suppose a user opens with:

> I'm looking for a movie tonight.

At this point the user is not yet ready to judge concrete items well. A broad attribute-based question is usually the right move, because it reduces uncertainty cheaply:

> What kind of movies are you in the mood for?

Now suppose the user says they want a comedy and mentions liking dry humor. Once we reach that point, asking another generic attribute question may not be the best next step. A comparison between concrete options can reveal much more:

> Which feels closer to what you want, *Jojo Rabbit* or *What We Do in the Shadows*?

That shift from broad exploration to concrete comparison is what we wanted the model to learn.

## Building InPE: A Dataset for Strategy-Aware CRS

Once the user study showed the stage-dependent pattern, the next problem was obvious: existing conversational recommendation datasets do not usually annotate **whether a turn needs elicitation** or **which elicitation strategy is best**.

To address that, we built **InPE**: **InSPIRED Preference Elicitation**.

InPE is an annotated and augmented version of the INSPIRED dataset. INSPIRED already provides **999 multi-turn movie recommendation dialogues** with an average of **21.15 turns per dialogue**, which makes it a strong foundation. But it lacks fine-grained labels for strategy-aware conversational decisions.

So we enriched it with annotations for:

- whether a turn requires preference elicitation
- which elicitation strategy is appropriate
- whether a strategy-aware rewrite improves the original system response

### The Annotation Pipeline

The annotation process is structured rather than ad hoc.

First, an LLM acts as an initial filter over the dialogue turns. Out of **10,576 total turns**, the LLM flagged **3,437 turns (32.5%)** as candidates for preference elicitation. Human annotators then validated these candidates. Among the flagged turns, **60.07%** were confirmed to genuinely require preference elicitation.

Then, for eligible turns, annotators labeled the preferred strategy as:

- attribute-based
- item-based
- hybrid

They also selected the best strategy-specific candidate response and judged whether it was actually better than the original response.

Each turn was annotated independently by **three annotators**, and majority voting was used for the final label. Where no majority was reached, an expert annotator resolved the case. That matters because these decisions can be subjective, especially in transitional conversational stages.

<figure class="paper-figure paper-figure--narrow">
  <img src="{{ '/assets/images/papers/when-how-ask-flow.png' | relative_url }}" alt="Annotation workflow for determining whether preference elicitation is needed and which strategy to use">
  <figcaption>
    The InPE workflow first asks whether a turn needs preference elicitation, then selects a strategy, generates strategy-specific candidates, and keeps the update only when it improves on the original response.
  </figcaption>
</figure>

### What the Dataset Looks Like

The final experimental dataset contains **6,719 turn-level samples** derived from INSPIRED.

One of the most interesting observations is the strategy distribution:

- **Hybrid:** 47.03%
- **Item-based:** 28.85%
- **Attribute-based:** 24.12%

This tells us something important. Effective elicitation is often not purely attribute-first or item-first. In many contexts, combining both kinds of signal is natural and useful.

The paper also examines whether strategy-aware rewrites are actually preferred by humans. The answer is strongly yes. Depending on strategy type, users preferred the strategy-aware versions in **89.34% to 95.5%** of cases. So the annotation work is not just descriptive; it also points toward better response behavior.

## From Dataset to Model: Why We Proposed COPE

Once we had a dataset with explicit labels for task type and elicitation strategy, the modeling direction became clearer.

Most LLM-based conversational recommender systems still handle heterogeneous behaviors inside one shared generation path. But "recommend an item", "ask a broad preference question", and "carry out general interaction" are not the same action. They may depend on different signals and require different generation behavior.

That motivated **COPE**: **COnversational Preference Elicitation via Mixture of Experts**.

## What COPE Actually Does

COPE treats conversational recommendation as a **structured sequential decision problem** rather than a plain next-utterance generation task.

At each turn, the system reasons over a structured action space that includes:

- a **system action**: elicit, recommend, or general interaction
- an **elicitation strategy** if elicitation is chosen
- a recommendation objective for recommendation turns

The architecture uses a shared frozen LLM backbone, but instead of relying on one monolithic pathway, it introduces **task-specialized experts** and a **two-stage router**.

### Stage 1: Task-Level Routing

The first router predicts the high-level function of the turn:

- **Elicit**
- **Recommend**
- **General**

This is the model's answer to "what kind of thing should I do right now?"

### Stage 2: Strategy-Level Routing

If the first-stage decision is **Elicit**, a second router selects the elicitation strategy:

- **Attr**
- **Item**
- **Hybrid**

This is the answer to the more specific question: "if I am going to ask, how should I ask?"

### Why Mixture of Experts Helps Here

COPE is not using mixture-of-experts just to scale parameters. The point is to separate **decision execution pathways**. In the paper's framing, different experts can learn different behavioral patterns without interfering with each other. That is a much better match to the structure of the task than forcing one generator to implicitly absorb everything.

Training also reflects this separation. During training, the router is supervised using the annotated labels, while expert activation follows the ground-truth decisions. During inference, the model uses the router predictions to activate experts dynamically.

## Experimental Setup

The experiments are run on the InPE dataset with a **6:2:2 train/validation/test split**.

Each turn-level sample includes:

1. a task label
2. an elicitation strategy label for elicitation turns
3. recommendation targets for recommendation turns
4. a preference-labeled response pair for response quality evaluation

The paper evaluates three aspects:

- **Recommendation quality** using Recall@k
- **Decision quality** using task accuracy and strategy accuracy
- **Response quality** using pairwise win rate and log-likelihood margin over human-preferred response pairs

I like this setup because it does not collapse everything into one metric. It lets us ask whether the model retrieves better items, makes better strategic decisions, and produces responses people would actually prefer.

## Main Results

### Recommendation Performance

COPE achieves the best recall across all reported cutoffs:

- **Recall@1 = 0.144**
- **Recall@10 = 0.314**
- **Recall@50 = 0.442**

The paper notes that the gains are especially visible at deeper cutoffs like Recall@10 and Recall@50. The strongest baseline, **ReFICR**, still underperforms COPE, even though it also uses an LLM backbone. That is a meaningful comparison because it suggests the benefit is coming from explicit strategy modeling, not just from using a larger or stronger language model.

Prompt-only LLM baselines such as Qwen3-8B and LLaMA3-8B-Instruct perform much worse, which again reinforces the idea that conversational recommendation needs task-specific structure rather than generic instruction-following alone.

### Response-Level Preference

On pairwise evaluation, COPE achieves:

- **Win Rate = 0.604**
- **Margin = 0.314**

That is the best among the reported models.

This result matters because retrieval gains alone are not enough. A conversational recommender can retrieve decent items and still interact badly. The pairwise results suggest that strategy-aware modeling improves the *quality of the system response itself*, not only its ranking behavior.

## What the Ablation Study Says

The ablation study is more interesting than it first appears.

The full model achieves:

- **Task Acc = 0.618**
- **Eli Acc = 0.711**
- **Recall@10 = 0.314**
- **Win Rate = 0.604**

Two partial variants remove either the recommendation expert or the elicitation expert. Both generally hurt the system. That fits the intuition that the model's components rely on one another and that strategic behavior is not something we can cleanly bolt on after the fact.

There is also a variant without any specialized experts. Interestingly, that version can achieve strong accuracy-style signals, but it performs worst in pairwise evaluation. The paper's interpretation is important: a unified model may predict labels reasonably well while still generating generic, less strategy-sensitive responses. In other words, coarse accuracy does not automatically translate to better conversational behavior.

I think that is one of the most useful lessons in the paper.

## The Mechanistic Analysis: Where the Bottleneck Is

The routing intervention experiment is one of my favorite parts of the paper because it clarifies what COPE is and is not failing at.

When the system is forced to use the correct routing decisions:

- **Ground Truth:** TaskAcc = 1.0, Eli@1 = 1.0, Recall@10 = **0.342**

Under normal predicted routing:

- **Prediction:** TaskAcc = 0.617, Eli@1 = 0.711, Recall@10 = **0.314**

Under random routing:

- **Random:** TaskAcc = 0.333, Eli@1 = 0.333, Recall@10 = **0.051**

This tells us two things.

First, the experts themselves are effective. If routing were perfect, performance would improve noticeably. Second, the current bottleneck is largely **strategy selection accuracy**, not the underlying capacity of the experts.

That is a very actionable conclusion for future work. If we want to push this line further, improving the router may matter even more than making the expert modules larger.

## What I Think the Paper Contributes

If I strip the paper down to its most important contributions, I would summarize them like this:

1. It shows, with user-study evidence, that preference elicitation strategy should vary across dialogue stages.
2. It provides a dataset, InPE, that makes this problem trainable and measurable.
3. It proposes a model, COPE, that explicitly separates "what action should I take?" from "what elicitation strategy should I use?"
4. It shows that this explicit strategy modeling improves both recommendation performance and response preference.

For me, the strongest conceptual takeaway is that conversational recommendation should be treated not just as ranking or generation, but as **interaction policy learning**.

## What I Still Find Interesting or Open

Even though I like the paper's story, I think there are still several open directions worth paying attention to.

One is the ambiguity of transitional turns. The paper already hints at this through the relatively low inter-annotator agreement on the initial decision of whether elicitation is needed. That is not a weakness of the annotation process so much as a sign that the problem itself is genuinely fuzzy.

Another is that hybrid strategies are very common. That suggests the field may have spent too much time thinking in a clean attribute-vs-item dichotomy, when in practice users and systems often benefit from mixed prompts.

And finally, the routing gap shows that expert specialization is only part of the problem. We also need better decision models for identifying the right conversational move under uncertainty.

## Final Thoughts

The paper started from a small question that sounds almost obvious once stated: in a conversation, should a recommender always ask the same kind of question? The empirical answer is no, and I think making that answer measurable is the main value of the work.

If you care about proactive conversational systems, the important message here is broader than movie recommendation. A useful interactive system should not only produce a response. It should choose the *right kind* of response for the current stage of interaction.

That is the lens I would keep from this paper: **when to ask and how to ask are first-class modeling problems**.

The InPE dataset is available here:

[https://github.com/juanfacabian/InPE](https://github.com/juanfacabian/InPE)
