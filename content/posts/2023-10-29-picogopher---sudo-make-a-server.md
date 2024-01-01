+++
title = "+mala's gopherhole: 2022-10-29 - PicoGopher - sudo make a server"
date = 2023-10-29
+++

```
    ----------------------------------------
     PicoGopher Part 2: sudo make a server
     October 29, 2022
    ----------------------------------------
     Written on my laptop
    ----------------------------------------
  
  
  Welcome to Part 2 of my PicoGopher journey! If you want to
  see how everything started, you can find the previous parts
  in my phlog at gopher://gopher.club/1/users/mala/ (also
  mirrored at gopher://gopher.3564020356.org). You can find the
  project's code at https://github.com/aittalam/PicoGopher.
  
  In Part 1 we talked about the why of PicoGopher (TL/DR: it is
  a supercool project and you definitely want to walk around
  your city while serving your phlog to strangers ;-P), and we
  got inspiration from (ie. blatantly copied) the official
  documentation of Raspberry Pi Pico W to build a simple mockup
  Gopher server. That was just a proof-of-concept, serving a
  text file which already contained what a server is expected
  to send to a Gopher client. The purpose of this post is to
  document the following step, i.e. how we can transform our
  mockup into an actual server, which gets a request for a
  given path and returns the corresponding file, gophermap, or
  error message.
   

  =========  Project status =========
  
  Below you can see the current status of the project. The
  steps marked as "x" have been completed, while those marked
  as "+" are described in this post. Namely we will focus on
  (1) translating gophermaps into the format that Gopher
  clients expect to receive and (2) parsing client requests and
  serving corresponding files.
  
  
    - [x] connect to the WiFi
    - [x] run a simple HTTP server
    - [x] run a mockup Gopher server
    - [x] load/save files
    - [+] make the Gopher server not a mockup anymore:
      - [+] translate gophermaps following gopher protocol
      - [+] load any file from disk
  
      -> at this point, you have a running Gopher server!
  
    - [ ] make the server a bit more resilient
      - [ ] enable async
      - [ ] better understand power saving
    - [ ] set up the pico as an access point for geolocalised
          access
  
    - [ ] powering PicoGopher
  
  
  
  ==== Interpreting gophermaps ====
  
  I think this is the point where a serious Gopher developer
  would provide a link to the RFC [1], describing the protocol
  in detail. But, honestly, there's not a lot to interpret out
  of gophermaps and to have something up and running quickly I
  preferred to look at the simplest server implementation I
  could find, which is exactly what I am going to describe
  here.
  
  Searching for "tiny gopher server python" I stumbled upon
  gofor [2], an unsurprisingly tiny gopher server written in
  python. Be warned, this might not be the best server you can
  find around: all the three commits to the repo occurred on a
  single day three years ago, and the author describes it as a
  "stupidly tiny, nonconformant (but functional) Gopher
  server". But, hey, I liked the honesty and, above all, the
  150 lines of super simple python code. 
  
  Most of the code we need resides in the "data_received" 
  function: first of all, it checks the request, making sure it
  matches a plausible Gopher selector (i.e. does not contain 
  tabs which are specific to Gopher+ [3] and points to a valid 
  path). Then, there are two possibilities: either a file was
  requested (so we need to directly serve it) or a directory, 
  in which case we look for a gophermap and interpret it.
  
  The transfer of generic files is quite straightforward: data
  is read from the file in blocks of size "block_size" and then
  sent through the socket. There are a couple of caveats for
  PicoGopher though which is worth knowing:
  
  - the device does not have much memory, so choosing a rather
    small value for block_size is preferable (I got some OOM
    errors with 64KB so I reduced it to 4, feel free to play
    with larger values if you want to find the sweet spot)
  
  - when I first reimplemented this code in micropython I
    naively used cl.send() to send data to the socket, as in
    the HTTP example shown in the Pico W tutorial. Only after
    testing it I realised that cl.send() might perform "short
    writes", ie. send through the socket fewer bytes than the
    actual length of the data. The solution is to use another
    function such as write(), which tries to send all data to
    the socket. OF COURSE gofor's code was already correct,
    so I should have just blatantly copied it :facepalm: :-)
  
  
  The following step, i.e. gophermap interpretation, is quite 
  simple as it does not really check for correctness of the 
  input file. It just assumes every row returned to the client 
  has to contain four columns as defined in the RFC: itemtype
  concatenated with some content, path, domain and port, all 
  separated by tabs. So whenever a line is read from a 
  gopherfile, it is split using the tab character as separator,
  and depending on how many columns we end up with one of five 
  things occurs: 
  
  - one column: it has to be text, so it is converted to "i" 
    (inline text type) + the content itself, then empty path, 
    domain, and port
  
  - two columns: they must be valid itemtype+content and path, 
    so we just add domain and port
  
  - three columns: we must have just missed the port so we just
    add it
  
  - four columns: just keep the row as it is
  
  - more than four columns: we only keep the first four
  
  
  After this, gofor code takes care of fixing relative paths. 
  For some reason there was a screwup with HTTP links: a "/"
  was prepended and Lagrange did not open them, so I decided to
  apply a small fix in my version of the code.
  
  
  ==== Accessing files on disk ====
  
  Gofor's code strongly relies on pathlib to evaluate whether
  the received path matches an existing and valid directory or 
  file. The research I did might have been quite superficial
  but I haven't found something equivalent in micropython, so
  I used the outputs of os.stat to implement my own tiny
  version which I called PicoPath. To do this I relied on the
  implementation of stat you can find here [4]. It is far from
  being complete, but it allows me to have code which is more
  readable and more similar to gofor's implementation.
  
  One more note regarding file access: I customised gofor's
  code so all requests are chrooted to a "gopher" directory,
  and kept all the extra checks as they were originally
  implemented. I am quite sure the system is not really secure
  so be careful with what you put on this thing! For now, I am
  more interested in making this work...
  
  
  ==== Running your server ====
  
  ... Aaand it works! :-)
  
  If you want to try serving your own gopherhole from a Pico W:
  
  - clone the PicoGopher GitHub repo [5]
  - copy your whole gopherhole in the "gopher" dir
  - open the 01_gopher.py file in Thonny and customize it with
    your WiFi ssid and password 
  - run the script
  
  
  The Pico will first try to connect to the WiFi, then you will
  see in Thonny's Shell pane which IP address it has been
  assigned. Open your gopher browser and connect to that IP!
  
  
  My post at https://fosstodon.org/@mala/109231106321211643 
  shows this very gopherhole being served by a Pico W. While 
  I am quite excited about this, I feel like we are not "there"
  yet. We do have a nice proof of concept, but before this has
  any chance to become something useful we should at least make
  it:
  
  - self-contained: it has to run without being connected to 
    a computer
  
  - portable: one should be able to put it into a backpack and
    bring it around, or leave it somewhere forever, without
    worrying whether it has enough power
  
  - ubiquitous: right now it relies on an existing wifi, but we
    should make it work everywhere by making it *provide* a 
    wifi to other devices
  
  - available: we should be able to choose whether we want to
    make it available only to people who have a gopher client
    installed or to anyone who has a browser
  
  
  There are already a few more things I would like to implement
  but I will stop here :-) Go try this version of PicoGopher
  and let me know what you think about it! Next time we will
  tackle some more of the missing steps and decide how to
  continue from there. If you have questions or ideas before
  that, I am very happy to hear them.
  
  
  
  [1] https://datatracker.ietf.org/doc/html/rfc1436
  [2] https://github.com/yam655/gofor
  [3] https://github.com/gopher-protocol/gopher-plus/blob/main/gopherplus.md
  [4] https://github.com/python/cpython/blob/main/Lib/stat.py
  [5] https://github.com/aittalam/PicoGopher

```
