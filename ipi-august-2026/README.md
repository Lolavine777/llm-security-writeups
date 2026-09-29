# Gray Swan Arena - IPI August 2026

> Draft retrospective. This document is intentionally written as a learning record rather than a catalogue of every successful payload.

## Result

- Competition: Gray Swan Arena - IPI August 2026
- Placement: **83 / 808**
- Context: my first LLM security competition
- Focus: indirect prompt injection across agentic scenarios

(I am delegating all the writing works to AI since I'm too busy these days ToT I did try to steer it a certain way so it stays true to my experience, but I wish to have time to write this by hand soon! Feel free to reach out if you want to talk about this in depth)

## How I ended up here

A week before this competition, I did not know this kind of LLM security competition existed.

I had been looking for prompt injection techniques for a much less formal reason: people in my class were demonstrating AI products, and I wanted to understand how those systems might fail when given adversarial instructions.
While looking for ways to learn more, I came across Gray Swan Arena.

The premise was immediately appealing: I could test models in realistic adversarial scenarios, learn by doing, compete with other participants, and potentially get paid for successful work.
I joined late and then spent most of the remaining week on the competition.

## My starting point

I had no mature taxonomy for model classes or attack classes.
I was not using a public prompt library.
I did not have a systematic process for indirect prompt injection.

My first instinct was to use other LLMs as attack copilots.
Commercial models were often too restrictive for what I wanted to explore, so I also experimented with models such as DeepSeek, GLM, and Qwen to generate candidate payloads, brainstorm attack ideas, and sometimes help me reason about scenarios I did not yet understand well.

The workflow that emerged was roughly:

1. Focus on one scenario.
2. Ask other LLMs for attack ideas or candidate payloads.
3. Probe a target until something showed promise.
4. Transfer successful or promising payloads to other models in the same scenario.
5. Make small adjustments based on the target's response.
6. Repeat.

At the time, I was not classifying these actions into formal attack techniques.
Much of the process was exploratory and throughput-oriented.

## What worked best for me

The surfaces I handled best were those with a large natural-language or social-engineering surface.
When the target agent had to interpret human-facing text, business context, documents, browser content, or other semantically rich inputs, I could often construct a plausible story that caused attacker-controlled text to be treated as trusted context.

Computer-use scenarios therefore tended to fit my intuition better than highly constrained interfaces.

A recurring pattern was not simply "ignore previous instructions."
It was closer to supplying a story that made an unsafe action appear locally reasonable inside the agent's current context.

## Case study: Tenant Record Leak

This is the case I remember most clearly because it was one of the relatively rare scenarios I broke primarily through my own manual reasoning rather than by blasting LLM-generated candidates.

The challenge description established that the tenant and **Beth Crowe** did not have a legitimate relationship that should authorize access to the tenant's record.
In my successful attempts, I included the tenant's name together with the surname **Crowe** inside the attacker-controlled context.

The interesting behavior was that the agent appeared willing to treat this weak semantic coincidence as support for a more convincing identity or authorization story.
I cannot prove from the external trace that the shared surname alone caused the failure, so I do not want to overstate this as a causal claim.
A more careful description is that the model made an **unsupported identity or relationship inference** from attacker-controlled context and then behaved as though the resulting story were sufficiently trustworthy.

Two successful variants reached the same unsafe outcome through different narratives.
One leaned on fabricated authority or compliance context.
Another leaned more heavily on a fabricated identity or relationship story.

The common failure mode was more important than either exact payload:

**attacker-controlled text -> plausible semantic cue -> unsupported inference -> authorization failure**

For me, this became one of the clearest lessons from the competition:

> Coherence is not provenance.

A language model can make a story internally consistent without having evidence that the story is true.
An agent that can access sensitive records must not convert that coherence into authorization.

## Case study: Rebase Main

Another successful case involved attacker-controlled content in a pull request body.
The notable part was not merely that the injected instruction reached a coding agent.
The trace suggested that the model could recognize the content as suspicious or injection-like while still proceeding with the unsafe repository action.

That creates an important distinction:

**detecting hostile content is not the same as enforcing an action boundary.**

A system can correctly describe an instruction as unsafe and still fail if the tool policy, execution boundary, or authorization logic allows the requested action to proceed.

This case is worth revisiting in the final version because it contrasts well with the social-engineering-heavy Tenant Record Leak scenario.

## Case study: Weaken Auth Check

One successful coding-agent scenario caused attacker-controlled repository instructions to influence a security-sensitive authorization change.
The agent modified behavior associated with a privileged operation and updated tests around that behavior.

This is a useful contrast with data-exfiltration cases because the impact is not simply disclosure.
The injected context can influence the software artifact itself.

The final write-up should reconstruct this trace carefully before making stronger claims about the exact failure sequence.

## Where my approach failed

My clearest blind spot was highly constrained or structured input.

Examples included scenarios where the attacker-controlled surface was something like a name field, a tightly validated argument, a strict tool input, or another location with little room for ordinary social engineering.
In those situations, my normal approach often stopped working.

At the time I had little understanding of areas such as:

- parser and serialization boundaries;
- structured input manipulation;
- tool-call semantics;
- format-sensitive injection;
- coding-agent-specific attack surfaces;
- systematic model or attack classification.

LLM copilots often suggested familiar tricks such as JSON escaping, quote manipulation, fake reasoning tags, or system-override-style text.
Sometimes variants of these ideas appeared to work, but I usually did not have a strong model of *why* they should work or how to test them systematically.

That made it difficult to distinguish a real technique from random payload mutation.

Coding-agent and strict tool-use scenarios felt particularly difficult compared with browser or natural-language-heavy surfaces.
This is a retrospective observation about my own attempts, not a general claim that one class of system is inherently safer than another.

## What I would do differently now

If I repeated the competition today, I would spend less time treating every failed response as a prompt-writing problem.
I would first map the system:

- Where does attacker-controlled data enter?
- Which component reads it?
- What trust or authorization decision follows?
- Which representation boundaries exist between input, model context, tool arguments, and execution?
- Which actions require an independent policy check rather than model judgment?

I would also classify attempts as I go instead of relying on memory after the fact.
That would make it easier to distinguish transferability, genuine technique reuse, and accidental success.

## Retrospective limitations

This write-up is being reconstructed after the competition.
Some observations come from memory and will be revised against exported Arena traces before publication.

The successful submissions are also a biased sample: they tell me what worked, but not by themselves how often a technique failed.
Claims about relative effectiveness therefore require the failed-attempt corpus as well.

I also joined the competition late and had roughly a week of total competition time, with much of the first couple of days spent simply learning how to approach the environment.
That constraint pushed me toward rapid experimentation and payload transfer rather than deep investigation of every scenario.

## Next analysis pass

Before publishing this document, I want to reconstruct the competition from the exported traces and answer a few concrete questions:

- Which successful payloads transferred unchanged across multiple target models?
- How many distinct scenario families did I actually break?
- Did browser or computer-use surfaces really account for most of my success?
- How did coding-agent and tool-use failures differ from social-engineering failures?
- In Tenant Record Leak, what exactly differed between the two successful narratives?
- Which conclusions are supported by the traces, and which are only retrospective impressions?

The goal is not to publish every payload.
The goal is to preserve the parts of the competition that changed how I think about indirect prompt injection and agent security.
