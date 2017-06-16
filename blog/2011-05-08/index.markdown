---
layout: page
status: publish
published: true
title: 'Exchanging fans on a HP DL 585 G6'
date: '2011-05-08 17:48:12 +0200'
date_gmt: '2011-05-08 17:48:12 +0200'
categories: [rnd]
order: 70
tags: []
---

This is a more hobby post, but I happened to acquire a HP DL 585 G6 server lately. It has pretty nice features for the price,
but the noise it makes with the default Nidec Beta V VA450DC fans is like a vacuum cleaner.

In order to be able to utilize the server in a conventional household, I decided to see if the fans could be switched to more silent ones. As I couldn't find 120mm x 38mm
fans that have reasonable noise levels (even if they have a high CFM value), I decided to use conventional 120mm x 25mm fans.

Knowing that Nanoxia Deep Silence 120mm has a pretty good noise/CFM -ratio, I decided to try these in the DL 585 G6. Someone else might want to try Noctuas or what not, it doesn't
really change how the fans are put there.

# Some Difficulties

The problem with the DL 585 G6 hot-swappable fan cages is that they're modeled for 120mm x 38mm fans. There are few things in the cage that make it impossible to fit a 120mm x 20mm fan to it. It's a bit stupid design maybe, because
it could have been made to accomodate fans of both sizes.

In order to fit a 120mm x 25mm fan in the cage, I figured that I had to remove all the plastic parts. And leave only the metal cage as well as the wiring.
Practically this is not too bad, because eventhough one cannot utilize the fans in as hot-swappable fashion as the original ones, one can still fit new fans in place of the default ones in a practical, usable way.

Additionally, after changing the main fans, I noticed that the PSUs also have fans. And they're noisy as well. There are two 60mm x 38mm fans from Nidec and they're rated > 8000 rpm. They do move a lot of air (they have max. CFM value of 72 I believe). Again, these need to be changed in order to imagine sitting in the same room with the server. Many manufacturers e.g. Nanoxia and Noctua do offer low-noise 60mm fans. Although their CFM rating is noticeably lower than of the original Nidecs, I don't really believe the PSUs require that high CFM values, eventhough they are also serving the role of exhaust fans (which although, would be considered a design failure in a conventional ATX case, because passing hot air through the PSUs requires the PSU cooling to work even harder). But since space is limited in this server, then this is probably why they decided to design it this way.

If one doesn't use all of the PCI(-e) ports, then it might be possible to fit at least one 120mm in place of the PCI(-e) slots, which could serve as an exhaust fan. I didn't see any additional fan power nearby, but of course one could always use a splitter cable.
