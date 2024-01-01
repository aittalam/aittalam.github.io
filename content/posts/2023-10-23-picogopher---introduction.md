+++
title = "+mala's gopherhole: 2022-10-23 - PicoGopher - Introduction"
date = 2023-10-23
+++

```
    ----------------------------------------
     PicoGopher - Introduction
     October 22, 2022
    ----------------------------------------
     Written on my laptop
    ----------------------------------------
  
  
  When I first heard about Gopher, I wondered how small its
  footprint was. Can you run a Gopher server on a very old 
  computer? Well, of course yes, given the protocol's been
  around for 30 years... Can you run it on a very small one? 
  Again, yes: gopher.3564020356.org runs on a RasPi 2, and if
  you wander enough around the gopherspace you will easily 
  end up in a gopherhole served by a Raspberry Pi Zero.
  
  Now the question is: how much can we shrink our resources 
  while still having a working Gopher server? In this post 
  (actually series of posts, if I want anything to be shared in
  a reasonable time) I show how to run your own gopherhole from
  a Raspberry Pi Pico W. And given the platform it runs on, the
  service it provides, and above everything my utter lack of 
  imagination, I have given this project the quite obvious name
  of PicoGopher :-) 
  
  The reasons why I am doing this are various, and probably the 
  most honest answer I could give is that I am trying to keep 
  myself distracted while everything around me is in turmoil.
  There are, however, more serious ones too: on the one hand, 
  I would like to somehow give back to the small internet
  community, by enabling more people to self-host and share 
  their own content in a less expensive way; on the other hand, 
  in a moment where all the main Internet services are becoming
  more and more centralised, I would like to plant a few ideas
  for an even more distributed Internet instead, one where 
  servers themselves can physically be displaced and even move
  with their owners.
  
  Physically move? Yes, one of the advantages of a very small,
  low-power server is that you just need a few small batteries
  to bring it along with you... And if you want your server to
  be always on, you can keep the batteries charged with a
  relatively small solar panel. This is actually something I 
  have always dreamed about doing: leaving some digital traces
  hidden around in the physical world. Think e.g. geolocalised
  riddles, something similar to what Cory Doctorow describes
  as part of the Harajuky Fun Madness challenges in his novel
  called "Little Brother"[0] which by the way I suggest you to
  read if you haven't done already. 
  Thinking about other possible applications of a cheap, 
  self-contained server that can be brought around (or left 
  somewhere) is left as an exercise to the reader ;-). Just
  do not let your imagination be restricted by the specific
  application of a Gopher server: a Pico has enough to run
  Gemini or HTTP too, to act as a proxy or as a gateway, to
  periodically download stuff for you and serve it in anther
  format. Have fun with it and let me know what you did :-)
  
  
  ==== The Raspberry Pi Pico W ====
  
  For all the details about the Raspi Pico W, I will leave you
  to the official datasheet [1] and the "Everything about the
  Raspberry Pi Pico" page [2]. To summarise:
  
  - it is tiny (as in thumb-sized)
  - it is cheap (as in ~7GBP/EUR/USD)
  - it has onboard WiFi (that's what the "w" stands for)
  - it has 2MB of flash memory (enough to hold quite some text)
  - it is based on the RP2040 microcontroller, which is super
    common now and available on a plethora of devices (for 
    instance, I think the Arduino Nano 33 BLE [3] could be used
    for this project instead of the Pico with similar results)
  
  
  ==== Steps to build PicoGopher ====
  
  After getting very basic information about how to program a
  Pico (you'll find plenty of good material about it both on
  the official website and in the links I shared previously),
  the following step was to get an idea about what I could do
  in terms of networking (both WiFi and TCP). RasPi's datasheet
  website provides a document [4] which has everything one 
  needs to get started, including code examples on how to 
  connect to the WiFi and how to build a simple HTTP server.
  
  Following this documentation I managed to run my first mockup
  Gopher server, which serves one single working gophermap 
  after loading it from flash memory. This is of course a super
  simplified example, but enough for us to know that we can now
  power this little thingie, let it connect to a WiFi, and then
  open our favorite Gopher browser and point to it to see some
  contents. This is 80% of what we expect our project to do,
  done in (less than) 20% of the time!
  
  You can find the mockup code in the PicoGopher GitHub repo
  [5]. Again it is not a lot, but if I want to be able to share
  something without disappearing for too long I need to keep
  these posts short (yeah I could also write more concisely,
  but you'd miss all the fun ;-P). 
  In my next PicoGopher posts I will try to apply the 80/20
  rule again while following these steps (the crossed ones are
  completed already), so you know what to expect each time. 
  
  
  - [x] connect to the WiFi
  - [x] run a simple HTTP server
  - [x] run a mockup Gopher server
  - [x] load/save files
  - [ ] make the Gopher server not a mockup anymore:
    - [ ] translate gophermaps following gopher protocol
    - [ ] load any file from disk
  
    -> at this point, you have a running Gopher server!
  
  - [ ] make the server a bit more resilient
    - [ ] enable async
    - [ ] better understand power saving
  - [ ] set up the pico as an access point for geolocalised
        access
  
  - [ ] powering PicoGopher
  
   
  ==== References ====
  
[0] https://www.gutenberg.org/files/30142/30142-h/30142-h.htm
    (look for "wifi" to find the HFM riddle)
[1] https://datasheets.raspberrypi.com/picow/pico-w-datasheet.pdf
[2] https://picockpit.com/raspberry-pi/everything-about-the-raspberry-pi-pico/
[3] https://store-usa.arduino.cc/products/arduino-nano-33-ble
[4] https://datasheets.raspberrypi.com/picow/connecting-to-the-internet-with-pico-w.pdf
[5] https://github.com/aittalam/PicoGopher

```
