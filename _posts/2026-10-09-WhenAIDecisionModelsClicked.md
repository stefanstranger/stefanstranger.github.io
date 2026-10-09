---
layout: post
title: When AI Decision Models Clicked
categories: [AI, AI Models, Automation]
tags: [AI, Decision Models, PowerShell, Unsloth, Laya, Jev]
comments: true
published: false
---
## Introduction

When I wrote about [agent harnesses](/2026/06/26/BuildingAnHarnessAgentWithMicrosoftAgentFramework/), the focus was on the infrastructure around a model. AI decision models made me look at a smaller part of that picture: what if I only need a judgment at one specific point in the workflow?

Take support tickets. I can write the routing workflow in PowerShell, but deciding which team should receive a message gets awkward when the customer describes several problems at once. That is the part where I would want help from a model.

Thinking about it as one call inside a PowerShell function made decision models easier for me to understand. I could ask which team should handle the ticket, check the answer, and continue with the script I already know how to write.

## TL;DR

* A decision model can select a team from a list you define. Your code checks the answer and decides whether to route the ticket or ask a person to review it.
* A regular LLM can do this too, including returning structured JSON. I want to compare how well each handles the job and what it costs to run.
* Jev is TypeSafe's flagship System One model. For the local experiment, I used Laya through Unsloth Studio's compatible Decision API.
* A valid team label can still be wrong. The probabilities and review threshold need testing on tickets representative of the actual workflow.

> [!NOTE]
> The terminal results below are from initial runs with Laya, GitHub Copilot CLI, and local Gemma. They show the workflows running, not a controlled benchmark. Both generated replies also claimed a routing action that the scripts never performed.

## What is an AI decision model

