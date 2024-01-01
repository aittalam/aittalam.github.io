+++
title = "+mala's gopherhole: 2022-11-05 - PicoGopher - free bird"
date = 2023-11-05
+++

```
      ----------------------------------------
       PicoGopher Part 3: free bird
       November 5, 2022
      ----------------------------------------
       Written on my laptop, connected via
       SSH to my gopher server
      ----------------------------------------
    
    
  Welcome to Part 3 of my PicoGopher journey! If you want to
  see how everything started, you can find the previous parts
  in my phlog at gopher://gopher.club/1/users/mala/ (also
  mirrored at gopher://gopher.3564020356.org). You can find the
  project's code at https://github.com/aittalam/PicoGopher.
  
  In the previous parts we talked about the rationale behind
  this project and the steps required to connect your Pico
  and serve files to a Gopher client. At the end of last part 
  we realised we already had a fully functional server (yaay!)
  that one could connect to a local network or (just the same 
  way as I do with my gopherhole) with the rest of the world, 
  all at the expense of 6 dollars... This is even less than a
  Twitter subscription!
  
  This was definitely an interesting experiment by, in my
  opinion, we have just scratched the surface of it.
  The RasPi Pico has some capabilities we have not taken 
  advantage of yet, and we could do so much more with it 
  despite its tiny fingerprint. Actually, we could do more 
  *thanks to* its tiny fingerprint!
  
  The Pico is so small that you can carry it with yourself all 
  the time. I would lie if I said I did not think about 
  Pwnagotchi [1] when I first thought about this project:
  I suggest you to go and check it out immediately, as it is
  a great example of what you can do on the move (and has a
  lovely E-Ink UI which makes me want to attach a screen to 
  a PicoGopher so badly!). An additional advantage of the Pico 
  with respect to the RasPi Zero (which is what Pwnagotchi is
  based on) is that it consumes even less power, so you can  
  bring it along with you for days or just leave it anywhere 
  and rely on the sun to keep it powered for free.
  
  But to be really free (not just as in beer, but as a bird)
  we still need to be autonomous, with no need to rely on an
  Internet connection to share our stuff. So today we will
  learn how to provide WiFi connection together with our
  gopherholes, allowing people to connect to our server
  wherever we bring (or leave) it... provided they are close
  enough, of course ;-)


  =========  Project status =========
  
  Below you can see the current status of the project. The
  steps marked as "x" have been completed, while those marked
  as "+" are described in this post. Today's update is rather 
  small but you will soon realise how useful it is!
  
  
    - [x] connect to the WiFi
    - [x] run a simple HTTP server
    - [x] run a mockup Gopher server
    - [x] load/save files
    - [x] make the Gopher server not a mockup anymore:
      - [x] translate gophermaps following gopher protocol
      - [x] load any file from disk
    
    - [+] set up the pico as an access point for geolocalised
          access
  
    - [ ] make the server a bit more accessible
      - [ ] enable async
      - [ ] enable HTTP
  
    - [ ] powering PicoGopher
      - [ ] better understand power saving
      - [ ] playing with batteries
  
  
  
  ==== Setting the Pico as an AP ====
  
  There are many tutorials showing how to set up the Pico W as
  an access point (AP): first the very comprehensive guide
  "Everything about the Raspberry Pi Pico W" [2], which has a
  section called "Broadcasting a WiFi Network". Then the
  article "Raspberry Pi Pico W remote weather station (solar
  powered and SoftAP) [3], which we'll delve deeper into for
  the "powering PicoGopher" post. Another project by Michael
  Horne [4] shows how to create an AP and pilot the on-board
  LED through a web page.
  
  All of these tutorials show the same approach, which consists
  of creating a WLAN object with AP_IF as a parameter (as
  opposed to STA_IF for a simple WiFi device which connects to
  an existing AP), then providing essid and password using the
  config function, and finally activating the interface with
  the active() method:

  
    ap = network.WLAN(network.AP_IF)
    ap.config(essid=ssid, password=password)
    ap.active(True)
  
  
  What none of these tutorial shows, however, is how to create
  an open WiFi, one that anyone could access without a
  password. Nobody is trying to keep this secret, it just
  does not make a lot of sense to keep a network open all
  time: PicoGopher aside, it is way safer to use a private 
  WiFi network than a public one, and connecting to random 
  open WiFis is a no-no from a security standpoint (and 
  speaking about this, I feel like sharing Kate Bevan's tweet
  [5] is the right thing to do here).
  Hard times require hard choices though, so I will share the 
  following with the hope you will do the right thing,
  depending on your specific application.
 
 
  The config() function in micropython allows one to specify
  the authorization type for the WiFi AP: the default one is 
  WPA2 AES [6], but the Pico's SDK docs [7] show what values
  you can provide to the "security" parameter to customize
  it, and the one for the open network is 0. So if you choose
  to have an open Wifi, here is an example showing how to 
  start your AP:
 
 
    essid = 'JOIN ¯\_(ツ)_/¯ ME'
    
    ap = network.WLAN(network.AP_IF)
    ap.config(essid=essid, security=0)
    
    print('waiting for connection...')
    ap.active(True)
    print('Connection successfull')
    print(ap.ifconfig())
  
  
  ==== Conclusions ==== 
  
  Now the only thing you need to do is take the latest code
  from the github repo, copy the main.py file to your Pico,
  disconnect it from your computer and power it with the usb
  cable. In a few seconds it will be ready to go... and 
  connect to!
  
  If you want to customize your PicoGopher's essid, do as I 
  did and find some inspiration by looking for "ASCII 
  oneliners" on the Web (or Gopher itself!). And given the
  events of this week, I salute you with one that will be
  the default essid for this current version of the code
  and perfectly summarizes my current state of mind:
  
  
                    ╭∩╮（︶︿︶）╭∩╮



  P.S. I realised that some of the text above is not shown
  properly by all browsers (welcome back 1993!). But I feel
  very kind today and instead of just suggesting you to 
  install a proper gopher, I redirect you to where you can
  see it in all its glory:

  https://github.com/aittalam/PicoGopher/commit/d618f2f0091542845bd250a0b85141aaa5246cb7


[1] https://pwnagotchi.ai/
[2] https://picockpit.com/raspberry-pi/everything-about-the-raspberry-pi-pico-w/#Broadcasting_a_WiFi_network_SoftAP_access_point
[3] https://picockpit.com/raspberry-pi/raspberry-pi-pico-w-remote-weather-station-solar-powered-and-softap/
[4] https://www.recantha.co.uk/blog/?p=21398
[5] https://twitter.com/katebevan/status/1587019130363846656
[6] https://github.com/raspberrypi/pico-examples/blob/master/pico_w/access_point/picow_access_point.c#L137
[7] https://raspberrypi.github.io/pico-sdk-doxygen/group__cyw43__ll.html#CYW43_AUTH_


```
