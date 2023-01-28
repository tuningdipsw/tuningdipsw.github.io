---
title: "Let's make a fighting game #0.8: Rollback (Pre-reading)"
date: 2023-01-27 21:08:30 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #0.5: Preparations](/fgtutorial/2023/01/09/article05-preparations.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D
fighting game using the Python game development library Pygame.

"Rollback netcode". "GGPO". ["Sajam".](https://twitter.com/jiyunaJP/status/1616477744556314625)
These are the words of the hour when it comes to good netplay, and fighting game players today
have raised their standards to the point where the inability to play the game
smoothly online is an instant deal-breaker for many.

If you're a small fighting game developer, you cannot afford to turn away
anyone who might be interested in your game, just because you think delay-based is good enough.

(Frankly, even if you are a large one, you might not be able to afford to turn those people away either.)

![Flag of the GBVS not-rollback announcement](/assets/images/article08/yikes.jpg)

*Battle standard of the GBVS not-rollback announcement*

Fortunately, implementing rollback netcode in your game is more than possible as a solo dev.  
Let's take a look at how.

![Flag of the GBVS rollback announcement](/assets/images/article08/nice.jpg)

*Battle standard of the GBVS rollback announcement*

<!--more-->

## Reading

Oh yeah. You're going to need to do a bunch of reading.
It's not enough for your understanding to be that rollback is "good" and "basically magic",
because you're going to have to implement it now.

Fortunately, by making it this far into this series, you've already mired yourself in a great big swamp of reading,
so this will hardly be the worst of it.

The thing to know about rollback netcode is that **implementing it is quite achievable, as long as you**
**plan your game and development around it from the start.**
Efforts to retrofit it into already-finished games built on delay-based netcode,
like *Mortal Kombat X* and *Guilty Gear Xrd* are far more daunting.
But you won't need to worry about that.

<aside>
Well, okay, <i>I'll</i> need to be able to answer the following questions.
<br>
You might be able to just copy all of my code and scrape on by.
</aside>

You'll need to be able to answer the following questions.

### - What is rollback netcode? How does it work?

To get our basic understanding of rollback netcode, we turn to the current best article on topic:

[![Image of Infilament's netcode article](/assets/images/article08/infil-article.png)](https://words.infil.net/w02-netcode.html)

- [Infilament's article on rollback netcode.](https://words.infil.net/w02-netcode.html)
(It's also where I got the links to many of the following resources.)

There is a [companion video](https://www.youtube.com/watch?v=1RI5scXYhK0) talking through the points of the article,
so if you have trouble picking things up by reading, it might help you to sit through one of those first
so that you can hopefully understand things a bit better on your read-through.

You still have to read the article though.

### - How might rollback affect my game design decisions?

Consult [page 6, *"Seems like a good time to branch into how the designer can make smart design choices..."*](https://words.infil.net/w02-netcode-p6.html)
of Infil's netcode article for some discussion on how visually-discernable rollbacks (rollback artifacts) can be designed around.

However, take note that even older games with very short move startups like GGXXACPR have benefited greatly
from the addition of rollback netcode. Fast games are still more than playable with rollbacking, so if
3-frame abare is part of your vision, you don't have to compromise it because of rollback netcode.

> What if the airdash had 2 or 3 frames of stationary or slower moving startup with the same total duration? Your game plays almost identically but the visual artifacts of rollback are way reduced.

This note will be useful in either case.

### - What do *I* need to do in order to support rollback?

[Zinac: Some of the systems affected by rollback's requirements](https://twitter.com/CMZinac/status/1181878689270599680)

> - Separate gameplay from presentation layer
> - Serializable game state
> - Particle simulation
> - Object lifetime
> - SFX
> - Animation system
> - UI
> - Desync detection

[Mauve: Requirements of the rollback-style network model](https://twitter.com/mauvecow/status/1182085152010203136)

> Game must be re-entrant:
> This means that your game has to be structured so that you can run an update,
> *without* affecting external things like video and sound.
> Separating game logic and game visuals is not always viable, especially in modern 3D engines.
>
> Again, if you haven't built for it from the beginning, separating these two can be difficult at best.
> It is not terribly hard in 2d games to just.. not draw things,
> but 3d games often make heavy use of stateful information and it would need to be be fully separated.

A pair of short Twitter notes written by engineers experienced with implementing rollback,
bringing up some areas of concern when building around rollback.

It's our responsibility to uphold the following:

1. Keep the game fully deterministic. Avoid desyncs due to floating point numbers.
2. Implement functions for saving and loading game state.
3. Ensure update and rendering steps are separated in our game loop.
4. Ensure game runs well enough to run several update steps in the space of a single 1/60th of a second.
5. Handle audio properly during rollbacks to prevent sound effect clipping or duplication.

Particle simulation, object lifetime, and animation systems aren't as problematic for us,
since we're not working with an engine that's inflexible about these things.

I'm not sure how UI applies to our case.

Desync detection might be covered by GGPO, but I am not sure.

[GGPO's developer guide](https://github.com/pond3r/ggpo/blob/master/doc/DeveloperGuide.md)  
GGPO's official guidance on how to call the GGPO library.

<aside>
Python can work with C and C++ libraries using <a href="https://realpython.com/python-bindings-overview/">Python bindings</a>,
a feature I am not familiar with right now but will speedily be looking at when I actually have to import the GGPO library.
</aside>

We will be making use of the open-source GGPO library to reduce some of the rollback systems we'll need to implement.
With this known-working library available to us, it is largely not worth the time/effort of trying to roll our own.

### Hints on implementation

- [GGPO feature in GDMag](https://drive.google.com/file/d/1nRa3cRBQmKj0-SEyrT_1VNOkPOJWNhVI/view)  
This article mostly retraces the explanations given by the Infil article, but it leaves some short tips for handling audio at the end.

<aside>
For those unfamiliar with navigating GitHub's web interface,
you can download the files by either clicking the green "Code" button -> "Download ZIP",
or looking through the "Releases" section linked on the right sidebar (if the repository has any).
<br>
<br>
If you just want to read the code without running it, the GitHub editor may be better than the base webpage.
Access it by pressing "." or ">" on the webpage.
</aside>

<a href="https://github.com/rcmagic/DemoFighterWithNetcode">
<img src="https://raw.githubusercontent.com/rcmagic/DemoFighterWithNetcode/master/screenshot.png" width="270" height="200">
</a>

- [*DemoFighterWithNetcode* on GitHub](https://github.com/rcmagic/DemoFighterWithNetcode)  
A minimal LOVE2D fighting game demo with netcode by Zinac.  
The code is open-source, for your perusal. It's a useful reference for some of the various systems we might
implement in our fighting game, not just the netcode.  
**It has some useful debug options we might want to crib off of.**  
[This video demo illustrating how desyncs cause one-sided rollbacks may be of interest.](https://www.twitch.tv/videos/493574286)

- [(Optional) Networking in *INVERSUS*](http://blog.hypersect.com/rollback-networking-in-inversus/)  
Developer discusses their own take on implementing rollback.  
Although *INVERSUS* is not a traditional 2D fighter, I thought their notes on implementing gamestate backups
and audio segregation were useful.

- [(Optional) NRS' GDC talk on retrofitting rollback into MKX](https://www.youtube.com/watch?v=7jb0FOcImdg)  
A long, technical overview of NRS' implementation, touching a lot on performance.  
Has a lot of details particular to low-level languages (not Python) and heavy game engine stuff (not our sprite-based fighter),
so it is optional, but quite interesting all the same.

### Additional readings for modding an existing game

These readings are mostly unnecessary for our task of implementing rollback into our new original game,
but if you are visiting this page to research how to retrofit rollback into an **older** game,
these readings might be of use. They're not exactly easy to find with web search, so I've taken the liberty of collecting them here.

- [**Game Hacking Resources**](https://github.com/super-continent/game-reversing-resources)

Useful primer and resource dump on getting into reverse engineering games, which you will need to do when retrofitting.  
(Unless your work is officially sanctioned by the developers, you will not have access to the source code.)

- **Mauve's writings on writing RollCaster from ~2010-2012**

  - [(1) Caster, an unofficial netplay client](https://web.archive.org/web/20180610000516/http://mizuumi.net/2010/10/24/caster-an-unofficial-netplay-client/)
  - [(2) Caster - How it's done](https://web.archive.org/web/20180429085318/http://mizuumi.net/2010/11/08/caster-how-its-done/)
  - [(3) Caster - The Making of RollCaster](https://web.archive.org/web/20180217125309/http://mizuumi.net/2010/11/27/caster-the-making-of-rollcaster/)
  - [(4) Investigating RollCaster’s Desyncs](https://web.archive.org/web/20170612204733/http://mauve.mizuumi.net/2012/02/05/investigating-rollcasters-desyncs/)
  - [(5) Investigating RollCaster's timing issues](https://web.archive.org/web/20180221091132/http://mauve.mizuumi.net/2012/02/22/investigating-rollcasters-timing-issues/)
  - [(6) How I didn't clip the sound effects.](https://web.archive.org/web/20180728043303/http://mauve.mizuumi.net/2012/03/08/how-i-didnt-clip-the-sound-effects/)
  - [(7) Understanding Fighting Game Networking](https://web.archive.org/web/20180824004523/http://mauve.mizuumi.net/2012/07/05/understanding-fighting-game-networking)

(6) is another source on how to handle audio during rollbacks.

Although (7) mostly echoes the rollback explanations from articles above, there are a few important passages worth noting.

> You can send multiple frames worth of input data for each frame, so that when one frame gets lost you don’t need to wait for a full resend cycle to complete in order to continue with the game. Input data’s pretty cheap so it doesn’t cost much to include 5-10 frames worth of data in a packet, so might as well include it. This is something that you should pretty much always do.
>
> [...]
>
> If one game is running slower than another, dropping a frame here or there, it **cannot** be treated the same as if it were a packet loss. This is because you lose that bonus extra buffer of delay you added to compensate for it, and get effectively nothing of value out of it. Obviously, this isn’t desirable.
>
> This leads to the most important rule of this sort of network code: The goal is to maintain complete synchronization with the other system. This **includes** performance issues. If one has a drop, this drop must be reflected in the other as well. Always. Anything else will lead to a desynchronization of the intended behavior.
>
> [...]
>
> And if you’re wondering, the rule from above regarding keeping synchronization with the other system’s performance must be upheld. If you detect that the other computer is running slowly and has dropped a frame, then you must also wait a frame to keep synchronization, even in a rollback setup. If you don’t do this the two computers will slowly drift out of timing and nobody wants that.

## At Last (For Real This Time)

Good. You now know have an idea of what you'll need to do to support rollback as we begin to code our engine.  
If you don't, you at least have a list of reading materials you can come back to when you get stuck on that.

With this under our belt, we're ready to start coding.

At last.

For real this time.
  
<br>
<br>

But not in this article.

![Iku throws a monitor out of a window](/assets/images/article08/monitor-away.png)

(TODO: Edit in link to next article)
<a href="/fgtutorial/2023/01/09/article05-preparations.html" align="right"> >> Next: #0.5: Preparations</a>
