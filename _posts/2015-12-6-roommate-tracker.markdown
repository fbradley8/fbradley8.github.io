---
layout: post
title:  "Raspberry Pi Roommate Tracker"
date:   2015-11-16
categories: projects featured
source: https://github.com/fbradley8/roommate-tracker
---
Let's say you have a roommmate; one who is typically in the middle of a nap when you get home from work. You have a bright and bubbly personality so, naturally, when you get home you begin your nighttime routine by singing to the cat while loudly cooking dinner. You won't even know that your roommate wants to kill you until he comes out of the bedroom wearing an eyes-half-open scowl.

After a few months of living like this I decided on a new purpose for my 7" USB touchscreen I purchased from mimomonitors.com. I wanted a quick way of knowing who was home as soon as I walked in the front door.

For this project I used:

-Raspberry Pi 1 Model B
-Mimo 720F 7" USB Touchscreen Monitor
-Edimax EW-7811Un Wi-Fi USB Adapter

If you're going to replicate this project, please don't use the same USB monitor I used. They're overpriced and linux support is terrible. Try out the [official Raspi touchscreen](http://www.element14.com/community/docs/DOC-78156?ICID=rpiaccsy-access-products) and let me know what you think.

#####The Project

Checkout the [source code]({{ page.source }}) from my GitHub. It's a node.js app running on localhost:3000. Start by running grunt. The script is simple and doesn't require a database. The server pings the predefined ip's in server.js and if it gets a response, the ip is sent to listening clients via websocket.

#####Issues

So here's the deal. Android devices will keep wifi on and respond to pings while the screen is off, but iPhones won't. The iPhone will switch back to cellular to save power. So if you've got an iPhone on the network, you'll need to disable this power saving feature.
