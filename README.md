# Random Numbers

Sometimes you'll need to generate values without having an exact amount in mind. It's effectively impossible to generate truly random values on a computer, so different engineers have come up with extremely complicated and confusing mathematical processes to simulate randomness, and fortunately, that work is done for us within library methods we can utilize.

Say we want to create a game that relies on chance. We're in a dungeon with 100 steps, and you're competing with your fellow gamers to see which one of you can get to the top first. Using a 6-sided die, you roll to see how many steps to take each turn. For one or two, you lose progress and go down a single step. For three, four, and five, you go one step up. For six, you roll again, and go up that second roll's amount of steps. You also have a .01% random enemy encounter, which will knock you back down all the way to the first step. The full one hundred steps is a perfect run, resulting in some secret loot, but to progress through the dungeon, you need to make it at least sixty steps within one hundred turns.

How can we simulate this process? How can we determine the likelihood of making it to the sixty steps within our alloted amount of turns? For starters, we'll need to use Python's random number generator, which we can access with NumPy.

## Rand

The get a random value, use NumPy's "random" package, specifically the `rand` method.

```py
import numpy as np

np.random.rand() # Generates psuedo-random numbers
```

The random numbers generated are a large float value between zero and one:

```py
print(np.random.rand()) # 0.06021292255610289

print(np.random.rand()) # 0.1569081665407469

print(np.random.rand()) # 0.5003967580708776

print(np.random.rand()) # 0.3178761941543087

print(np.random.rand()) # 0.1308164031459278
```

Results will, obviously and by design, vary. The numbers are randomized based off a `seed`, which Python chooses automatically, but we can actually set it manually if we want to:

```py
np.random.seed(123)

print(np.random.rand()) # 0.6964691855978616

print(np.random.rand()) # 0.28613933495037946

np.random.seed(123)

print(np.random.rand()) # 0.6964691855978616

print(np.random.rand()) # 0.28613933495037946
```

As you can see, the numbers aren't actually random, they're procedurally generated, as they will generate the same response based on the starting seed, and will produce the same values when they have the same seed. This is why the numbers are considered "psuedo-random" instead of truly random. One might think it defeats the purpose of generating random numbers if you decide to generate the same random numbers each time, but this is useful if you need to replicate your results in a codebase that uses random values.

## Randint

If you want something a little cleaner than a big decimal value, you can use the `randint` method instead. The parameters are similar to the range function, but rather than returning a list of all numbers within the range, it selects a single value in that range - you guessed it, at random - and returns that value. Just like the range function, this method is non-inclusive, meaning it will stop at the second number, not after the second number:

```py
np.random.randint(1, 11) # Produces a random number between 1 and 10
```

```py
def flip_coin():
    if np.random.randint(0, 2) > 0:
        return "heads"
    return "tails"
    
print(flip_coin()) # tails

print(flip_coin()) # heads

def roll_die():
    return np.random.randint(1, 7)
    
print(roll_die()) # 4

print(roll_die()) # 3

print(roll_die()) # 4
```

Note again that because the seed was manually entered, you'll get the same results every time you run these randomly generated values.

## Random Walk

Back to the dungeon game, if every step you take is a random step, then the collection of random steps can be referred to as a `random walk`. Anytime an object or value increments from State A to State B through a series of uncertain values, this process is a random walk. Everything from the travel path of a molecule to the financial status of a gambler can be encompossed and simulated this way.

The fastest way to simulate a random walk is by tying a random step to a for loop. Think of the coin toss, we can toss the coin multiple times, and keep track of each of our flips with a dictionary:

```py
def coin_flip_chain(amt):
    flips = range(0, amt)
    results = {
        "heads": 0,
        "tails": 0
    }
    for flip in flips:
        if np.random.randint(0, 2) > 0:
            results['heads'] += 1
        else:
            results['tails'] += 1
    return results
    
print(coin_flip_chain(100))

# {'heads': 47, 'tails': 53}
```

But this is just a collection, not necessarily a walk. With the coin flip, a coin either is or isn't heads, or it is or isn't tails. This doesn't mean anything unless it's tied to some kind of progression. For a walk, we can utilize the die roll and track our progress in the dungeon:

```py
def dungeon_game():
    rolls = range(0, 100) # we roll the die 100 times
    step = 0 # track the current step
    path = {
        "rolls": [], # track the value of each roll
        "walk": [] # track our progression
    }
    for r in rolls:
        if np.random.randint(0, 1000) == 1:
            step = 0
            path['walk'].append(step)
            continue # 0.1% chance to fall back to the beginning
        
        roll = roll_die() # capture the roll value
        
        if roll <= 2: # if roll is 1 or 2, one step back
            step = max(step - 1, 0) # step will be 0 or step -1, whichever value is higher
            path['walk'].append(step)
            path['rolls'].append(roll)
        elif roll >= 3 and roll <= 5: # if roll is 3, 4, or 5, one step forward
            step += 1
            path['walk'].append(step)
            path['rolls'].append(roll)
        else:
            reroll = roll_die() # if roll is 6, roll again to determine steps
            step += reroll
            path['walk'].append(step)
            path['rolls'].append((roll, reroll)) # capture initial 6 and reroll as tuple
    return path
```

So for each roll, we utilize the roll_die() function we created earlier to consistently generate a random number between 1 and 6, taking our appropriate steps forward and back along the way. Our `path` dictionary tracks the value of each roll (including tuples to catch the reroll on 6's), as well as the actual step we were on for each roll along the way. The dictionary looks a little messy as is, but we can convert it into a data frame for better readability:

```py
import pandas as pd

dungeon_crawl = pd.DataFrame(dungeon_game())

#    rolls  walk
# 0      2     0
# 1      4     1
# 2      4     2
# 3      2     1
# 4      2     0
# ..   ...   ...
# 95     5    54
# 96     1    53
# 97     5    54
# 98     1    53
# 99     1    52

# [100 rows x 2 columns]
```

If you want to see all of the columns, you have to override the settings with

```py
pd.set_option("display.max_rows", None)
```

But you can still see from this abbreviated result that under the 123 seed, we didn't actually make it to step 60.
