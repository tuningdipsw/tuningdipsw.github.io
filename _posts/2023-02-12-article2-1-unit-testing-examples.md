---
title: "Let's make a fighting game #2.1: Unit testing examples"
date: 2023-02-12 15:00:00 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #2: Inputs](/fgtutorial/2023/02/06/article2-inputs.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D
fighting game using the Python game development library Pygame.

It is technically possible to write code without testing, in the same way that it
is technically possible to draw with your eyes closed.

Both regular testing, by running your game, and automated testing,
by writing a suite of tests after making changes to your game,
allow you to be more confident about the correctness of your code,
and help catch bugs your new changes might introduce.

You might feel that writing tests is an unnecessary chore.  
I also felt that way before I wrote these tests,
but I think I wouldn't have that gut reaction if I'd just found
just the tutorial I was looking for to show me how to do it.

For that reason, I'll be sharing a brief account of what I did to set up
unit testing for article #2. I hope you find it useful.

<!--more-->

## Manual testing

is just the act of running your game, or the relevant section of it,
and testing that whatever change you just made to the code:

- doesn't immediately crash
- does what you expected your code to do
- doesn't cause any obvious changes in the rest of your game

And so, your normal development process (of any code, not just games)
is just a cycle of writing bits of your desired code in steps that should work,
running it and fixing the errors when it doesn't, and slowly building on each of those steps
until the desired functionality has been achieved.

Coding becomes a lot harder when you're unable to figure out how to subdivide your problem into testable steps,
or if you try to write too much code in one go without testing; learning how to do this is part of learning to code.

I believe my development process of article #2 went something like this:

- Cribbed a basic game loop and FPS counter off of a [tutorial](https://coderslegacy.com/python/display-fps-pygame/)
that I knew would work

- Set up the `keybinds` dictionary

- Figured out how to convert `pg.key.get_pressed()` into a `dict[Button, bool]` using `keybinds` in `keysPressedToInput()`

- Started printing this dictionary of Buttons to console in `main.py`, so I could see the buttons I pressed appear

- Added `cleanSocdButtons()` to `keysPressedToInput()`

- Started blitting the rendered text of each dictionary of Buttons to the screen with `directionsToArrow()` and `attackButtonsToLetters()`

- Fiddled around with the format of `keybinds`, changing it from `key: str` to `key: list(str)`
as a way of implementing macro keys nicely

- Defined the `Input` class as a proper form of the `dict[Button, bool]`

- Defined the `InputHistory` class to replace the `list[dict[Button, bool]]` I was using in `main.py`, and formalized
the `append()` and `render()` functions there

- Replaced the strings in `keybinds` with safer `Button` Enums

- Reverted `keybinds` to `key: Button` and implemented the macro keys as a separate dictionary, because it made
the `keysPressedToInput()` logic neater

...all the while hitting the VSCode "Run without Debugging (Ctrl+F5)" button until I had a working input system,
and then going a little further with some refactoring at the end.

<aside>
The bug was that P,K,S would not appear if the Punch, Kick, or Slash keybinds
were pressed on their own, only from pressing the macro_PKS button.
<br>
<br>
When the `for` loop finally iterated to the macro_PKS keybind after handling the rest of the keybinds,
the macro_PKS not being down overwrote frame_buttons[Button.PUNCH] to `= False`, from the `= True` that had been set by the Punch keybind
(and so on).
</aside>

However, I had actually introduced a bug that I didn't notice in my manual testing for some time
when I changed `keybinds` from `key: str` to `key: list(str)`.
The details of the bug are not important, but the fact that I'd missed it for even a short period of time
made me think "I really need to set up some automated unit tests, I shouldn't put this off any longer."

As your program gets larger and your codebase more complex, it becomes harder to hold
it all in your head and more tiring to test. At some point you'll start missing things.
This is where you need to start automating your testing.

## Automated testing

More specifically, *regression testing* is the act of checking that things that you had working in the past
haven't stopped working (haven't regressed) after you've made a change. As you can imagine, the amount of
regression testing you need to perform only goes up as you code;
in order to keep that workload manageable, you have to automate it.