[TypeSafe describes System One](https://docs.typesafe.ai/concepts/system-one) as models for bounded, structured decisions. You supply the application state and typed questions. The model returns answers and probabilities rather than a generated explanation.

In the ticket example, "state" means the customer's message. I supply the list of teams and ask the model to choose one.

The interface supports different kinds of questions:

| Question type | What you ask it to do                 | Example                         |
| ------------- | ------------------------------------- | ------------------------------- |
| `choice`    | Select from developer-defined options | Which team handles this ticket? |
| `noul`      | Assess a yes/no question              | Does this message ask for help? |
| `score`     | Rate against a defined ordered scale  | How urgent is this request?     |

A `choice` question returns a selected option and option probabilities. A `noul` question provides the probability of yes. A `score` question provides a score and associated probability information.

I still have to decide what my application is allowed to do with the answer. Selecting `billing`, for example, should send the ticket to a queue. It shouldn't give the model permission to issue a refund!

### System One, Jev, and Laya

[Jev](https://docs.typesafe.ai/models) is TypeSafe's flagship System One model, available through its hosted API. [Laya](https://huggingface.co/convaiinnovations/laya) is a local decision-model option. [Unsloth Studio documents how to serve Laya](https://unsloth.ai/docs/models/decision-laya) through a Jev-compatible Decision API.

The hosted TypeSafe endpoint is `POST https://api.typesafe.ai/v1/systemone`. Unsloth's local Decision API also uses the `/v1/systemone` route.

One detail to watch: Jev and Laya calculate confidence differently, despite the compatible interface. I would need to retest a confidence threshold when switching between them.

For more on Jev and agent architecture, there is [The ClawCast, Episode 12, &#34;Jev and OpenClaw Enterprise&#34;](https://www.youtube.com/watch?v=ufLTkFzgw2k). It covers the approach discussed here, though I haven't independently tested its product claims.

## Why not use a PowerShell switch or a regular LLM

If a ticket already contains a reliable department field, I would route it with ordinary code.

But what should the script do with this?

> I can't upgrade my plan: checkout crashes and my card was charged.

Is this sales because the customer wants to upgrade? Technical support because checkout crashes? Billing because their card was charged?

A keyword-based script can match all three. I would need a policy for which issue takes priority, then a way to judge the message against that policy. This is where I want to try a decision model.

Here is how I think about the options:

| Approach       | Where it fits                               | What to watch for                            |
| -------------- | ------------------------------------------- | -------------------------------------------- |
| Ordinary code  | Explicit rules over known values            | Ambiguous language is hard to encode         |
| Decision model | Bounded classification, routing, or tagging | A valid answer can still be the wrong answer |
| Standard LLM   | Generation, explanation, and synthesis      | Validate output and evaluate task quality    |

You might be thinking: I can already ask an LLM to return `{"team":"billing"}`. And you can. Depending on the model and server, you can also constrain that output with a schema.

That gives me a useful baseline for the experiment. I want to find out whether the decision model gets the routing right, how long it takes, how much memory it uses, and whether its probabilities help me decide when to ask for review. The API documentation alone cannot answer those questions for my setup.

## Where this fits in an agent harness

The [agent harness](/2026/06/26/BuildingAnHarnessAgentWithMicrosoftAgentFramework/) is one place I could put this check. The harness can call the decision model whenever a ticket arrives. An ordinary script could do the same; this doesn't require an agent.

For the ticket example, the flow could look like this:

1. Receive the ticket.
2. Ask the decision model which team should handle it.
3. Validate the answer and apply the application's review policy.
4. Route the ticket using ordinary code, or send it for human review.
5. Use a standard LLM if a reply or explanation needs to be drafted.

I would make step 2 a required call in the application, rather than leave it to the general-purpose LLM to decide whether to use that check.

[MCP](https://modelcontextprotocol.io/specification/2025-11-25) could provide the connection to tools and context in this setup. It doesn't change the reliability of the model's judgment or replace the application's validation and authorization checks.

## The local experiment I want to run

If you read my post about [the AI tools I use daily](/2026/08/14/AIToolsIUseDaily/), you probably won't be surprised that I want to try this in Windows Terminal. I'll compare Laya followed by an LLM with an LLM that classifies and replies in one request.

### Before running the scripts

You'll need PowerShell 7 and [Unsloth Studio](https://unsloth.ai/docs/new/studio/install). Start Studio with `unsloth studio -p 8888` and keep it running.

* Enable **Settings > API > Decision API > Serve requests** and load Laya.
* Load a regular chat model and enable its OpenAI-compatible API.
* Have your Studio API token ready, or enable localhost keyless inference.

Both scripts below are complete. Paste one into each pane; neither depends on the other. Laya answers typed questions; the regular LLM classifies the same ticket and drafts a reply. The Copilot follow-up adds a reply to Laya's workflow.

The examples use Studio on port 8888. If your chat endpoint uses another port, change its URL. The [Unsloth API guide](https://unsloth.ai/docs/basics/api) explains the serving settings.

> [!WARNING]
> Keep both endpoints on loopback and disable server-side tools for this demo. Token entry is hidden, but the bearer header contains the token in memory. Don't print `$Headers` or include it in a recording.

### Split Windows Terminal

To get that layout, use this [Windows Terminal command](https://learn.microsoft.com/en-us/windows/terminal/command-line-arguments) from PowerShell with PowerShell 7 installed:

```powershell
wt -w new --maximized new-tab --title "Laya | Decision Model" --suppressApplicationTitle pwsh -NoLogo `; split-pane -V --title "Standard LLM" --suppressApplicationTitle pwsh -NoLogo
```

The escaped semicolon separates the Windows Terminal commands. `-V` creates a vertical divider, with the panes beside each other.

Both panes are clients. Splitting the terminal does not require two separate model servers.

### Left pane - Laya decision model

This is the direct REST call. The ticket goes into `state`; the three questions go into `questions`. The only setup at the prompt is your API token. Press Enter instead if localhost keyless inference is enabled.

```powershell
$ErrorActionPreference = 'Stop'
$Ticket = 'Hi, I was charged twice for my March invoice (#4411). Please refund the duplicate today.'
$Token = Read-Host 'Studio API token (Enter for enabled keyless access)' -AsSecureString
$Headers = @{}
if ($Token.Length -gt 0) {
    $Headers.Authorization = 'Bearer ' + [System.Net.NetworkCredential]::new('', $Token).Password
}

$Body = @{
    model = 'laya'
    state = $Ticket
    questions = @{
        team = @{
            type = 'choice'
            instructions = 'Which team should handle this? If several apply, prioritize technical, billing, sales, then other.'
            criteria = @{
                billing   = 'invoices, payments, refunds'
                technical = 'bugs, outages, errors'
                sales     = 'pricing, new plans'
                other     = 'everything else'
            }
        }
        refund = @{
            type = 'noul'
            instructions = 'Does the customer ask for a refund?'
        }
        urgency = @{
            type = 'score'
            instructions = 'How urgent is this?'
            criteria = @('not urgent', 'soon', 'today')
        }
    }
} | ConvertTo-Json -Depth 10

$Timer = [System.Diagnostics.Stopwatch]::StartNew()
$Response = Invoke-RestMethod `
    -Uri 'http://localhost:8888/v1/systemone' `
    -Method Post -Headers $Headers `
    -ContentType 'application/json; charset=utf-8' `
    -Body $Body -TimeoutSec 180 -MaximumRedirection 0

if ($Response.answers.team.type -cne 'choice' -or
    $Response.answers.refund.type -cne 'noul' -or
    $Response.answers.urgency.type -cne 'score') {
    throw 'Expected team, refund, and urgency answers from the Decision API.'
}

$Timer.Stop()
[pscustomobject]@{
    model = $Response.model
    answers = $Response.answers
    usage = $Response.usage
    elapsed_ms = [math]::Round($Timer.Elapsed.TotalMilliseconds, 1)
} | ConvertTo-Json -Depth 10
```

The output includes Laya's model, answers, probabilities, usage, and `elapsed_ms`. `$Response` keeps the full parsed API response available for inspection.

`laya` selects the decision model configured in Studio. Record the returned model name and the loaded checkpoint when comparing runs.

### Right pane - Regular LLM

This script sends the customer's message directly to Unsloth's chat endpoint. Enter the exact chat model ID shown by Studio or its `GET /v1/models` endpoint, then your token.

The LLM gets the same team definitions and priority rule as Laya. It briefly shows its classification, then drafts a customer-facing reply in the same request. The output is plain text, not JSON.

Use a general-purpose instruct/chat model for this example. [Phi-4-mini-reasoning](https://huggingface.co/microsoft/Phi-4-mini-reasoning) is designed and tested for math reasoning, rather than customer-service conversations.

```powershell
$ErrorActionPreference = 'Stop'
$Ticket = 'Hi, I was charged twice for my March invoice (#4411). Please refund the duplicate today.'
$Model = Read-Host 'Exact chat model ID from Unsloth Studio'
if ([string]::IsNullOrWhiteSpace($Model)) { throw 'A chat model ID is required.' }
$Token = Read-Host 'Studio API token (Enter for enabled keyless access)' -AsSecureString
$Headers = @{}
if ($Token.Length -gt 0) {
    $Headers.Authorization = 'Bearer ' + [System.Net.NetworkCredential]::new('', $Token).Password
}

$Instructions = @'
Read the customer's ticket and classify it:

- Team: billing (invoices, payments, refunds), technical (bugs, outages, errors),
  sales (pricing, new plans), or other.
  If several apply, prioritize technical, billing, sales, then other.
- Refund requested: yes or no.
- Urgency: not urgent, soon, or today.

Briefly show the classification, then write a customer-facing reply using it.
The classification describes the request; it does not verify account
information or mean that any action has been taken.
'@

$Reply = $null
$Body = @{
    model = $Model
    messages = @(
        @{ role = 'system'; content = $Instructions }
        @{ role = 'user'; content = $Ticket }
    )
    stream = $false
    temperature = 0
    max_tokens = 1024
} | ConvertTo-Json -Depth 10

$Timer = [System.Diagnostics.Stopwatch]::StartNew()
$Response = Invoke-RestMethod `
    -Uri 'http://localhost:8181/v1/chat/completions' `
    -Method Post -Headers $Headers `
    -ContentType 'application/json; charset=utf-8' `
    -Body $Body -TimeoutSec 180 -MaximumRedirection 0

if (@($Response.choices).Count -eq 1 -and
    $Response.choices[0].finish_reason -ceq 'length') {
    $Timer.Stop()
    throw (("Generation reached a length limit after {0} output tokens ({1:N1} ms). " +
        "No complete reply was received. Check max_tokens and the server context limit, " +
        "or use a general-purpose chat model.") -f
        $Response.usage.completion_tokens, $Timer.Elapsed.TotalMilliseconds)
}
if (@($Response.choices).Count -ne 1 -or
    $Response.choices[0].finish_reason -cne 'stop' -or
    $Response.choices[0].message.refusal -or
    $Response.choices[0].message.tool_calls) {
    throw 'Expected one completed text answer, without refusal or tool calls.'
}
$Reply = $Response.choices[0].message.content
if ([string]::IsNullOrWhiteSpace($Reply)) { throw 'The model returned no reply.' }

$Timer.Stop()
"Model: $($Response.model)"
$Reply
"Elapsed: {0:N1} ms" -f $Timer.Elapsed.TotalMilliseconds
```

The script prints the returned model, classification and reply, and elapsed time. `$Response` keeps the full API response, including usage data when supplied.

If generation reaches a length limit, the script reports an error with the token count and elapsed time. It leaves `$Reply` empty rather than presenting partial output as a completed answer.

In my Phi-4-mini-reasoning run, the server returned `finish_reason: "length"` and `completion_tokens: 1024`. The output stopped inside `<think>`, before a final reply. That run belongs in the results as an incomplete response.

To experiment with that reasoning model, increase `max_tokens` and allow a longer HTTP timeout if needed. More tokens may let it finish, but don't guarantee a useful reply. Removing the `<think>` text cannot recover an answer it never generated.

The API reported zero `reasoning_tokens` even though reasoning text appeared in `message.content`. Those words still consumed the output-token budget. The reply itself is plain text; there is no classification JSON to parse.

### Reading the two results

Laya returns a team choice, the probability that a refund was requested, and an urgency score with a legend and probabilities. These are values a script can use to decide what happens next.

The regular LLM shows a brief classification and writes a reply in one request. It uses the same routing policy, but returns prose rather than Laya's typed answers and probabilities.

The complete workflows are Laya followed by a reply-generating LLM, and an LLM that classifies and replies on its own. Compare Laya's time plus the follow-up request with the single LLM request, not classification time alone.

For a controlled comparison, use the same reply-generating model in both workflows. The examples use Copilot after Laya and a local model on its own, so differences can also come from the response models and serving environments.

Neither script routes a ticket or issues a refund. The LLM has no account access, so review its reply rather than treating any claim about a charge or refund as verified.

To try the ambiguous case, replace `$Ticket` in both scripts with `"I can't upgrade my plan: checkout crashes and my card was charged."` Both models use the same routing priority; the regular LLM also drafts a reply.

Both timers measure the HTTP call, response parsing, and completion checks. Token entry, request preparation, and output formatting are excluded. Laya prints `elapsed_ms`; the LLM prints `Elapsed` in milliseconds.

Run the requests one after the other when measuring latency. A first request may include model loading, so record cold and warm runs separately. Check whether both models can stay loaded before comparing repeated runs.

### Send the decision results to GitHub Copilot CLI

To turn the classification into a reply, run this block in the same PowerShell pane after the Laya script. It reuses `$Ticket` and `$Response`. Don't run the regular LLM script in that pane first, since it overwrites `$Response`.

You'll need GitHub CLI with `gh copilot` installed and authenticated. This sends the ticket and decision results to GitHub Copilot; this step is no longer local inference.

The prompt explains the result fields without prescribing the reply. The UTF-8 settings prevent curly apostrophes from becoming garbled when PowerShell captures the CLI output.

```powershell
$ErrorActionPreference = 'Stop'
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$OutputEncoding = [Console]::OutputEncoding

if ([string]::IsNullOrWhiteSpace($Ticket) -or
    $Response.answers.team.type -cne 'choice' -or
    $Response.answers.refund.type -cne 'noul' -or
    $Response.answers.urgency.type -cne 'score') {
    throw 'Run the Laya decision script in this pane first.'
}

$DecisionJson = $Response.answers | ConvertTo-Json -Depth 10
$Prompt = @"
Respond to the customer's message using the accompanying decision-model results as context.

How to read the results:
- choice: the selected option; probabilities contains the probability for each option.
- noul: the probability of yes for the question asked.
- score: a numeric rating; legend maps the scale to labels.
- These results classify the message. They do not verify account information or indicate that any action has been taken.

Customer message:
$Ticket

Decision-model results:
$DecisionJson
"@

$Timer = [System.Diagnostics.Stopwatch]::StartNew()
$Reply = gh copilot -- -p $Prompt --silent
$ExitCode = $LASTEXITCODE
$Timer.Stop()

if ($ExitCode -ne 0) {
    throw "GitHub Copilot CLI failed with exit code $ExitCode."
}
if ([string]::IsNullOrWhiteSpace($Reply -join "`n")) {
    throw 'GitHub Copilot CLI returned no reply.'
}

$Reply -join "`n"
"Elapsed: {0:N1} ms" -f $Timer.Elapsed.TotalMilliseconds
```

This prints the reply directly, without `Format-List` indentation. The elapsed time includes CLI startup and the Copilot request, not the earlier Laya call. Review the generated reply before using it.

## What came back in the terminal

Here are the two reply-generation paths on screen. The upper-right pane shows the Copilot follow-up after Laya; the lower-left shows Gemma classifying the ticket and replying in one request. The other panes show server logs.

![Windows Terminal showing the Copilot follow-up reply, Gemma classification and reply, elapsed times, and local server logs](/assets/2026-10-09-01.png)

### Classification and replies

Gemma, running as `unsloth/gemma-4-12B-it-qat-GGUF`, returned:

* Team: billing
* Refund requested: yes
* Urgency: today

That agrees with the earlier Laya result for this ticket: `billing`, a refund-request probability of `0.9794`, and a probability of `0.9045` for the urgency level "today". Laya's urgency score was `1.9005` on the zero-to-two scale.

The replies were different. Copilot wrote:

> I'm sorry for the trouble. I've routed your report about invoice #4411 to our billing team to review the duplicate charge. We'll confirm what happened and whether a refund is due; I can't confirm that a refund has been issued or promise it will be completed today.

Gemma showed its classification first, then included this in the reply:

> I have forwarded your request to our billing team to investigate the duplicate charge and process the refund for you as quickly as possible. We will notify you as soon as the refund has been completed.

This is the bit I would stop at before sending either reply to a customer. Both say the ticket has been passed to billing. My scripts did no such thing. Gemma also talks about a refund as though processing it is already the next step.

The Copilot prompt explained that classification does not mean an action was taken, but the reply still claimed routing. Passing structured results to an LLM does not guarantee that its prose will stay within those facts.

For a real workflow, I would have code perform the authorized routing action and record the result before asking for a reply about it. Until then, these are drafts to review, not confirmations to send.

### Observed times

The terminal uses Dutch number formatting: `14.552,5 ms` means 14,552.5 milliseconds.

| Request                              | Elapsed time | What the timer covers                       |
|--------------------------------------|--------------|---------------------------------------------|
| Copilot follow-up after Laya          | 14,552.5 ms  | CLI startup and reply request, excluding Laya |
| Gemma classification and reply       | 73,687.1 ms  | Local chat request and response checks       |

The Copilot follow-up took about 14.6 seconds; Gemma took about 73.7 seconds. These are individual observations from different models and serving environments, not evidence that decision models are faster than chat models.

The full Laya-to-Copilot path also includes the Laya request. Its client elapsed time is not shown in this screenshot. The earlier Laya output showed 1,828.6 ms, but that was a separate run, so I haven't combined them into one measured total.

The server logs contain their own `process_time_ms` values. Those measure a different part of the request and should not replace the client timings. The screenshot also doesn't establish whether models were warm or whether other work was running.

## What I still want to measure

A handful of tickets will help me check the requests and responses. To compare accuracy, I'll need a representative set of tickets with expected answers.

For measurements beyond this demonstration, I want to:

* Warm up both models and measure cold starts separately.
* Run timed requests sequentially so they do not compete for shared resources.
* Measure end-to-end client latency, including parsing and validation.
* Repeat requests and report median latency, plus tail latency with enough samples.
* Give both models the same classification task and labeled test set if comparing classification accuracy.
* Evaluate generated replies separately for usefulness and unsupported claims.
* Report invalid outputs and errors instead of dropping failed requests.
* Report review rate and accuracy among accepted cases when applying a review threshold.
* Measure memory use before claiming a resource advantage.

I'll record input length and chat generation settings alongside the hardware, server version, checkpoint, and quantization so the comparison can be repeated. A fixed temperature doesn't guarantee identical outputs. Model aliases such as `jev-latest` can also change, so a reproducible run needs a pinned version.

## How much should I trust the probability

[Unsloth&#39;s Laya guidance](https://unsloth.ai/docs/models/decision-laya) advises thresholding the selected choice's probability rather than its `confidence` field. In the response, look up the selected team under `answers.team.probabilities`.

An answer with a probability of `0.80` can still be wrong. If a model is well calibrated on my tickets, I would expect roughly 80% of the answers assigned probabilities around 0.80 to be correct across many examples. That says something about a group of predictions, not a guarantee about the ticket in front of me. [TypeSafe&#39;s confidence documentation](https://docs.typesafe.ai/confidence) explains calibration in more detail.

I'll need labeled examples to choose a threshold for this workflow. Too low, and I risk routing more tickets incorrectly. Too high, and I may send nearly everything to a person. I want to measure both the review rate and the accuracy of the tickets that get routed automatically.

I also would not compare Laya's option probabilities directly with an LLM writing "I am 95% confident." A generated confidence percentage is not the same measurement.

## When would I use this

Ticket routing is where I would start, because I can define the available teams and check the answers against a policy. Tagging or triage could be worth trying for the same reason, provided the task is similarly bounded.

For reliable fields and explicit rules, I'll keep using ordinary code. For a drafted reply or a summary, I'd use an LLM. In the ticket workflow I can imagine using both models: one to select the team, then the LLM to help draft a response after the routing and review checks.

Next I'll repeat the runs with ambiguous tickets and use the same reply-generating model in both paths. There's also a guide to training a decision model with [Unsloth](https://unsloth.ai/docs/basics/train-your-own-decision-model-with-unsloth), but I'll leave that for later.

## Further reading

* [TypeSafe System One](https://docs.typesafe.ai/concepts/system-one)
* [How to build with TypeSafe](https://docs.typesafe.ai/concepts/how-to-build-with-system-one)
* [TypeSafe API reference](https://docs.typesafe.ai/api)
* [TypeSafe confidence and calibration](https://docs.typesafe.ai/confidence)
* [Unsloth Laya and Jev API guide](https://unsloth.ai/docs/models/decision-laya)
* [Unsloth Studio installation](https://unsloth.ai/docs/new/studio/install)
* [Unsloth API endpoint guide](https://unsloth.ai/docs/basics/api)
* [Laya model card](https://huggingface.co/convaiinnovations/laya)
* [The ClawCast, Episode 12 on YouTube](https://www.youtube.com/watch?v=ufLTkFzgw2k)
* [Windows Terminal command-line arguments](https://learn.microsoft.com/en-us/windows/terminal/command-line-arguments)
