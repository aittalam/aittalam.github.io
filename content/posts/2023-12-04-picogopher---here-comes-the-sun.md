+++
title = "+mala's gopherhole: 2022-12-04 - PicoGopher - here comes the sun"
date = 2023-12-04
+++

```
      ----------------------------------------
       PicoGopher Part 6: here comes the sun
       December 04, 2022
      ----------------------------------------
       Written on my laptop, at a desk under 
       a pile of electronic junk
      ----------------------------------------
    
    
  Welcome to Part 6 of the PicoGopher journey! If you want to
  see how everything started, you can find the previous parts
  in my phlog at gopher://gopher.club/1/users/mala/ (also
  mirrored at gopher://gopher.3564020356.org). You can find the
  project's code at https://github.com/aittalam/PicoGopher 
  (commit 39b2777).
  
  In the previous parts I showed how to develop PicoGopher from
  scratch, serve a gopherhole via Gopher and HTTP, redirect
  user connections to its IP via a captive portal, and power it
  with different kind of batteries depending on the use you
  want to make of it and its estimated consumption.
  
  Today we are going to power PicoGopher with solar. One of the
  main reasons I decided to jump in this field, which is new and
  completely unexplored for me, is that I'd like to think about
  PicoGopher as a sustainable way for people to build their own
  "geolocalized servers". These servers would also need to be as
  cheap as possible, as the plan would be to leave them more or
  less hidden somewhere, and generally unattended, potentially 
  for long periods of time... I guess "not having your server 
  stolen" is a reasonable extra problem to consider, but I will 
  leave it as an exercise to the readers :-)
  
  Ultimately, I would like not just to prove this thing is
  possible in theory, but also have some back-of-the-envelope
  calculations to quantify how likely it is to work, how stable
  it will be, and so on. The good news is that there's quite
  some interesting material around and my work can rely on the
  experiments of many other people who attempted something
  similar. Just to name a few, the incredible Low-tech Magazine
  has a version of its website which is running on solar, and
  they documented both its creation [1] and its power analysis
  [2]. I particularly like how they show, in each page of the
  website, the battery level of the server, so everyone can
  estimate for how long it will be available before some
  sunlight is needed again. Another very interesting project 
  is the one by Krzysztof Jankowski [3], who built a Gemini 
  capsule on a solar-powered ESP8266 [4] (which made me want
  to either build a Gemini server for PicoGopher or a Gopher 
  server on the ESP8266! ;-)). Last but not least, [5] shows
  how to build a solar-powered weather station and it has been
  my reference (as well as Krzysztof's, looking at the solution
  he implemented) for a solar prototype.
  
  
  ==== Steps to build PicoGopher ====
  
  Below you can see the current status of the project. The
  steps marked as "x" have been completed, while those marked
  as "+" are described in this post. 
  
  
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
  
    - [x] powering PicoGopher
      - [x] better understand power consumption
      - [x] playing with batteries
      - [+] playing with solar
      - [+] monitoring power
      - [+] better understand power saving
  
  
  As you will see in the following, powering our project with
  solar is not a tremendously complicated task per se. However,
  as it involves a bit more electronics than I am comfortable 
  with and depends on external factors such as having enough
  sunlight, I felt the need to add a few thoughts on monitoring
  power (so you can understad what is happening in realtime)
  and power saving (so you can deal with moments when there is 
  just not enough sun to keep PG running).
  
  
  ======== Playing with solar =======
  
  Solar power for DIY projects now seems to come as a turnkey
  solution from many online electronics stores. Being a total
  newbie in this field, I first looked on Pimoroni for a
  ready-to-use circuit and ended up buying Adafruit Universal 
  USB/DC/Solar Lithium Ion/Polymer charger [6]. For a little 
  less than 15GBP, this provides a hassle-free solution that 
  you just need to connect to a 6V solar cell (e.g. [7]), a 
  3.7~4.2V battery (e.g. [8]), and of course the RasPi Pico.
  
  This approach significantly reduces the amount of things one
  has to solder (provided all the components have the proper 
  plugs, i.e. 2-pin JST to connect battery+Pico and 5.5x2.1mm 
  DC plug for power) but of course you pay for your peace of 
  mind: the total cost for all the components is ~32 GBP, of 
  which just 6 are for the Pico itself. 
  
  Looking for a cheaper alternative, I also built an equivalent
  of what is presented in [4,5]: this makes use of a TP4056 [9] 
  and a Schottky diod [10] for a cost of about 2 GBP, bringing
  the grand total below 20 Pounds. The only problem I found is
  that the solution is controversial to say the least. [11]
  warns that the TP4056 should not be used both as a charger
  and as a load driver at the same time, because when a load is
  present the charger might not detect charge completion so it
  might overcharge. Comments on [5] suggest to directly use a
  solar power management module like Waveshare's [12], which I
  have not tried but might be a good solution for our task at
  a smaller price than Adafruit's. Finally many, many tutorials
  on YouTube show a load which is directly connected to the
  same TP4056 outputs used for the battery.
  
  Long story short, I realised that the more I delved deeper
  into this rabbit hole the more I was getting inconsistent
  information, so I decided to try and solve a different
  problem. If I want to compare different approaches, how can
  I *measure* how battery charging/discharging is going?
  
  
  ========= Monitoring power ========
  
  Being able to monitor incoming power has many advantages.
  First of all, you can measure empirically whether the time
  estimates we made in the previous post were realistic, i.e.
  how much time PicoGopher will leave with a given set of 
  batteries. Then you can estimate battery health by looking
  at their charge/discharge curves [13,14] (and perhaps also 
  get some insight into whether they are being overcharged). 
  Last but not least, you can take decisions about what to do
  with your system depending on your battery charge level:
  for instance you can put it in standby mode, send a warning
  message to your readers, etc.
  
  For all this reason, I looked into how to make battery info
  available to the Pico and found that it is possible to use
  one of its analog pins (ADC0, ADC1 or ADC2, respectively
  available as GP26, GP27, and GP28 - see pinout in [15]) to
  read a voltage input. The value which is read from an ADC pin
  is an unsigned 16-bits int, so it will range between 0 and
  65535 to represent voltages in the 0~3.3 Volts interval.
  
  Note, however, that while the maximum input voltage is 3.3V
  the Pico can be powered with voltages up to 5.5V. For this
  reason, we must find a way to reduce the input voltage to an
  ADC pin to a value which will not damage the Pico. The
  solution is using a voltage divider [16], which is nothing
  more than two resistors connected in series, with the input
  voltage (the one we are powering the Pico with) applied
  across the resistors pair and the output voltage (the one we
  are sending to the ADC pin) emerging from the connection
  between the two of them:
  
  
                  Vin ---+
                         |
                         R1
                         |
                         +---- Vout
                         |
                         R2
                         |
                  GND ---+
  
  
  The relationship between the input voltage Vin and the output
  voltage Vout is:
  
  Vout = Vin * R2 / (R1+R2)
  
  This means that:
  
  - if R1 and R2 have the same resistance, then Vout = 1/2 Vin
  - if R1 = 2*R2, then Vout = 1/3 Vin
  - provided we know the relationship between R1 and R2, their
    value does not matter for the sake of calculating Vout. It
    is important though to control the current consumption, so
    it is best to use relatively large resistance values (e.g.
    100K Ohm) to avoid wasting too much current for the
    measurement.
  
  
  The schematics and a few examples of connections with the
  Pico are available in [17,18]. For PicoGopher I chose two
  identical 100KOhm resistors and ADC3 (GP28) for tracking the
  battery charge values, so I had to multiply by 2 all the
  voltage readings and my conversion factor becomes
  
    conversion_factor = 2 * 3.3 / 65535
  
  Note that this will change depending on which resistors you
  use in the voltage divider. Just to give you an example, the
  Pimoroni Pico Lipo Shim seems to have R1 = 2*R2 because they
  multiply the voltage read by 3 [19]. Their code also provides
  an estimate on the battery percentage left which is quite
  convienient: I blatantly copied into my code, adapting it for
  my Li-Ion batteries' voltage.
  
  Once I could read the voltage properly, I decided to track it
  in two different ways: the first one was by logging it into a
  `gopher/picopower.log` file; the second was by showing it
  live on an e-ink display (Pimoroni's Pico inky). The code for
  both is available on GitHub.
  
  The display allowed me to check the battery level at any given
  time, and from its latest update before the Pico turned off I 
  could confirm that I can easily run PicoGopher for more than 
  12 hours on a fully charged 3.7V 1200mAh Li-Ion battery. This 
  was a bit less than my initial estimation but I think it was 
  reasonable, especially considering the fact that I added some
  extra machinery including the voltage divider, the TP4056 and
  the display.
  
  With the logs I was able to plot a discharge profile showing
  me the impact of solar cells during the day. This was
  particularly useful as it made me realise that London's sun
  in December is definitely not enough for a 1W solar cell to
  recharge my battery! Looking for a way to measure which kind
  of cells I would need for my task, I found [20] which allowed
  me to make a first estimate which I am trying to improve now
  (see [21] or, long story short, I will need at least 4W for a 
  decent performance here).
  
  
  ========== Power saving ===========
  
  The Rp2040 datasheet [22] shows that the Pico can be put into
  different power saving states: SLEEP and DORMANT. It also
  provides C "hello_sleep" and "hello_dormant" examples showing
  how to do it. [23] shows how to do it in micropython instead,
  using light or deep sleep and consuming as low as 1.4mA during
  sleep. The story behind this though is quite interesting and
  it includes various experiments and patches [24,25] that were 
  applied to micropython before the function was officially 
  built into it. If you are still alive after this huge amount
  of references, I'd suggest you to take a look at it :-)
  
  I personally have not implemented power saving mode into
  PicoGopher yet, but my plan is quite simple: after voltage
  reading, if the value is below some threshold I will just put
  PG to sleep for a while. Ideally I would like to use the
  battery's voltage itself as a trigger to wake the Pico up,
  but I think I will settle for a fixed amount of time first.
  You will likely see this first implementation in the next
  update to picopower.py!
  
  
  =========== Conclusions ===========
  
  In this post I tried to summarize my attempt at making a
  solar-powered version of PicoGopher. Solar is a super
  interesting subject which was completely new to me and a
  rabbit hole which was definitely worth falling into. 
  
  I think I am still quite far from the ideal solution I had 
  in mind -mainly because the requirement for a bigger solar 
  panel makes it a bit harder to leave PicoGopher wherever 
  you like- but at the same time I am quite excited to know 
  it is not just possible, but also way easier to implement 
  in warmer and sunnier places. 
  
  My soldering skills have definitely improved in the process
  and I have documented everything with photos, which I will
  share somewhere (I guess on my fosstodon account) and link 
  here soon. 
  
  This said, I think there are plenty of ways I could still
  improve this side of the project, and I would be very happy
  to hear feedbacks (and suggestions!) from you all. Feel free
  to reach out to me on Mastodon or via email, and many thanks
  to those of you who have already done it!
  
   
============ References =========== 

[1] https://solar.lowtechmagazine.com/2018/09/how-to-build-a-lowtech-website.html
[2] https://solar.lowtechmagazine.com/2020/01/how-sustainable-is-a-solar-powered-website.html
[3] https://krzysztofjankowski.com/
[4] gemini://gemini.p1x.in:1966/
[5] https://picockpit.com/raspberry-pi/raspberry-pi-pico-w-remote-weather-station-solar-powered-and-softap/
[6] https://www.adafruit.com/product/4755
[7] https://www.amazon.co.uk/gp/product/B0BDX551VC
[8] https://shop.pimoroni.com/products/lipo-battery-pack?variant=20429082183 
[9] https://www.amazon.co.uk/gp/product/B07BSVS842
[10] https://www.amazon.co.uk/gp/product/B087C7QRXF
[11] https://www.best-microcontroller-projects.com/tp4056.html
[12] https://www.waveshare.com/solar-power-manager.htm
[13] https://learn.adafruit.com/li-ion-and-lipoly-batteries
[14] https://batteryuniversity.com/article/bu-501a-discharge-characteristics-of-li-ion 
[15] https://datasheets.raspberrypi.com/picow/pico-w-datasheet.pdf
[16] https://en.wikipedia.org/wiki/Voltage_divider
[17] http://raspi.tv/2013/controlled-shutdown-duration-test-of-pi-model-a-with-2-cell-lipo
[18] https://blog.rareschool.com/2022/08/how-can-you-make-your-picopico-w.html
[19] https://github.com/pimoroni/pimoroni-pico/blob/main/micropython/examples/pico_lipo_shim/battery_pico.py
[20] https://re.jrc.ec.europa.eu/pvg_tools/en/tools.html
[21] https://fosstodon.org/@mala/109439846884942035
[22] https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf
[23] https://ghubcoder.github.io/posts/pico-w-deep-sleep-with-micropython/
[24] https://ghubcoder.github.io/posts/deep-sleeping-the-pico-micropython/
[25] https://github.com/tomjorquera/pico-micropython-lowpower-workaround

```
