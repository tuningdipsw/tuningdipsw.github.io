---
title: "Let's make a fighting game #1: Introducing our game loop"
date: 2023-01-30 12:00:00 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #0.8: Rollback (Pre-reading)](/fgtutorial/2023/01/27/article0-8-rollback-prep.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D
fighting game using the Python game development library Pygame.

Teaching the game loop design pattern is a common part of any game programming tutorial,
so there's already plenty of coverage on this topic on the internet,
but we'll go over it again, since it is quite important, and it sets up our next few
topics quite nicely.

I won't be regurgitating the lessons of authors who've done it better, but I hope to
add some useful commentary on how the game loop looks in the context of our fighting game.

<!--more-->

## Distracting aside

You know, I've come to realize that these writings are probably going to read more like a devblog than a tutorial.

If I knew how to do the things I'm setting out to do already, then I would have the opportunity to
put together some quite clean and curt no-frills writing, as befits a tutorial. Getting to the point
is quite important, and it respects time of the reader who just wants to paste it and go. But as
it stands, I don't quite know what will work or what will be good until I've tried to write it,
and perhaps not even then. So it is a little dishonest for me to try to write as if the information
I'm about to tell you is quite authoritatively excellent, or the code that I'm about to reproduce
is absolutely up-to-standard, best-practices stuff really, when I'm actually wracking my brain really
hard to try and reason out what I need to do before every new feature.

Sometimes I don't even do that
and I just type some of the doggest lines I've ever written to see if it works (which is a pretty good
approach to get yourself started and writing code, so let us not knock it). Really, trying to write a
tutorial without knowing what I'm about to do isn't very easy on my end either.

<aside>
Incidentally, the Fall 2022 anime <a href="https://en.wikipedia.org/wiki/Do_It_Yourself!!"><i>Do It Yourself!!</i></a>
was an excellent watch that more than lived up to my iyashikei expectations for the season.
Quite light and quite fun. A worthwhile watch for those looking to fill their own iyashikei needs.
</aside>

Anyways, I just wanted to get that out there before I place some of the smartest-looking dumb words ever
to be put together before you. I think I'm going to end up writing a lot of segments in post to correct
or improve the things I'm about to put forth. But that's the fun of Doing It Yourself, I guess. Let's learn a thing or two, together.

And I apologize to you, my readers who must sit through my long-winded foolishness in order to glean
even a spark of potentially useful insight.  
And I thank you for reading through it anyways.  
I would love to assemble a cleaner version of this text when I've put this whole thing together.

And now, we return you to our feature programming.

## Game loop reading

