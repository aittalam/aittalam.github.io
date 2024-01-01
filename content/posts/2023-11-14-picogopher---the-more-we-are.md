+++
title = "+mala's gopherhole: 2022-11-14 - PicoGopher - the more we are"
date = 2023-11-14
+++

```
      ----------------------------------------
       PicoGopher Part 4: the more we are, 
                          the funnier it is.
       November 14, 2022
      ----------------------------------------
       Written on my laptop, offline, during
       a flight and a train trip
      ----------------------------------------
  
  
  Welcome to Part 4 of the PicoGopher journey! If you want to
  see how everything started, you can find the previous parts
  in my phlog at gopher://gopher.club/1/users/mala/ (also
  mirrored at gopher://gopher.3564020356.org). You can find the
  project's code at https://github.com/aittalam/PicoGopher.
  
  In the previous posts I described the steps I followed to 
  create PicoGopher from scratch. Thanks to the large amount of
  available code and documentation, it was not too hard to
  crank up some simple implementation of the Gopher protocol
  and allow anyone to serve their own contents from a RasPi
  Pico W. While this is an interesting hack, though, I think
  that requiring people to (1) have a Gopher client installed
  and (2) know your server's IP address is, perhaps, a bit of
  a stretch. And for PicoGopher to be actually useful and used,
  I think it should be accessible to a wider audience.
  
  For this reason, today's post is dedicated to extending
  PicoGopher so it can provide contents via (a very simplified
  version of) an HTTP server and automatically redirect users' 
  requests to the AP's IP address. All of these options can be
  turned on or off at will, allowing for different level of
  access - from the most esoteric one (protected WiFi, Gopher
  only, non-default IP address) to the most open (open WiFi,
  HTTP access, automatic redirection to server IP).


  =========  Project status =========
  
  Below you can see the current status of the project. The
  steps marked as "x" have been completed, while those marked
  as "+" are described in this post. You will first read about
  going async (required as soon as we start running more than
  one server listening on different ports), then a few more
  details about the new HTTP and DNS servers.
  
  
    - [x] connect to the WiFi
    - [x] run a simple HTTP server
    - [x] run a mockup Gopher server
    - [x] load/save files
    - [x] make the Gopher server not a mockup anymore:
      - [x] translate gophermaps following gopher protocol
      - [x] load any file from disk
    
    - [x] set up the pico as an access point for geolocalised
          access
  
    - [+] make the server a bit more accessible
      - [+] enable async
      - [+] enable HTTP
      - [+] captive portal with PicoDNS
  
    - [ ] powering PicoGopher
      - [ ] better understand power saving
      - [ ] playing with batteries
  
  
  =========== Going Async ===========
  
  The translation of our server code to async becomes necessary
  when we want to simultaneously run more than one server on the 
  Pico (e.g. both Gopher and HTTP). This allows our servers to
  listen to requests on different ports and answer them without
  blocking each other.
  
  The datasheet "Connecting to the Internet with Raspberry Pi
  Pico W" [1] shows an example of an (HTTP) async server. The
  main difference wrt the non-async version is that socket
  binding, listening, and accepting are all dealt with by the
  `uasyncio` micropython lib [2], and our main() code will boil
  down to starting a new server, defined by a callback method,
  and host ip + port to listen to.
  
  The callback function is passed reader and writer streams by
  default, and we can use them to communicate with clients
  connecting to our servers. This does not change the way our
  Gopher server works in a dramatic way, but for the sake of
  readability I moved all its code to a PicoGopher class and
  created a similar one for HTTP (PicoHTTP). For DNS instead, 
  I just copied/pasted the code from P Doyle's Micropython 
  DNSServer Captive Portal project [4]. Yeah that's quite ugly,
  but I was too eager to see that working! I hope you can
  understand what I mean... And if you don't, let me cite Larry
  Wall: "Laziness, Impatience, Hybris: the three virtues of the
  programmer" :-)
  
  
  =========== HTTP Server =========== 
  
  Our HTTP server is a bit of a Frankenstein's creature, but we
  love it anyway :-) It is because it's a patchwork of code
  coming from the datasheets (for the async part), from the
  Gopher listener (to parse gopherfiles) and, erm, stack
  overflow (if I remember well!) to deal with URL decoding.
  We love it because it is another great example of the 80:20
  rule, and it shows us that you can get a reasonably working
  website running on a Pico, without the need to write one line
  of HTML (well, just because I did it for you... but look at
  the source code and you will convene with me that's a really
  small amount anyway).
  
  How can we get an HTML-free website? Given our gopherhole is
  (at least for now) made only of gopherfiles and plain text
  files, all we have to do is convert a gopherfile into an HTML
  page which links to the other files. The picohttp.py file
  shows how I did it - it is still very simple and limited, but
  I think it can be expanded rather easily to support more 
  advanced features. If you look for `gophermap` in the code,
  you will find the logic behind this: at the moment I do 
  nothing more than copying plain text rows as they are, and 
  translating 2-column links into HTML <a href...> links.
  Text formatting is managed by some improvised CSS that works 
  decently on Chrome&co but screws everything on Lynx, which
  means I have already added it to the list of things that
  need to be fixed (well, yeah, the more the 80% grows in 
  features, the more the 20% of things to fix grows too...)
  
  The other main difference when compared to Gopher is that the
  HTTP server gets multiline requests (with a set of headers
  for each request, that we currently just ignore), prepends
  resource descriptors with a GET (because GET requests are the
  only ones we accept), and encodes URLs by converting many
  characters into their hex equivalent (if you ever saw a URL
  containing a `%20` whenever you had originally typed a space,
  that's URL encoding for you). This last part is taken care of 
  by the `urldecode()` function (yeah that's the one I copied
  from stack overflow). 
  
  
  === DNS server + Captive Portal ===
  
  The concept of captive portal is one I had no clear idea
  about until just a few weeks ago, when I stopped after work
  to have a drink with some colleagues and friends (back when
  it was possible to invite guests at Twitter, but also back
  when I still had a job at Twitter). After the second pint 
  I came out with the question "how can I pop up a gopherhole 
  on someone's screen the same way those login pages appear 
  when you connect to some WiFis?". 
  
  A colleague (thanks Matt!) introduced me to the fabulous
  world of captive portals and once I knew the right terms to
  search (as in +fravia's arrows [3]) I was able to find their
  Micropython implementation. The one in [4] is the simplest
  one I found - and decided to use for a quick and dirty
  experiment with PicoGopher. 
  
  The captive portal in PicoGopher is implemented as (yet)
  another server, a DNS, that basically replies to every domain
  name resolution request with the same IP address, that is the
  one of your Pico (which is 192.168.4.1 by default). When a
  Mac or an iPhone detects this, it will also try to open a
  `hotspot-detect.html` page on the captive portal, so I
  updated the HTTP server to redirect the request to the root
  of our gopherhole.
  
  There are still a few caveats to this captive portal. First
  of all, this is just one of different ways of implementing it
  and I am not sure whether this is the best one yet. Then, I
  realised that when *all* DNS requests from a laptop or a
  phone are redirected to a Pico then the device might have a
  hard time replying to all of them (and even in the best case 
  when nothing breaks, it might still take way more power). 
  Finally, I found it hard to understand how the response 
  payload was built using the code and comments alone. If you 
  are feeling the same, I found [4] a bit more detailed and 
  with better references to DNS documentation and RFC [5,6]. 
  
  
  =========== Conclusions =========== 
  
  The code in the latest commit (4d085f4) implements all the
  changes described in this post. In `main.py` there is now a
  set of variables that you can customize not just to set up
  your AP but also to enable / disable both the HTTP and the 
  DNS server.
  
  Once again I think there's still some work to do here, but I
  hope this is enough to get started with some interesting new
  functionalities.
  
  
  ============ References =========== 

[1] https://datasheets.raspberrypi.com/picow/connecting-to-the-internet-with-pico-w.pdf
[2] https://docs.micropython.org/en/latest/library/uasyncio.html
[3] https://fravia.2113.ch/targets.htm
[4] https://github.com/p-doyle/Micropython-DNSServer-Captive-Portal
[5] https://github.com/jczic/MicroDNSSrv/blob/master/microDNSSrv.py
[6] https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml
[7] https://www.rfc-editor.org/rfc/rfc1035.html

```
