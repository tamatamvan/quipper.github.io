---
layout: post
title: "How much (characters) is that doge in your window?"
date: 2016-08-15 19:00:00
author: kristm
comments: true
---

A funny thing happened one ordinary night as i went home and started to get settled for bed.
a `devsupport` issue from our github has caught my eye. the title went something like "Strange Characters on Qlink".
something about strange things has always fascinated me, and this was not one of the usual places where these things would happen. I HAD to take a look. 
The issue reporter went on to say that an image of a cat had appeared on a teacher's class management page. As it turns out, it wasn't a cat.
but that good ole canine from the doge meme. and it was in some form of ASCII art.

![](assets/article_images/2016-08-15-doge-art-in-your-window/tercero_b.png)

### WHOA? HOW DID IT END UP THERE?

At first I thought someone had found a way to execute a piece of javascript into the page. kids are always finding ways to subvert the system,
and if there's some new tricks these tricksters are playing, my job tells me that I should be on top of it. I turned off javascript in the browser. cat is still there.
If it's not javascript, must be something that's been hacked into the view. could it be some kind of ruby black art? I get a little rush out of the possibility I might be on to something. I open up the view for that particular page. nothing out of the ordinary. I look at the controllers; still nothing. 
it seems to be getting rendered out of our data in the database.

I fired up the console and tried to poke around. I never would have guessed what I would see next.

![](assets/article_images/2016-08-15-doge-art-in-your-window/first_name.gif)

### TERMINAL ANIMATION OR ASCII SORCERY?

What I initially thought of as a new style of animation was apparently the terminal printing out a large data that's been sneaked
into the `User#first_name` field by one *crafty* user. Which is what ends up getting rendered as our cat/doge.

I got a data dump of the user attributes in a [text file] (https://drive.google.com/open?id=0Bx634Fb8elPgSmowS2tLQlROZkk) so I can inspect it closer in ruby.
<img width="788" alt="screen shot 2016-04-21 at 10 10 39 am" src="https://cloud.githubusercontent.com/assets/375711/14695943/694c67b8-07a9-11e6-8858-dfc7d6f44074.png">

turns out that the first_name field got some 79617 worth of characters.
I got a brief introduction to Ruby's unpack and pack methods which seems useful [here] (http://blog.bigbinary.com/2011/07/20/ruby-pack-unpack.html)

<img width="1068" alt="screen shot 2016-04-21 at 9 39 30 am" src="https://cloud.githubusercontent.com/assets/375711/14696039/6325414c-07aa-11e6-87db-733704719431.png">

by retrieving the UTF-8 character, seems bulk of the data is coming from 100 - 900 range (I seem to recall most printable english characters don't go over the 150 mark <sup>[1] (#footnote1)</sup> ). I also noticed that most of the characters above the 900 mark are in the 9600 and 7800 range, so I tried printing that out

<img width="950" alt="screen shot 2016-04-21 at 9 39 59 am" src="https://cloud.githubusercontent.com/assets/375711/14696146/35836ace-07ab-11e6-8781-e42bb1d6c870.png">

Ah! These are the brushes used for the Doge art!

### THE UNCHARTED REGIONS OF THE UTF-8 UNIVERSE

Up until this moment my knowledge of utf-8 has been limited to the latin character set, and maybe some of the more popular asian sets like chinese or japanese. But there's
a big utf-8 world out there. Possibly to represent all known writing systems.

Japanese kana seem to be in the 12000 range
<img width="940" alt="screen shot 2016-04-21 at 9 53 26 am" src="https://cloud.githubusercontent.com/assets/375711/14696261/13cee6f0-07ac-11e6-9aec-efa94b7f442d.png">

Thai

Vietnamese


*TL;DR Put a size limit on your input fields!<sup>[2] (#footnote2)</sup>*

**NOTES**

<a name="footnote1">1</a>: I was actually thinking of ASCII, which has [128 characters] (http://stackoverflow.com/questions/19212306/whats-the-difference-between-ascii-and-unicode)
<a name="footnote2">2</a>: MongoDB won't magically cure your text limit issues
