# Own your AI agent: running open source agents on your terms

*Based on a talk given with David de la Iglesia Castro at OxML 2026.*

This is the written version of a talk we gave on building and running AI agents you actually control. The goal is not to convince you that commercial AI services are bad — they are, in many cases, excellent — but to show that you don't have to depend on them, and that you learn a lot more when you don't.

The post is structured the way the talk was: a motivating story, a short philosophical detour, a walk through the five primitives that every modern agent is built from, and then a series of concrete experiments with local models and tools.

---

## A parable, to set the scene

![Slide 4: Ada and Zangemann](slides/slide-04.jpg)

The desktop image above is from *Ada and Zangemann*, a children's book by Matthias Kirschner and Sandra Brandstätter (illustrated by David Revoy). The story features a man, Zangemann, who is a genius inventor. He builds wonderful tools for everyone — but each tool carries his own preferences, his own biases, his own decisions about what users should and shouldn't be allowed to do. The tools cannot easily be modified.

Ada is a little girl who pulls broken Zangemann devices out of the dumpster, takes them apart, and assembles her own custom tools from the pieces.

I (Davide) related strongly to Ada. Around her age I had my first business card that read *inventor*, and the way she approaches technology — by tinkering, by understanding, by adapting things to her own needs — is exactly the relationship I want to have with AI.

### Twenty years ago: power browsing

![Slide 5: Powerbrowsing metaphor](slides/slide-05.jpg)

In 2005 I gave a talk about something I called *power browsing*. The metaphor was simple: we see reality through our eyes, but our eyes can be helped. If we have myopia, glasses correct what we see back to what's really there. Sunglasses let us see in conditions where we'd otherwise be blinded.

I wanted to apply the same idea to the web. In the early 2000s, websites were drowning in pop-ups and ads — we have, in some ways, returned to that. My "eye with bad eyesight" was Internet Explorer. The "corrected eye" was Firefox with an ad blocker. And the "sunglasses" were custom bots that would crawl a site, extract the actual content I cared about, and serve it to me cleanly.

![Slide 6: The actual setup, a laptop in a drawer](slides/slide-06.jpg)

I built these tools in Perl, running on an old Compaq Armada that lived in a drawer in my house, connected to a slow DSL line and always on so I could query it from a Symbian phone. It was a nice experiment, and I learned a tremendous amount doing it.

### One year ago: vibe reversing

![Slide 8: Vibe reversing with Claude](slides/slide-08.jpg)

Last year I tried the same exercise again, but with Claude. The target was *Viaggiatreno*, the Italian railway website, which holds train timetables for every station in Italy. I wanted to read the timetables without clicking through the website's menus and ads.

Twenty years ago this took me weeks: I had to learn the site, learn Perl, learn regular expressions, write the crawler from scratch. This time I opened Claude, described the problem, and pasted a screenshot of the network tab. Claude found the JSON endpoints, suggested how to call them, and even built me a small UI. It worked basically out of the box.

So — is reverse engineering dead? Should we just ask Claude to do everything?

### The hard truth

![Slide 9: It worked, but...](slides/slide-09.jpg)

It worked. But four things bothered me:

- **I had already done this before.** I had enough prior knowledge to know whether Claude's suggestions were right. Without that prior knowledge, would I have been able to verify the answer?
- **The artifacts lived on Claude's platform.** If I had only asked "what time does this train leave?", Claude would have answered — and I would have been dependent on Claude for every subsequent question. I had to remember to explicitly ask for the code, the script, the thing I could take home.
- **Claude gave me excellent learning references.** Which, if I had only wanted the quick answer, I would have skipped entirely.
- **I wrote zero lines of code, and I learned nothing.** Twenty years ago I came out the other side with knowledge of HTTP, of curl, of regular expressions — knowledge I have reused for two decades. This time I came out with… nothing.

![Slide 10: The strongest critique](slides/slide-10.jpg)

When I wrote a blog post about this, I asked Claude to give the strongest possible criticism of my position. The reply was: *"You're a technical person telling non-technical people to make their lives harder to solve problems that mostly exist in your head."*

That stings. So let me ask: are these problems just in my head?

![Slide 11: Just in my head?](slides/slide-11.jpg)

