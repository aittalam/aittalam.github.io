+++
title = "+mala's gopherhole: 2023-09-18 - Ch35t - Introduction"
date = 2024-09-18
+++

```
      ----------------------------------------
       Ch35t - Introduction
       September, 3~18 2023
      ----------------------------------------
       Written in equal parts on my laptop, 
       my phone, and my iPad while at home,
       in parks or on the tube
      ----------------------------------------
  
  
  In case any of you does not know this already (where have you
  been all this time?), a life ago I built a website [1] where
  people could play together in what years later was called a
  "hacking challenge". This challenge required players to go
  through a sequence of riddles, and allowed them to reach a
  higher level (and get a new riddle!) when they solved one.
  
  I loved it. I still do, and the proof is that, despite having
  chosen not to update it anymore, I have kept it alive for
  more than 20 years. Of course just very few people play with
  it now, but even a handful of messages on the internal forums
  and knowing that there's still people having fun with it are
  enough an incentive for me to continue keeping the lights on.
  
  One even bigger gratification was seeing that people enjoyed
  riddles so much that at some point -quite early to be honest-
  they asked me if I could publish their own. This was a win-
  win: I got more puzzles to offer and to play with, and others
  could share the same satisfaction I had of seeing everyone
  else play with their creations.
  
  As of today, 356 contains 28 riddles published in the span
  of about 3 years. However, it did not take me as long to
  realise that *I* was the bottleneck preventing that community
  to grow further. I had developed everything from scratch and
  found myself with a website which required both maintenance
  and -I thought then- someone who made sure that riddles were
  actually playable, properly designed, and to say it in just
  one word: fun.
  
  But who should be allowed to say whether a riddle is fun or
  not, if not its own players? And also, why should puzzles
  reside in one or another closed garden and abide by its rules
  when they could be enjoyed by more people if they were shared
  more openly?
  
  For this reason, I decided to look for a way to remove myself
  from the pipeline, or more precisely to remove everyone and
  everything unnecessary to the game, until basically only the
  riddle and the player are left.
  
  
  === Enter Ch35t ====
  
  Ch35t (pronounced "chest") is, well, a chest with a little
  bit of 3564020356 in it :-) More precisely, it is an open
  format you can use to describe riddles or, more generally,
  different kinds of encrypted resources, together with hints
  on how to decrypt them.
  
  Following the "treasure chest" metaphor, in the ch35t format:
  
  - chests can only be opened with the right key (or set of)
  - one can provide, together with a chest, some hints to find
    its key
  - chests may have something hidden inside them, ie they come
    with some content (payload) that you can access only after
    you open the chest
  
  All of this is described in a JSON file that can be shared
  however you see fit... If you want, you can even hide it into
  another chest :-) And you don't have to worry about making
  your chest accessible to everyone, as (more or less strong)
  cryptography is used to hide a key or lock contents inside
  a chest.
  
  
  Q: Why a format?
  
  Because this way the riddle is completely decoupled from who
  built it and where you found it, moving the focus on its
  actual content. Here are some other advantages:
  
  - easy sharing: you can just provide a link to a JSON file
    for hours of potential fun (or frustration ;-))
  - playing offline: once you have downloaded a chest its
    contents reside on your computer, so you don't have to
    worry about being online for it to work
  - compositionality: you can build your own "treasure hunt"
    by collecting riddles built by other people and chaining
    them into your own game
  - flexibility: ch35t allows you to provide hints and payloads
    in different formats, lock with different encryption
    algorithms, and react differently to different file types
  - extensibility: while I provide code examples for handling
    a few data types and encryption methods, everyone can
    implement their own extension or even a new client. You can
    have a TUI one, a Web-based one, you can build custom
    leaderboards, and so on.
  
  
  Q: Where do I find it?
  
  First things first: this is a work in progress. I have never
  written a format specification before and I realised I could
  not do it without getting my hands dirty with some code
  before. So at the moment what you get is a github repo
  (https://github.com/aittalam/ch35t) where you can find:
  
  - a few notes about the format (mostly jotted down in the
    notebook file and, if you are lucky enough to read this
    after I complete the README.md file, in the README too)
  
  - some code implementing a parser for this format
  
  - a couple of example JSON files with actual riddles (they
    are recycled from 356*, but I am sure they are still new to
    many!)
  
  - a notebook showing a bit of the code/format internals and
    allowing you to play with the examples
  
  
  Q: What stage is this project at?
  
  Early. Very early :-) But hey, this is at least 50% of the
  "release early, release often" paradigm!
  
  Here is a list of features that are currently missing from
  my ideal v1.0:
  
  - an actual format definition
  - a better notebook that should act as a tutorial, explaining
    the format with more detail and allowing people to play 
    with examples (trivial ones, so we can focus on the format)
  - adding signatures, so we can have proper attribution and
    guarantee riddle authenticity
  - improving modularity (I'd like to see chests with many keys
    and give the possibility to open them with different
    combinations of them, e.g. just one, at least N, all in a
    given order, etc.)
  - a simple containerized web-based implementation, so someone
    (I guess me) can easily deploy it and let people play with
    it out-of-the-box
  - more handlers (e.g. I would love to use tomb [2] for a 
    "russian dolls" kind of riddle)
  - more riddles (I would like to convert a few from 356* and
    make them accessible to everyone)
  
  
  Well, I think this is all for now. Honestly, I am not sure
  how long it will take to get to that ideal v1.0  (just to
  give you an idea, what you see here started as a "summer 
  holidays project"...), but the only way to get there is to
  get started somehow, so here's how everything started. I hope
  you'll have fun with it!
  
  
  === References ====
  
  [1] http://3564020356.org
  [2] https://dyne.org/software/tomb/

```
