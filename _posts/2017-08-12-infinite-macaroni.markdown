---
layout: post
title: "Infinite Macaroni"
date: 2017-08-12 00:00:00 -0400
categories: cs
---

Browsing [/r/perfectloops](https://www.reddit.com/r/perfectloops) I came across
this fun animation:

<iframe src='https://gfycat.com/ifr/NecessaryWideAlpaca' frameborder='0'
scrolling='no' allowfullscreen width='640' height='480'></iframe>

Not only is it fun to watch, but you quickly start to wonder what it looks like
on the inside after a few loops. It's constantly being distended, so whatever 
the walls of this infinite macaroni are made of keep getting thinner.  Does 
the cross section change? If so, what's the pattern?

Let's assume that when the animation starts, the macaroni ring we see is
completely solid. When the animation has completed its first loop (heh), we're
left with another ring, but this time it's hollow. The center of the ring has
been drawn out into a tube and closed onto itself, leaving an open space.
Kind-of like a pool noodle whose ends are made to touch.

With this image in our heads, let's start the second loop of the animation.
This one might be the trickiest to visualize, so bear with me. Keeping in mind
the idea of a pool noodle with touching ends, pause the animation a few frames
after it's begun. What we have now is sort of like one of those water-tube-toys
you could buy at the aquarium:

<img src="https://images-na.ssl-images-amazon.com/images/I/31kNjmC-WdL.jpg"/>

The only difference being that instead of being filled with a green fluid, our
macaroni is filled with empty space. And as the animation completes, the ends
of the water-tube-toy like shape merge, leaving the inside of the tube-toy
disconnected from the outside! And now we're left with two concentric hollow
macaronis! A cross section would look like an `O` inside of an `O`. 

So what happens with the next loop? Luckily, we've already worked out what
happens when this animation runs for a hollow macaroni! It produces another
hollow macaroni inside of itself... So each time this animation runs, we double
the number of concentric `O`s.

Depending on how these concenctric macaronis distribute themselves, as the
number of iterations tends towards infinity we'll approach a completely solid
macaroni again! What a loop.
