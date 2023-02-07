---
title: "Let's make a fighting game #1.1: Game loop tangents"
date: 2023-01-31 12:00:00 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #1: Introducing our game loop](/fgtutorial/2023/01/30/article1-gameloop.html)

This is a bit of a side article to article #1 (Introducing our game loop), filled with bits
I though were ultimately unnecessary to the main tutorial and ended up cutting
during my editing pass.

Here are some fun bits of writing and hopefully informative I couldn't quite bring myself to cut completely.
If you're stuck in a tutorial-reading haze, I would really recommend just skipping to #2
instead of reading through this, but if you'd just like to read a little more,
I've left this here for you.

<!--more-->

## Game loop readings

The best literature on what the game loop pattern looks like, and how to do it well,
comes from two main articles:

1. Robert Nystrom's [*Game Programming Patterns*.](https://gameprogrammingpatterns.com/game-loop.html)
2. Glenn Fiedler's [*Gaffer On Games: Fix Your Timestep!*](https://gafferongames.com/post/fix_your_timestep/)

(2) might be a little tricky to understand if you don't have existing code
in front of you, but I think reading through (1) first will help you grasp the big ideas.  

Neither of these pages are particularly long, so I encourage you to give them a read, but
if you just want to crunch through the tutorials here, I'm hardly going to force you to do a reading assignment.

## Some definitions

### ("FPS" and "Frames" are overloaded terms)

<aside>
<a href="https://wiki.gbl.gg/w/Dong_Dong_Never_Die/FAQ#.22The_frame_data_seems_off.2C_usually_I_can_react_to_a_25f_startup_move.22">
Dong Dong Never Die's 100 FPS</a> is a unique exception to this in fighting games.
The 60FPS standard has been ingrained in fighting games for a long time, so I don't recommend switching.
<br>
<br>
Another fun example of a non-60FPS game is
<a href="https://shmups.wiki/library/Cho_Ren_Sha_68K">ChoRenSha68k</a>, a shmup programmed on 55FPS hardware;
when played on Windows PC instead of its native X68000, the player must handle ChoRenSha's challenges in
ever so slightly less time than they were designed to be handled in. Still doable, but just a little bit
more saucy.
<br>
<br>
<a href="https://www.youtube.com/watch?v=_w8SBUWuzek&list=PL6PHQCxAqpJTA3R5hgkqVJChfIuecx2gh">
Has an excellent FM chiptune soundtrack as well.</a>
Reading the <a href="https://shmuplations.com/chorensha68k/">developer's interview</a> about this game really impressed me
with what they were able to accomplish at the time.
</aside>

60FPS is a common performance minimum for most games.
We don't expect to run into a major performance bottleneck with our relatively simple 2D graphics,
but if our game were to utilize systems like complex 3D models and lighting engines, we might have to start worrying about this.

PC gaming enthusiasts have the privilege
of enjoying games at high frame rates (often 120FPS, possibly higher);
although even most modern fighting games these days cap their frame rate at 60FPS,
we could theoretically allow our game to run above 60FPS visually, for the benefit of
some theoretical buttery-smooth graphical effects.

(We're going to be making a sprite-based 2D fighter, where we have to supply each frame of the animation by hand,
so we don't really have any buttery-smooth graphical effects to speak of. But it's an idea.)

