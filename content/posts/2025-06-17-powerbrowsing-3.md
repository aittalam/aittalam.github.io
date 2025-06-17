+++
title = 'Powerbrowsing 3.0 (or not?)'
date = 2025-06-17
draft: true
+++

[Some time ago](http://davide.eynard.it/malawiki/PowerBrowsing.html) I
decided to start teaching people the concept of *Powerbrowsing*.
The rationale behind it was that everyone at that time already knew how to
*browse* the Web, but companies on the Web were already taking too much
control of our browsing experience... And I wanted to get that control back.
The good news was: everything runs on your computer, you have (at least in
theory) full control of what runs on it. And following this principle I
enjoyed teaching people how to reverse-engineer websites, how to crawl them,
how to extract structured information from unstructured text and use that
information to build new useful things.

The bad news is: more than 20 years passed, and we are still letting companies
take control of our experience not just on the Web but, due to Internet's
pervasiveness, basically everywhere. And the thing is, my feeling now is like
they took ideas that were dear to me -as they represented, in my thoughts, a
way to liberate people- and twisted them in a way that now makes everyone
pissed off not even at those companies, but at those ideas themselves.

One example? Crawling. I taught crawling as digital self-defense - a way to
extract the information you needed without submitting to manipulative
interfaces packed with ads and dark patterns. Now that same technique hammers
small, self-hosted websites with AI training requests, forcing individuals to
pay bandwidth costs while tech giants build billion-dollar models.

AI is another thing I devoted 20 years of my life to, and while I hereby
unapologetically share with you my love for this subject, I also want to be
mindful about the implications it currently carries with its more or less
recent developments. And as with crawling (and powerbrowsing in general), I
think it can have a very negative impact on people if used in the wrong way,
but it can also be a very powerful tool to take back control of our lives.

# enthusiasm


# concerns

## laziness, just shove everything in

- problem is not just losing touch with what's happening
- objective issue with the agents themselves

## connect anything with anything

- interoperability dream, but at the same time it gives tools which are *too* powerful in the hands of *too* many people
- what about control? how easy does it become?

## enshittification path (see below)

- strong incentives into locking
- it's already very difficult for open solutions to compete with closed ones
- even when MCP servers run locally, if you trust a served model the (possibly personal) information you are feeding your agent might leak

# a survival guide to enshittification

- empower us when searching information
- first experiments with adversarial interop
- I see a lot of potential

# conclusions

- own your AI: run local models when possible
- learn your tools: do not be lazy! Check your inference engine logs (to understand if things are working properly), customize the way it works (e.g. change default context length on ollama)
- optimise your agent: simpler/shorter prompts (beware the context size!), more specific/explicit instructions, choose from a subset of tools which work best for your use case (select them from the MCP server!)
- think: who is this tool going to empower? is this gonna be a centaur or reverse centaur case?

* * *

- enshittification path:
    
    - things "just work" if you use closed, more expressive models so there's an incentive into locking
    - the MCP ecosystem is great, as it allows you to abstract from how / where their code runs. The issue could be that you don't realise it but you are leaking your personal information somewhere or locking into some service which will, at some point, become a pay-for-use or a subscription
    - even when MCP servers run locally, if you trust a served model the (possibly personal) information you are feeding your agent might leak
- "just shove everything in, let the agent figure it out approach":
    
    - context size issues: huge prompts are passed to the LLM with very very long lists of available tools (AFAIK lists of whatever is available, not something linked to your specific prompt/use case)
    - ambiguity: a "web search tool" might be used instead of a more specific tool, or it is not clear whether one should be preferred to another. (from few experiments it looks like more ad-hoc tools which provide structured results might provide more deterministic output, even with less expressive models)
- "connecting everything with anyything"
    
    - I was super enthusiastic about this when talking about powerbrowsing
        - at the time it was "to allow people be in control of their tools" (the browser, information available on the Web)
        - the main principle was being able to get structured information from unstructured data (HTML, APIs, etc) and use them for your needs
    - how are these tools used now instead?
        - increase control over people (the very same tool we are building for the hackathon is not ideal if put into the hands of an employer)
        - reverse centaur phenomenon
- how could this become a part of a survival guide to enshittification?
    
    - help with search
    - help move out of some system you have been locked into
    - help to reverse engineer stuff (is it us breaking the law if the agent is doing that without us hinting about that?)
    - improve interoperability among closed systems
- interesting points / conclusions
    
    - read Cory Doctorow's post (https://pluralistic.net/2025/05/27/rancid-vibe-coding/)
    - the future is here, it's just not evenly distributed
    - risks: lean into laziness, get us even more dependent and locked in
    - advantages: centaur model (can we make everyone, not just developers, a bit more like centaurs?)
- high-level principles:
    
    - own your AI: run local models when possible
    - learn your tools: do not be lazy! Check your inference engine logs (to understand if things are working properly), customize the way it works (e.g. change default context length on ollama)
    - optimise your agent: simpler/shorter prompts (beware the context size!), more specific/explicit instructions, choose from a subset of tools which work best for your use case (select them from the MCP server!)
    - think: who is this tool going to empower? is this gonna be a centaur or reverse centaur case?