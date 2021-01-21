---
title: Creating music with code
description: 
date: 2020-07-27
tags:
  - music
  - sound design
layout: layouts/post.njk
---

I've been learning about music theory for the past 3 months with the help of a musician. I've always wanted to experiment with music creation but my past attempts have failed because of lack of structure and not looking for guidance from someone who knows the territory.

One of my favorite related practice activities has been using Sonic Pi to experiment creating sounds with code and mixing it with my understanding of music as I learn.

## On music tools

I'm impressed with the quality of the existing music tools. I'm using Ableton Live but most "DAW's" have polished interfaces, complex plugin systems and lots of features. It must be interesting to know more about the internals of those tools.

Most UI's are meant to be used with the mouse and skeuomorphism is everywhere which makes a lot of sense as knobs and buttons have a direct relation to hardware devices.

## The creative process

I've had a lot of fun specially because there are not fixed workflows. Everyone has their own set of tools and ways to use them and that feels refreshing. 

## On using code

I guess this approach is not that attractive for musicians or people that don't have experience with programming but I see a lot of opportunities to use code to create sounds, composition and experimentation. As I continue to learn the established tools (Live in my case), I can use code to do something quickly.

For example this is how to create a sound that resembles a sea wave:

```ruby
in_thread do
  live_loop :sea do
    synth :pnoise, attack:3, sustain:2, release:3, amp:0.8
    sleep 10
  end
end
```

If you want to experiment, paste that code in Sonic Pi and press play. 

Isn't that very cool?

My favorite part so far has been experimenting with composition, creating different layers to add more complexity to the track and the trial and error process. I'm aware that as I keep learning more of the fundamentals, the quality will keep improving.


<iframe width="560" height="315" src="https://www.youtube.com/embed/USdH8HPDzqA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



