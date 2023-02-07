---
title: "Let's make a fighting game #2: Inputs"
date: 2023-02-06 14:00:00 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #1: Introducing our game loop](/fgtutorial/2023/01/30/article1-gameloop.html)

[<< Prev: #1.1: Game loop tangents](/fgtutorial/2023/01/31/article1-1-gameloop-tangents.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D
fighting game using the Python game development library Pygame.

The first thing we need to create is the input system of our game.
Although it is an intriguing idea, we won't be taking after *NetHack* or *Dwarf Fortress*
and binding actions to single buttons (no 'q' to quaff). We must
instead find a way to decide whether a player's (eg.) "236P" input should trigger a
special move or a command normal (6P).

<!--more-->

> Numpad notation will be used in these articles. See
> [Dustloop](https://www.dustloop.com/w/Notation) for a refresher on how to read
> numpad notation.

## Eventual goals

- **Display inputs on screen.**

It may help us to reference how other games show their inputs.

![XRD's training mode](/assets/images/article2/xrd-input-display.png)
Guilty Gear XRD*'s input display system. (source: own screenshot)*

The slightly darker colour of some direction/button inputs indicate that the
direction/button is still being held down from the previous input state.

The actions that these inputs evaluated to can also be displayed,
although the buttons are ordered chronologically bottom to top,
while the actions are top to bottom.

![UNICLR's training mode](/assets/images/article2/uni-input-display.png)
Under Night*'s input display system.* [*(video source)*](https://www.youtube.com/watch?v=I42_NwoYdQw)

Does not visually distinguish between invalid or buffered inputs and valid inputs.
However, showing frame counts to indicate how long the player's inputs remained in each state looks useful.

- **Avoid hardcoding keybinds into our business logic.**

![+R's key mapping menu](/assets/images/article2/plus-r-key-mapping-gui.png)
Guilty Gear XXAC+R*'s key mapping menu. For some reason, +R has a keyboard menu that maps to controller keys,*
*and then a controller menu that maps those controller keys to game buttons, which is a little confusing.*

We will need to support a number of input devices and customizable controls for our fighting game,
so we must avoid hardcoding specific keys into our business logic.

A GUI for setting these keys in-game is a must, but we'll put it off for now.

- **Contextually parse inputs into actions.**

Special inputs are a famous fixture of the fighting game genre, ranging from the simple quarter-circle forward (236) motion
to the infamous [pretzel motion](https://glossary.infil.net/?t=Pretzel%20Motion) (1632413).

<aside>
There exist arguments both for and against them as a barrier to entry.
<a href="https://www.youtube.com/watch?v=1qEguZXyWjw">(1) </a>
<a href="https://www.youtube.com/watch?v=xBKnlyMpp1s">(2) </a>
<br>
<br>
Some games such as GBVS and Fantasy Strike have also experimented with easier 1-button specials.
We'll be using the most traditional type of special inputs in this game, but I invite you to experiment with this idea.
</aside>

The fighting game inputs are highly contextual even among fighting games;
for many moves, the game has to consider our previous inputs as well as our current ones
in order to determine whether (eg.) 6P should lead to the command normal 6P, or the special move 236P.

The exact logic of how to parse an input into one of multiple specific game actions, especially when the player has pressed
more buttons than necessary, as in the case of many [Option Selects](https://glossary.infil.net/?t=Option%20Select),
can become extremely complex.

Dustloop's *BlazBlue* page contains a reference of [input priority](https://www.dustloop.com/w/BBCF/Miscellaneous#Input_Priority) that
shows just how complex this system might become.
For special moves, we'll lean on [*Soku*'s simpler hierarchy](https://hisouten.koumakan.jp/wiki/Controls#Skills) for now,
since *BlazBlue*'s hierarchy varies between characters.

For the time being, we'll keep the inputs we accept fairly simple. We can reference the section on *BlazBlue*'s
["Input Requirements"](https://www.dustloop.com/w/BBCF/Miscellaneous#Input_Requirements) for some sensible specs on how to parse
a special input.

- **Accept buffered inputs.**

An extension of the input system that allows more leniency during combos and defense.
The leniency of the buffer system varies between games, ranging from generous to
[nearly non-existent](https://twitter.com/TheFuntax/status/1321983553966669824),
giving a slightly different feel to each game.

See ["Buffer Window"](https://glossary.infil.net/?t=Buffer%20Window) on the Fighting Game Glossary.

Some inputs don't cause an action (immediately), due to the character
being committed to some other action, or possibly the requested action being invalid during
the current state.

If done close enough to the earliest valid frame that the action could take place,
a valid input may be "buffered" by the game to trigger the requested action on that earliest frame.

However, if the input is performed too many frames before that earliest valid frame, or "outside of the buffer window",
it is considered invalid and the game will not translate it into an action.

## Starting with the basic game loop

I've borrowed a version of the game loop with a FPS counter from
[this tutorial](https://coderslegacy.com/python/display-fps-pygame/).
I've taken the liberty of separating some global constants and the FPS counter
class into separate files:

```python
# constants.py
WINDOW_WIDTH = 640
WINDOW_HEIGHT = 480
FRAME_RATE_CAP = 60

WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
```

```python
# fps_counter.py
import pygame as pg
import constants

class FpsCounter:
    def __init__(self):
        self.clock = pg.time.Clock()
        self.font = pg.font.SysFont("Verdana", 10)
        self.text = self.font.render(str(self.clock.get_fps()), True, constants.WHITE)
        
    def render(self, display):
        fps = int(self.clock.get_fps())
        self.text = self.font.render(f"{fps}FPS", True, constants.WHITE)
        display.blit(self.text, (constants.WINDOW_WIDTH - 50, 5))
```

```python
# main.py
import pygame as pg
import sys

import fps_counter as fps
import constants

pg.init()
window = pg.display.set_mode((constants.WINDOW_WIDTH, constants.WINDOW_HEIGHT))
fpsCounter = fps.FpsCounter()

def processInput():
    pass

def update():
    pass

def render():
    window.fill(constants.BLACK)
    fps_counter.render(window)
    
    pg.display.update()

running = True
while running:
    processInput()
    update()
    render()
    fpsCounter.clock.tick(constants.FRAME_RATE_CAP)
```

Running this game only displays a black screen with an FPS counter,
but this is a nice skeleton of a program for us to springboard off of.

## Establishing some terminology

<aside>
Not an official terminology, I'm inventing it on the spot right now.
</aside>

Our ultimate goal with the input system is to translate *keys* to *buttons* to *inputs* to *actions*:

**Keys**  
The raw inputs of an input device such as a keyboard, joystick, or controller.  
We'll start with keyboard support only for simplicity.

<aside>
Note: Don't get confused by the other macros,
a dash macro button should be handled as an unique button than a combination of multiple buttons.
</aside>

**Buttons**  
The "buttons" that your game supports.  
In our case, this is the 4 cardinal direction keys (keyboard),
a 2-dimensional direction from a joystick,
the 5 attack buttons "Punch", "Kick", "Slash", "Heavy Slash", "Dust",
and macro buttons for combinations like P+K+S or dash macro.

We map our keys to our buttons.

**Inputs**  
The processed form of the buttons, after [SOCD cleaning](https://www.hitboxarcade.com/blogs/support/what-is-socd)
and macro button combinations have been applied.

<aside>
Technically, we will be storing the diagonal directions as a combination of two
cardinals, but logically we'll pretend those are a single diagonal.
</aside>

For leverless devices like the keyboard, multiple buttons for contradictory directions can be pressed
(Simultaneous Opposite Cardinal Directions),
so we need to clean SOCDs and translate them into one of the 8 directions, or the neutral direction.

Devices with levers/joysticks can't do this, so we just translate the 2-dimensional input into the
closest of the 8 directions, or neutral if the stick is in some defined deadzone.
It might be okay to handle that in "Buttons" instead of "Inputs", but it probably doesn't really matter.

**Actions**  
The actions performed by the character in game after parsing the inputs.

NB: We will consider inputs independent of state, and actions dependent on it.
So inputs will use the absolute directions of left and right,
while actions will have to consider which side of the opponent a character is on,
and convert inputs' "left or right" into forward or back.

Due to this dependence on game state, the parsing of inputs into actions will
occur in `update()`, not `processInput()`. This way, `processInput()` does not need
to know anything about game state.

## Should we get input from pygame.key.get_pressed() or the event queue?

While filling in `processInput()`, we hit a fork in the road.

```python
# main.py
def processInput():
    # Method (1): Use key.get_pressed()
    parseKeysPressed()
    
    # Method (2): Event queue (not used)
    for event in pg.event.get():
        if event.type == pg.QUIT:
            cleanupGame()
            break

def cleanupGame():
    '''
    Run any cleanup before exiting the game.
    '''
    pg.quit()
    sys.exit()
```

In Pygame, we have [one of two choices](https://www.pygame.org/docs/tut/newbieguide.html#managing-the-event-subsystem)
in how we handle input:  
We can either ask Pygame for a list of keys that being pressed at this very moment,
or we can check Pygame's event queue for a list of all the key presses and key releases
that have occurred since the last time we checked the queue (ie. called `processInput()`).

As a queue, the event queue method will catch all inputs, even if performance drops.
If the performance of the game loop drops to 1 FPS, the queue will capture all inputs
during that frame, while `key.get_pressed()` can only ever capture 1 frames' worth of inputs.

However, because it can capture multiple frames' worth of inputs, the logic of `processInput()`
becomes harder to get right as well. Even if it did only capture one frames' worth of inputs,
handling simultaneous button presses with the event queue requires juggling knowledge
of key presses and key releases, which is also a little tricky.

<aside>
I probably will revisit the event queue version of the `processInput()` eventually,
even if we never hit performance issues, just because it will make a good lesson
for the tutorial.
</aside>

As stated in article #1, we will not spend unnecessary time worrying about performance before we have to.
We'll go with the simpler `key.get_pressed()` method for now just to keep us moving forward;
if our game starts to have trouble maintaining 60 FPS, I will revisit this to make the switch.

## Defining the keybinds

Let's create a new file `inputs.py` to store our key, button, and input-related functions.

```python
# inputs.py
from __future__ import annotations
# https://stackoverflow.com/questions/33533148
# For type hinting support.
from enum import Enum

import pygame.locals as locals
import pygame as pg
import constants

class Button(Enum):
    LEFT = 0
    DOWN = 1
    RIGHT = 2
    UP = 3
    PUNCH = 10
    KICK = 11
    SLASH = 12
    HEAVY = 13
    DUST = 14
    MACRO_PK = 20
    MACRO_PD = 21
    MACRO_PKS = 22
    MACRO_PKSH = 23

# define keybindings manually here
# TODO: eventually replace with a proper menu interface for rebinding keys
keybinds = {}
keybinds[locals.K_7] = Button.LEFT
keybinds[locals.K_8] = Button.DOWN
keybinds[locals.K_9] = Button.RIGHT
keybinds[locals.K_SPACE] = Button.UP
keybinds[locals.K_z] = Button.PUNCH
keybinds[locals.K_x] = Button.KICK
keybinds[locals.K_c] = Button.SLASH
keybinds[locals.K_v] = Button.HEAVY
keybinds[locals.K_d] = Button.DUST
keybinds[locals.K_f] = Button.MACRO_PKS
keybinds[locals.K_g] = Button.MACRO_PK
keybinds[locals.K_h] = Button.MACRO_PD
keybinds[locals.K_j] = Button.MACRO_PKSH
# ...
```

A (literal) key-value pair dictionary stores our key mappings.
I've used Python's [Enum](https://docs.python.org/3/howto/enum.html#) feature to define our buttons.

You can use literal strings ("punch", "kick", etc) instead of Enums, as I did initially, but if you ever misspell one of these strings
("puncg") in your logic later down the line, the logic may fail silently, which is troublesome to debug.
Enums will cause an error at runtime if misspelled, so they are safer to use.

```python
# inputs.py
# ...
macro_defs = {}
macro_defs[Button.MACRO_PK] = [Button.PUNCH, Button.KICK]
macro_defs[Button.MACRO_PD] = [Button.PUNCH, Button.DUST]
macro_defs[Button.MACRO_PKS] = [Button.PUNCH, Button.KICK, Button.SLASH]
macro_defs[Button.MACRO_PKSH] = [Button.PUNCH, Button.KICK, Button.SLASH, Button.HEAVY]

def keysPressedToInput(current_frame: int) -> Input:
    '''
    Takes an int current_frame,
    checks pygame.key.keys_pressed() for all keys currently pressed,
    and returns an Input created with a dict of all assigned Buttons pressed
    (after SOCD cleaning) and current_frame.
    '''
    keys_pressed = pg.key.get_pressed()
    frame_buttons: dict[Button, bool] = {}
    for (key, button) in keybinds.items():
        frame_buttons[button] = keys_pressed[key]
        
    for macro_button in macro_defs.keys():
        if frame_buttons.get(macro_button) and frame_buttons[macro_button]:
            for button in macro_defs[macro_button]:
                frame_buttons[button] = True
    
    frame_buttons = cleanSocdButtons(frame_buttons)
    return Input(frame_buttons, current_frame, current_frame + 1)

#...
```

We define some macro buttons as well, and `keysPressedToInput()`, which neatly
translates the keys of `pg.key.get_pressed()` into our Buttons.

It eventually creates an Input,

```python
# inputs.py
# ...
class Input():
    def __init__(self, buttons: dict[Button, bool], start_frame: int, end_frame: int):
        self.buttons = buttons
        self.start_frame = start_frame
        self.end_frame = end_frame
        
    # https://stackoverflow.com/questions/390250
    def __eq__(self, other):
        if isinstance(other, Input):
            return self.buttons == other.buttons and self.start_frame == other.start_frame and self.end_frame == other.end_frame
        return NotImplemented

# ...
```

after running the buttons through `cleanSocdButtons()`.

```python
# inputs.py
# ...
def cleanSocdButtons(frame_buttons: dict[Button, bool]) -> dict[Button, bool]:
    '''
    Takes a dict with each assigned Button pressed this frame, 
    and returns a copy of it with SOCD cases handled:
    - UP + DOWN = neutral, remove both
    - LEFT + RIGHT = neutral, remove both
    
    Note that these don't both have to be handled the same way.
    '''
    cleaned_inputs = frame_buttons
    if cleaned_inputs[Button.LEFT] and cleaned_inputs[Button.RIGHT]:
        cleaned_inputs[Button.LEFT] = False
        cleaned_inputs[Button.RIGHT] = False
    if cleaned_inputs[Button.UP] and cleaned_inputs[Button.DOWN]:
        cleaned_inputs[Button.UP] = False
        cleaned_inputs[Button.DOWN] = False
    return cleaned_inputs

# ...
```

These functions are all we need to convert from Keys to Input.
We call these functions in `main.py`'s `parseKeysPressed()` to create an Input.

```python
# main.py
# ...
import inputs
# ...
current_frame = 0
inputHistoryP1 = inputs.InputHistory("P1")
inputHistoryP2 = inputs.InputHistory("P2")
# ...
def parseKeysPressed():
    '''
    Checks what keys are currently being pressed, 
    and creates a corresponding Input in input_history.
    '''
    input = inputs.keysPressedToInput(current_frame)
    inputHistoryP1.append(input)
```

<aside>
Download the Segoe UI Symbol font <a href="https://freefontsdownload.net/free-segoeuisymbol-font-135679.htm">here</a>
and add it to an `assets` folder.

Thanks to <a href="https://stackoverflow.com/questions/61974883/unicode-characters-not-showing-when-displaying-to-pygame">
this StackOverflow answer</a>
for how to render the Unicode directional arrows.
</aside>

We'll store these Inputs in a slightly modified container class, InputHistory.
This allows us to implement some custom logic when adding new Inputs in `append()`,
and it also allows us to give it its own `render()` method that we can slot right into `main.py`.

```python
# inputs.py
# ...
class InputHistory():
    def __init__(self, player: str):
        self.player = player # "P1" or "P2"
        self.inputs: list[Input] = []
    
    def append(self, input: Input) -> None:
        '''
        Takes an Input and adds it to self.inputs,
        and removes old inputs from self.inputs.
        '''
        if len(self.inputs) > 0 and input.buttons == self.inputs[-1].buttons:
            # If input buttons are the same as last input,
            # combine it with last input instead of making a duplicate.
            self.inputs[-1].end_frame = self.inputs[-1].end_frame + 1
        else:
            self.inputs.append(input)
            
        # Don't need to keep inputs after a certain amount of time has passed.
        # Down/back charge history will be stored in game state,
        # so deleting old inputs has no effect on charge moves.
        if len(self.inputs) > 30:
            self.inputs.pop(0)
    
    def render(self, display: pg.surface.Surface) -> None:
        font = pg.font.Font("assets/seguisym.ttf", 20)
    
        for i in range(len(self.inputs)):
            # Print the newest inputs first, closer to the top.
            input = self.inputs[len(self.inputs) - 1 - i]
            
            arrow_direction = directionsToArrow(input.buttons)
            attack_buttons = attackButtonsToLetters(input.buttons)
            input_string = f"{arrow_direction} {attack_buttons} {input.end_frame - input.start_frame}"
            
            text = font.render(f"{input_string}", True, constants.WHITE)
            if self.player == "P1":
                x = 10
            else:
                x = constants.WINDOW_WIDTH - 80
            display.blit(text, (x, 15 * i))
```

(The logic behind deleting old inputs is probably not perfect, but it probably doesn't have to be, either.)

We also implement some basic helper methods to convert Inputs into a text form, for InputHistory's `render()`.

```python
# inputs.py
def directionsToArrow(buttons: dict[Button, bool]) -> str:
    '''
    Takes a dict of buttons pressed,
    and returns a character of the arrow corresponding to their direction
    (or ' ' for neutral).
    
    It is assumed that the buttons have already been SOCD cleaned.
    '''
    if buttons[Button.LEFT]:
        if buttons[Button.UP]:
            return '↖' #
        elif buttons[Button.DOWN]:
            return '↙'
        else:
            return '←'
    elif buttons[Button.RIGHT]:
        if buttons[Button.UP]:
            return '↗'
        elif buttons[Button.DOWN]:
            return '↘' #
        else:
            return '→'
    elif buttons[Button.DOWN]:
        return '↓'
    elif buttons[Button.UP]:
        return '↑'
    else:
        return ' '

def attackButtonsToLetters(buttons: dict[Button, bool]) -> str:
    '''
    Takes a dict of buttons pressed
    and returns a string of letters (or ' ' for nothing)
    representing all attack buttons pressed.
    '''
    string = ""
    if buttons[Button.PUNCH]:
        string = string + "P"
    if buttons[Button.KICK]:
        string = string + "K"
    if buttons[Button.SLASH]:
        string = string + "S"
    if buttons[Button.HEAVY]:
        string = string + "H"
    if buttons[Button.DUST]:
        string = string + "D"
    return string

```

We can now add this into our `main.py` game loop:

```python
# main.py
# ... 
def render():
    window.fill(constants.BLACK)
    fpsCounter.render(window)
    inputHistoryP1.render(window)
    inputHistoryP2.render(window) # no Inputs being added here right now
    
    pg.display.update()

running = True
while running:
    processInput()
    update()
    render()
    
    current_frame = current_frame + 1
    fpsCounter.clock.tick(constants.FRAME_RATE_CAP)
```

And that's all we need to get Inputs running and displayable.

![Gif of our game, inputs displaying in all their glory](/assets/images/article2/input-display.gif)

Nothing too glamorous, but it gets the job done.  
Ah, pushing buttons to make all sorts of text spool out rapidly is quite fun.

## Stopping point

Not a bad place to stop - our Inputs system seems to work fine,
and is decoupled enough from other systems that we shouldn't need to
touch it much when we begin work on `update()`.

We are missing a few things in the Inputs system, like a second set of keybindings for player 2,
there are some [magic numbers](https://en.wikipedia.org/wiki/Magic_number_(programming))
littered around that need to be cleaned up at some point,
and we will need to bosh it around some more for GGPO, rollback, and netplay,
but it's enough for now.

[![How to download from GitHub](/assets/images/download-on-github.png)](https://github.com/tuningdipsw/fighting-game-tutorial/tree/article2-inputs)

*`python main.py` when in the same directory as `main.py` to run it.*

[I've uploaded the code for this article on GitHub here](https://github.com/tuningdipsw/fighting-game-tutorial/tree/article2-inputs),
where you can read it in uninterrupted form, or download it to try it yourself.

If we look back at our "Eventual goals", we've currently managed "Display inputs on screen" and
"Avoid hardcoding keybinds into our business logic".

But before we can work on parsing our Inputs into actions, we'll need to build up our game state.
We'll begin tackling this in article #3.

Coming to a blog, which is this one, at a time when it's ready, which is soon (I hope). Look forward to it.

TODO: edit in link to #3

<a href="/fgtutorial/2023/02/01/article3-game-state.html" align="right"> >> Next: #3: Game state</a>

### Bonus: Setting up some unit tests

I've also written a short mini-article on adding a few unit tests for `inputs.py`.  
This is not necessary in order to build our game, but it is an important part of programming at large,
and I couldn't find a direct mention of setting up tests within the Pygame tutorials,
so I thought I would share something about how I set mine up, and some of the resources I drew upon to do so.

It's more of a general Python or programming topic than a Pygame one, but I think it'll be quite helpful to cover.

TODO: edit in link to #2.1

<a href="/fgtutorial/2023/02/01/article2-1-unit-testing-primer.html" align="right"> >> Next: #2.1: Unit testing primer</a>

### Article meta

Code fragments make up the body of this article, but I'm not sure how effective it was.

I think it's better than writing pure abstract theory of what needs to be done,
but most of my non-code comments turned out rather light.
I am reasonably proud that my code is neat and well-commented enough to be self-documenting, at least in my opinion,
but it did make my non-code comments feel a bit obvious when I wrote them.

But I think there is some merit to providing a logical order for looking through the code.  
It feels more approachable than throwing a whole GitHub repository at the reader.  
Learning to read through one of those is a useful skill, but it's always a bit daunting.

If you have any complaints or feedback about the format, please let me know in the comments below.