Are you concerned about UX that changes from one week to the next? About inconsistent model performance? About sudden pricing changes — the thing that cost $20 last month and costs $50 this month? About sustainability, sharing personal data, the requirement to always be online, ads, lack of control?

If you answered yes to any of these, this post is for you. Just to add one example: while preparing this talk, the news came out that Elon Musk was offering compute to Anthropic. Whether or not that deal happens, the dependencies between providers and the people who own infrastructure mean nothing is guaranteed to be there for you tomorrow. I sleep better knowing at least some of what I do, I can do entirely under my own control.

---

## A short philosophical detour

When I studied AI, my professor — Marco Somalvico — preached philosophy of AI to us almost every class. At the time I wasn't sure I'd ever use any of it. Twenty years on, I use it almost daily. Two ideas in particular.

### The machine is a place

![Slide 13: The Mechanical Turk](slides/slide-13.jpg)

Somalvico used to say: *the machine is a place*. The Mechanical Turk wasn't a metaphor — it was literally a place with a person inside, moving the chess pieces. For us today the place is more abstract, but the principle holds: when you run an AI tool, there is always someone (you or someone else) doing the work inside. They can have more or less autonomy. The question worth asking is: *who is inside this machine, and on whose terms?*

### The Chinese Room

![Slide 14: The Chinese Room](slides/slide-14.jpg)

John Searle, 1980. A person inside a room receives questions in Chinese through a slot. They don't speak Chinese — but they have a very thick book mapping every possible input to a correct output. From the outside, the room appears to understand Chinese perfectly.

The argument is usually used to debate machine consciousness — and frankly, I don't care whether the room "really" understands. What I care about is the opposite move: can I be fooled into thinking the room understands more than it does? Can I be betrayed by surface-level competence?

This is why probing the room — testing where it breaks, what it actually knows — is worth doing, even when you treat the model as a black box.

### The right comparison

![Slide 16: The open source AI models room](slides/slide-16.jpg) ![Slide 17: The commercial AI services room](slides/slide-17.jpg)

When people compare open-source LLMs to commercial AI services, they often do something unfair: they run a local LLM in a chat interface, ask it a few questions, and compare the result to ChatGPT or Claude. But — as Anthropic's own "Building Effective Agents" diagram shows — a commercial service is not just an LLM. It's an LLM plus retrieval, plus tools, plus memory, plus a lot of engineering wrapped around all of it.

So the fair comparison is not "local LLM vs. commercial service". It's "local LLM with the same scaffolding vs. commercial service". The rest of this post is about how to build that scaffolding yourself.

---

## Every agent is the same five things

![Slide 20: Every agent is the same five things](slides/slide-20.jpg)

This section comes from David de la Iglesia Castro's part of the talk, based on two projects: [agent.cpp](https://github.com/mozilla-ai/agent.cpp) (a ~1100-line C++ runner using local GGUF models via llama.cpp) and [any-agent / tinyagent](https://github.com/mozilla-ai/any-agent) (a Python framework-agnostic interface over OpenAI Agents, smolagents, LangChain, Google ADK, LlamaIndex, and Agno).

Two completely different codebases. Same primitives. Same loop.

### The thesis: an agent is a loop

![Slide 21: An agent is a loop](slides/slide-21.jpg)

The whole thing reduces to two alternating phases:

1. **`model.generate`** — the LLM looks at the conversation so far plus the tool catalogue, and decides what to do next. It either calls one or more tools, or it produces a final answer.
2. **`tool.execute`** — if the model called tools, we (the runtime, not the model) execute them and append the results to the conversation.

Loop until the model emits no tool calls. Done.

Everything else — streaming, tracing, MCP, multi-agent, KV caching, error recovery — is plumbing around this loop.

### The five primitives

![Slide 22: The five primitives](slides/slide-22.jpg)

1. **Model** — the brain. Takes messages and tools, returns the next message.
2. **Tools** — the hands. Named, typed, callable functions the model can invoke.
3. **Instructions** — the mission. The system prompt.
4. **Callbacks** — the hooks. Lifecycle observers that can also rewrite what flows through the loop.
5. **Loop** — the engine that drives the alternation.

Names vary across frameworks. Semantics don't. Once you can name these five things, you can read any agent codebase.

### Primitive 1: Model