The basic game loop that you'll find in any game library tutorial, like
[this one](https://www.patternsgameprog.com/discover-python-and-patterns-8-game-loop-pattern)
or [that one](https://coderslegacy.com/python/display-fps-pygame/)
looks something like this:

```python
while True:
    processInput()
    update()
    render()
    clock.tick(60)

# Ok, this is a bit of a simplification, but pseudocode is fine for now.
```

## Adding rollback to the mix

Neatly enough, our basic game loop already has updating state and rendering state cleanly separated.
That core requirement of rollback isn't hard for us to fulfill in Pygame.

We actually have seen what our game loop might look like with rollback in our earlier rollback reading:

<aside>
Interesting note: The presenter mentions that skipping saving the game state in all but the last simulated frame
allowed them to eke out some performance gains in the worst-case scenario.
The tradeoff is that they had to rollback more frames in the average rollback scenario.
<br>
<br>
See <a href="https://youtu.be/7jb0FOcImdg?t=1879">31:19 of the NRS GDC talk</a>.
I don't expect performance to be one of our main worries, so we'll just save it in each of ours.
</aside>

![Game loop diagram breakdown of what happens in a rollback frame, from NRS' GDC talk](/assets/images/article1/nrs-gameloop-tick-diagram.png)

*Game loop breakdown of what happens in a rollback frame, from [NRS' GDC talk](https://youtu.be/7jb0FOcImdg?t=1412)*.

The rough code for a gameloop accounting for rollback probably looks something like this:

```python
while(running):
    # --- processInputs() starts here ---

    processLocalInputs()
    sendLocalInputs()

    # Rollback prediction: if no remote inputs are received this frame,
    # rollback fills in a repeat of last known input
    receiveRemoteInputs()

    inputs = (local_inputs, remote_inputs)

    # If remote inputs were received, check that they match the prediction.
    # Then return the frame number of the last correct frame.
    # (If the prediction has been correct, this is the current frame.)
    frame_to_rollback_to = checkRemoteInputs()

    # --- processInputs() ends here ---
    # --- update() starts here ---

    if frame_to_rollback_to != current_frame:
        loadFrameState(frame_to_rollback_to)
    
    # Simulated frames
    while frame_to_rollback_to != current_frame:
        advanceGameState(inputs, frame_to_rollback_to)
        saveFrameState(frame_to_rollback_to)
        frame_to_rollback_to = frame_to_rollback_to + 1

    # Real frame
    advanceGameState(inputs, current_frame)
    saveFrameState(current_frame)
    current_frame = current_frame + 1

    # --- update() ends here ---
    
    render(game_state)
    clock.tick(FRAME_RATE_CAP) 
```

Actually, consulting the
[GGPO Developer Guide](https://github.com/pond3r/ggpo/blob/master/doc/DeveloperGuide.md#synchronizing-local-and-remote-inputs),
a lot of the parts like `receiveRemoteInputs()` and when to call `saveFrameState()` and `loadFrameState()` are handled by GGPO.
So our loop will look a little different when we integrate that in.

But this is the rough look of the logic underneath, if you need to implement the things GGPO does yourself.

## Could we do better?

If you're familiar with some of the literature on game loops, you might notice that
this version of the game loop is a little imperfect, as `update()` is still
somewhat bound to `render()`. `update()` does have the ability to handle multiple frames of updates per loop,
but it's doing it more to handle rollbacks than to handle a slow `render()` function.

<aside>
And as anyone who has played against me on my goopy laptop, struggling to hit 60FPS in Strive or GBVS, can attest.
</aside>

But I'm going to argue that it's not the end of the world for now if we bind these together:
it's not uncommon for fighting games to slow down to the speed of the performance of the slower computer
when playing online, as we alluded to at the end of the rollback article (Mauve's articles).

> If one game is running slower than another, dropping a frame here or there, it **cannot** be treated the same as if it were a packet loss. This is because you lose that bonus extra buffer of delay you added to compensate for it, and get effectively nothing of value out of it. Obviously, this isn’t desirable.
>
> This leads to the most important rule of this sort of network code: The goal is to maintain complete synchronization with the other system. This **includes** performance issues. If one has a drop, this drop must be reflected in the other as well. Always. Anything else will lead to a desynchronization of the intended behavior.

Note that this refers to drops in the rate that the game state is updating
(and transmitting the inputs that influence how game state updates).
If the drops are only in the rendering rate and not the inputs/updates, they are of course not affecting the game state,
so you wouldn't have to force the remote computer to reflect that drop.

[Infil's article, part 5](https://words.infil.net/w02-netcode-p5.html)

> Solving the problem is relatively straightforward. Most rollback systems will briefly pause the player who is ahead for 1 or 2 frames so the player who is behind can catch up. If the game keeps on top of this system and never lets the games drift very far apart, the correction will be quick and painless. Note that these pauses are not due to missing input or other networking difficulties; they are designed to fix the fact that different computers will run software at slightly different rates in unpredictable ways, and losing 1 frame every 10 seconds to maintain the sync is unnoticeable to even the most astute players.

It's not ideal that how fast our game moves can be slowed down by how fast it can render,
but it is passable.

We're going to start off with this naive game loop for now, just so we can get building.

The main thing we're worried about is `render()` being too slow to hit 60FPS and dragging down our game speed,
but it's not a good idea to spend too much time worrying about performance before it actually appears,
at least in a very lightweight, 2D case such as ours.

So we will put the high-level decoupled game loop ideas on hold until it becomes necessary.
I originally wrote a spot more about this, but since we're going with this naive loop for now,
it turned into unnecessary clutter.

I've confined that section to #1.1, an appendix article.
If you'd like to read more about game loops, you can read it there, or consult the
more expert articles I have cited there.

<a href="/fgtutorial/2023/01/31/article1-1-gameloop-tangents.html" align="right"> >> Optional: #1.1: Game loop tangents</a>

## See you next time

Next time on <b>FIGHTING GAME P</b>, we'll start by implementing the input processing part of our game loop.
I'm partway through writing it right now and it's certainly getting a little messy, so it promises to be a meaty one.
I might even have to break it up into multiple parts. So look forward to that.

TODO: edit in link to #2.

<a href="/fgtutorial/2023/01/30/article1-gameloop.html" align="right"> >> Next: #2: Inputs</a>

---
