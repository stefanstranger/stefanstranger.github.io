---
layout: post
title: What are AI Agent Evaluators and Why They Matter
categories: [AI, Agentic AI, Observability]
tags: [AI Agents, Evaluators, Multi-Agent Systems, Azure AI Foundry]
comments: true
---

## Introduction

I recently bought the book [Designing Multi-Agent Systems: Principles, Patterns, and Implementation for AI Agents by Victor Dibia](https://buy.multiagentbook.com/).

![Screenshot of book cover](/assets/07-01-2026-01.png)

In chapter 10 "Evaluating Multi-Agent Systems" he discusses how to build evaluation frameworks with LLM-as-judge and metrics. This was a new topic for me and wanted to learn more about what Agent Evaluators are and why they matter. This is why I wrote this blog post. I hope this is also an interesting topic for more people who are building Agentic AI Solutions.

I highly recommend reading this book written by [Victor Dibia](https://www.linkedin.com/in/dibiavictor/)!

## Observability

Before we can start explaining evalutors we first need to discuss observability. According to Victor's book users want to observe what agents did (actions) and why. This is related to the question, how we can trust AI in applications. For that we need to access and monitor the content created by AI systems.

That's where [observability](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/observability?view=foundry&preserve-view=true#what-is-observability) comes into play. AI observability refers to the ability to monitor, understand, and troubleshoot AI systems throughout their lifecycle. It involves collecting and analyzing signals such as evaluation metrics, logs, traces, and model and agent outputs to gain visibility into performance, quality, safety, and operational health.

Now it's time to discuss evaluators.

## Evaluators

[Evaluators](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/observability?view=foundry&preserve-view=true#what-are-evaluators) are specialized tools that measure the quality, safety, and reliability of AI responses. By implementing systematic evaluations throughout the AI development lifecycle, teams can identify and address potential issues before they impact users.

When you build Agentic Solutions your agent or agents explores a series of actions, and you want to know which steps take place and if these steps lead to success or failure. With an evaluation framework you want to be able to capture the end-to-end process, what is called "trajectory" according to Victor Dibia,. This is a sequence of reasoning messages and actions that are executed by your Agentic solution.

With systematic evaluation we want to build trust. Victor shows how to create an evaluation framework with his Python library called [picoagents](https://github.com/victordibia/designing-multiagent-systems/tree/main/picoagents/src/picoagents/eval). In this blog post I'll be using an other evaluation framework, the [Microsoft Foundry's evaluation functionality](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/evaluate-generative-ai-app?view=foundry).

With the the [Azure AI Evaluation SDK](https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/evaluation/azure-ai-evaluation/README.md) lets you run evaluations locally on your machine and in the cloud. In this blog post I'm using the [local evaluations](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/agent-evaluate-sdk?view=foundry). In this blog post, you learn how to run built-in evaluators locally on simple agent data or agent messages.

## Pre-requisites

- Python 3.10 or later
- Azure AI Evaluation client library for Python package
- Microsoft Agent Framework Python package
- Access to Github Models
- Azure AI Foundry Project and model deployed

## Azure AI Evaluation Evaluators

The following evaluators are available in the Azure AI Evaluation SDK:

### Built-in evaluators

Built-in evaluators are out of box evaluators provided by Microsoft:

| Category  | Evaluator class |
|-|-|
| [Performance and quality](https://learn.microsoft.com/azure/ai-studio/how-to/develop/evaluate-sdk#performance-and-quality-evaluators) (AI-assisted)  | `GroundednessEvaluator`, `RelevanceEvaluator`, `CoherenceEvaluator`, `FluencyEvaluator`, `SimilarityEvaluator`, `RetrievalEvaluator` |
| [Performance and quality](https://learn.microsoft.com/azure/ai-studio/how-to/develop/evaluate-sdk#performance-and-quality-evaluators) (NLP)  | `F1ScoreEvaluator`, `RougeScoreEvaluator`, `GleuScoreEvaluator`, `BleuScoreEvaluator`, `MeteorScoreEvaluator`|
| [Risk and safety](https://learn.microsoft.com/azure/ai-studio/how-to/develop/evaluate-sdk#risk-and-safety-evaluators) (AI-assisted)    | `ViolenceEvaluator`, `SexualEvaluator`, `SelfHarmEvaluator`, `HateUnfairnessEvaluator`, `IndirectAttackEvaluator`, `ProtectedMaterialEvaluator`                                             |
| [Composite]()| `QAEvaluator`, `ContentSafetyEvaluator`                                             |

For more in-depth information on each evaluator definition and how it's calculated, see [Evaluation and monitoring metrics for generative AI](https://learn.microsoft.com/azure/ai-studio/concepts/evaluation-metrics-built-in).

In this blog post I will start with a [simple textual similarity evaluator](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/evaluation-evaluators/textual-similarity-evaluators?view=foundry-classic#similarity).

Similarity measures the degrees of semantic similarity between the generated text and its ground truth with respect to a query. Compared to other text-similarity metrics that require ground truths, this metric focuses on semantics of a response, instead of simple overlap in tokens or n-grams. It also considers the broader context of a query.


#### Similarity Evaluator

The Similarity Evaluator provides the following metrics.

| Property | What it means | Common uses |
|-|-|-|
| similarity | 1–5 semantic alignment score | KPI, gating, aggregation |
| gpt_similarity | Legacy mirror of similarity | Backward compatibility; migrate away |
| similarity_result | "pass"/"fail" vs threshold | CI/CD gates; pass‑rate trends |
| similarity_threshold | Decision boundary (default 3) | Tune per scenario; document version |
| similarity_prompt_tokens | Tokens in evaluator input | Cost & latency tracking; prompt hygiene |
| similarity_completion_tokens | Tokens in evaluator output | Sanity checks on completion size |
| similarity_total_tokens | Sum of prompt+completion | Cost dashboards; outlier detection |
| similarity_finish_reason | LLM stop code (e.g., length) | Troubleshoot truncation & limits |
| similarity_model | Evaluator model ID | Reproducibility; drift checks |
| similarity_sample_input | Serialized evaluator input | Auditing; re‑runs; dataset mapping checks |
| similarity_sample_output | Serialized evaluator output | Low‑level trace of evaluator result |

## Demo of Similarity Evaluator

Below the scenario I choose to demo the similarity evaluator.

### Why the Similarity Evaluator?

I chose the Similarity Evaluator for this demonstration because it perfectly illustrates a critical challenge in AI development: **different models may answer the same question differently, even when both answers are semantically correct**.

The test case uses a seemingly simple question: "How many times does the letter 'e' appear in 'Mercedes-Benz'?" with a ground truth of "4 times (M-e-rc-e-d-e-s-B-e-nz)". While this has a definitive answer, different LLMs might:

- Phrase the answer differently ("There are 4 e's" vs "The letter e appears 4 times")
- Include or exclude the explanation showing where the letters appear
- Use different formatting or verbosity levels

This is where the Similarity Evaluator stands out. Traditional software tools often fail if the wording isn't an exact match, or they only count how many words look the same. The Similarity Evaluator is different because it uses AI to act like a human judge, understanding the actual **meaning** of the answer rather than just the words used. It recognizes that "4 times" and "The letter e appears four times in Mercedes-Benz" mean the same thing, even though they are written differently.

By testing multiple models (GPT-4o, Phi-4, GPT-5, Mistral-Small) with the same question and evaluating their responses against the ground truth, we can:

1. **Build trust** by systematically comparing model outputs
2. **Identify consistency** across different model architectures
3. **Make informed decisions** about which models align best with expected outputs
4. **Demonstrate semantic evaluation** rather than just exact matching

This multi-model comparison demonstrates how evaluation frameworks help us understand model behavior and build confidence in AI systems—exactly the kind of systematic evaluation Victor Dibia emphasizes in his book for building trustworthy multi-agent systems.

![Similarity Evalutor Diagram](/assets/ai-agent-evaluation-workflow.drawio.png)

### The Code

The full python script `multi_model_evaluation.py` is available in this [GitHub Gist](https://gist.github.com/stefanstranger/5d29964a82594ad710af930bd4c3d4c7).

Here are the key components of the evaluation script:

### 1. Chat Client and Agent Initialization

First, we initialize the client for GitHub Models and create our agent.

```python
    # Initialize GitHub Models Client for this specific model
    openai_chat_client = OpenAIChatClient
        model_id=model_id,
        api_key=os.environ.get("GITHUB_TOKEN"),
        base_url=os.environ.get("GITHUB_ENDPOINT"),
    )

    # Create AI Agent
    agent = ChatAgent(
        chat_client=openai_chat_client,
        instructions=instructions,
        stream=True
    )
```

### 2. Test Case Definition

We define a clear query and a comprehensive ground truth. The ground truth doesn't just give the number; it explains the reasoning which helps the evaluator judge semantic correctness even if the phrasing varies.

```python
    # Define the test case   
    query = "How many times does the letter 'e' appear in 'Mercedes-Benz'?"
    ground_truth = "4 times (M-e-rc-e-d-e-s-B-e-nz)"
```

### 3. LLM as Judge Configuration

We configure an Azure OpenAI model (e.g., GPT-5-chat) to act as the "Judge". You cannot use reasoning models for evaluation.

```python
    # Configure Azure OpenAI for the evaluator (LLM-as-Judge)
    model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=os.environ.get("AZURE_OPENAI_ENDPOINT"),
        api_key=os.environ.get("AZURE_OPENAI_KEY"),
        azure_deployment=os.environ.get("AZURE_OPENAI_DEPLOYMENT"),
        api_version=os.environ.get("AZURE_OPENAI_VERSION"),
    )
```

### 4. Agent instruction

```python
instructions = "You are being evaluated on your ability to answer questions accurately and follow instructions precisely."
```

### 5. SimilarityEvaluator

We initialize the `SimilarityEvaluator` with our judge configuration. We set a threshold (e.g., 3 out of 5) to determine pass/fail criteria.

```python
    # Create SimilarityEvaluator with threshold of 3
    similarity = SimilarityEvaluator(model_config=model_config, threshold=3)
    
    # ... inside evaluation loop ...

            # Evaluate the response
            eval_result = similarity(
                query=query,
                response=agent_answer,
                ground_truth=ground_truth
            )
```

### 6. Results

Finally, we parse the evaluation results. The evaluator returns a score (1-5) and a boolean pass/fail result based on our threshold.

```python
            # Store results
            score = eval_result.get('gpt_similarity', eval_result.get('similarity'))
            result = eval_result.get('similarity_result', 'N/A')
            
            print(f"Similarity Score: {score}/5")
            print(f"Result: {result.upper()}")
```

<a href="https://stefanstranger.github.io//assets//07-01-2026-02.png"><img width="540" height="1170" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io//assets//07-01-2026-02.png" border="0" /></a>

07-01-2026-03.png<a href="https://stefanstranger.github.io//assets//07-01-2026-03.png"><img width="540" height="1170" title="image" style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border-width: 0px" alt="image" src="https://stefanstranger.github.io//assets//07-01-2026-02.png" border="0" /></a>

## Conclusion

By implementing this `SimilarityEvaluator`, we moved from 'feeling' that our agent is working to 'knowing' it is working based on quantifiable metrics. This is the first step towards building robust, production-ready Agentic AI systems.

In the next blog post, I'll explore how to use evaluators as a feedback loop to **improve prompt instructions**. I also plan to investigate how we can use these metrics to **compare different agent architectures**—testing whether complex workflows or tool-enabled agents actually deliver better results than simpler implementations.

Let me know iņ the comments how you are using evaluations.

## References

- [Designing Multi-Agent Systems: Principles, Patterns, and Implementation for AI Agents by Victor Dibia](https://buy.multiagentbook.com/)
- [Microsoft Foundry - What are evaluators](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/observability?view=foundry-classic#what-are-evaluators)
- [Microsoft Foundry - Observability in generative AI](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/observability?view=foundry&preserve-view=true)
- [Github - Azure SDK for Python - Evaluation Examples](https://github.com/Azure/azure-sdk-for-python/tree/8513df47960817e6315619fcd5741f7260bfea4b/sdk/ai/azure-ai-projects/samples/evaluation)
- [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview)
- [Microsoft Agent Framework Quick-Start Guide](https://learn.microsoft.com/en-us/agent-framework/tutorials/quick-start?pivots=programming-language-python)
- [Evaluate generative AI models and applications by using Microsoft Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/evaluate-generative-ai-app?view=foundry)
- [Github - Azure AI Evaluation client library for Python](https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/evaluation/azure-ai-evaluation/README.md)
- [Github Models Marketplace](https://github.com/marketplace?type=models)