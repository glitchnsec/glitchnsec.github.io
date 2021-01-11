---
layout: post
date:   2019-07-07 12:56:00 -0500
categories: research
title: SpiderMonkey Research - The beginning
---

For a while now, I have been really interested in getting into browser exploitation but the thought alone has been daunting. I checked out<a href="https://www.amazon.ca/Browser-Hackers-Handbook-Wade-Alcorn/dp/1118662091" target="_blank"> Browser Hacker's Handbook</a> briefly but that scared me too. I then decided to contribute to Mozilla to get my hands dirty. Although I have not contributed as much as I would love to, I have been fortunate to e-meet Matthew Gaudet. Matthew is a really friendly, smart and helpful dude. I also stumbled across <a href="https://www.mgaudet.ca/technical" target="_blank">his blog</a> while scouting for resources on SpiderMonkey<br />
<br />
Few days ago while checking on <a href="https://liveoverflow.com/" target="_blank">LiverOverflow</a>, I saw he has just began his browser exploitation journey! This was amazing, his first video spoke to me on a personal level. As a young researcher, it is easy to see what other known researches are up to and feel like you'll never get there because time waits for no one and the industry is rapidly changing. Seeing LiveOverflow explain his own challenges and take on it re-inspired me. I thought to myself; I can start with him, with discipline, by the time he has become advanced, I would too! Maybe not as good but I will be somewhere better than where I am now.<br />
<br />
Fortunately, I had Mozilla's Firefox repo since I was contributing to that. I decided to use SpiderMonkey as the research Engine on Windows. This is pretty different from LiveOverflow's setup but that's the fun part!<br />
<br />
<b><u>Approach</u></b><br />
<br />
My goal is to mirror his research closely. Discover the (near) equivalence or differences between both engines, pick a similar bug to his, walk-through it and share my part of the story like he is doing. I'm excited and scared at the same time but this is gonna be fun. This blog would help keep me accountable and responsible.<br />
<br />
Every post on this topic would contain an "Into the weeds" section where I hope to elaborately describe my challenges and mistakes. It would typically be at the end of the post and the intention is make the posts less magical and more realistic.<br />
<br />
<b><u>Honourable Mentions </u></b><br />
<br />
<a href="https://twitter.com/stankoja" target="_blank">Stanko Jankovic</a> a Bugcrowd researcher is also doing similar work with V8 and Windbg. <a href="https://medium.com/@stankoja/v8-bug-hunting-part-1-setting-up-the-debug-environment-7ef34dc6f2de" target="_blank">Check it out!</a><br />
<br />
<b><u>Conclusion</u></b><br />
<br />
LiveOverflow, I'm just one of the many people you inspire and teach indirectly. Once again, thank you for what you are doing in and for the community. Shout out to other researchers bringing knowledge that once rested among gods to the average man,&nbsp; encouraging and inspiring us to work hard and consistently. I look forward to contributing more than I have received/been given.<br />
<b><u> </u></b><br />
Anyone, finally, if you see anything wrong, weird, etc. Feel free to correct me. Without further ado, let's get to it!<br />
<br />