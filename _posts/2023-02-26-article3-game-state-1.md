---
title: "Let's make a fighting game #3: Game state (1)"
date: 2023-02-26 14:00:00 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #2: Inputs](/fgtutorial/2023/02/06/article2-inputs.html)

[<< Prev: #2.1: Unit testing examples](/fgtutorial/2023/02/12/article2-1-unit-testing-examples.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D
fighting game using the Python game development library Pygame.

We've created a system to translate key presses into logical buttons,
but they don't *do* anything yet; we have no concept of an action,
nor even a concept of a character with which to perform one.

Let's start getting that set up.

<!--more-->

## Eventual goals

In addition to the holdover goals of "parsing inputs into actions"
and "accept buffered inputs" from article #2, we can add a few more goals
around our game state.

The goals for our input system were easier to scope down by comparison, but,
well, the game state sort of encompasses everything a fighting game is --
the rules and numbers for everything that you CAN do, so I am at risk of just writing

"Make a fighting game."

here and dusting my hands off, walking away, leaving everyone baffled.

But, instead, I will do my best to spell out how each bit of game state we define will turn
our game into the fighting game we're all familiar with.

### **Implement saveable, loadable state, as required by rollback.**

The task of serializing things, making saves of our state, and setting our state to that of a saved state
isn't actually a very complicated or difficult problem for us to solve in Pygame,
but I'll write this down anyways.

### **Define characters and their attacks in terms of hitboxes and hurtboxes.**

Near all that a fighting game is, is defined by hitboxes, hurtboxes, and frame data.

> For a recap on hitboxes and hurtboxes, consult Dustloop's [hitboxes article](https://www.dustloop.com/w/Hitboxes).

If your attack hitbox collides with the opponent's character hurtbox,
you apply the hit, and you reduce the opponent's HP.  

Doesn't sound so complicated, right?  
You could say this simple principle of collision detection holds of many other games, such as Pong, or Space Invaders.

But the fine nuance of interaction in a fighting game lies in the third dimension of time.  
In order to codify the frames (unit of time) of the frame data into our game,
we need a particular object called a StateTimeline.

<aside>
Disclosure: I didn't have a full understanding of
how to handle state, animations, and time until I read through Zinac's
<a href="https://github.com/rcmagic/DemoFighterWithNetcode/blob/master/game/StateTimelines.lua">
DemoFighter,
</a>
from which I am taking the term "state timeline" from.
I draw upon his open-source work with respect and gratitude.
</aside>

## (What is a StateTimeline?)

[In short, it's what this code represents.](https://github.com/rcmagic/DemoFighterWithNetcode/blob/master/game/StateTimelines.lua)  
If you were able to get a sense for what that object was used for, feel free to skim through the following section.  
But, for completeness, allow me to explain in excruciating detail.

A fighting game allows its characters to perform a great variety of actions to choose from,
a tremendous offering of all sorts of attacks and movement.
In comparison, a game like a shmup offers the player only elementary actions.

But the difference lies in *commitment* -
the limitations on what actions a character can take while they're performing another one.

For example, a shmup requires the player maintain fine control of their ship at all times -
though the player is only offered two axes of movement, a shot button, and perhaps a bomb button,
they're given great control over the granularity of how much or how little they want to move, fire, or bomb.  
Not only that, all these systems can be operated simultaneously.  
In general, high-commitment actions in STGs are quite rare, and the relative loss of fine control feels extremely wrong.

Meanwhile, almost *every* action in a fighting game has some degree of commitment attached to it.

![+R Potemkin 5H from Dustloop](/assets/images/article3/pot-5h-frame-data.png)
*Example data of a fighting game attack, from [Dustloop.](https://www.dustloop.com/w/GGACR/Potemkin#5H)*

Every attack can be divided into startup, active, and recovery frames,
and for the most part, you only get to "cancel" your attack into something else after it's hit something.

So the startup frames are already a commitment - only when your attack comes into contact with your opponent
(active frames) are you allowed to take any possible cancel options (for some defined "cancel window").  
You are not free to perform *any* action afterward, either -
you only have a choice from a limited list of valid cancel options,
else you'll have to wait out the rest of your recovery frames like normal.

![Strive gatling chart](/assets/images/article3/strive%20gatling%20table%20-%20NBSilentShadow%20-%20status%201404937791889416193.jpg)

*Chart of Strive's valid "gatling" cancels for each attack.*
*You don't need to memorize this for this article.*
*[(Source)](https://twitter.com/NBSilentShadow/status/1404937791889416193)*

<aside>
These are general rules - there are edge cases where a whiffed move's recovery
or a move's startup can be canceled (see
<a href="https://glossary.infil.net/?t=Purple%20Roman%20Cancel">Strive PRC</a> and
<a href="https://glossary.infil.net/?t=Kara%20Cancel">kara cancelling</a>).
</aside>

And, if the attack never comes into contact with the opponent, or "whiffs",
you'll be unable to take any action at all until the end of the recovery frames.

And, of course, there are some times when your character can't perform any actions at all,
like when you're in hitstun, blockstun, or have been knocked down.

So in comparison to the shmup, where you can just do pretty much anything at any time,
the limitations on which actions can be taken at which times lends itself well to
modelling our fighting game with states.

That's the "state" part of "state timeline" -
so, as you might expect, the "timeline" captures the dimension of time.

For each state we have in mind, be it a dash, an idle stance, or an attack,
each with their own hitbox and hurtbox dimensions,
animation sprites,
and cancel options,
we also attach the timelines, or the timings, for each property.

- When do the hitboxes first appear?  
- Do they change in shape on a later frame?  
- Do the hurtboxes change over the course of the state?  
- Every state has its own animation, of course -
which sprite images appear at which times of the state, and for how long?  
- Perhaps the attack causes the character to move around as well, like a teleport or a transport attack -
when does the movement begin? How much does it accelerate the character by?

The StateTimeline is an object that captures both the properties and the timings of a state.
Without it, our fighting game character can't even perform an attack.

With those properties explained, DemoFighter's simple
[StateTimeline](https://github.com/rcmagic/DemoFighterWithNetcode/blob/master/game/StateTimelines.lua)
should hopefully make a lot more sense to you.  
Note that DemoFighter only implements two states as timelines: an attack state and an idle state.
Cancel options aren't included in their model -- we'll have to work that into our own implementation of the object.

## More eventual goals

### **Store and handle character data with a scalable workflow.**

Almost all games are really just composed of a lot of numbers,
and despite them looking like they're just about two characters punching each other,
fighting games are no exception.

Those timelines we showed off earlier are almost all composed of numbers,
so we can expect to have to manage quite a few of them.
Here's an example of just a [single character's frame data.](https://www.dustloop.com/w/GGACR/Potemkin/Frame_Data)
As you can see, there are quite a few numbers there.

So you'll have to come up with a way to manage all those numbers in a way
that's easy to read, easy to update, and easy to generate.  
Writing those numbers into your code directly will not do you any favours.  
Hunting for a particular value will be difficult,
and the file probably won't be set up so that you can easily compare one value to another by eye.

Spreadsheet software handles numbers wonderfully.  
Masahiro Sakurai (of Super Smash Bros. directing fame) talks a bit about this
in his video about [setting up parameters](https://www.youtube.com/watch?v=nGaajB8m5Q0).

[![Image link to Sakurai's video](/assets/images/article3/sakurai-on-parameters.png)](https://www.youtube.com/watch?v=nGaajB8m5Q0)

If we want to wrangle large amounts of data, we need a good tooling workflow.

Our plan is to:
1. Store our character data and frame data in a spreadsheet that we can easily manipulate.
2. Export and convert that into a format that our Python game can read (StateTimelines).
3. Have our game use those StateTimelines to give our attacks the desired properties.

### **Find or write a useful tool for generating hitbox and hurtbox information.**

Having a spreadsheet will be useful, but there's another wrinkle in our workflow we'll have to smooth out.

If this game is just a proof of concept for our tutorial, it's okay, and feasible, for us to
write exactly one attack and lay down a few numbers to define the
dimensions of each hitbox and hurtbox associated with it.

However, as soon as we start wanting our game to have *plural numbers of attacks*,
we will soon come to realize that attempting to define our character timelines by hand,
through manual data entry, is extremely painful.  
Even though Zinac's DemoFighter really only has an idle state and one attack,
you can still see how he had to write a whole bunch of strange magic numbers into
[his data file](https://github.com/rcmagic/DemoFighterWithNetcode/blob/master/game/StateTimelines.lua)
to define character hit/hurtboxes.

```python
# StateTimelines.lua
# ...
    damageBoxes = 
    {
        {start = 0, last = 26, x = -40, y = 80, r = 40, b = 0}
    },

    attackBoxes = 
    {
        {start = 8, last = 11, x = 32, y = 80, r = 120, b = -20}
    },
```

If you go over to [Spriter's Resource](https://www.spriters-resource.com/pc_computer/blazbluecentralfiction/sheet/108864/)
and download (eg.) Azrael from *BlazBlue Centralfiction*'s sprites, you can "borrow" all the sprite images you want -

But you'll notice that the dimensions of each sprite aren't consistent:

![](/assets/images/article3/az202_04.png)

![](/assets/images/article3/az202_05.png)

*Two consecutive frames of Azrael's 5C attack.*

So, where should the anchor point of our game state x- and y-coordinates be, relative to this sprite?  
And where should the hit-and-hurtboxes of this attack be, relative to our x,y-coords?  
-- Measuring which pixel should go where to define all those numbers is not practical.

It's not intuitive for humans to convert between raw positional numbers and images in their heads.
**I will not be calculating and entering 5,000 numbers by hand**, and I hope you will not be either.

I want to use a tool where I just click or drag boxes on my sprite image, and it prints out the Rect data for what I just did.
I do that a couple of times for the however many frames of my attack animation, and I have the h-boxes of my timeline.

Can we find an existing tool?  
Hmm. The MUGEN enthusiasts have to work with sprites all the time,
I imagine they've got to have a tool they use to define their hitboxes.

A search for MUGEN hitboxes led me to the
[MUGEN Fighters Guild forums](https://mugenguild.com/forum/topics/the-importance-hitboxeshurtboxes-and-why-they-matter-193823.0.html),
which mentioned the term CLSN. (Collision?)
That term led me to [this tutorial](https://www.youtube.com/watch?v=EDDIKSgXayw) for defining CLSN boxes
in a tool called [Fighter Factory 3](http://fighterfactory.virtualltek.com/).

So we may wish to give this tool a try.
I will experiment with this when we start to move from a proof of concept into a scaled-up fighting game.
If it's something we can incorporate into our workflow, awesome.  
If not, then we'll probably just have to make one ourselves.  
But we may still be able to draw some inspiration from this tool in terms of usability,
so we should still give it a try.

## Simple game state, drawing, and movement

With our larger goals in mind, let's start making changes to our code.

<aside>
* Q: Is this the post-edited version of the code? <br>
A: ...
</aside>

Ok, prescient disclaimer from the future here - we have a lot of stuff to put together ahead of us,
and it's not always going to be obvious, or even apparent, what the best way to write it is, until we write it.  
So to prevent ourselves from going into a catatonic state of indecision from not knowing how to do it best,
we're going to hack together some atrocious first-pass code,
giving ourselves something to work with that vaguely does what we want,
and then we're going to delete 50% of it in post.*

So don't scream if you see some really ugly functions in the next 20 minutes,
just tell yourself the author's going to fix that in the future, like I'm doing right now,
and the fear and revulsion should subside.

Let's make a GameState class to store our game state properties.
Off the top of my head, we'll put the round timer and the two characters' states in there.  
Let's also move the `current_frame` variable we had lying around in `main.py` into there as well.

```python
# gamestate.py
from __future__ import annotations
import pygame as pg

import constants
import inputs
from inputs import Button

class GameState():
    def __init__(self, inputHistories: dict[str, inputs.InputHistory]) -> None:
        self.current_frame: int = 0
        self.inputHistories = inputHistories
        
        self.characters: dict[str, Character]= {}
        self.characters["P1"] = Character(inputHistories["P1"], self, "P1")
        self.characters["P2"] = Character(inputHistories["P2"], self, "P2")
        
    def update(self) -> None:
        for character in self.characters.values():
            character.update(self.current_frame)
            
        self.current_frame = self.current_frame + 1
        
    def render(self, display: pg.surface.Surface) -> None:
        # render characters
        for character in self.characters.values():
            character.render(display)
```

And let's add the Character class underneath it:

```python
# gamestate.py
# ...
class Character():
    def __init__(self, inputHistory: inputs.InputHistory, \
                 gameState: GameState, player: str) -> None:
        # Pass in references to other systems Character needs to know about
        self.inputHistory = inputHistory
        self.gameState = gameState
        
        if player == "P1":
            self.xpos: int = 50
        else: # P2
            self.xpos: int = constants.WINDOW_WIDTH - 50
        self.ypos: int = constants.WINDOW_HEIGHT - 50
        
        # TODO: probably want a special load_character helper
        # for loading char's sprites en masse
        self.surface: pg.Surface = pg.image.load('assets/guy2.png').convert_alpha()
        
        # TODO: arbitrary placeholder values,
        # would like to load this in from a character data file
        self.maxHp: int = 200
        self.hp: int = self.maxHp
        self.walkspeed: int = 10
        
        self.facingLeft = False
        
    def update(self, frame_number: int) -> None:
        '''
        Takes a frame_number, and updates self based on the Buttons pressed on that frame.
        '''
        # NOTE: Added a helper function in inputs.py to get the buttons dict for frame X
        frame_buttons = self.inputHistory.getFrameButtons(frame_number)
        if frame_buttons[Button.LEFT]:
            self.xpos = self.xpos - self.walkspeed
        elif frame_buttons[Button.RIGHT]:
            self.xpos = self.xpos + self.walkspeed
        
        # Limits on xpos
        self.xpos = min(constants.WINDOW_WIDTH, self.xpos)
        self.xpos = max(self.xpos, 0)
    
    def render(self, display: pg.surface.Surface) -> None:
        pass
```

We've added a very simple `update()` function that uses our `inputHistory` to move our `Character`s left and right.

You might notice that there's not actually very much game state that exists outside of the `Character`s
(ie. exists only in `GameState`) apart from the round timer.

We could choose to put `Character`-created objects (ie. projectiles like fireballs)
in `GameState`, but it might make more sense to attach them to the `Character` class instead,
to make their ownership clearer.

<aside>
Although stage aesthetic can sometimes indirectly affect gameplay by making some attacks
harder to see, eg.
<a href="https://streetfighter.fandom.com/wiki/Kanzuki_Beach#Trivia">Kanzuki Beach</a>'s
waves obscuring certain character projectiles.
</aside>

> The stage itself might have some relevant state -
> although it's more common in traditional fighters that the stage itself has no direct impact on gameplay,
> some games like Smash Bros or Injustice feature [stage interactables](https://youtu.be/ZTeFLQlS6EI?t=195).  
> That state should go in a separate Stage class within `GameState`, to mark it as one of our rollbackable objects.
>
> If our stage is purely aesthetic, though, it's not part of the game state that needs to be saved as part of rollback,
> so we could choose to put that off to some outside class that only touches our main.py render() loop.

We do need to render *something* on our window just so we have some feedback
to see how `update()` is working.  
This calls for art.

![Guy sprite 1](/assets/images/article3/guy.png)

Perfect.

...

![Guy sprite 2](/assets/images/article3/guy2.png)

Perfect.

```python
# gamestate.py
# ...
class Character():
    # ...
    def render(self, display: pg.surface.Surface) -> None:
        rect = pg.Rect(self.surface.get_rect())
        rect.midbottom = (self.xpos, self.ypos)
        
        surface_facing = self.surface
        if self.facingLeft:
            surface_facing = pg.transform.flip(surface_facing, True, False)

        display.blit(surface_facing, rect)
```

With some simple rendering code, we get something like this:

![The first moving guys](/assets/images/article3/render-guys-1.gif)

> NB: `rect.midbottom = (self.xpos, self.ypos)`
> is a neat trick to align the middle of the sprite image's bottom edge to (xpos,ypos).  
> For now, this is okay, but as we discussed above, the anchor point of most of our sprites won't be so consistent.  
> We'll need to modify this logic in the future.

<aside>
Disclosure: A slight adjustment to our input system was made to support 2 players properly between articles.
It was simple enough that I feel confident leaving it as an exercise to the reader.
Alternatively, check `inputs.py` in the code repository.
</aside>

### Knowledge couplings

Let's take a moment to look at some of the couplings we're planning with our classes.
We don't necessarily need to torment ourselves over this prematurely,
but it also doesn't hurt to take a step back from time to time
and see if we'll write, or even, we've written, ourselves into a maintenance swamp.

**InputHistory does not need to know anything about GameState.**
Great! That's what we set out to do at first, and that hasn't changed.

**Character (an object stored within GameState) needs to know about its player's InputHistory, but not the other player's.**
That's what we expected.

**Character needs to know about the opponent's Character.**
This is necessary in order to determine:
- whether to "turn around"/flip its sprites
- whether relative direction "back" is left or right
- whether opponent is low enough HP to (eg.) perform an [Astral Finish](https://glossary.infil.net/?t=Astral%20Heat)

**Character may need to know some GameState information.**
For most applications, you probably won't need this, but there *are* a few edge cases where you could use GameState:
- for stage interactables
- for the [rare move influenced by round timer](https://www.dustloop.com/w/GGACR/Zappa#Hello,_Three_Centipedes)

Neat-o. The couplings seem manageable, perhaps even expected, so we'll put up with them for the time being.  
`GameState` has access to the pair of inputHistories when it's created in `main.py`,
and we can pass inputHistory and gameState into `Character.__init__()`.

```python
# gamestate.py
class GameState():
    def __init__(self, inputHistories: dict[str, inputs.InputHistory]) -> None:
        self.inputHistories = inputHistories

        self.characters: dict[str, Character]= {}
        self.characters["P1"] = Character(inputHistories["P1"], self, "P1")
        self.characters["P2"] = Character(inputHistories["P2"], self, "P2")
        self.characters["P1"].assignOpponent(self.characters["P2"])
        self.characters["P2"].assignOpponent(self.characters["P1"])   
# ...

class Character():
    def __init__(self, inputHistory: inputs.InputHistory, \
                gameState: GameState, player: str) -> None:
        self.inputHistory = inputHistory
        self.gameState = gameState
        if player == "P1":
            self.xpos: int = 50
        else: # P2
            self.xpos: int = constants.WINDOW_WIDTH - 50
        
# ...
```

To make getting the opposing Character's attributes a little easier to type,
we *do* do something slightly irresponsible:

```python
class Character():
    # ...
    def assignOpponent(self, opponent: Character) -> None:
        '''
        Part of initialization, but deferred until after both Characters have been created
        to prevent P1 from referring to P2 before P2 has been created.
        '''
        self.opponent = opponent
```

Note that this is technically poor practice in Class design, because if `Character` gets used in the future by some other library,
it is possible for them to create a valid `Character` without remembering to call `assignOpponent` afterward, causing errors.
It would be best if this could automatically be handled in initialization.
But since we don't expect the `Character` class to be passed around much,
only within `gamestate.py` and its tests, we'll ease up a little here.

### Walking

```python
# gamestate.py, Character()
def update(self, frame_number: int) -> None:
    '''
    Takes a frame_number, and updates self based on the Buttons pressed on that frame.
    '''
    # NOTE: Added a helper function in inputs.py to get the buttons dict for frame X
    frame_buttons = self.inputHistory.getFrameButtons(frame_number)
    if frame_buttons[Button.LEFT]:
        self.xpos = self.xpos - self.walkspeed
    elif frame_buttons[Button.RIGHT]:
        self.xpos = self.xpos + self.walkspeed
```

We dumped our movement code above into `Character.update()` in a very straightforward way,
but it would be nice if we could move that into its own function.
Not only will it make `update()` a little cleaner, it will be easier to write tests for if it has its own function.

```python
# gamestate.py, Character()
def update(self, frame_number: int) -> None:
    '''
    Takes a frame_number, and updates self based on the Buttons pressed on that frame.
    '''
    frame_buttons = self.inputHistory.getFrameButtons(frame_number)
    if (frame_buttons[Button.LEFT] or frame_buttons[Button.RIGHT]):
        self.walk(frame_buttons)
    # ...

def walk(self, buttons: dict[Button, bool]) -> None:
    if self.facingLeft:
        if buttons[Button.LEFT]:
            self.xpos = self.xpos - self.forwardWalkspeed
        else:
            self.xpos = self.xpos + self.backwardWalkspeed
    else:
        if buttons[Button.LEFT]:
            self.xpos = self.xpos - self.backwardWalkspeed
        else:
            self.xpos = self.xpos + self.forwardWalkspeed
```

I'm not sure how common it is for forward and backwards walk speeds to be
[different](https://hisouten.koumakan.jp/wiki/Movement_Data#Walking),
but I decided to throw it in here for fun.  
To my knowledge, they're usually made the same, but my understanding is that they have slightly uses in gameplay -  
High backwalk speed helps you make your opponent's attacks whiff,
while high forward walk speed helps you ensure your attacks don't.

If you wanted to make a character good at one of these but not the other,
you might choose to give them different walkspeeds.

### character.facingLeft

```python
# gamestate.py
class Character():
    def __init__(self, inputHistory: inputs.InputHistory, gameState: GameState, \
                 player: str) -> None:
        # ...
        self.facingLeft = False
```

We haven't done anything with it yet, but we did define the `Character` property `facingLeft`
to determine whether to flip our `Character`.
It'
s actually a very pertinent part of state, not just a visual marker that flips our sprite horizontally.

```python
# gamestate.py
class Character():
    # ...
    def update(self, frame_number: int) -> None:
        # ...
        self.faceOpponent()

    def isRightOfOpponent(self) -> bool:
        '''
        Note that the blocking direction check uses this directly,
        but general input flip (forward vs back), sprite flip, hit/hurtbox flip
        are determined by self.facingLeft instead,
        a value that isn't always updated every frame.
        '''
        return self.xpos >= self.opponent.xpos
    
    def faceOpponent(self) -> None:
        '''
        Sets self.facingLeft based on Character's position relative to opponent Character.
        '''
        self.facingLeft = self.isRightOfOpponent()
```

For now, we just make each `Character` face their opponent on every frame,
but our desired logic is a little bit more complex. (See the end of chapter of aside for more information.)

### Rendering UI

Let's add our first UI elements to our game, HP bars and a round timer.

Although the HP bars reference an attribute that belongs to the `Character`s,
I think of `Character.render()` as the function that renders the `Character` sprite, and the sprite only.
So I feel that we should put our UI rendering in `GameState.render()` instead.

```python
# gamestate.py
class GameState():
    # ...
    def render(self, display: pg.surface.Surface) -> None:
        # render UI
        self.renderRoundTimer(display)
        self.renderHpBars(display)
        
        # render characters
        for character in self.characters.values():
            character.render(display)
```

Our round timer update logic will be a little basic, but it'll get the job done for now.
With a bit of Rect finagling, we can easily put it on the top of the screen.

```python
    def update(self) -> None:
        # TODO: may add more nuance to round timer than this
        self.round_timer = self.round_timer - 1.0 / constants.FRAME_RATE_CAP
        # ...

    def renderRoundTimer(self, display: pg.surface.Surface) -> None:
        '''
        Basic font-based method of rendering the round timer.
        '''
        rounded_timer = "{:.0f}".format(self.round_timer)
        self.text = self.font.render(rounded_timer, True, constants.WHITE)
        rect = self.text.get_rect()
        rect.midtop = (int(constants.WINDOW_WIDTH / 2), 5)
        display.blit(self.text, rect)
```

HP bars aren't too difficult either. We'll draw a coloured, "full" rectangle of varying length
on top of a white, "empty" rectangle of static length.

```python
    def renderHpBars(self, display: pg.surface.Surface) -> None:
        '''
        Basic HP bar UI.
        '''
        hp_bar_length = constants.WINDOW_WIDTH / 2
        
        # P1
        missing_hp_rect = pg.Rect(0, 0, hp_bar_length, 20)
        p1_hp_proportion = max(0.0, self.characters["P1"].hp / self.characters["P1"].maxHp)
        current_hp_rect = pg.Rect(missing_hp_rect)
        current_hp_rect.width = int(hp_bar_length * p1_hp_proportion)
        # For P1, the last bit of HP is the rightmost one
        current_hp_rect.topright = missing_hp_rect.topright
        
        pg.draw.rect(display, constants.WHITE, missing_hp_rect)
        pg.draw.rect(display, constants.RED, current_hp_rect)
        
        # P2
        missing_hp_rect = pg.Rect(0, 0, hp_bar_length, 20)
        missing_hp_rect.topright = (constants.WINDOW_WIDTH, 0)
        
        p2_hp_proportion = max(0.0, self.characters["P2"].hp / self.characters["P2"].maxHp)
        current_hp_rect = pg.Rect(missing_hp_rect)
        current_hp_rect.width = int(hp_bar_length * p2_hp_proportion)
        # For P2, the last bit of HP is the leftmost one (no change required)
        
        pg.draw.rect(display, constants.WHITE, missing_hp_rect)
        pg.draw.rect(display, constants.BLUE, current_hp_rect)
```

We don't have any way to actually lose HP yet, so let's add a temporary little hack to `Character.walk()`.

```python
# gamestate.py
    def walk(self, buttons: dict[Button, bool]) -> None:
        if self.facingLeft:
            if buttons[Button.LEFT]:
                self.xpos = self.xpos - self.forwardWalkspeed
            else:
                self.xpos = self.xpos + self.backwardWalkspeed
                self.hp = self.hp - 1 # TODO: temporarily added to show HP loss
        else:
            if buttons[Button.LEFT]:
                self.xpos = self.xpos - self.backwardWalkspeed
                self.hp = self.hp - 1 # TODO: temporarily added to show HP loss
            else:
                self.xpos = self.xpos + self.forwardWalkspeed
```

Now backwalking causes you to lose HP. A strange game design choice, but for now,
it's just the thing to help us visualize HP loss.

With our character flipping and UI changes done, our game now looks like this:

![Walking, flipping, and UI](/assets/images/article3/walking-and-ui.gif)

## Stopping point

This article is starting to get a bit long now, so we'll cut it off here.  
We introduced a fair few lofty long-terms goals, but we've hardly even started the immediate ones.  
We have some basic actions taken based on input, and the skeletons of our game state classes,
but we've yet to add an actual attack to our fighting game.  
But in article #3.1, we'll actually implement our first `StateTimeline`. Look forward to it.

[Code for this article on GitHub.](https://github.com/tuningdipsw/fighting-game-tutorial/tree/article3-game-state-1)

TODO: edit in link to #3.1  
[>> Next: #3.1: Game state 2](/fgtutorial/2023/02/01/article3-1-game-state-2.html)

## Aside: When or when not to flip a Character

The common rules for when or when not to flip `Characters` are...
I'm literally testing this out as I write this part out, they're darn complicated.  
You might want to boot up a fighting game to test these out yourself instead of just reading it,
it may be a bit tricky to parse through text alone.

- Characters performing an attack generally don't change their facing direction.

  - This often means that if you do an anti-air attack right before your opponent jumps over you,
and it has a hitbox that's more in front of your head than above or behind it, the lack of flip might cause it to miss.

- However, doing a move that passes through the opponent to cross them up
(eg. [Jam's crossup 236S~H](http://www.dustloop.com/w/GGACR/Jam_Kuradoberi#Senri_Shinshou))
might specifically change your facing direction.

- Characters midjump do not change their facing direction until they land, jump in midair, or recover from an attack(?).
This has some implications on aerial attacks, like [tacos.](https://glossary.infil.net/?t=Taco)

- A character being put in hit/blockstun does not change their facing direction
until they exit stun (return to neutral) or are put in hit/blockstun again.

However, the check for inputting the correct blocking direction (always the direction *away from the opponent*)
is done on attack contact and does **not** use facingLeft's value.
This gives us a further complication when parsing inputs:

- If you jump over your opponent and do an (eg.) air DP, your character will not flip,
so you **don't** have to flip your DP input directions.

- If you jump over your opponent and try to block, it is not parsed as a direct action.  
As with a normal jump, your character will not flip,
but you **do** have to block in the away direction if you get hit by an attack, despite your character not flipping.  
(Getting hit by an attack will put your character in hit/blockstun and then cause your sprite to flip.)

- If you were in hit/blockstun and your opponent crosses over you, your character will not flip (immediately),
but you **do** have to flip your blocking direction.

- There may be [crossup protection](https://glossary.infil.net/?t=Cross-up%20Protection) to help you block, though.

Generally it seems like these rules are a little fast and loose at times - here are some specific examples of quirks between games.

- In GBVS, pressing the Guard button midjump will flip your character as well.
Blocking with the Guard button will always blocks in the correct direction, so this is sort of a moot point for GBVS.

- In GG, the input to [FD](https://glossary.infil.net/?t=Faultless%20Defense) in midair
(backwards direction and two attack buttons that are not S+H or ?+D) will not be flipped (like the DP input example),
but FDing will not cause you to flip direction.

  - FD is maintained if you hold down the initial two attack buttons even if you release "back",
and by remaining in this blocking state you can keep blocking attacks despite not holding a "backwards" direction.

  - After blocking an attack this way, your character will flip because you entered a new state of blockstun,
but again, you will still correctly block even without holding a "back" direction by staying in FD state
(by holding down the two attack buttons).

- MBAACC rules offer the defender some crossup protection by let them block both by holding backwards for their character *or*
in the same direction that the offensive character is facing.
However, there's still a trick for the attacker to get around this -
see [Sandori](https://glossary.infil.net/?t=Sandori) for more details.

As you can see, these rules are quite complicated indeed.  
A little too complicated for now.  
We'll leave them be for this update, but we will have to account for them in the future.

---

TODO: edit in link to #3.1  
[>> Next: #3.1: Game state 2](/fgtutorial/2023/02/01/article3-1-game-state-2.html)
