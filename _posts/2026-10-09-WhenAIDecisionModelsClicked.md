---
layout: post
title: When AI Decision Models Clicked
categories: [AI, AI Models, Automation]
tags: [AI, Decision Models, PowerShell, Unsloth, Laya, Jev]
comments: true
published: false
---
## Introduction

When I wrote about [agent harnesses](/2026/06/26/BuildingAnHarnessAgentWithMicrosoftAgentFramework/), I focused on the infrastructure around a model.

Decision models made me look at one smaller question: what if I need AI judgment at only one point in a workflow?

Take support tickets. PowerShell can route a ticket, but deciding which team should receive an ambiguous message is harder to express as fixed rules.

I could ask a decision model to classify the message, validate its answer, and let PowerShell continue the workflow.

The terminal test exposed an even more useful boundary. Neither script routed the ticket, yet both generated replies claimed that routing had happened.

That is the argument of this post: a model can make a judgment, but code must own the action. Generated prose should describe only actions the application actually recorded.

## TL;DR

* Laya returned typed decisions for team, refund intent, and urgency.
* A regular LLM classified the same ticket and drafted a reply.
* GitHub Copilot used Laya's results to draft another reply.
* Both replies invented a routing action that the scripts never performed.
* Typed output constrains an interface. It does not prove correctness or make later prose truthful.

> [!NOTE]
> These are initial workflow demonstrations, not a controlled benchmark. The models, runtimes, and request paths differ, and the test uses one synthetic ticket.

## What is an AI decision model