![Slide 23: Anything that implements one method](slides/slide-23.jpg)

A model is anything that implements one method: given a list of messages and a list of tool definitions, return the next assistant message. The returned message contains content (text for the user), or tool calls (structured requests to invoke tools), or both.

The model abstraction is what makes the loop portable. The loop doesn't know whether the model runs locally via llama.cpp on a GGUF file or remotely via the OpenAI API. It doesn't know how the chat template is rendered, or how the KV cache is reused. All of that is hidden behind the interface.

### Primitive 2: Tools

A tool is three things, always:

- A **name** — a stable identifier that the model emits when it wants to invoke this tool.
- A **definition** — a free-form description (the model uses it to decide *when* to call the tool) plus a JSON Schema for the arguments.
- An **execute** function — the actual code that runs when the tool is invoked.

It's worth being precise about how this works: the model cannot directly call your code. The model emits a JSON object saying "I would like to call the tool named `X` with these arguments". Your runtime parses that JSON, validates it against the schema, dispatches to your function, captures the return value, and feeds it back into the conversation as a tool message. The model then sees the result on the next iteration.

### Primitive 3: Instructions

The simplest primitive. It's a string — the system prompt. It goes first in the message list, and it tells the model who it is and how to behave. There are newer-sounding paradigms (skills, etc.) that look more complicated, but at the end of the day they're all instructions: clever ways of loading the right text into context at the right time without overwhelming the model.

### Primitive 4: Callbacks

![Slide 27: Six hooks, every argument mutable](slides/slide-27.jpg)

There are six points in the loop where callbacks fire:

- `before_agent_loop` — seed history
- `before_llm_call` — trim or inject context before the model runs
- `after_llm_call` — edit the parsed message
- `before_tool_execution` — approve, edit, or skip the tool call
- `after_tool_execution` — recover from errors
- `after_agent_loop` — final answer

The crucial design choice is that every callback argument is passed by *mutable reference*. Callbacks aren't just observers — they're rewriters. This is what makes the same six hooks powerful enough to implement all of the following:

![Slide 28: What mutable callbacks let you build](slides/slide-28.jpg)

- **Logging and tracing** (read-only before/after pairs)
- **Context engineering** — trim or summarize the message list before each model call
- **Guardrails** — inspect proposed tool calls, refuse dangerous ones
- **Human-in-the-loop** — show the user the proposed call, edit args, or skip
- **Error recovery** — turn tool exceptions into recoverable data

Almost everything beyond the core loop is a callback, not a change to the loop itself.

### Primitive 5: The loop

![Slide 29: The whole orchestrator in 14 lines](slides/slide-29.jpg)

```text
function run_loop(messages):
    ensure_system_message(messages)
    fire(before_agent_loop, messages)
    loop:
        fire(before_llm_call, messages)
        parsed = model.generate(messages, tool_defs)
        fire(after_llm_call, parsed)
        messages.append(parsed)
        if parsed.tool_calls is empty:
            fire(after_agent_loop, messages, parsed.content)
            return parsed.content
        for tc in parsed.tool_calls:
            execute_one_tool_call(messages, tc)
```

A few things worth noting:

1. The loop stops only when there are no tool calls. There is no built-in turn cap or timeout. If you want one, write a callback that raises.
2. Tool exceptions don't crash the loop — they become tool results with error content, the model sees them on the next iteration, and often recovers.
3. The caller owns the messages list. `run_loop` mutates it in place but stores no state between invocations.
4. Hooks fire in a fixed order, every iteration. Registration order is a contract.

### The recovery dance

![Slide 30: The recovery dance](slides/slide-30.jpg)

The way the loop handles tool errors is worth lingering on, because it's what makes agents resilient. If a tool throws, the exception is captured into an error result. The `after_tool_execution` callback gets a chance to recover (e.g., retry with different arguments). If the error is still there after recovery, the loop raises. Otherwise, the error is appended to the conversation as a tool result and the model sees it on the next iteration — typically the model then adjusts and tries again.

The principle: dump errors back to the model as data, not as crashes. The loop never sees the failure.

### Mapping real systems

![Slide 32: Mapping real systems to the primitives](slides/slide-32.jpg)

