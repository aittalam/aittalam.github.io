+++
title = "Building a Custom Feed on ATProto"
date = 2025-04-13
+++

# Building a Custom Feed on ATProto

![A part of Bilibin's "Vasilisa the beautiful", showing a woman walking in the woods holding a skull-shaped lamp.](/images/atproto_vasilisa.png)

*NOTE: this is the first post of a series, check out Part 2: [Text-to-Feed with BYOTA and Embedding Similarity]({{< ref "2025-07-20-text-to-feed-with-byota.md" >}})*.

Last weekend I was reading Vicki's 
[You can just hack on ATProto](https://vickiboykis.com/2025/01/23/you-can-just-hack-on-atproto/)
and decided I wanted to learn how custom feeds work. One of the main reasons is that a friend 
recently asked me whether [BYOTA](https://github.com/mozilla-ai/byota) would have worked with
ATProto and, while I was fairly confident it would have, I had no idea how to get started :-).
Another reason is that I would like to find an easy way for people to bring their custom timelines
from a BYOTA prototype "to production" (i.e. see them running on their social networks), and 
I want to get exposed to as many alternative protocols as possible and evaluate their pros and
cons.

If you have already built a custom feed with ATProto, this post will probably not teach you
anything new. But since while playing with it I realised not all the information I needed was
available in the same place, I decided to write something down so (1) folks experimenting with
it would have it easier, but most of all (2) I would have a place to look back when I forget
how I did it... :innocent:

## Resources

The first link I'd like to share is [ATProto Browser](https://atproto-browser.vercel.app/) 
by [Tom Sherman](https://tom-sherman.com/). The reason is that, even if you don't now anything
about the protocol, you'll immediately be able to get at least a general understanding of how it 
works just by [pasting your handle in it](https://atproto-browser.vercel.app/at/aittalam.bsky.social)
and browsing through the data you are presented.

The second, I'd say obligatory, is the [Custom Feeds page](https://docs.bsky.app/docs/starter-templates/custom-feeds)
inside Bluesky docs. It gives you a general overview of how custom feeds work and links to a (yay!)
starter kit in (ugh!) Typescript. A piece of information that I originally missed while skimming
through this doc but is instead super important is
[the following one](https://docs.bsky.app/docs/starter-templates/custom-feeds#deploying-your-feed): 

![A paragraph titles "Deploying your feed" whose content says: "Your feed will need to be accessible at the value supplied to the FEEDGEN_HOSTNAME environment variable. The service must be set up to respond to HTTPS queries over port 443."](/images/atproto_feed_deploy.png)

Last but not least, and this is what I have actually used for my custom feed experiment, is the python
[ATProto Feed Generator](https://github.com/MarshalX/bluesky-feed-generator) powered by [The AT Protocol SDK for Python](https://github.com/MarshalX/atproto). The instructions were easy enough for me to see something running
(at least locally) in a few minutes.

## Running the custom feed online

Running your custom feed online requires a few extra steps from trying it locally. As, I guess,
at some point you would like to actually use it in your Bluesky app, here's how I got it working.

1. Make sure you can run your server on port 443 (this often requires you to have root access on
the machine where it runs) and that it supports HTTPS (this means you need to have a TLS certificate,
and the certificate maps to a domain name so you'll need a valid domain name for your server too!).
I managed to fix this by (1) adding an ad-hoc A section to my DNS records and (2) using 
[Let's Encrypt](https://letsencrypt.org/) to get a TLS certificate (with certbot, following the
instructions I found [here](https://certbot.eff.org/instructions?ws=other&os=pip)). If you complete
these steps successfully, you should have the following new files on your server's disk:

```
/etc/letsencrypt/live/your.domain.name/fullchain.pem
/etc/letsencrypt/live/your.domain.name/privkey.pem
```

2. Clone the repo:

```
git clone https://github.com/MarshalX/bluesky-feed-generator.git
```

3. Create a new python env:

```
sudo apt install python3-pip
sudo pip install virtualenv
./setupvenv.sh
```

4. Copy `.env.example` to `.env` and customize it. **NOTE** that while you can easily test
your code locally or referencing your server via its IP address, for it to work with bsky.app
you'll need to provide a valid hostname (see point 1 above). You can ignore the `FEED_URI`
field at the beginning (as it will be returned after you publish your feed with `publish_feed.py`).
For the `PASSWORD` you can get an app password [at this URL](https://bsky.app/settings/app-passwords)
(or following Settings -> Privacy and Security -> App passwords in the Web app).

5. Publish your feed

```
python publish_feed.py
```

6. Run the server (after customizing its code - e.g. right now I just changed the filter string,
but my next step will be getting posts only from the people I follow...)

```
flask run -h 0.0.0.0 -p 443 --cert /etc/letsencrypt/live/your.domain.name/fullchain.pem --key /etc/letsencrypt/live/your.domain.name/privkey.pem
```

At this point, your feed should be reachable at `https://bsky.app/profile/<your_handle>/feed/<your_feed_name>`. 
Note that after you publish the feed you can already look for information about it (even before you start it) 
on the ATProto Browser (under the `app.bsky.feed.generator` PDS collection, see e.g. 
[mine](https://atproto-browser.vercel.app/at/did:plc:l7nvdunbdfruxmgwbfdgfbwm/app.bsky.feed.generator)).
I found this very useful, as comparing my feed generator to existing ones I realized that there was something
wrong with my setup and that's when I learned about HTTPS.

# Conclusions and Next Steps

Well, that's it... I told you there was not much new ;-) but I thought it was useful to point
out the need for TLS certificate and domain name if you want your feed server to actually work
outside of your terminal.

Now, as the claim with my friend was "I think BYOTA would work, in principle, on ATProto too",
I guess there will be follow-ups at some point. Just be aware that while keeping experimenting
with this, I'd like to prioritize ActivityPub for now (if you wonder why, Cory Doctorow explains
it [here](https://pluralistic.net/2024/11/02/ulysses-pact/#tie-yourself-to-a-federated-mast)
way better than I could ever do). If you have any idea regarding this and want to talk, you
know where to find me!