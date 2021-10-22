---
layout: post
title: "How You Look At It"
description: "Ostensibly, this is about how to remove a pink cast in a picture."
category: [general]
tags: [mental models]
draft: true
---

{::options parse_block_html="true" /}
## "Oh, there's more pink than I realized" - The Wife

She needed a picture as background for her Pintober series on Instagram.

I said I might be able to do it tomorrow, and then went the kitchen. This was the third
time starting to clean the litter. Bag in hand, I nearly passed the fridge;
Oh, that's just flattening a curve LAB color. Which curve? I had once again not
gotten to the litter.

My brain said, incorrectly, the A channel was Cyan to Magenta (it's actually Red to Green). 
While I remembered the color mapping incorrectly, it still was the A channel that I needed
to flatten. Searching the A channel for the culprit pinks to subdue, acquired & flattened.

This works because LAB represents color as Luminosity (L channel, intensity), A/B channel. 
The "color" as we think of it in RGB is a mix of AB but its brightness is in the L 
it represents color. 

In RGB the color / intensity are tied. To get pink you have some amount of red 
(intensity and color) and then some amount of Green and Blue. To reduce pink, you
reduce red. So if you attempt to remove a color cast in RGB, there's a lot of 
collateral damage.

10 minutes later I had a new 8.5 x 11" printed with the pink cast nearly gone, and I did
manage to avoid turning a red door orange with a few more curve adjustments.

This different representation makes a transformation that would be difficult in the 
RGB color space trivial in the LAB color space. 

How trivial? (windows shortcuts):
* Alt I, M, L (Image:Mode:Lab color)
* Ctrl - M (curves)
* A channel tab
* guessed the pink was in the middle (it was), flattened the curve in the mid range
* noticed a red door was orange, increased slope of curve around that color
* Alt I, M, R

Print.

I learned all of this from [Photoshop LAB Color](https://www.amazon.com/Photoshop-LAB-Color-Adventures-Colorspace/dp/0321356780).
I read this probably 15 years ago, but every so often I use a few techniques to remove
fog in images, or to do what the eye does naturally, separate similar colors to see
distinction. A camera loses some of this, but LAB color makes fixing this quick.

I got to the litter. I had planned to stretch, work on my back a bit, and go to bed.
I wrote this instead.

LAB color makes me think back to a class in Electrical Engineering on signal theory.
What little I remember is that certain formulas can be estimated in calculus but
not really solved, per se. However, the Fourier Transformation comes to the rescue.
(To actual Engineering Students, this was in 1991, and I barely remember the class,
I hope I'm not entirely off my rocker, but it's possible).

Perform a Fourier Transformation, then the problem becomes a solving a sum over a range.
That results in solution, you then reverse Fourier Transform.

In the first case, something difficult in RGB is trivial in LAB. In thge second,
estimating a solution versus solving the equations for FM signal broadcast. 
Transform the representation, to make the impossible possible. 

This works with human interaction as well.

Have you come across a problem where transforming the approach, view, reforestation,
made a huge difference?