This same five-primitive structure shows up everywhere. Mozilla's wasm-agents (one HTML file, Pyodide, OpenAI Agents SDK). Mariozech's pi-mono (~680 lines of TypeScript). Nous Research's hermes-agent. Steinberger's openclaw. Anthropic's Claude Code itself. Same loop. Bigger agents just add more tools and more callbacks — they don't change the loop.

### A worked example: memory as three tools

To show how little code this actually takes, here's a complete memory-enabled agent from agent.cpp's examples. The instructions tell the assistant that memory is available about the user. Three tools are exposed: `list_memories`, `read_memory`, `write_memory`. No callbacks. Just the loop.

When the agent starts a fresh conversation, it calls `list_memories`, sees what's stored, and decides whether to read anything. When the user shares a new fact ("I love Galician cuisine"), the agent calls `write_memory` to persist it. The memory itself is just a JSON file on disk — it could equally be a database or a remote service.

Once you have this skeleton, you can see how memory might evolve: instead of three tools the agent has to remember to use, you could write a `before_llm_call` callback that automatically reads all memories and injects them into the context. Both implementations are reasonable; the right choice depends on your model and your token budget. The point is that *you can see the choice clearly*, because the primitives are visible.

---

## Know your tools

![Slide 35: Know your tools](slides/slide-35.jpg)

The rest of this post is about what happens when you actually run agents built this way. We will look at several things that go wrong, and several things that go right, with small local models.

### The bricks matter more than the language

We will move between code in C++, Python, JavaScript, and HTML in this section, because the point is not which language to write an agent in — Claude Code can translate between any of them for you. The point is to know what the *bricks* are: what a model call looks like on the wire, what a tool definition looks like, where context comes from, where it goes.

### A first run: counting Rs in StRawRbeRRRy

One of the demos in Mozilla's [wasm-agents-blueprint](https://github.com/mozilla-ai/wasm-agents-blueprint) is a single HTML file that runs a Python agent in your browser via Pyodide. It connects to a local LLM server (llama.cpp, Ollama, LM Studio — anything that speaks the OpenAI completions API) and exposes three tools: a character counter, a webpage fetcher, and a Tavily web search.

The classic test: *how many times does the letter R occur in the word "strawberry"?* — except misspelled, so the tokenizer can't help. With Qwen 3.5 9B running locally and the character counter tool available, the agent:

1. Receives the system prompt, the user prompt, the list of tools, and the model name.
2. Emits a tool call: `count_character_occurrences(character="r", word="StRawRbeRRRy")`.
3. The runtime executes the (one-line Python) tool. It returns `5`.
4. The result is appended to the conversation; the model is called again; it produces "There are 5 R's".

The interesting move here is that we have shifted *what we trust*. We are no longer trusting an LLM's stochastic guess about how many Rs are in a word. We are trusting a piece of Python code that we wrote ourselves and can read. The LLM's job is to *decide when to call it*, not to do the counting. For a brief moment last year, this setup with a 9B local model gave more correct answers than GPT online — because GPT wasn't reaching for a tool, and ours was.

### Context is all you need

![Slide 36: Context is all you need](slides/slide-36.jpg)

Here is a failure mode you only see if you read the logs. The user asks "how many stars does mozilla-ai/any-agent have on GitHub?". The agent reaches for the web search and fetch tools. But the default context window in some local inference servers (Ollama, until recently) is 4096 tokens. A GitHub page can easily blow past that. The result: the server silently truncates the input prompt, the agent loses the original question, and answers a completely different one — telling you how to install the `any-agent` library instead of how many stars it has.

Two lessons:
- Always check the context size of your inference engine, and set it explicitly. Defaults can be tiny.
- Some tools fail loudly when they run out of context. Others fail quietly, by just… working worse. The quiet failure is the dangerous one.

### Model expressiveness: same task, three models

![Slide 37: Qwen3-0.6B fails to use tools](slides/slide-37.jpg)

