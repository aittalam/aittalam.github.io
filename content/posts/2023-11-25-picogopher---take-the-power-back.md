+++
title = "+mala's gopherhole: 2022-11-25 - PicoGopher - take the power back"
date = 2023-11-25
+++

```
      ----------------------------------------
       PicoGopher Part 5: take the power back
       November 25, 2022
      ----------------------------------------
       Written on my laptop after a flu, while
       brew upgrade is running
      ----------------------------------------
   
   
  Welcome to Part 5 of the PicoGopher journey! If you want to
  see how everything started, you can find the previous parts
  in my phlog at gopher://gopher.club/1/users/mala/ (also
  mirrored at gopher://gopher.3564020356.org). You can find the
  project's code at https://github.com/aittalam/PicoGopher.
  
  We now have a tiny server which runs both on Gopher and HTTP
  protocols. Both implementations are super simplified and can
  definitely be improved, however I decided to postpone this in
  favor of another aspect of this project. When I started this
  journey [1] I claimed that, being low-power, PicoGopher could
  be brought around while being powered with small batteries or
  left somewhere indefinitely while being recharged with solar
  panels. And in the last weeks you might have seen photos of
  my "to go" prototypes on GitHub or Mastodon/Twitter. At the
  end, though, a question still remains: how feasible is this?
  That is, how much power does PG really take to run, how much 
  is it going to last on batteries, and is it realistic to
  think it will just keep on working when recharged by solar
  cells?
  
  To answer these questions, today we will finally talk about 
  power: not the one which is abused by billionaires ;-) but 
  the one we will unleash to freely share information with
  others (yeah that sounded a bit cheesy, but you got the gist
  of it!)
  
  
  =========  Project status =========
  
  Below you can see the current status of the project. The
  steps marked as "x" have been completed, while those marked
  as "+" are described in this post. The plan for today's post
  was to completely cover the "power" section, but as soon as
  I started playing with solar I realised that there's so much
  to say about it that it deserves an episode on its own. So
  please bear with me, and I will make the wait worth it :-)
  
    - [x] connect to the WiFi
    - [x] run a simple HTTP server
    - [x] run a mockup Gopher server
    - [x] load/save files
    - [x] make the Gopher server not a mockup anymore:
      - [x] translate gophermaps following gopher protocol
      - [x] load any file from disk
    
    - [x] set up the pico as an access point for geolocalised
          access
  
    - [x] make the server a bit more accessible
      - [x] enable async
      - [x] enable HTTP
      - [x] captive portal with PicoDNS
  
    - [+] powering PicoGopher
      - [+] better understand power consumption
      - [+] playing with batteries
      - [ ] playing with solar
      - [ ] better understand power saving
  
  
  Speaking about worthiness, I do care a lot about making
  something that is useful to others, both for what concerns
  the project and its documentation which includes these posts.
  The main concern I have about this now is that with anything
  hardware related I am in uncharted territories, so bear with
  me if I am writing something silly that just spawns out of my
  inexperience and please please please give me feedback on
  that, so I will be able to fix it and make these posts a bit
  better!
  
  And now with the uncharted territories...
  
  ======== Power consumption ========
  
  The first thing we need to understand is what power the RasPi
  needs to run PicoGopher. This splits into two main questions:
  
  1. how can we measure how much power PicoGopher needs?
  2. how can we translate this value to some practical metric
     such as "I need this amount of batteries to run it for x
     hours/days"?
  
  The thread at [2] answers both of these questions and a few
  more, such as "what is the simplest way to power a Pico". A
  multimeter is used to measure the power consumption while
  running code, and a simple back of the envelope calculation 
  is done to estimate how long it will take to drain a given
  set of batteries. 
  
  At the cost of stating the obvious, power (P) is measured in
  Watts and is the product of voltage (V) and current (I),
  respectively measured in Volts and Amperes. This means that
  once you have two of them, you can always easily calculate
  the third one. Now, the back of the envelope calculation in 
  the above thread is as follows: 
  
  "Power consumption when running this code is approximately 
  0.1W (19mA at 4.99V), so 4 x AA batteries (@ 2,000mAh each) 
  would keep the Pico running for well over 4 days"
  
  Note that the calculation is done by simply focusing on the
  current (2000mAh / 19mA = ~100hrs). This is perhaps overly
  simplified (as eg. it does not take into consideration the
  battery efficiency) but I think it is good for a first
  estimate. [3] shows a different approach which also takes
  efficiency into consideration, to only later realise that
  the project lasted ~60% of what they expected! So yeah, there
  definitely is a science behind all of this and definitely
  other variables we should take into consideration but, once
  more, I am okay with something reasonable even if still rough.
  
  About the number of batteries, note that as they are
  connected in series (to get to a reasonable voltage for the 
  Pico) the current is not affected. If you are curious about
  the voltage stuff, we'll get there in a sec.
  
  
  =========== Measurements ==========
  
  For the measurements I used an AT35 multimeter (15GBP on
  Amazon, see https://www.youtube.com/watch?v=BK83N3X2oVs for 
  a review). To make a fair estimation of the consumption in a
  real case scenario, I ran as much stuff simultaneously as 
  possible (not true: just some), spinning up Gopher, HTTP, 
  and captive portal, and sending requests repeatedly from 
  three different terminals. Each terminal requested something
  different, respectively the main gopherfile, a large (174KB)
  text file via Gopher, and the same file via HTTP. 
  
  Small trivia: the large file is the txt version of [4], the 
  11th document in Project Gutenberg and the first novel 
  uploaded there, aka Alice in Wonderland. The file is now part
  of PicoGopher and available in the GitHub repo for testing... 
  and sharing!
  
  The three commands I ran to measure consumption under heavy
  load are: 
  
  - while true; do printf "11-0.txt\r\n" | nc 192.168.4.1 70; done
  - while true; do printf "\r\n" | nc 192.168.4.1 70; done
  - while true; do wget http://192.168.4.1/11-0.txt; done
  
  The outcome of my measurements showed that PicoGopher usually
  consumes about 60mA at rest (with WiFi on), and under heavy
  load it went up to peaks of ~88mA a few times but in general
  it was within a 70~74mA range.
  
  
  ======== Adding batteries =========
  
  There are different methods to power a RasPi Pico on the
  move. Pico's datasheet [5] provides a good description of
  how to do it with one or multiple power sources or adding a
  battery charger). The simplest way is also shown in [2]: you
  just connect the power source to pins 39 and 38 (VSYS and 
  GND)... and that's it. The only problem left is understanding
  how many batteries to use and choose the proper ones.
  
  Simplest solution: regular AA/AAA batteries. AA hold some 
  2000mAH, while AAA about half ot it. Again this is super
  approximated, and you might want to check [6] and [7] to get
  more info about it (oof no matter what of these tiny details
  one looks into, there's a whole new world behind each of
  them!). To give you a rough idea, if we take an average
  consumption of 80mA, AA batteries should last for 2000/80=25 
  hours.
  
  How many batteries? The working voltage of the RasPi Pico W
  is in the range of ~1.8V to 5.5V. This means that you can
  use almost anything in range (neat!). As the voltage is not
  constant but decreases while batteries discharge, you will
  want to start with something which is way above the lower
  threshold. This means you should be fine with 3xAA cells
  (~3.0V to ~4.8V) or 3xAAA (smaller fingerprint, but will last
  less). You can also use rechargeable ones: 3 or 4 as they
  typically have lower voltages (~4.4V to ~5.2V), and you might
  get as much as 2800mAh and 1300mAh with NiMH AAs and AAAs
  respectively.
  
  Speaking about rechargeable batteries, Li-Ion are a good
  choice if you want something compact (you can pack a higher
  voltage in a much smaller space) and that you can recharge in
  many different ways. One option is using a LiPo SHIM for the 
  Pico [8]: its main advantage is that it is tiny, quite well
  integrated with the Pico, and already manages both battery
  overcharge and charging while simultaneously powering the
  device. The main cons are that it's a bit pricy (7.50GBP) and
  you need headers on the Pico.
  
  A cheaper alternative with a bit less soldering to do 
  (#SolderItIntoExistence is my hashtag for the week!) is using
  a TP4056 (see e.g. [9]) with a Schottky diod as explained in 
  [10]: we will delve deeper into this project next time as it 
  has been the reference for my solar prototype, but I wanted 
  you to know that you can also use it with USB as its main
  power source.
  
  
  =========== Conclusions ===========
  
  We should now be able to estimate how much power PicoGopher
  needs to run and for how long it should be able to run on a
  given power source among different types of batteries. I have
  collected some specs below to get an idea of what the best
  choice could be for different applications and while e.g.
  rechargeable AAs are great, Li-Ion (perhaps with a bigger
  form factor and relative mAh) could be a good choice for a
  self-contained, solar solution which what I will describe in
  the next post.
  
  Type            mAh     hrs@80mA
  AA (x3-4, r)    2800    35      
  AA (x3, nr)     2000    25      
  Li-Ion (r)      1200    15      
  AAA (x3-4, r)   1100    13.75   
  Li-Ion (r)      500     6.25    
  
  
  As usual I think I barely scratched the surface here, and
  there are many things I would like to delve deeper into (in
  addition to solar of course), including
  
  - reading how much charge the battery holds
  - using the above, testing whether my prediction of battery 
    duration were reasonable :-)
  - low power mode (e.g. when the battery level drops below
    a given point the pico could go in sleep mode and restart
    later when hopefully it has been recharged)
  
  ... ok, ok, I'll stop here. See you next time!
  
  
  ============ References =========== 

[1] gopher://sdf.org:70/0/users/mala/phlog/2022-10-23%20-%20PicoGopher%20-%20Introduction
[2] https://forums.raspberrypi.com/viewtopic.php?t=300676
[3] http://raspi.tv/2013/controlled-shutdown-duration-test-of-pi-model-a-with-2-cell-lipo
[4] https://www.gutenberg.org/ebooks/11
[5] https://datasheets.raspberrypi.com/picow/pico-w-datasheet.pdf
[6] https://www.batteryequivalents.com/aa-batteries-size-types-and-equivalents.html
[7] https://www.batteryequivalents.com/aaa-batteries-size-chemistry-types-and-replacements.html
[8] https://shop.pimoroni.com/products/pico-lipo-shim 
[9] https://www.amazon.co.uk/gp/product/B07BSVS842
[10] https://picockpit.com/raspberry-pi/raspberry-pi-pico-w-remote-weather-station-solar-powered-and-softap


```