(If hand-drawing or hand-supplying the sprites for each frame of animation sounds like a lot of work -- it is!  
That's one of the main reasons why you might choose to work with 3D models instead.  
I'm not so familiar with how to create, animate, and render 3D objects,
and I admit I have a fondness for hand-drawn animation, so I'll be going with the 2D sprite-based route.)

(...We'll have to skimp out on going all out drawing these animations in this tutorial,
just so we can finish our code before 3023.)

If we did so, we'd need to ensure that the actual logical frames of our fighting game,
the frames that dictate how long our characters' actions take, remain at the fighting game standard of 60FPS,
in order to maintain the convention that players understand and expect.

It might be nice to decouple the timing of frame-rendering from our input processing and our game state updating.

- If our game performance suddenly tanks to 15FPS because our antivirus just launched a scheduled scan
and our computer suddenly doesn't have the resources to lovingly render the sebum glands of (eg.) Mario's nose,
it kind of suck to see our gameplay (eg. the timing we press buttons to in our combo) slow down to quarter speed.  
- If we want to implement something like [Tekken 7's cinematic slowdown](https://youtu.be/F6Fx0T_3IWY?t=704),
we want the speed at which the characters take action to slow down without affecting the visual frame rate.  
We probably wouldn't want to change the speed at which input parsing is handled during these situations.
- The time at which the game moves might even stop completely.
There are situations like [superflashes](https://glossary.infil.net/?t=Super%20Flash)
and most games' round starts where the game state is paused, but inputs can continue to be entered in order to buffer moves.

This separation between the game-*performance* frames that our graphics live in, the game-*logic* frames that
the game state lives in, and the *real-time* 1/60ths of a second our inputs live in is worth mulling over in your head.

To restate this, the following three definitions of "frames"/"FPS" are different:

- **The number of "frames" (pictures drawn to screen) shown per second, the *visual performance frames*.**

Normally 60FPS, but it'd be nice if it could be uncapped higher for visual smoothness,
or if performance drops during rendering didn't affect the gameplay.

- **The number of "frames" (smallest unit of in-game time) per second, within which the game state can change.**

Constant 60FPS, by convention.  
However, the rate at which game state advances may slow down or be frozen during certain game mechanics.  
It might also be forced to slow down in online situations where the connection between two players is inconsistent.  
(If your game stops receiving the opponent's inputs, the game state shouldn't advance.)

- **The rate at which player inputs are processed by the game.**

You could make this arbitrarily fast instead of 60FPS,
but 60FPS is more than enough fidelity to parse human input,
and making this different from the rate of game state change might introduce some easy-to-mess-up logic to the input parser.

---

Yes, these three definitions line up nicely with the `processInput()`, `update()`, and `render()`
functions of our game loop, just in reverse order.

As it stands, our current naive implementation of the timing clock at the base of the Game Loop ties these three together.
That doesn't seem ideal.
Although it's probably not the end of the world if they're not all completely decoupled...

## ...Can we make it better?

```c
# https://gameprogrammingpatterns.com/game-loop.html#play-catch-up
double previous = getCurrentTime();
double lag = 0.0;
while (true)
{
  double current = getCurrentTime();
  double elapsed = current - previous;
  previous = current;
  lag += elapsed;

  processInput();

  // Isn't this a little bit familiar?
  while (lag >= MS_PER_UPDATE)
  {
    update();
    lag -= MS_PER_UPDATE;
  }

  render();
}
```

Let's look over this C code from article (1) for a game loop with fixed update time step and
variable rendering for a moment.

Our game state is behind 1 or more fixed time steps (frames) where it needs to be,
so we advance it one MS_PER_UPDATE (frame) at a time until we're up to date.
Finally, now that we're in the frame we need to be in, we can render it.

Hm. That sounds familiar.

It sounds a little like the catch-up we do when we rollback our game state.

Like we mentioned in article #1, we're not going to worry about integrating this special
fixed-timestep update loop into our code quite yet. I know I have a tendency to get lost in
overthinking too much before I even start to code, so for now, we'll start with
the naive game loop in article #2.

But it's a nice idea to keep in mind.

## Other questions

### ...VSync?

My understanding of how VSync plays into our FPS theory isn't too good, but I should point out that
the community recommended PC settings for minimal input delay in
[*BBCF*](https://twitter.com/Super_Myoro/status/1468005468349943809) and
[*Xrd*](https://twitter.com/Hursh191/status/1574203001681588224) both have VSync off,
so it shouldn't be a major deal-breaker if we don't support it.
We might add support for it or play around with some forced screen-tearing examples in a bonus chapter later on.

### Could we use multi-threading to decouple processInput(), update(), and render() instead?

...Maybe?
That sounds plausible...

I'm not confident enough in my programming ability to try it, though.  
I think it'll be easier to get this right if we stick to a single-threaded approach for now.

## Go to the next article

<a href="/fgtutorial/2023/02/06/article2-inputs.html" align="right"> >> Next: #2: Inputs</a>