Any programming language that people use to code *will* have some libraries for testing.
Python's main ones are *unittest* and *pytest*.

> As I am not an experienced coder, game programmer, or Pygame user,
> I suspect my approach is likely imperfect. I'd love to hear more about how you've
> set up your own Python and Pygame tests in the comments section below the article.

### What functions should we test?

Although it's tempting to say "everything", and perhaps if you're very idealistic you said that too,
it is also possible to write more tests than you strictly need to. It's not the end of the world
if you do, but if you're smart and **intentional** about which ones you skip, you may be able to save
some of your mental strength for other, more important tasks.

I didn't write tests for all of my functions -
for example, `directionsToArrow()` and `attackButtonsToLetter()` are untested because
they are basically helper functions simple enough to reason correctness,
and they are of low importance, since they don't affect game logic, only rendering.  
So I don't expect to modify them in the future, and I don't feel worried about skipping them.

A further caveat we might run into is that some parts of games don't translate well to automated testing.
Take our `render()` functions -

It's easy enough for us to verify by hand and by eye that `render()` is correct by running our program and checking
that it displays such-and-such, that it "looks right",
but it's hard to define what "correct" or "looks right" is at the code level.
Audio cues ("sounds right") or responsiveness ("feels right") have the same issue.

As far as I know, we must take it on faith that Pygame will handle drawing, audio, and inputs
correctly; we trust that these aspects of this library we're making use of have already been tested.

That doesn't mean that we can't write any tests if our functions happen to touch Pygame even once,
but the easiest tests for us to write will be for code whose logic is cleanly separated from Pygame elements.

### How do we test?