[TypeSafe describes System One](https://docs.typesafe.ai/concepts/system-one) models as tools for bounded, structured decisions.

The application supplies state and typed questions. The model returns answers and probabilities instead of a generated explanation.

In this example, the state is the customer's message. I provide the available teams and ask the model to choose one.

The interface supports different kinds of questions:

| Question type | What you ask it to do                 | Example                         |
| ------------- | ------------------------------------- | ------------------------------- |
| `choice`    | Select from developer-defined options | Which team handles this ticket? |
| `noul`      | Assess a yes/no question              | Does this message ask for help? |
| `score`     | Rate against a defined ordered scale  | How urgent is this request?     |

A `choice` question returns an option and its probability distribution.

The API calls its yes/no question type `noul`; it returns the probability of yes. A `score` question returns a position on an ordered scale and probability information.

My application still decides what to do with those answers. Selecting `billing` may route a ticket to a queue. It must not give the model permission to issue a refund.

### System One, Jev, and Laya

[Jev](https://docs.typesafe.ai/models) is TypeSafe's hosted System One model. [Laya](https://huggingface.co/convaiinnovations/laya) is a local alternative.

[Unsloth Studio can serve Laya](https://unsloth.ai/docs/models/decision-laya) through a Jev-compatible `/v1/systemone` API.

The compatible interface does not imply identical behavior. Jev and Laya calculate confidence differently, so a threshold tuned for one cannot be copied safely to the other.

## Why not use a PowerShell switch or a regular LLM

If a ticket already contains a reliable department field, I would route it with ordinary code.

But what should the script do with this?

> I can't upgrade my plan: checkout crashes and my card was charged.

Is this sales because the customer wants to upgrade? Technical support because checkout crashes? Billing because their card was charged?

A keyword script can match all three. I need a priority policy and a way to judge the message against it. This is where a decision model may help.

Here is how I think about the options:

| Approach       | Where it fits                               | What to watch for                            |
| -------------- | ------------------------------------------- | -------------------------------------------- |
| Ordinary code  | Explicit rules over known values            | Ambiguous language is hard to encode         |
| Decision model | Bounded classification, routing, or tagging | A valid answer can still be the wrong answer |
| Standard LLM   | Generation, explanation, and synthesis      | Validate output and evaluate task quality    |

An LLM can also return `{"team":"billing"}`. Depending on the server, a schema can constrain that output.

The distinction is therefore not "JSON versus prose." It is whether a purpose-built decision interface, including its probabilities, is useful inside an application policy.

## Where this fits in an agent harness

An [agent harness](/2026/06/26/BuildingAnHarnessAgentWithMicrosoftAgentFramework/) could require this check whenever a ticket arrives. An ordinary script can do the same; an agent is not required.

For the ticket example, the flow could look like this:

1. Receive the ticket.
2. Ask the decision model which team should handle it.
3. Validate the answer and apply the application's review policy.
4. Route the ticket using ordinary code, or send it for human review.
5. Use a standard LLM if a reply or explanation needs to be drafted.

The application requires the decision call. It does not leave a general-purpose LLM to decide whether the check should happen.

## The local workflow

I used Windows Terminal to demonstrate two paths:

1. Laya classifies the ticket, then GitHub Copilot drafts a reply from the decision results.
2. A local Gemma model classifies the ticket and drafts a reply in one request.

This demonstrates workflow shape, not relative model quality. A controlled comparison would use the same reply model, environment, inputs, and repeated runs.

### Before running the scripts

The examples require PowerShell 7 and [Unsloth Studio](https://unsloth.ai/docs/new/studio/install). I started Studio with `unsloth studio -p 8888`.

* Enable **Settings > API > Decision API > Serve requests** and load Laya.
* Load a regular chat model and enable its OpenAI-compatible API.
* Have your Studio API token ready, or enable localhost keyless inference.

Each script is self-contained. Laya answers typed questions. Gemma classifies the same ticket and drafts a reply. A third block sends Laya's results to Copilot.

The examples use Studio on port 8888. If your chat endpoint uses another port, change its URL. The [Unsloth API guide](https://unsloth.ai/docs/basics/api) explains the serving settings.

> [!WARNING]
> Keep both endpoints on loopback and disable server-side tools. Token entry is hidden, but the bearer header exists in memory. Do not print `$Headers` or expose it in a recording.

> [!CAUTION]
> The Copilot step sends the ticket and decision results to an external service. Real tickets may contain personal, payment, or contract data. Use only an approved data path and apply your organization's redaction and retention policies.

### Split Windows Terminal

To get that layout, use this [Windows Terminal command](https://learn.microsoft.com/en-us/windows/terminal/command-line-arguments) from PowerShell with PowerShell 7 installed:

```powershell
wt -w new --maximized new-tab --title "Laya | Decision Model" --suppressApplicationTitle pwsh -NoLogo `; split-pane -V --title "Standard LLM" --suppressApplicationTitle pwsh -NoLogo
```

The escaped semicolon separates the Windows Terminal commands. `-V` creates a vertical divider, with the panes beside each other.

Both panes are clients. The split does not imply separate model servers.

### Left pane - Laya decision model

This REST call sends the ticket as `state` and asks three typed questions. Enter the API token, or press Enter if localhost keyless inference is enabled.

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

The output includes the model, answers, probabilities, usage, and `elapsed_ms`. `$Response` retains the parsed API response.

`laya` selects the decision model configured in Studio. Record the returned model name and the loaded checkpoint when comparing runs.

### Right pane - Regular LLM

This script sends the message to Unsloth's chat endpoint. Enter the exact model ID shown by Studio or `GET /v1/models`, followed by the token.

The LLM gets the same team definitions and priority rule as Laya. It briefly shows its classification, then drafts a customer-facing reply in the same request. The output is plain text, not JSON.

Use a general-purpose instruct model. [Phi-4-mini-reasoning](https://huggingface.co/microsoft/Phi-4-mini-reasoning) is designed and tested for mathematical reasoning, not customer-service conversations.

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

The script prints the model, classification, reply, and elapsed time. `$Response` retains the full API response.

> [!NOTE]
> My first model choice, Phi-4-mini-reasoning, consumed all 1,024 output tokens inside `<think>` and never produced a final reply. Its model card targets mathematical reasoning, so I replaced it with Gemma.

The script rejects length-limited output instead of presenting partial text as a completed reply.

### Reading the two results

Laya returns a team choice, refund probability, and urgency score. These values can feed application policy.

Gemma classifies and replies in one request. It uses the same routing policy but returns prose rather than typed decisions.

The complete paths are Laya plus a reply model, and one LLM doing both jobs. End-to-end timing for the first path must include both requests.

The demonstration uses Copilot after Laya and local Gemma alone. Any difference may come from the model, hosting environment, network, or number of requests.

Neither script routes a ticket or issues a refund. The LLM has no account access. Any claim that an action occurred is unsupported.

To try the ambiguous case, replace `$Ticket` in both scripts with `"I can't upgrade my plan: checkout crashes and my card was charged."` Both models use the same routing priority; the regular LLM also drafts a reply.

Both timers cover the HTTP call, parsing, and completion checks. They exclude token entry, request construction, and output formatting.

Run requests sequentially when measuring latency. Record cold and warm runs separately, and verify whether both models remain loaded.

### Send the decision results to GitHub Copilot CLI

Run this block in the same pane after Laya. It reuses `$Ticket` and `$Response`. Do not run the Gemma script there first because it overwrites `$Response`.

This requires an authenticated GitHub CLI with `gh copilot`. It sends the ticket and decision results to GitHub Copilot, so this step is not local inference.

The prompt explains the fields without prescribing the reply. The UTF-8 settings prevent PowerShell from corrupting curly apostrophes in captured output.

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

The timer includes CLI startup and the Copilot request, but not Laya. Review the reply before using it.

## What came back in the terminal

Here are the outputs from the two reply paths.

![GitHub Copilot CLI reply generated from Laya's decision results](/assets/2026-10-09-02.png)

![Gemma classification and customer reply from one local request](/assets/2026-10-09-03.png)

### Classification and replies

Gemma, running as `unsloth/gemma-4-12B-it-qat-GGUF`, returned:

* Team: billing
* Refund requested: yes
* Urgency: today

This agrees with Laya: `billing`, refund probability `0.9794`, and probability `0.9045` for "today." Laya's urgency score was `1.9005` on a zero-to-two scale.

The replies were different. Copilot wrote:

> I'm sorry for the trouble. I've routed your report about invoice #4411 to our billing team to review the duplicate charge. We'll confirm what happened and whether a refund is due; I can't confirm that a refund has been issued or promise it will be completed today.

Gemma showed its classification first, then included this in the reply:

> I have forwarded your request to our billing team to investigate the duplicate charge and process the refund for you as quickly as possible. We will notify you as soon as the refund has been completed.

I would reject both drafts. Both say the ticket was sent to billing. The scripts did no such thing. Gemma also describes refund processing as though it had started.

Copilot received an explicit warning that classification does not indicate action. It still claimed routing. Structured context did not keep the generated prose within the recorded facts.

In production, code should perform the authorized action and record its outcome. Only then should a reply model describe that outcome.

### Observed times

The terminal uses Dutch number formatting: `14.552,5 ms` means 14.6 seconds.

| Request                        | Observed time | What the timer covers                  |
|--------------------------------|---------------|----------------------------------------|
| Copilot follow-up after Laya   | 14.6 s        | CLI startup and reply, excluding Laya  |
| Gemma classification and reply | 73.7 s        | Local chat request and response checks |

These are single observations from different models and environments. They do not show that decision models are faster than chat models.

The Laya-to-Copilot path also includes Laya. A separate Laya run took 1.8 seconds, but I have not combined measurements from different runs into an end-to-end total.

The server's `process_time_ms` measures a different boundary and cannot replace client timing.

### Test environment and limitations

| Detail                  | Recorded value                            |
|-------------------------|-------------------------------------------|
| Operating system        | Windows                                   |
| Shell                   | PowerShell 7                              |
| Decision serving        | Unsloth Studio, version not recorded      |
| Decision model          | Laya multilingual, checkpoint not recorded |
| Local reply model       | `unsloth/gemma-4-12B-it-qat-GGUF`         |
| Copilot CLI version     | Not recorded                              |
| CPU, GPU, RAM, and VRAM | Not recorded                              |
| Warm-up state           | Not recorded                              |
| Number of measured runs | One per displayed result                  |

Those gaps prevent a reproducible performance comparison. The run shows that the workflows execute and exposes a grounding failure; it does not establish accuracy, cost, or latency rankings.

## What I would measure next

A representative labeled set is required to compare classification quality. Several hand-picked tickets only verify that the workflow runs.

For measurements beyond this demonstration, I want to:

* Use the same classification task and labeled test set.
* Use the same reply model in both complete workflows.
* Separate cold starts from warm, sequential requests.
* Report median and tail latency over repeated runs.
* Record invalid outputs, review rate, and accepted-case accuracy.
* Evaluate replies for usefulness and unsupported claims.
* Measure memory before making resource claims.

A reproducible run also needs hardware, server version, checkpoint, quantization, input length, and generation settings. Model aliases can change, so the model version should be pinned.

## How much should I trust the probability

[Unsloth's Laya guidance](https://unsloth.ai/docs/models/decision-laya) advises using the selected option's probability rather than its `confidence` field.

An answer at `0.80` can still be wrong. If the model is calibrated on my ticket distribution, predictions near `0.80` should be correct at roughly that rate across many examples.

I have not tested whether Laya's probabilities are calibrated on my tickets. [TypeSafe's confidence documentation](https://docs.typesafe.ai/confidence) explains the concept, but my workflow still needs its own labeled evaluation.

A low threshold risks incorrect routing. A high threshold may send nearly everything to review. The useful measures are review rate and accuracy among automatically accepted cases.

An LLM writing "I am 95% confident" does not provide a comparable probability.

## When would I use this

Ticket routing is a reasonable starting point because the choices and policy can be defined and evaluated. Bounded tagging and triage may fit for the same reason.

For reliable fields and explicit rules, I would use ordinary code. For drafting or summarization, I would use an LLM.

The combined workflow is the interesting one: a decision model judges, code validates and acts, and an LLM drafts a reply from the recorded outcome.

The useful boundary was not decision model versus LLM. It was judgment versus action.

Laya could classify the request, and an LLM could explain it. Neither should claim that my application did something it never did.

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
