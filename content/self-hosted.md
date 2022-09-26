---
title: "Going Self-Hosted"
date: 2022-09-26T23:28:14+01:00
draft: false
---

So I've decided to go self-hosted. It's quite easy and cheap to start hosting your own stuff.
I started doing this by taking [this Vultr promo](https://www.vultr.com/promo/try150/), where I would get 150$ free to use (without time limit - I can use it for 30 months).

For you the reaeder, if you don't want to invest money, there are other options.
- [Google Cloud](https://cloud.google.com) - Offers you a VM totally for free (but without static IP).
- [Oracle's Cloud](https://www.oracle.com/cloud/free/) - Offers you a quite powerful VM (for some reason they didn't accept my card).
- [Replit](https://www.replit.com) - Place where you can host unlimited(?) "nix" containers. Is where I host my Whoogle instance. They just force your repo to be open source (although you could still host there binaries which source would be hidden lol).

Another option could be hosting on a old laptop/mobile phone.


# How has been the experience

I was more hyped than I should have been when I started to self-hosted. However, I still think this was a good investment.
I learnt a bit more about ssh (more in terms on how to make your server secure) and won experience on deploying stuff.

I believe everyone that has interest in linux/backends/deployment should do it once, since it is accessible.

## What I'm hosting

I wanted to have more ideas on what I could host. Currently I'm hosting:
- DNS server (AdGuard Home)
- Git Server - `ssh vps.miguelneedssleep.ml -p 23231`
- [Whoogle](https://google.miguelneedssleep.ml)


# How do I get started in self-hosting

I would recommend you to start on replit. You can host there stuff for absolutely free.
On Whoogle's GitHub page you even have a one click deploy to create your instance.

Once you find yourself limited to what if offers you, rent a server and start hosting this basic stuff that I also host, cause you most likely would use it too.