Although I expressed the difficulty of writing automated tests around Pygame, we're not entirely helpless.
We can make some use of [`unittest.mock`'s functions](https://docs.python.org/3/library/unittest.mock.html#the-mock-class)
in the `mock.call_count()` and `mock.assert_called_with()` family.

That is, although our tests cannot verify how the results handled by the Pygame library "look" or "sound",
we can write tests that verify that certain Pygame functions such as `surface.blit()` are being called
`X` number of times and with `Y, Z...` parameters.

The actual "automated" part of running the tests can then be done with a single keyboard shortcut (in VSCode: `Ctrl + ; then A`)
whenever you feel the need to check for regressions (e.g. right before you do a manual test as part of your normal development).

The tests themselves, through the use of the `unittest`/`pytest` libraries, are pretty straightforward.
We'll see what some of the actual tests look like shortly.

### What tests should we write?

As the name "unit testing" implies, we start with the smallest functions, the most atomic parts
of our code, and write tests to verify their correctness. We don't worry so much about how
different modules work together, we plan to handle that with "integration tests".

<aside>
See <a href="https://blog.miguelgrinberg.com/post/how-to-write-unit-tests-in-python-part-2-game-of-life">
Practical unittest and pytest example, part 2, </a>
also represented in Additional reading below.
</aside>

Test coverage is the idea that your tests should cover 100% of the paths your code can take, and there are some
plugins that you can set up to calculate this percentage for you.
I didn't set one up myself, but it is an option.

In any case, covering all the code paths is the goal of your testing.  
Test single cases, test overlapping cases, test functions that use other functions, and so on.

> Custom user input requires especially careful input validation, although games don't have as much of this as,
> say, websites or apps do.
>
> Think carefully about what inputs are possible to pass to a function, which ones you want to accept
> and which ones should be rejected.
> A good practice is for your function to begin by making assertions about its inputs, even those not from users,
> just to define and check your assumptions about the inputs.

### `test_cleanSocdButtons()`

I started with one of the simpler functions, since I wasn't familiar with `pytest` yet.

Although I don't have its original form any more, I probably wrote it as something like this at first:

```python
# test/test_inputs.py
def test_cleanSocdButtons_LR_neutral():
    input = {Button.LEFT: True, Button.RIGHT: True, Button.UP: False, Button.DOWN: True}
    expected = {Button.LEFT: False, Button.RIGHT: False, Button.UP: False, Button.DOWN: True}
    output = inputs.cleanSocdButtons(input)
    assert output == expected
```

This can be generalized to multiple test cases with `pytest`'s `parametrize` annotation:

```python
# test/test_inputs.py
cleanSocdButtons_testcases = [
    # L+R = neutral
    ({Button.LEFT: True, Button.RIGHT: True, Button.UP: False, Button.DOWN: True},
     {Button.LEFT: False, Button.RIGHT: False, Button.UP: False, Button.DOWN: True}),
    
    # U+D = neutral
    ({Button.LEFT: True, Button.RIGHT: False, Button.UP: True, Button.DOWN: True},
     {Button.LEFT: True, Button.RIGHT: False, Button.UP: False, Button.DOWN: False}),
    
    # both rules at the same time
    ({Button.LEFT: True, Button.RIGHT: True, Button.UP: True, Button.DOWN: True},
     {Button.LEFT: False, Button.RIGHT: False, Button.UP: False, Button.DOWN: False})
]

@pytest.mark.parametrize("test_input, expected", cleanSocdButtons_testcases)
def test_cleanSocdButtons(test_input, expected):
    assert inputs.cleanSocdButtons(test_input) == expected
```

This one wasn't too hard, since the tested function doesn't call any other functions,
interact with complex classes, and doesn't even have many cases to cover. So we've got the hang of it now.
Maybe we had to do a little finagling with settings to get VSCode to detect the test (I think I had to
create a blank `__init__.py` file in the `test` folder), but we were able to get a simple test running.

### `test_keysPressedToInput()`

Now for the tricky one -- this one has messier inputs and uses mocks.

The first mock we'll need to use is for `pygame.key.get_pressed()`.

Ok, we don't actually need it - we could instead write our way around this
by rewriting the `keysPressedToInput()` function in `inputs`
to take in a list of key names as a parameter, instead of calling `pygame.key.get_pressed()` within the function.
Then we just make that list the testcase input instead of intercepting `pygame.key.get_pressed()` with a mock in this test.

I didn't do it this time because I wanted to learn how to get this mock to work,
but I don't believe there's a strong reason against doing so.

Either way, we write a little helper function to convert
lists like `[locals.K_z, locals.K_x, locals.K_c, locals.K_v, locals.K_d]`
into the Pygame `key.get_pressed()` format (with all the empty spaces for keys unpressed):

```python
# test/test_inputs.py
def create_key_mock(pressed_key_list):
    '''
    Takes a list of key names (pygame.locals, which are technically ints)
    and returns a simulated pygame.key.get_pressed() output with those keys pressed.
    '''
    tmp = [0] * 300
    for key in pressed_key_list:
        tmp[key] = 1
    return tmp
```

The other mock we'll need is for `inputs.keybinds`:
While we could use `inputs.keybinds` to determine what `pygame.locals.[key]`s to feed into
our testcase inputs, if we think ahead, we're eventually going to have to support
rebinding keys, so this might cause a temporary break in these tests when we make that change
and remove the hardcoded `keybinds` definition in `inputs`. Sure, we can fix that when we get there,
but we can also try to handle that ahead of time.

We'll instead fix `keybinds` to a dictionary in this test file and mock that into `keysPressedToInput()`
when we test it, so that our `pygame.locals.[key]` test inputs are unaffected if
we change `inputs.keybinds` in the future.

```python
# test/test_inputs.py
default_keybinds = {}
default_keybinds[locals.K_7] = Button.LEFT
default_keybinds[locals.K_8] = Button.DOWN
default_keybinds[locals.K_9] = Button.RIGHT
default_keybinds[locals.K_SPACE] = Button.UP
default_keybinds[locals.K_z] = Button.PUNCH
default_keybinds[locals.K_x] = Button.KICK
default_keybinds[locals.K_c] = Button.SLASH
default_keybinds[locals.K_v] = Button.HEAVY
default_keybinds[locals.K_d] = Button.DUST
default_keybinds[locals.K_f] = Button.MACRO_PKS
default_keybinds[locals.K_g] = Button.MACRO_PK
default_keybinds[locals.K_h] = Button.MACRO_PD
default_keybinds[locals.K_j] = Button.MACRO_PKSH
# yeah, this is a copy of what's in inputs.py right now
```

Disclosure: It took me some time to figure out how to mock `keybinds`,
because I initially tried to do it with

```python
@mock.patch.object(inputs, "keybinds")
def test_keysPressedToInput(mock_keybinds, test_input_keys, expected_buttons):
    mock_keybinds.return_value = default_keybinds
    # ...
```

but this caused a strange KeyError when running the test.

The annotation format of `@mock.patch.object(inputs, "keybinds", default_keybinds)`
is (one of) the correct way(s) to patch out `keybinds` for a different object `default_keybinds`,
I had apparently confused `return_value` for the object itself.

The final, parametrized version of `test_keysPressedToInput()` looks like this:

```python
# test/test_inputs.py
keysPressedToInput_testcases = [
    # single key
    ([locals.K_z],
     [Button.PUNCH]),
    
    # all single keys
    ([locals.K_z, locals.K_x, locals.K_c, locals.K_v, locals.K_d],
     [Button.PUNCH, Button.KICK, Button.SLASH, Button.HEAVY, Button.DUST]),
    
    # macro keys
    ([locals.K_f],
     [Button.PUNCH, Button.KICK, Button.SLASH, Button.MACRO_PKS]),
    
    # multiple macro keys
    ([locals.K_f, locals.K_h],
     [Button.PUNCH, Button.KICK, Button.SLASH, Button.DUST, Button.MACRO_PKS, Button.MACRO_PD]),
    
    # macro + single
    ([locals.K_z, locals.K_f],
     [Button.PUNCH, Button.KICK, Button.SLASH, Button.MACRO_PKS]),
    
    # simple SOCD
    ([locals.K_7, locals.K_9],
     [])
]

# Note: Mocking keybinds to default prevents test from breaking when hardcoded keys change
@pytest.mark.parametrize("test_input_keys, expected_buttons", keysPressedToInput_testcases)
@mock.patch.object(inputs, "keybinds", default_keybinds)
@mock.patch.object(key, "get_pressed")
def test_keysPressedToInput(mock_key_get_pressed, test_input_keys, expected_buttons):
    mock_key_get_pressed.return_value = create_key_mock(test_input_keys)
    current_frame = 1
    
    output = inputs.keysPressedToInput(current_frame)
    expected = inputs.Input(create_buttons_dict(expected_buttons), 1, 2)
    
    assert output == expected
```

Note that we added another helper function `create_buttons_dict()` to convert our Button list in the `expected_buttons` parameter
into a proper dict of Buttons (with `False` items for Buttons, since KeyErrors would occur if (eg.) Button.LEFT is missing
from the Dict when `cleanSocdButtons()` runs).

```python
# test/test_inputs.py
def create_buttons_dict(pressed_buttons: list[Button]) -> dict[Button, bool]:
    '''
    Takes a list of Buttons and returns a proper dict[Button, bool].
    '''
    buttons = {}
    for button in list(Button):
        buttons[button] = button in pressed_buttons
    return buttons
```

That helps us write a cleaner set of inputs and expected outputs in our test cases.

> Note that we have to make a small change to the `Input()` class in `inputs` to handle the `==` equality operator
> in `assert output == expected`:
> [(See this post for more details.)](https://stackoverflow.com/questions/390250/elegant-ways-to-support-equivalence-equality-in-python-classes)

```python
# inputs.py
def __eq__(self, other):
    if isinstance(other, Input):
        return self.buttons == other.buttons and self.start_frame == other.start_frame and self.end_frame == other.end_frame
    return NotImplemented
```

### `test_inputHistory_append()`

```python
# test/test_inputs.py
def test_inputHistory_append():
    ih = inputs.InputHistory("P1")
    ih.append(inputs.Input(create_buttons_dict([Button.PUNCH]), 1, 2))
    ih.append(inputs.Input(create_buttons_dict([Button.PUNCH]), 2, 3))
    
    # test input deduplication
    expected = inputs.Input(create_buttons_dict([Button.PUNCH]), 1, 3)
    assert len(ih.inputs) == 1
    assert ih.inputs[-1] == expected
    
    # regular case
    ih.append(inputs.Input(create_buttons_dict([Button.KICK]), 3, 4))
    expected = inputs.Input(create_buttons_dict([Button.KICK]), 3, 4)
    assert len(ih.inputs) == 2
    assert ih.inputs[-1] == expected
```

Our last test is pretty easy as well, as it doesn't require any mocks.
The main behavior of `append()` that we're interested in is the input deduplication,
so we check both the duplicate and non-duplicate case.

I think it's fine in this case because it makes sense for the testcases to be sequential,
but I think it's slightly preferable to separate your tests into different cases instead of combining them into a single
mega-testcase function for function X, because if multiple tests fail at the same time,
the failure logs will only show the first failure.

<br>
<br>
<br>

Those are the tests I wrote. Rather light for now, but I'll definitely need to write quite a few more
when I start laying out the complex particulars of the game state, and the game logic of the fighting game.

For ease of viewing, the tests for my code can be found in the `test/` folder of my fighting game tutorial's
[GitHub repository,](https://github.com/tuningdipsw/fighting-game-tutorial/tree/main/test)
and will be updated with new tests as I publish new chapters.

### Bonus: The debugger, a useful IDE feature

A great feature of contemporary IDEs is the ability to easily set breakpoints in your code.

![VSCode's debugger](/assets/images/article2-1/debugger.png)

In VSCode, you set or unset breakpoints by clicking the red circles to the left of the line numbers (1),
and the debugger will kick in when you choose to "Start Debugging (F5)", or run a test with "Debug Test" instead of "Run Test".

After hitting a breakpoint, you can use the cassette buttons (2) to resume program flow, to step one line forward, etc, and
check what's in each variable in the Variable pane (3).

The debugger is generally easier to handle than adding `print()` statements, finagling f-strings into the right formats to get
those variables' values, reading through all the printed logs at once, and then remembering to delete those `print()`
statements afterwards; it's an invaluable tool for helping you figure out why your program or your tests aren't working the way
you expect them to, and I recommend giving it a try.

(Although I admit that adding a quick `print()` statement is my first instinct at times, too.)

## Additional reading

The main notes I found when searching for guides on writing tests for Pygame
were a pair of comments by u/bitcraft on the r/pygame subreddit.
- [(1) unit testing for pygame](https://www.reddit.com/r/pygame/comments/5h177k/comment/dayky59)
- [(2) How are you guys unit testing your game(s)?](https://www.reddit.com/r/pygame/comments/jqujoa)

I found this post as well, and credit it for the `create_key_mock()` helper function,
although it doesn't offer much more insight than that.
- [StackOverflow post that suggested mocking pg.key.get_pressed()](https://stackoverflow.com/questions/52917479/unit-testing-in-python-pygame-for-key-get-pressed)

The remaining links are more general, but very helpful guides on how to use unittest and pytest.
- [VSCode's guide on how to set up unit testing in Python](https://code.visualstudio.com/docs/python/testing)
- [DataCamp's tutorial on how to use PyTest, fixtures, and parametrize](https://www.datacamp.com/tutorial/pytest-tutorial-a-hands-on-guide-to-unit-testing)
- [Practical unittest and pytest example, part 1](https://blog.miguelgrinberg.com/post/-how-to-write-unit-tests-in-python-part-1-fizz-buzz)
- [Practical unittest and pytest example, part 2](https://blog.miguelgrinberg.com/post/how-to-write-unit-tests-in-python-part-2-game-of-life)
- [Python docs on how to use unittest.mock](https://docs.python.org/3/library/unittest.mock.html)

While looking into ways to actually automate the running of my unit test suite, I found
[this tutorial on how to set up a GitHub action](https://medium.com/thelorry-product-tech-data/unit-testing-and-continues-integration-ci-in-github-action-for-python-programming-c8ad57fae3a1)
to run the test and reject failing code after any push to the GitHub repository,
but this seemed excessive for a personal project that I was going to
test on my own local setup, so I did not attempt this.

## Advance to the next article

I hope this article was able to help you set up tests for your own game, and perhaps
realize that doing so isn't as hard as you might have feared it was.

Let's return to developing our game in article #3.

TODO: edit in link to #3  
[>> Next: #3: Game state](/fgtutorial/2023/02/01/article3-game-state.html)
