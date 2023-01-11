---
title: "Let's make a fighting game #0: The heart"
date: 2023-01-06 21:08:30 -0000
category: fgtutorial
author: minogame
toc: true
---

Good day. In this series of articles, I'll be attempting to program a simple 2D fighting game using 
the Python game development library Pygame, documenting the process as I do so, working through the 
problem, the logic, and the code. There might be some arbitrary flavour tangents included as well, 
as I am not being paid to omit them.

<!--more-->

...And here's one coming up fast...!

*(Note: This is a long one. Feel free to skip to #0.5 or a later segment using the Table of Contents above.)*

## Why are you here?
![](/assets/images/article0/yes-you.png)

<aside>
Keep it down with that "<i>Smash isn't a real fighting game</i>" talk you know what I'm talking about
</aside>

The fighting game genre boasts a storied legacy that few others can match, with its *Street Fighter*s and 
its *Tekken*s, its *King of Fighters* and its *Smash Bros*.
And it doesn't take long for onlookers to understand what a fighting game is about;
the thrill of combat between two foes standing face-to-face,
the back-and-forth drama of two rivals going blow-for-blow can pull them in without explanation. 
It's famous, simple, and it's been around for quite a long time.
Between those three factors, I think it is a contender for the most recognizable genre there is.

And yet, despite the purchase the fighting game has in our cultural consciousness, the fighting 
game genre has a reputation for being one of the hardest to get into, and in recent years,
it seems like one that only a handful of big companies dare attempt.

Yes, recent fighting games have had their successes, as of late.   
*Guilty Gear Strive* - [1 million copies sold.](https://www.eventhubs.com/news/2022/aug/10/guilty-gear-1-million/)  
*Tekken 7* - [10 million copies.](https://www.gematsu.com/2022/12/tekken-7-sales-top-10-million-tekken-series-sales-top-54-million)  
*Smash Ultimate* - [28 million copies.](https://www.eventhubs.com/news/2022/may/11/ssbu-sold-28-million-units/)

However, this is but a sample of the biggest games, made by the most skilled companies...
and only of sales, not of playbase.

I don't want to bring out the Steam Charts(tm), but
you don't have to look far to see how players flock and have flocked to other genres, like the FPS,
[the Facebook game](https://en.wikipedia.org/wiki/The_Sims_Social#Reception), 
[the MOBA](https://en.wikipedia.org/wiki/Multiplayer_online_battle_arena), 
[the gacha game](https://en.wikipedia.org/wiki/Gacha_game), 
[the battle royale](https://en.wikipedia.org/wiki/PUBG:_Battlegrounds), 
[the auto-battler](https://en.wikipedia.org/wiki/Dota_Auto_Chess#Reception), 
[the Vampire Survivors...-like](https://www.youtube.com/watch?v=poGpNrQn6vc), 
whatever breakout hit you see creating its own genre of derivatives and imitators...  

and you might wonder to yourself, why make a fighting game?

Who is going to play my game?

Sometimes I wonder how fighting games are doing. Perhaps you wonder sometimes as well.

The golden age raised a generation of fighting game players we still tell stories about today.
The arcade generation, the four Japanese gods, those *Umehara Fighting Gamers...*  
The revival era sparked by *SF4*, too, spawned its own renaissance of competition, where it seemed like
*SF4* was the world.  
But those days are behind us, of course.

Now, today, *Street Fighter 6*, *Tekken 8*, and *Project L* loom on the horizon with great promises and 
expectations for each -- New mechanics! New characters! Smoother netcode! New *old* characters! Better 
features, a better package! Photorealistic images of Ryu's sweat glands! A fresh new intake of players!
-- with the hopes that finally, with this title, the golden age might return again.   

...Perhaps.

As multiplayer games, I think fighting games have it rough.

We core fighting gamers 
go into them looking only for their multiplayer, ignoring or downplaying the package, the single-player elements,
chasing the memory of that fresh new competitive scene we've been longing for, just like the one
that made us fall in love with fighting games... Ha... ha ha hahahahahahahaha...!!!

Soon we're crying for balance, crying for content, new characters, old characters, calling for the heads of 
the developers... Where's Mileena, Ed Boon?? Daisuke's vision, am I right?! When's rollback, Kamone?!?!
Oh thank you so very much, Mori...!!!!!

But the modern fighting game had to evolve, has to patch, has to bow to the crashing waves, 
because they live and they die by the love of their playerbase......

The fickle dissatisfaction that can drive a thousand bitchmad Twitter users to call a game dead
lives in the same heart as the love that lets fifty immovable Discord users continue to boot up and play
sets with one another, peer to peer, long past the point of end-of-service,
longer than any developer could realistically dream that their game could be played.

Atop the pile of a thousand defunct live service games (some middling fighters among them), lies that
enviable summit which the most immortal of fighting games have achieved -  
those that have inspired people to play them into eternity.

Look at [*Eternal Fighter Zero*'s Latin American community](https://www.youtube.com/watch?v=Vh28VMbjkPk),
look at [the arcade goer who plays *Street Fighter II Turbo* every day, for thirty years](https://www.youtube.com/watch?v=2yT30H8vK2Q) --   
and I can't help but think that the core fighting game player has something in them in common
with a speedrunner.

The pursuit of perfection that drives a player to endlessly hone themselves, the dream of someday becoming
unbeatable through one's depth of experience and intimate mastery; that, at least to me, is the romance
of the fighting game genre -- and it abrases with the idea of an eternal revolving door of new fighting games.

Every new fighting game that comes out today must compete, in at least some way, with the decades of
legacy that comes before it.

<aside>
The <i>Will it Kill?</i> clip is pretty good though. Actually, there were at least two at the time of writing,
I misremembered. 
<a href="https://youtu.be/Twq279Rg8pk?t=2707">(1)</a>,
<a href="https://youtu.be/iXd-HmrcbZc?t=2065">(2)</a>
</aside>

Your game, if you choose to go through with making it, must do its utmost to make the world care about
it for even a moment. It's fighting against *Melty Blood Actress Again Current Code*, against
*Street Fighter 6*, against <del>Yomi</del>...**Y**our **O**nly **M**ove **I**s Hustle; it's fighting against
*Phantom Breaker Omnia*, although this is a game that to my knowledge has little more legacy than its
exceptionally camp [release date trailer](https://www.youtube.com/watch?v=MgQAPDAq3Go), one
*Vortex Gallery* tournament, and a *Will it Kill?* clip, so this is not an exceptionally high bar to clear.

<aside>
...until perhaps one day, they let go of even that, and retire from competition, retire from 
fighting games... 
<a href="https://twitter.com/honzogonzo/status/1422224258717999105">to the great retirement home in the sky</a>
<br>
<br>
...I don't think there's any shame in that. However tragic it may be on a distant level, if it's something that 
you've grown out of, if moving on ultimately brings your current self closer to happiness, I cannot 
help but celebrate it with you.
<br>
<br>
Also your hands might explode if you play Naoto for too long
</aside>

But even supposing that the core fighting game player gives it an honest try, and perhaps even enjoys
the [discovery honeymoon](https://www.youtube.com/watch?v=Y-b6SbGQSoI) --
In the end, those oldheads, who once were young, who've already found the fighting game *they* love, 
will surely seep back to the embrace of their old lover...

I don't wish for you to take these musings as doomsaying, that I believe fighting games are dying.  
I don't believe that fighting games will die.  
There's so much enthusiasm and passion for them today, in this silver age of rollback netcode and 
(comparatively) positive discourses.  
I think there'll always be a faction of *[goinmul](https://youtu.be/C3q5nSqGXr4?t=92)* players who'll keep 
playing their games for as long as they can. I expect great things from the upcoming generation 
of AAA fighting games.  
But, I'm not convinced that they can sweep the world again like contemporary or future genres have, or will.  
Even though they're such beautiful games.

When I look around the fighting game space, there's a lot of things that make me uneasy about its future.

<br>

<aside>
<a href="https://www.youtube.com/watch?v=2ovCHgAPXSc">"This is the hardest time to be a fighting game player"</a>,
<a href="https://twitter.com/2dJazz/status/1357086600316948480">(2)</a>,
<a href="https://twitter.com/2dJazz/status/1571867774573768706">(3)</a>
</aside>

Netplay can be alienating, yet offline play may be uncertain.

<br>

<aside>
<a href="https://twitter.com/ysemulti/status/1273184356186669056">(1)</a>,
<a href="https://twitter.com/fubarduck/status/1579758025442996224">(2)</a>,
<a href="https://www.youtube.com/playlist?list=PLExsOrWTHnDYK74QrMSsMBRVTuVy6I9zm">(3)</a>
</aside>

Arcades are already on their way out,

<br>

<aside>
<a href="https://www.youtube.com/watch?v=k_fm2hCLjVs">"The Smash World Tour Situation is a Big Deal Even for the FGC"</a>,
<a href="https://www.youtube.com/watch?v=dcmhYEZ_SUY">"The Bigger Culprit in the Smash World Tour Shutdown Hasn't Got Enough Smoke"</a>
</aside>

the professional scene survives in no small part on the good graces of corporate eatsports interests,

<br>
<br>

<aside>
<a href="https://twitter.com/nycfurby/status/1587288086177845250">(1)</a>, 
<a href="https://twitter.com/KITVandy/status/1551642913263812608">(2)</a>
</aside>

and the tremendous effort required to run the events that carry our scene cannot be taken for granted.

<br>

Yes. I feel a little uneasy about the future of fighting games.

<br>
<br>

But you don't care about that.
<br>

You mustn't care about that.
<br>

If you're here reading this, then you must already want to make a fighting game. You may not be aiming 
to make the next *Street Fighter*, or a title so successful it earns you ten morbillion dollars, or a game 
so excellently balanced that will-be oldheads point to it in internet arguments about how amazing/crappy 
the actual next *Street Fighter* is -- but surely, on some level, you must have a love for the game. 
You must have a love for watching two characters speak to one another, through the rule of beasts, and 
that precious spark within you, wishing to make that happen yourself.

I cannot promise that making a fighting game will make you rich, or even that it's a sustainable way 
to support yourself.  
It is unreasonable to expect that your self-made fighting game will be able to compete with the 
titans of the genre.  

Nor can I even say that making a fighting game is a good idea.  
I write this text only because I am less sane than anyone whose judgement might be better.

But if you have a character you want to see expressed through the language of a game --  
[a character from a series you love,](https://www.gematsu.com/2014/11/blazblue-dengeki-bunko-fc-devs-want-make-idolmaster-love-live-fighting-games) 
[an original character of your own that you can call a child,](https://dic.pixiv.net/a/%E3%81%86%E3%81%A1%E3%81%AE%E5%AD%90)  
if you have a love for that character,   
if you have love,   
then make this game, and pray that that feeling can transmit to someone else out there, out in the world.

If there's a part of you that screams when you witness an unbelievable combo on *Will it Kill?*, a part 
of you that could love a kusoge, that can feel the electricity running between the hands on your controller 
and the character on your screen, that wishes to see a game achieve the miracle of existence, 
against all odds...

If you have a love for the game, like I do,  
then let's make a game.  

Let's make a beautiful game.

<br>

<br>

<br>

## AWESOME, THOSE WORDS YOU JUST WROTE WERE TRULY GRIPPING. NOW ONTO THE TUTORIAL

Firstly, I must say this.  
My programming and fighting game player credentials: I must shamefully disclose that 
I don't have any. Fraudulent? Disappointing? Yes. Perhaps.

<aside>
* See <a href="https://youtu.be/egcPOGDIk6Y?t=277">"Sajam Discusses Fighting Game Content: We Need More of It"</a>.
<br>
<br>
** These tend to emerge naturally as a pasttime of debate between those experts. Tis the intellectual
pleasure of showing off one's expertise to the world, married with the urge to correct some other moron
(smooches. Smooches. We're all friends. All in good fun) who hath made an error, or even just an omission
of technical detail, that wills these threads into existence.
<br>
<br>
<i>"As the strongman exults in his physical
ability[...], so glories the analyst in that moral activity which disentangles. He derives pleasure from
even the most trivial occupations bringing his talents into play." </i>
 - Edgar Allen Poe, <i>The Murders in the Rue Morgue</i>. 
<br>
<br>
"Infodumping is pretty fun." I only know this 
quotation because it was used in <i>Forbidden Scrollery</i> chapter 40. This is my infodump flex.
<br>
<br>
...I'll shelve this particular aside now.
</aside>

But I think many subjects suffer from a lack of mid-level educational content.\*
While there's tons of tutorials for newcomers, and maybe even a handful of high-level, highly-specific
technical discussions by subject matter experts\*\*, 
a journeyperson who's broken out of the beginner-level "tutorial zone" may find themselves frustrated 
by a lack of obvious, discoverable resources suitable for their level.

I've found this to be true of both programming and fighting games, at least for myself. That's why 
I'm starting this series. Though I have no accomplishments nor expertise to my name, the act of putting 
this resource together might still prove useful to someone, if only myself; and if it fails, stalls, or 
falls short, then, well, probably nobody will see this, so it shall be no sweat off my brow anyway.
Even then, I might learn something in doing so.

<br>

Let's begin by answering some preliminary questions.

*Sorry, no, this isn't the tutorial either.*

## YEAH, THAT MAKES SENSE

### Why a 2D fighter?

This is because *Guilty Gear XX Accent Core +R* is my current favourite fighting game, and also because 
2D fighters are the ones I have the most knowledge of. In comparison, I know much less about 3D and 
platform fighters; at best, I might be able to write the occasional addendum or errata where one of my 
2D-centric comments does not hold in 3D or platform fighters. I also have a terrible gap in knowledge 
about tag fighters. But we'll get there when we get there.

I will do my best to write a few comments on the differences between the various subcategories of 
fighting games, but in the end it will fall to you to fill in the gaps. I hope that the rigors of 2D 
that I provide will be enough to tee you up for whatever task you choose for yourself.


### Why PyGame?

It is not necessary to learn how to program at the low level of any programming language in order to 
create a fighting game; many excellent and free game engines, libraries, and frameworks are available 
out there with a wealth of tutorialiterature to learn from, although I cannot guarantee that they will 
have up-to-date tutorials specific to fighting games, and it feels a little disingenuous to ask you to 
learn **All of (eg.) Unity** through overly-general tutorials just to learn how to make your fighting game.

- *Unity* has a reputation for being the big, classic indie game engine, and the 
[Universal Fighting Engine](http://www.ufe3d.com/doku.php) toolkit, used in titles like *Fight of Animals*, 
is an adept tool to create a fighting game. However, it comes with the caveat that UFE is not free. 
In addition, I am obliged to report that *Unity* (the company) has taken a questionable direction in recent years 
[(1)](https://www.fool.com/investing/2022/07/24/most-troubling-thing-about-unitys-ironsource-deal/) 
[(2)](https://www.vice.com/en/article/y3d4jy/unity-workers-question-company-ethics-as-it-expands-from-video-games-to-war), 
so you may wish to look for alternatives going into the future.

![](/assets/images/article0/amongus-arena.png)

Fight of Animals *and* Fight of Machines *were made with Unity + UFE;*
Among Us Arena Ultimate *was made in Unity directly.*

- *Godot Engine* is one such alternative, offering similar power to Unity while sticking to communal 
open-source roots. It lends itself well to 3D assets in ways that other tools might not. Working in this
engine might be a good experience for making other kinds of games, too.

![](/assets/images/article0/yomi-hustle.png)

*Although not a traditional fighting game title, <strike>Yomi</strike>*
**Y**our **O**nly **M**ove **I**s Hustle *was made in Godot.*

<aside>
If you'd like to weigh the quality of the MUGEN tool yourself, a spreadsheet of MUGEN games maintained by 
Tumblr user <a href="https://mugenfinder.tumblr.com/">mugenfinder</a> can be found 
<a href="https://docs.google.com/spreadsheets/d/1FMHlikel6MjlbojchEdS82VsE-ScQVhpkz9cVU_UL8M/edit#gid=0">here</a>.

<br>
<br>

* As told to me by mugenfinder, you can't sell games made with MUGEN for profit, but you can with IKEMEN GO.
</aside>

- *MUGEN*, the classic fighting game character kitchen sink, also offers immediate structure for those who 
just want to slap their characters into a working thing immediately. I'm told it has a vibrant community 
of *MUGEN* enthusiasts and SaltyBettors backing it up, but I have not investigated the quality of the 
program(s), the ease of use, or the tutorial support for making changes. Certainly I myself feel a stigma 
that "proper" fighting games don't come from *MUGEN*, but I don't know enough about it to say that for 
certain.

![](/assets/images/article0/battle-craze.png)

[Sarvets All-Stars](https://www.youtube.com/watch?v=SpAg5rcEVh4), *one of the auction tournament games of 
Combo Breaker 2018, was made using MUGEN, and* [Battle Craze](https://www.playbattlecraze.com/), *as seen 
on [Will it Kill Episode 17](https://www.youtube.com/watch?v=iXd-HmrcbZc), uses SUEHIRO's I.K.E.M.E.N GO engine.*

Then, why use a programming library like Pygame? I'm not going to lie - it's no small deal of grief to 
work with code directly. "True" originality is overrated, and though it may be satisfying to your ego, 
as a developer and a programmer, you ought to avoid reinventing the wheel when possible.

However, even so, there is something to be said about the satisfaction that comes from building something 
from the ground up, the degree of control and understanding afforded to the programmer who starts from scratch.
I think there's a lot that we can learn in doing so, even if it may be faster and/or easier to start 
with a more approachable offering.

<aside>
* This is a subjective opinion. But I believe it fully.

<br>
<br>
** If you don't know what Delphi is, don't worry:
 neither do I. This probably says more about my lack of dev knowledge than it does about its popularity 
as a programming language. But the point is that against all odds, video games get made, and against 
even further odds, people don't even do it with the tools you'd expect them to. Occasionally you hear 
about a cartel of extremely gifted wizards simulating a fully-featured copy of <i>Dwarf Fortress</i>
in <i>Minecraft</i>, or a miniature black hole. Or something. This world is full of miracles. 

</aside>

As for the choice of Python and Pygame -- let it be said that if there is a will, there is a way to make 
a game with any tool given to you. For example, the cult classic shmup *Hellsinker.*, one of the greatest 
solo indie games of all time\*,
[was written in Delphi](https://web.archive.org/web/20180819171301/http://mauve.mizuumi.net/2012/06/01/retrospective-hellsinker-translation-development#more-441)\*\*.
[Just finishing a game, any game at all](https://www.derekyu.com/makegames/deathloops.html), 
setting aside the merits and difficulties of whichever particular medium you choose, is an achievement 
that deserves celebration.

But the last thing I will ask you to do is write a video game in (eg.) C, because
1. that is an unkind thing to ask someone to do,
2. I also would have to write a video game in (eg.) C, and I don't want to.

You can use any tool you like, but you probably shouldn't choose something that only adds pain.

There might be some things you truly can't do anything about, but unrelated, 
unfair pain points can and will wear down your willingness to work on your project, and 
**you absolutely must guard your motivation with your life.**
If you run out of motivation, there is a very real risk that your project will remain unfinished forever. 

Don't agonize over the decision forever, but if you take a bit of time to pick your tools with some thought,
you can save a lot of pain.
And when you do come across pain points, take a bit of time to stop and identify them, and
perhaps invest a bit of time into reducing the pain to the best of your ability.

Now, I consider Python to be a good starter language for a host of reasons:
- You don't have to write a small suite of esoteric-looking boilerplate code to print "Hello World".
- You don't have to complete a small ritual to compile and build it.
- Lists and dictionaries are within arms' reach.
- You don't have to learn how to manage memory immediately. You don't need to know what a pointer is.
- It has a host of excellent learning resources, like 
[Automate the Boring Stuff](https://automatetheboringstuff.com/#toc).
- If you don't finish your project, you can jump ship and use your newfound Python knowledge to 
write programs for machine learning or statistical analysis. Probably.
- It's the language I started with, thus making it the best.

<aside>
It would have been <i>so sick</i> if I recommended LOVE2D right after talking about love
for ten paragraphs, but I'm not quite comfortable enough with Lua to use it myself.
</aside>

There are surely a feast of wonderful languages that fulfill these criteria -- 
like Lua, for example, which offers the excellent [LOVE2D library](https://love2d.org/) 
used in the top-notch shmup [BLUE REVOLVER](http://bluerevolvergame.com/) (among others). 
I've chosen Python and Pygame for its comfort factor, and in truth, this is the most important factor. 

<aside>
However, sometimes being stubborn really only just results in unnecessary pain that brings you closer 
to the point of giving up or losing motivation. Thus there are also times where you must recognize 
that you should bend the knee and do things the easy way, perhaps just temporarily, perhaps for the 
rest of the way. You must learn these limits for yourself.
</aside>

Overcoming the learning curve of other, perhaps clunkier languages can definitely be done with a 
little bit of stick-with-it stubbornness; if this is what you choose, you should still be able to 
get something out of the problem statement and logic portions of each update, with just a little bit 
of extra elbow grease to adapt the Pygame code to your language. But you can do it. 
I have a baseless faith in your ability. Nevertheless, I encourage you.

<br>
<br>
<br>

*Please feel free to write in with YOUR factual opinion about what the BEST first programming language 
for new devs is. Or make a YouTube video with a loud thumbnail on it and reap those 15 cents of 
AdSense revenue. Either/or.*

\* *A technical article addressing PyGame and its reputation for being a slow library can be found
[here](https://blubberquark.tumblr.com/post/630054903238262784/why-pygame-is-slow).*

### Can I steal this code?

You are free to use the code in this tutorial however you like. (That's the MIT License included in this
site's repository.) An acknowledgement of credit would be a 
nice gesture, but I can't really stop you if you don't add one, and in the end it's not a big deal. 
But if these articles were useful to you, then I would love to hear about it. Write in! ...Somewhere! 
I'll have to figure out an inbox or a comments section for this at some point.

If you steal and claim credit for writing this tutorial I'll be pretty miffed at you. 
I'm not going to send you death threats or hire a dark web hitman or whatever, but like, come on. Let me have this.

### Why did(n't) you do it like this?

If something isn't clear to you, leave a comment and I'll do my best to answer.

If you know a better way, don't keep it to yourself. I am not an expert programmer.
And there's lots of things I don't know! If the suggestion is good, I'm happy to update the relevant posts 
with better code or better information. 

<a href="/fgtutorial/2023/01/09/article05-preparations.html" align="right"> >> Next: #0.5: Preparations</a>