To show what model size buys you, here is the same prompt — "When was Davide Eynard born?" — run against three local models, with two tools available: `scan_current_dir` (find files matching a pattern) and `read_file` (read a file's contents). My birthdate isn't on Wikipedia, so no model should know it from training data alone. The only way to get it is to find the `birthdays.csv` file on disk and read it.

**Qwen3-0.6B**, with no hint: ignores the tools entirely and replies "I don't have access to Davide Eynard's birth date information." That's a fine answer, but the tools went unused.

![Slide 38: Qwen3-0.6B hallucinates](slides/slide-38.jpg)

Run the same model again: this time it hallucinates a confident, wrong date. The tools still went unused.

![Slide 39: Qwen3-0.6B with a hint](slides/slide-39.jpg)

Give it a hint — "look for the birthdays file to find out" — and it calls `scan_current_dir` with the literal pattern `davide_eynard_birthdate.txt`. No such file exists. Empty output. "I don't know."

![Slide 40: Qwen3-0.6B with a stronger hint](slides/slide-40.jpg)

Give it a stronger hint — "look for the birthdays CSV file" — and it finally gets there: `read_file("birthdays.csv")`, gets back the contents, returns the correct date.

The lesson is not "Qwen3-0.6B is bad". It's that the agentic behaviour we take for granted with Claude or GPT — figuring out which tool to use without being told — is not free. It's a function of model size and training. For a 0.6B model, you may need to tell it exactly which tool to use.

![Slide 45: Qwen3.5-0.8B figures it out](slides/slide-45.jpg)

**Qwen3.5-0.8B**, only marginally bigger but six months newer: with the hint "look for birthdays on disk", it tries `.txt` first, finds nothing, broadens to `*birthday*`, finds `birthdays.csv` and `agent_birthday.py`, picks the CSV (correctly judging it more likely to contain the answer), reads it, returns the date.

![Slide 47: Qwen3.5-9B fights wrong data](slides/slide-47.jpg)

**Qwen3.5-9B**: doesn't need any hint. Tries `*.md` first, then `*`, finds the CSV, reads it, returns the answer. But here's the interesting part — I deliberately put Alan Turing's *death* date in the CSV as if it were his birthdate, to see what would happen. The 9B model refused. It knew Turing's actual birthdate from training, and would not accept the CSV's contradicting value. Only when I explicitly said "trust the tool's answer" did it return the CSV's date, with the qualifier *"according to the birthdays.csv file"*.

This is worth pausing on. A larger model has more knowledge baked in, which means it will *fight* incorrect data fed to it via tools. That's a feature if your tool is buggy. It's a bug if the person you're asking about really does share a name with someone famous — say, an Alan Turing who is not *the* Alan Turing — because the model will keep insisting on the wrong answer.

### Overthinking: strawberry fields literally forever

![Slide 48: Qwen3:8b think mode overthinks "strawberry"](slides/slide-48.jpg)

Some models are trained with a "think mode" that produces extensive internal reasoning before answering. Here is what Qwen3:8b in think mode does with our R-counting question: it spends most of a screen debating whether "strawberry" might be a typo for "strawbery", whether there are three Rs or five, manually counting letter by letter, then second-guessing the count. The only reason it eventually gets to the right answer is that — eventually — it gives up reasoning and calls the tool.

Tools as guardrails: even an overthinking model produces a correct answer if it ends up calling the tool, because the tool's output is deterministic. The model has to decide to trust it.

### Who owns the search tool?

![Slide 49: GPT-OSS abuses search](slides/slide-49.jpg)

GPT-OSS, OpenAI's open-weights model released last year, has a different failure mode: it searches the web compulsively. Asked for five trending TV shows in 2025, it first searches "trending TV shows 2025", then for each show in the result it searches the release date separately, then searches the whole list again. Many searches for one question.

![Slide 50: Without search, GPT-OSS hits Wikipedia](slides/slide-50.jpg)

Take the search tool away and the model fetches Wikipedia pages directly. Some of the URLs it constructs are valid; many are 404s — it confidently invents page titles that don't exist. About a year ago, Wikipedia closed off non-browser user agents, partly in response to this kind of traffic. Your locally-running agent can become a small DDoS source if you aren't careful.

![Slide 51: Who owns search tools?](slides/slide-51.jpg)

And then there's the question of who owns the search tool itself. When Ollama released GPT-OSS support, they added a built-in web search that was on by default. The "offline" inference server was no longer offline. The search endpoint was opaque — the relevant code wasn't open source — so you couldn't see where your queries were going. Ollama also added an "airplane mode" toggle, off by default.

This is the point at which philosophical questions about ownership become concrete: who is in the room with you when you run a local model that does web search?

### What if search is your tool?

A more hopeful experiment. [SearXNG](https://github.com/searxng/searxng) is a free meta-search engine that aggregates results from many providers — Google, Wikipedia, Startpage, others — without keeping logs of your queries. You can run it locally (it fits on a Raspberry Pi, in a Docker image). Combined with a small local model and the `pi` agent framework as a host, you get an agent that does web search where *you* are in the room: you can see exactly which upstream services were called, you control which ones are enabled, your queries don't get logged for ads.

### And what if Wikipedia itself is your tool?

The [Kiwix](https://kiwix.org/) project distributes the entire Wikipedia in a single `.zim` file. The full version with images is around 50 GB; a text-only version is around 10–15 GB. An MCP server (`zim-mcp-server`) exposes the Zim file as a tool an agent can query. With this setup, "who was X born?" — for an X obscure enough that small models don't know but well-known enough to be on Wikipedia — works offline, with a small model, on a laptop.

### Building a small knowledge base

The other extreme is your own data. I keep notes in [Joplin](https://joplinapp.org/) — not because Joplin is special, but because it has an API. I built a small wiki about the `llamafile` project I work on, then asked a 9B local model with read access to those notes to summarize the main GPU-acceleration issues I'd worked on. The output was a good summary, with sections matching the documentation I'd actually written.

The pattern here — building a personal "wiki" populated by an LLM reading and restructuring your existing notes — comes from a recent gist by Andrej Karpathy. The point is that you do not need a state-of-the-art model to do useful retrieval over a few hundred of your own notes. A small local model and a few tools is enough.

---

## A new problem: stalking agents

![Slide 59: A new problem - stalking agents?](slides/slide-59.jpg)

One last experiment, half-joke and half-warning. I asked an agent to search the web for *my* birthdate and report both the date and the URL where it found it. The agent eventually located the right answer in a PDF CV hosted on a Lugano university website. I'm fine with that — I put the CV online myself.

But pause and consider: an agent that never gets tired, never stops searching, can correlate scraps of personal information across many sites in a way that nobody would have the patience to do by hand. The technology that lets you build a local agent for benign tasks lets anyone build one for surveillance. This is not a reason not to build agents. It is a reason to think carefully about what tools you give them and what you publish online.

---

## Learnings

To compress everything above into something portable:

![Slide 58: Learnings](slides/slide-58.jpg)

- **There's tinkering with AI and there's tinkering with AI.** You'll solve problems more quickly using AI as a tool. You'll learn more when AI is the object of your tinkering. Both are fine. Be deliberate about which one you're doing.
- **No one-size-fits-all solution.** The right setup depends on your task, your compute, your model's training, and the tools you need. There is no answer to "which model should I use?" without those four pieces of context.
- **Easy isn't useful.** Friendly UX is evolving fast and breaks often. Sometimes the bottom-up route — building from primitives — is the route that pays off long-term.
- **Think small.** Limit context growth. A handful of well-chosen tools goes a very long way. Most of what I showed above used just *fetch a web page* and *search*.
- **Agents are slot machines.** We remember the times they worked and forget the times they didn't. Try to tune them so the success rate is high enough to stop being a gamble, and have realistic expectations when it isn't.
- **Prevent enshittification.** With every choice — every tool, every model, every library — ask what data, freedoms, and control you're giving away. Especially for the tools that look free.

---

## Where to go from here

![Slide 62: And now? Be like Ada!](slides/slide-62.jpg)

Some concrete next steps if any of this resonated:

- Check out the [mozilla-ai GitHub organisation](https://github.com/mozilla-ai).
- Play with different agents — ours or other people's, doesn't matter as long as they're open source.
- Try writing your own "any-agent". 400 lines of Python is enough for the loop.
- Test different local models against the same task. The differences are educational.
- Try MCP servers — including ones you host yourself.
- Host a tool or service for your community.

Or do none of those. The point of this post isn't a to-do list. The point is to give you enough scaffolding to look at any agent — Claude Code, Cursor, your local llama.cpp script — and know exactly which of the five primitives you're looking at, where the context comes from, where it goes, and who is in the room with you.

Be like Ada.
