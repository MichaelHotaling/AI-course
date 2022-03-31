---
layout: post
title: Solving Wordle Using Information Theory
author: Michael Hotaling]
category: [Blog]
tags: [blog, information_theory]
---

# Solving Wordle Using Information Theory

[Wordle](https://www.powerlanguage.co.uk/wordle/) has gone pretty viral in the last couple months. It's a very simple game where you have to guess a five letter word within six guesses. Each guess reveals some information about what letters are in the word and in what positions. The faster you get the word, the better your score. 

It's an interesting game to analyze from a programatic standpoint. Is there a strategy that can ensure that we guess the word of the day in as few guesses as possible? What is the best starting word? What's the worst starting word? 


```python
import itertools
import string
import requests
import datetime
import numpy as np
import pandas as pd
```

![Wordle](solvewordgamesusingmath.jpg)

## The Data

Wordle requires the player to input an actual five-letter words, so it forbids random assortment of letters like `prsmt` or `aeiou` which, theoretically would be good starting words. 

There's are two lists of words that you are allowed to enter in the game that are considered valid guesses. The pool of allowable entries is a little less that 13,000 words long, but these is another curated list that contains about 2,300 possible answers. These lists are actually present in the source code on the website in the order that they appear, so it is technically possible to just look up tomorrows answer, but that'd be cheating. 

We will pull the lists from the JS file and sort them to remove the order information. Unfortunately, there isn't a great way to pull a variable or array from a JavaScript file into Python, so some hacky string manipulation will have to do.


```python
#response = requests.get("https://www.nytimes.com/games/wordle/main.4d41d2be.js")
# Legacy JS

url = "https://web.archive.org/web/20220101000805js_/https://www.powerlanguage.co.uk/wordle/main.db1931a8.js"
response = requests.get(url)
data = response.text

#valid_entries = data.split(",Oa=[")[1].split("]")[0].replace('"',"").upper().split(",")
#wordle_answers = data.split("var Ma=[")[-1].split("]")[0].replace('"',"").upper().split(",")

wordle_answers = data.split("var Aa=[")[-1].split("]")[0].replace('"',"").upper().split(",")
valid_entries = data.split(",La=[")[-1].split("]")[0].replace('"',"").upper().split(",")
valid_entries = sorted(list(set(wordle_answers + valid_entries)))

wordle_answers = sorted(wordle_answers)

print(f"There are {len(wordle_answers)} possible answers in Wordle")
print(f"There are {len(valid_entries)} possible words that can be entered in Wordle")
```

    There are 2315 possible answers in Wordle
    There are 12972 possible words that can be entered in Wordle
    

## Game Mechanics

The game first starts with an empty 5x6 grid. We have to choose a starting word to begin the game. After we hit enter with our first guess, hidden information is revealed about the secret word using the letters of our submission. All five of our letters will change color based on if the letters are present in the hidden word. 

![ocean](ocean.JPG)

If our letter turns green, it means that the letter is present in the hidden word and in the correct position. Yellow means that the letter is in the word, but in the incorrect position. Gray means that the letter is not in the hidden word at all.

To begin with our analysis, we can create a function that returns the color sequence for a particular combination of words. We can start by generating a list of n length with each of the spaces representing a white space. The function will then check the position of each of the letters between two words of the same length. If the letters match up, it will overwrite the white space with a green space. If the letter is present in the word, but the positions don't match, it will overwrite the white space with a yellow space. 


```python
def indices(arr:list, value:str) -> list:
    """A function that takes in a list and a string and returns a list of index positions within the input list
    
    Args:
        arr (list): List to pull the index values from
        item (str): String to search for in the input list

    Returns:
        list: list of index positions of item

    """
    return [ind for ind, x in enumerate(arr) if x == value]

def color_sequence(guess:str, answer:str, emoji=True) -> str:
    """A function that takes returns a color sequence of a guess based on a predetermined answer
    
    Args:
        guess  (str): 
            The guess for the game
        answer (str):
            The answer for the game
        emoji (bool): True
            Returns a string of green, yellow and white square emojies
            False will return letters G,Y, and W

    Returns:
        str: Color sequence 

    """
    guess = list(guess)
    answer = list(answer)
    
    # Start with all spaces absent
    colors = ["W"] * (len(answer))
    
    # If correct
    for ind, (g_letter, a_letter) in enumerate(zip(guess, answer)):
        if g_letter == a_letter:
            colors[ind] = "G"
            guess[ind] = ""
            answer[ind] = ""
    
    # If present
    for ind, (g_letter, a_letter) in enumerate(zip(guess, answer)):
        if g_letter != a_letter and g_letter in answer and g_letter != "":
            g = indices(guess, g_letter)
            a = indices(answer, g_letter)
            if not any(i in g for i in a):
                colors[ind] = "Y"
                answer[answer.index(g_letter)] = ""
                
    colors = "".join(colors)
    
    if emoji:
        colors = (colors
                  .replace("G","🟩")
                  .replace("Y","🟨")
                  .replace("W","⬜"))
    
    return colors
```

## All Possible Color Sequences

We have a total of three different colors to choose from with a total of five positions. 
This yields a total of $3^5$ combinations, or 243 total combos. However, there are a few combinations that can't be returns like 🟩🟩🟩🟩🟨, 🟩🟩🟩🟨🟩, 🟩🟩🟨🟩🟩 and so on.


```python
combinations = ["".join(i) for i in itertools.product("⬜🟨🟩", repeat = 5)]
remove = ["".join(i) for i in itertools.product("🟩🟨", repeat = 5)]
remove = [i for i in remove if len(indices(i,"🟩")) > 3]

combinations = set(combinations)-set(remove[1::])

for ind, i in enumerate(sorted(combinations), start = 1):
    print(i, end =" ")
    if ind % 7 == 0:
        print("")
```

    ⬜⬜⬜⬜⬜ ⬜⬜⬜⬜🟨 ⬜⬜⬜⬜🟩 ⬜⬜⬜🟨⬜ ⬜⬜⬜🟨🟨 ⬜⬜⬜🟨🟩 ⬜⬜⬜🟩⬜ 
    ⬜⬜⬜🟩🟨 ⬜⬜⬜🟩🟩 ⬜⬜🟨⬜⬜ ⬜⬜🟨⬜🟨 ⬜⬜🟨⬜🟩 ⬜⬜🟨🟨⬜ ⬜⬜🟨🟨🟨 
    ⬜⬜🟨🟨🟩 ⬜⬜🟨🟩⬜ ⬜⬜🟨🟩🟨 ⬜⬜🟨🟩🟩 ⬜⬜🟩⬜⬜ ⬜⬜🟩⬜🟨 ⬜⬜🟩⬜🟩 
    ⬜⬜🟩🟨⬜ ⬜⬜🟩🟨🟨 ⬜⬜🟩🟨🟩 ⬜⬜🟩🟩⬜ ⬜⬜🟩🟩🟨 ⬜⬜🟩🟩🟩 ⬜🟨⬜⬜⬜ 
    ⬜🟨⬜⬜🟨 ⬜🟨⬜⬜🟩 ⬜🟨⬜🟨⬜ ⬜🟨⬜🟨🟨 ⬜🟨⬜🟨🟩 ⬜🟨⬜🟩⬜ ⬜🟨⬜🟩🟨 
    ⬜🟨⬜🟩🟩 ⬜🟨🟨⬜⬜ ⬜🟨🟨⬜🟨 ⬜🟨🟨⬜🟩 ⬜🟨🟨🟨⬜ ⬜🟨🟨🟨🟨 ⬜🟨🟨🟨🟩 
    ⬜🟨🟨🟩⬜ ⬜🟨🟨🟩🟨 ⬜🟨🟨🟩🟩 ⬜🟨🟩⬜⬜ ⬜🟨🟩⬜🟨 ⬜🟨🟩⬜🟩 ⬜🟨🟩🟨⬜ 
    ⬜🟨🟩🟨🟨 ⬜🟨🟩🟨🟩 ⬜🟨🟩🟩⬜ ⬜🟨🟩🟩🟨 ⬜🟨🟩🟩🟩 ⬜🟩⬜⬜⬜ ⬜🟩⬜⬜🟨 
    ⬜🟩⬜⬜🟩 ⬜🟩⬜🟨⬜ ⬜🟩⬜🟨🟨 ⬜🟩⬜🟨🟩 ⬜🟩⬜🟩⬜ ⬜🟩⬜🟩🟨 ⬜🟩⬜🟩🟩 
    ⬜🟩🟨⬜⬜ ⬜🟩🟨⬜🟨 ⬜🟩🟨⬜🟩 ⬜🟩🟨🟨⬜ ⬜🟩🟨🟨🟨 ⬜🟩🟨🟨🟩 ⬜🟩🟨🟩⬜ 
    ⬜🟩🟨🟩🟨 ⬜🟩🟨🟩🟩 ⬜🟩🟩⬜⬜ ⬜🟩🟩⬜🟨 ⬜🟩🟩⬜🟩 ⬜🟩🟩🟨⬜ ⬜🟩🟩🟨🟨 
    ⬜🟩🟩🟨🟩 ⬜🟩🟩🟩⬜ ⬜🟩🟩🟩🟨 ⬜🟩🟩🟩🟩 🟨⬜⬜⬜⬜ 🟨⬜⬜⬜🟨 🟨⬜⬜⬜🟩 
    🟨⬜⬜🟨⬜ 🟨⬜⬜🟨🟨 🟨⬜⬜🟨🟩 🟨⬜⬜🟩⬜ 🟨⬜⬜🟩🟨 🟨⬜⬜🟩🟩 🟨⬜🟨⬜⬜ 
    🟨⬜🟨⬜🟨 🟨⬜🟨⬜🟩 🟨⬜🟨🟨⬜ 🟨⬜🟨🟨🟨 🟨⬜🟨🟨🟩 🟨⬜🟨🟩⬜ 🟨⬜🟨🟩🟨 
    🟨⬜🟨🟩🟩 🟨⬜🟩⬜⬜ 🟨⬜🟩⬜🟨 🟨⬜🟩⬜🟩 🟨⬜🟩🟨⬜ 🟨⬜🟩🟨🟨 🟨⬜🟩🟨🟩 
    🟨⬜🟩🟩⬜ 🟨⬜🟩🟩🟨 🟨⬜🟩🟩🟩 🟨🟨⬜⬜⬜ 🟨🟨⬜⬜🟨 🟨🟨⬜⬜🟩 🟨🟨⬜🟨⬜ 
    🟨🟨⬜🟨🟨 🟨🟨⬜🟨🟩 🟨🟨⬜🟩⬜ 🟨🟨⬜🟩🟨 🟨🟨⬜🟩🟩 🟨🟨🟨⬜⬜ 🟨🟨🟨⬜🟨 
    🟨🟨🟨⬜🟩 🟨🟨🟨🟨⬜ 🟨🟨🟨🟨🟨 🟨🟨🟨🟨🟩 🟨🟨🟨🟩⬜ 🟨🟨🟨🟩🟨 🟨🟨🟨🟩🟩 
    🟨🟨🟩⬜⬜ 🟨🟨🟩⬜🟨 🟨🟨🟩⬜🟩 🟨🟨🟩🟨⬜ 🟨🟨🟩🟨🟨 🟨🟨🟩🟨🟩 🟨🟨🟩🟩⬜ 
    🟨🟨🟩🟩🟨 🟨🟨🟩🟩🟩 🟨🟩⬜⬜⬜ 🟨🟩⬜⬜🟨 🟨🟩⬜⬜🟩 🟨🟩⬜🟨⬜ 🟨🟩⬜🟨🟨 
    🟨🟩⬜🟨🟩 🟨🟩⬜🟩⬜ 🟨🟩⬜🟩🟨 🟨🟩⬜🟩🟩 🟨🟩🟨⬜⬜ 🟨🟩🟨⬜🟨 🟨🟩🟨⬜🟩 
    🟨🟩🟨🟨⬜ 🟨🟩🟨🟨🟨 🟨🟩🟨🟨🟩 🟨🟩🟨🟩⬜ 🟨🟩🟨🟩🟨 🟨🟩🟨🟩🟩 🟨🟩🟩⬜⬜ 
    🟨🟩🟩⬜🟨 🟨🟩🟩⬜🟩 🟨🟩🟩🟨⬜ 🟨🟩🟩🟨🟨 🟨🟩🟩🟨🟩 🟨🟩🟩🟩⬜ 🟨🟩🟩🟩🟨 
    🟩⬜⬜⬜⬜ 🟩⬜⬜⬜🟨 🟩⬜⬜⬜🟩 🟩⬜⬜🟨⬜ 🟩⬜⬜🟨🟨 🟩⬜⬜🟨🟩 🟩⬜⬜🟩⬜ 
    🟩⬜⬜🟩🟨 🟩⬜⬜🟩🟩 🟩⬜🟨⬜⬜ 🟩⬜🟨⬜🟨 🟩⬜🟨⬜🟩 🟩⬜🟨🟨⬜ 🟩⬜🟨🟨🟨 
    🟩⬜🟨🟨🟩 🟩⬜🟨🟩⬜ 🟩⬜🟨🟩🟨 🟩⬜🟨🟩🟩 🟩⬜🟩⬜⬜ 🟩⬜🟩⬜🟨 🟩⬜🟩⬜🟩 
    🟩⬜🟩🟨⬜ 🟩⬜🟩🟨🟨 🟩⬜🟩🟨🟩 🟩⬜🟩🟩⬜ 🟩⬜🟩🟩🟨 🟩⬜🟩🟩🟩 🟩🟨⬜⬜⬜ 
    🟩🟨⬜⬜🟨 🟩🟨⬜⬜🟩 🟩🟨⬜🟨⬜ 🟩🟨⬜🟨🟨 🟩🟨⬜🟨🟩 🟩🟨⬜🟩⬜ 🟩🟨⬜🟩🟨 
    🟩🟨⬜🟩🟩 🟩🟨🟨⬜⬜ 🟩🟨🟨⬜🟨 🟩🟨🟨⬜🟩 🟩🟨🟨🟨⬜ 🟩🟨🟨🟨🟨 🟩🟨🟨🟨🟩 
    🟩🟨🟨🟩⬜ 🟩🟨🟨🟩🟨 🟩🟨🟨🟩🟩 🟩🟨🟩⬜⬜ 🟩🟨🟩⬜🟨 🟩🟨🟩⬜🟩 🟩🟨🟩🟨⬜ 
    🟩🟨🟩🟨🟨 🟩🟨🟩🟨🟩 🟩🟨🟩🟩⬜ 🟩🟨🟩🟩🟨 🟩🟩⬜⬜⬜ 🟩🟩⬜⬜🟨 🟩🟩⬜⬜🟩 
    🟩🟩⬜🟨⬜ 🟩🟩⬜🟨🟨 🟩🟩⬜🟨🟩 🟩🟩⬜🟩⬜ 🟩🟩⬜🟩🟨 🟩🟩⬜🟩🟩 🟩🟩🟨⬜⬜ 
    🟩🟩🟨⬜🟨 🟩🟩🟨⬜🟩 🟩🟩🟨🟨⬜ 🟩🟩🟨🟨🟨 🟩🟩🟨🟨🟩 🟩🟩🟨🟩⬜ 🟩🟩🟨🟩🟨 
    🟩🟩🟩⬜⬜ 🟩🟩🟩⬜🟨 🟩🟩🟩⬜🟩 🟩🟩🟩🟨⬜ 🟩🟩🟩🟨🟨 🟩🟩🟩🟩⬜ 🟩🟩🟩🟩🟩 
    

We will also want to create a function that takes in a guess and a color sequence to find the words in the corpus that match the sequence. 


```python
def get_words_using_sequence(input_word, sequence, corpus):
    return [word for word in corpus if color_sequence(input_word, word) == sequence]
```

There is a special mechanic for duplicate letters. If a guess contains duplicate letters but the answer only has one of the letters, the first letter in the guess will appear yellow and the second will appear white. If there are two duplicate characters and they are both misplaced, they will both appear yellow. The same goes for combinations of matching and present characters. We can verify our function returns the correct values for the the following guess `HELLO` when the answer varies in the number of Ls. 


```python
print("Game Mechanics for Duplicate Letters")
print(f"      H  E  L L  O")
print("HYDRO", color_sequence(guess = "HELLO", answer = "HYDRO"), "No L's")
print("HAZEL", color_sequence(guess = "HELLO", answer = "HAZEL"), "One L: misplaced")
print("GOLEM", color_sequence(guess = "HELLO", answer = "GOLEM"), "One L: correct")
print("NOBLE", color_sequence(guess = "HELLO", answer = "NOBLE"), "One L: correct")
print("LLAMA", color_sequence(guess = "HELLO", answer = "LLAMA"), "Two L's: both misplaced")
print("ALLEY", color_sequence(guess = "HELLO", answer = "ALLEY"), "Two L's: one correct, one misplaced")
print("TROLL", color_sequence(guess = "HELLO", answer = "TROLL"), "Two L's: one correct, one misplaced")
print("JOLLY", color_sequence(guess = "HELLO", answer = "JOLLY"), "Two L's: both correct")
```

    Game Mechanics for Duplicate Letters
          H  E  L L  O
    HYDRO 🟩⬜⬜⬜🟩 No L's
    HAZEL 🟩🟨🟨⬜⬜ One L: misplaced
    GOLEM ⬜🟨🟩⬜🟨 One L: correct
    NOBLE ⬜🟨⬜🟩🟨 One L: correct
    LLAMA ⬜⬜🟨🟨⬜ Two L's: both misplaced
    ALLEY ⬜🟨🟩🟨⬜ Two L's: one correct, one misplaced
    TROLL ⬜⬜🟨🟩🟨 Two L's: one correct, one misplaced
    JOLLY ⬜⬜🟩🟩🟨 Two L's: both correct
    

## Calculating the Entropy

Now, we can generate a list of words that match the sequence we recieve from the game. This won't always be very helpful. We may get stuck in a loop of possible answers, which a common way to fail the game. For example, if we use `SPARK` and get the 🟩⬜🟩🟩⬜ sequence, we get back several possible answers. 


```python
reduced_corpus = get_words_using_sequence('SPARK', "🟩⬜🟩🟩⬜", wordle_answers)
print(reduced_corpus)
```

    ['SCARE', 'SCARF', 'SCARY', 'SHARD', 'SHARE', 'SMART', 'SNARE', 'SNARL', 'STARE', 'START', 'SWARM']
    

Instead of trying each of the possible answers, we can use a turn to reduce the amount of possible answers. For example, `THYME` would be a great word to use in this case. All but two of the possible answers will have unique sequences which we can use to narrow down the answer much faster in a much more repeatable way.


```python
print("       T H  Y M  E")
for word in reduced_corpus:
    print(word, color_sequence(guess = "THYME", answer = word))
```

           T H  Y M  E
    SCARE ⬜⬜⬜⬜🟩
    SCARF ⬜⬜⬜⬜⬜
    SCARY ⬜⬜🟨⬜⬜
    SHARD ⬜🟩⬜⬜⬜
    SHARE ⬜🟩⬜⬜🟩
    SMART 🟨⬜⬜🟨⬜
    SNARE ⬜⬜⬜⬜🟩
    SNARL ⬜⬜⬜⬜⬜
    STARE 🟨⬜⬜⬜🟩
    START 🟨⬜⬜⬜⬜
    SWARM ⬜⬜⬜🟨⬜
    

We can use this principle on all the the available words to find which one fits the best. Since we have the complete list of possible entries for Wordle, we can now iterate through the list, finding how many different combinations of sequences there are for each word and counting how many times each sequence occurs. Using that information, we can calculate the probability of each sequence occuring and also calculate the entropy that each sequence returns. 

Entropy in this case is a measure of how far we reduce the possible pool of answers. It can be calculated by taking the log-base 2 of the percentage of answers given that are possible for a given sequence. 

$$
I(E) = \log_2\left(\frac{1}{p(E)}\right)
$$

For example, if we use the word `PIZZA` as a starting word, which would instictually be a bad starting word due to the double Zs, and get ⬜⬜⬜🟩⬜, we reduce our possible pool of answers from 12,972 to 25. That provides an entropy value of 9.02 bits. 

> `BONZE`,
 `BOOZE`,
 `BOOZY`,
 `CLOZE`,
 `COOZE`,
 `CROZE`,
 `DOOZY`,
 `FEEZE`,
 `FORZE`,
 `FROZE`,
 `FURZE`,
 `FURZY`,
 `GLOZE`,
 `GONZO`,
 `HEEZE`,
 `JEEZE`,
 `KUDZU`,
 `LEEZE`,
 `NEEZE`,
 `NUDZH`,
 `TOUZE`,
 `TOUZY`,
 `TOWZE`,
 `TOWZY`,
 `WOOZY`


If instead, we got a more likely response of ⬜⬜⬜⬜⬜, we would reduce our pool of answers by a little over half, from 12,972 to 4,145. 



```python
def count_of_combos(guess:str, answers:list, sort = False, probability = True, verbose = False) -> dict:
    """A function that takes in a list and a string and returns a list of index positions within the input list
    
    Args:
        guess (str): 
            A string to compare all the answers against
            
        answers (list): 
            A list containing all possible combinations of answers
        
        sort (bool): False
            Sorts the return dictionary by value
            
        probability (bool): True
            Converts the values in the dictionary to a probability

    Returns:
        list: 
            list of index positions of item

    """
    results_dict = {}

    increment_value = 1
    
    if probability is True:
        increment_value /= len(answers)
        
    for ans in answers:
        seq = color_sequence(guess, ans)

        if seq in results_dict:
            results_dict[seq] += increment_value
        else:
            results_dict[seq] = increment_value
    if sort:
        results_dict = dict(sorted(results_dict.items(), key=lambda item: item[1], reverse=True))

    return results_dict
```


```python
word = "PIZZA"
word_combinations = count_of_combos(word, valid_entries, probability=True, sort = True)

print(f"Data for Each Color Combinations for the Word {word}\n")
print(f"{'Sequence':^12}  {'Count':<8} {'%':<5} {'Bits'}     "*3)
for ind, (key, value) in enumerate(word_combinations.items(), start = 1):
    print(f"{key}| {round(value*len(valid_entries)):<5}|{f'{value*100:.2f}%':>6s}| {f'{np.log2(1/value):.2f}':>5s}", end = "    ")
    if ind % 3 == 0:
        print()
    
```

    Data for Each Color Combinations for the Word PIZZA
    
      Sequence    Count    %     Bits       Sequence    Count    %     Bits       Sequence    Count    %     Bits     
    ⬜⬜⬜⬜⬜| 4145 |31.95%|  1.65    ⬜⬜⬜⬜🟨| 3167 |24.41%|  2.03    ⬜🟨⬜⬜⬜| 1190 | 9.17%|  3.45    
    ⬜🟩⬜⬜⬜| 956  | 7.37%|  3.76    ⬜🟨⬜⬜🟨| 576  | 4.44%|  4.49    🟨⬜⬜⬜⬜| 442  | 3.41%|  4.88    
    ⬜⬜⬜⬜🟩| 408  | 3.15%|  4.99    🟩⬜⬜⬜⬜| 358  | 2.76%|  5.18    🟨⬜⬜⬜🟨| 309  | 2.38%|  5.39    
    🟩⬜⬜⬜🟨| 219  | 1.69%|  5.89    ⬜🟩⬜⬜🟨| 136  | 1.05%|  6.58    🟨🟨⬜⬜⬜| 126  | 0.97%|  6.69    
    ⬜🟨⬜⬜🟩| 84   | 0.65%|  7.27    🟩🟩⬜⬜⬜| 81   | 0.62%|  7.32    🟩🟨⬜⬜⬜| 79   | 0.61%|  7.36    
    ⬜🟩⬜⬜🟩| 62   | 0.48%|  7.71    🟨🟩⬜⬜⬜| 58   | 0.45%|  7.81    🟩⬜⬜⬜🟩| 44   | 0.34%|  8.20    
    ⬜⬜🟨⬜⬜| 42   | 0.32%|  8.27    ⬜⬜🟩⬜🟨| 41   | 0.32%|  8.31    🟨🟨⬜⬜🟨| 37   | 0.29%|  8.45    
    ⬜⬜🟩⬜⬜| 36   | 0.28%|  8.49    ⬜⬜🟨⬜🟨| 35   | 0.27%|  8.53    🟩🟨⬜⬜🟨| 34   | 0.26%|  8.58    
    🟨⬜⬜⬜🟩| 30   | 0.23%|  8.76    ⬜⬜⬜🟩🟨| 26   | 0.20%|  8.96    ⬜⬜⬜🟩⬜| 25   | 0.19%|  9.02    
    🟩🟩⬜⬜🟨| 20   | 0.15%|  9.34    ⬜🟩🟨⬜⬜| 17   | 0.13%|  9.58    ⬜🟨🟨⬜⬜| 15   | 0.12%|  9.76    
    ⬜🟩🟩⬜⬜| 14   | 0.11%|  9.86    ⬜🟨🟨⬜🟨| 13   | 0.10%|  9.96    ⬜⬜⬜🟩🟩| 12   | 0.09%| 10.08    
    ⬜⬜🟩🟩⬜| 9    | 0.07%| 10.49    ⬜🟨🟩⬜⬜| 8    | 0.06%| 10.66    ⬜🟨⬜🟩🟨| 7    | 0.05%| 10.86    
    ⬜⬜🟨⬜🟩| 7    | 0.05%| 10.86    ⬜🟩🟩🟩⬜| 6    | 0.05%| 11.08    ⬜🟨⬜🟩⬜| 6    | 0.05%| 11.08    
    🟩🟩⬜⬜🟩| 6    | 0.05%| 11.08    🟨🟨⬜⬜🟩| 5    | 0.04%| 11.34    ⬜🟩🟩⬜🟨| 4    | 0.03%| 11.66    
    ⬜🟨🟨🟩⬜| 4    | 0.03%| 11.66    ⬜🟩⬜🟩⬜| 4    | 0.03%| 11.66    ⬜🟨🟩⬜🟨| 4    | 0.03%| 11.66    
    🟩🟨⬜⬜🟩| 4    | 0.03%| 11.66    🟨⬜🟨⬜🟨| 4    | 0.03%| 11.66    ⬜⬜🟩🟩🟩| 3    | 0.02%| 12.08    
    ⬜⬜🟩🟩🟨| 3    | 0.02%| 12.08    🟨🟩⬜⬜🟩| 3    | 0.02%| 12.08    ⬜🟩🟨⬜🟨| 3    | 0.02%| 12.08    
    ⬜⬜🟨🟩🟨| 2    | 0.02%| 12.66    ⬜🟨⬜🟩🟩| 2    | 0.02%| 12.66    🟨🟩⬜⬜🟨| 2    | 0.02%| 12.66    
    ⬜🟨🟩🟨⬜| 2    | 0.02%| 12.66    🟩🟨⬜🟩⬜| 2    | 0.02%| 12.66    🟩🟩🟩⬜⬜| 2    | 0.02%| 12.66    
    ⬜🟨🟨⬜🟩| 2    | 0.02%| 12.66    🟨🟩🟨⬜⬜| 2    | 0.02%| 12.66    ⬜🟩🟩🟨⬜| 2    | 0.02%| 12.66    
    🟨⬜🟨⬜🟩| 2    | 0.02%| 12.66    🟨🟨🟨⬜🟨| 1    | 0.01%| 13.66    ⬜🟩⬜🟩🟨| 1    | 0.01%| 13.66    
    ⬜🟨🟩🟨🟨| 1    | 0.01%| 13.66    ⬜🟨🟩🟩🟨| 1    | 0.01%| 13.66    ⬜🟩⬜🟩🟩| 1    | 0.01%| 13.66    
    🟩⬜⬜🟩🟨| 1    | 0.01%| 13.66    🟩🟨🟨🟩⬜| 1    | 0.01%| 13.66    🟩🟩⬜🟩⬜| 1    | 0.01%| 13.66    
    🟩🟩🟩🟩🟩| 1    | 0.01%| 13.66    🟩⬜⬜🟩🟩| 1    | 0.01%| 13.66    🟩⬜🟨⬜⬜| 1    | 0.01%| 13.66    
    🟩⬜⬜🟩⬜| 1    | 0.01%| 13.66    🟩⬜🟩🟩⬜| 1    | 0.01%| 13.66    🟩⬜🟩⬜⬜| 1    | 0.01%| 13.66    
    🟩⬜🟨🟩🟨| 1    | 0.01%| 13.66    ⬜⬜🟨🟩⬜| 1    | 0.01%| 13.66    🟨⬜⬜🟩🟩| 1    | 0.01%| 13.66    
    🟨⬜🟨🟩🟨| 1    | 0.01%| 13.66    🟨🟨🟨⬜⬜| 1    | 0.01%| 13.66    🟨🟨🟩⬜⬜| 1    | 0.01%| 13.66    
    ⬜⬜🟨🟩🟩| 1    | 0.01%| 13.66    ⬜⬜🟩🟨🟨| 1    | 0.01%| 13.66    ⬜⬜🟩🟨⬜| 1    | 0.01%| 13.66    
    ⬜🟩🟨⬜🟩| 1    | 0.01%| 13.66    🟨⬜🟨⬜⬜| 1    | 0.01%| 13.66    

We can now take the weight each combinations entropy by the probability of it occuring to get the entropy for each word and summing them all together to get the total entropy calculated for each word. 

$$H(X) = \sum {p} \log_2(\frac{1}{p}) $$


```python
def sum_entropy_calculation(combinations):
    entropy = 0
    for key, value in combinations.items():
        entropy += value * (-1 * np.log2(value))
    return entropy
```


```python
print(f"The calculated entropy for the {word} is {sum_entropy_calculation(word_combinations):.2f} Sh")
```

    The calculated entropy for the PIZZA is 3.27 Sh
    

Now that we can calculate the entropy for one word, we can calculate the entropy for all of them by iterating over the list of words in the corpus.


```python
def entropy_df(inputs, corpus):
    entropy_dict = {}
    for i in inputs:
        combs = count_of_combos(i, corpus)
        entropy_dict[i] = sum_entropy_calculation(combs)
    df = pd.DataFrame([[key, val] for key, val in entropy_dict.items()], 
                      columns = ['WORD','ENTROPY']).sort_values("ENTROPY", ascending=False)
    df.index = df['WORD']
    df.drop(['WORD'], axis = 1, inplace=True)
    return df
```


```python
df = entropy_df(valid_entries, wordle_answers)
```

Here are the top 10 starting words based on entropy.


```python
df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>SOARE</th>
      <td>5.885960</td>
    </tr>
    <tr>
      <th>ROATE</th>
      <td>5.882779</td>
    </tr>
    <tr>
      <th>RAISE</th>
      <td>5.877910</td>
    </tr>
    <tr>
      <th>RAILE</th>
      <td>5.865710</td>
    </tr>
    <tr>
      <th>REAST</th>
      <td>5.865457</td>
    </tr>
    <tr>
      <th>SLATE</th>
      <td>5.855775</td>
    </tr>
    <tr>
      <th>CRATE</th>
      <td>5.834874</td>
    </tr>
    <tr>
      <th>SALET</th>
      <td>5.834582</td>
    </tr>
    <tr>
      <th>IRATE</th>
      <td>5.831397</td>
    </tr>
    <tr>
      <th>TRACE</th>
      <td>5.830549</td>
    </tr>
  </tbody>
</table>
</div>



and here are the 10 worst starting words. Unsuprisingly, many of them have duplicate and triplicate letters.


```python
df.tail(10).sort_values('ENTROPY')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>QAJAQ</th>
      <td>1.891836</td>
    </tr>
    <tr>
      <th>JUJUS</th>
      <td>2.038309</td>
    </tr>
    <tr>
      <th>IMMIX</th>
      <td>2.052600</td>
    </tr>
    <tr>
      <th>XYLYL</th>
      <td>2.192001</td>
    </tr>
    <tr>
      <th>YUKKY</th>
      <td>2.205266</td>
    </tr>
    <tr>
      <th>PZAZZ</th>
      <td>2.214171</td>
    </tr>
    <tr>
      <th>JAFFA</th>
      <td>2.234170</td>
    </tr>
    <tr>
      <th>FUFFY</th>
      <td>2.243640</td>
    </tr>
    <tr>
      <th>KUDZU</th>
      <td>2.264022</td>
    </tr>
    <tr>
      <th>GYPPY</th>
      <td>2.285137</td>
    </tr>
  </tbody>
</table>
</div>



## Follow Up Guesses
Since we know the best starting word is `SOARE`, we can calculate what the next best word will be based on the sequence we get from `SOARE` The sequences below and their associated word yield the best entropy for the second term. Most of the time, the answer can be achieved in 3 guessses. 


```python
word = "SOARE"

ind = 0
d = count_of_combos(word, wordle_answers, probability=True)

for sequence in sorted(d.keys()):

    corpus = wordle_answers
    corpus = get_words_using_sequence(word, sequence, corpus)
    
    super_dict = {}
    for answer in wordle_answers:
        combs = count_of_combos(answer, corpus)
        super_dict[answer] = sum_entropy_calculation(combs)
        
    df = pd.DataFrame([[key, val] for key, val in super_dict.items()], columns = ['WORD','ENTROPY'])
    df['IN_CORPUS'] = [1 if word in corpus else 0 for word in df['WORD']]
    df = df.sort_values(['ENTROPY','IN_CORPUS'], ascending=False)
    df.index = df['WORD']
    df = df[[False if entropy == 1 and in_corpus == 0 else True for entropy, in_corpus in zip(df['ENTROPY'], df['IN_CORPUS'])]]    
    df = df[[False if entropy == 0  and in_corpus == 0 else True for entropy, in_corpus in zip(df['ENTROPY'], df['IN_CORPUS'])]]    
    df = df.reset_index(drop=True)    
    
    print(f'{sequence} {df.iloc[0]["WORD"]}: {df.iloc[0]["ENTROPY"]:.2f}', end = "  ")
    ind += 1
    if ind % 4 == 0:
        print()

```

    ⬜⬜⬜⬜⬜ LUNCH: 5.24  ⬜⬜⬜⬜🟨 BETEL: 5.02  ⬜⬜⬜⬜🟩 GUILT: 5.07  ⬜⬜⬜🟨⬜ GLINT: 4.66  
    ⬜⬜⬜🟨🟨 DETER: 4.40  ⬜⬜⬜🟨🟩 TRUMP: 3.81  ⬜⬜⬜🟩⬜ TOUCH: 3.55  ⬜⬜⬜🟩🟨 BEEFY: 3.32  
    ⬜⬜⬜🟩🟩 THERE: 2.00  ⬜⬜🟨⬜⬜ MINTY: 4.87  ⬜⬜🟨⬜🟨 CANAL: 4.59  ⬜⬜🟨⬜🟩 LAUGH: 3.87  
    ⬜⬜🟨🟨⬜ RADAR: 4.21  ⬜⬜🟨🟨🟨 BETEL: 3.59  ⬜⬜🟨🟨🟩 CAROL: 2.81  ⬜⬜🟨🟩⬜ MARCH: 3.10  
    ⬜⬜🟨🟩🟨 ALTAR: 2.32  ⬜⬜🟨🟩🟩 AFIRE: 1.00  ⬜⬜🟩⬜⬜ CLINK: 4.60  ⬜⬜🟩⬜🟨 DEPTH: 3.75  
    ⬜⬜🟩⬜🟩 CLUNG: 3.12  ⬜⬜🟩🟨⬜ GLINT: 4.02  ⬜⬜🟩🟨🟨 ADMIT: 2.00  ⬜⬜🟩🟨🟩 DITCH: 2.97  
    ⬜⬜🟩🟩⬜ AUDIT: 3.39  ⬜⬜🟩🟩🟨 HANDY: 2.73  ⬜⬜🟩🟩🟩 BAGEL: 2.00  ⬜🟨⬜⬜⬜ GLINT: 4.81  
    ⬜🟨⬜⬜🟨 DEMON: 3.91  ⬜🟨⬜⬜🟩 CLINK: 3.78  ⬜🟨⬜🟨⬜ TUNIC: 3.95  ⬜🟨⬜🟨🟨 DETER: 3.03  
    ⬜🟨⬜🟨🟩 PINTO: 2.69  ⬜🟨⬜🟩⬜ CYNIC: 3.00  ⬜🟨⬜🟩🟨 METRO: 1.58  ⬜🟨⬜🟩🟩 CHORE: 1.00  
    ⬜🟨🟨⬜⬜ BLUNT: 4.28  ⬜🟨🟨⬜🟨 CAMEO: 2.00  ⬜🟨🟨⬜🟩 ABLED: 2.75  ⬜🟨🟨🟨⬜ BARON: 3.63  
    ⬜🟨🟨🟩⬜ ABACK: 2.58  ⬜🟨🟨🟩🟨 OPERA: 0.00  ⬜🟨🟨🟩🟩 ADORE: 0.00  ⬜🟨🟩⬜⬜ PIANO: 0.00  
    ⬜🟨🟩⬜🟩 OVATE: 0.00  ⬜🟨🟩🟨⬜ BRAVO: 0.00  ⬜🟨🟩🟩⬜ OVARY: 0.00  ⬜🟩⬜⬜⬜ LUNCH: 4.24  
    ⬜🟩⬜⬜🟨 MONTH: 3.43  ⬜🟩⬜⬜🟩 CLOUD: 3.32  ⬜🟩⬜🟨⬜ MUNCH: 3.71  ⬜🟩⬜🟨🟨 WATCH: 2.40  
    ⬜🟩⬜🟨🟩 FIGHT: 2.95  ⬜🟩⬜🟩⬜ ADULT: 2.58  ⬜🟩🟨⬜⬜ DELTA: 3.42  ⬜🟩🟨🟨⬜ BALMY: 3.00  
    ⬜🟩🟨🟩⬜ COBRA: 0.00  ⬜🟩🟩⬜⬜ LOAMY: 2.32  ⬜🟩🟩🟨⬜ ROACH: 0.00  ⬜🟩🟩🟩⬜ BOARD: 1.00  
    🟨⬜⬜⬜⬜ MIGHT: 4.25  🟨⬜⬜⬜🟨 BUILT: 3.62  🟨⬜⬜⬜🟩 INEPT: 3.00  🟨⬜⬜🟨⬜ TRUCK: 3.62  
    🟨⬜⬜🟨🟨 WITCH: 3.27  🟨⬜⬜🟨🟩 COUNT: 2.81  🟨⬜⬜🟩⬜ USURP: 0.00  🟨⬜🟨⬜⬜ UNITY: 3.90  
    🟨⬜🟨⬜🟨 ANKLE: 2.32  🟨⬜🟨⬜🟩 THUMP: 3.14  🟨⬜🟨🟨⬜ HARSH: 2.00  🟨⬜🟨🟨🟩 ARISE: 1.58  
    🟨⬜🟩⬜⬜ GULCH: 3.32  🟨⬜🟩⬜🟨 LEFTY: 2.32  🟨⬜🟩⬜🟩 BELCH: 2.81  🟨⬜🟩🟨⬜ PATCH: 2.52  
    🟨⬜🟩🟨🟩 ERASE: 0.00  🟨🟨⬜⬜⬜ ALONG: 2.81  🟨🟨⬜⬜🟨 ONSET: 1.58  🟨🟨⬜⬜🟩 BATCH: 2.32  
    🟨🟨⬜🟨⬜ DIGIT: 2.32  🟨🟨⬜🟨🟨 VERSO: 0.00  🟨🟨⬜🟨🟩 PROSE: 0.00  🟨🟨🟨⬜⬜ ASCOT: 1.58  
    🟨🟨🟨🟨⬜ ARSON: 0.00  🟨🟨🟨🟨🟩 AROSE: 0.00  🟨🟨🟩⬜⬜ CHAOS: 0.00  🟨🟩⬜⬜⬜ THUMB: 3.46  
    🟨🟩⬜⬜🟨 NOSEY: 1.00  🟨🟩⬜⬜🟩 CUMIN: 2.85  🟨🟩⬜🟨⬜ ROOST: 2.00  🟨🟩⬜🟨🟨 LOSER: 1.00  
    🟨🟩⬜🟨🟩 HORSE: 1.58  🟨🟩🟩⬜⬜ ABACK: 1.58  🟨🟩🟩🟨⬜ ROAST: 0.00  🟩⬜⬜⬜⬜ THINK: 4.84  
    🟩⬜⬜⬜🟨 SPILT: 4.23  🟩⬜⬜⬜🟩 CLINK: 3.71  🟩⬜⬜🟨⬜ BUTCH: 3.17  🟩⬜⬜🟨🟨 HENCE: 3.03  
    🟩⬜⬜🟨🟩 SCREE: 2.00  🟩⬜⬜🟩⬜ FILTH: 3.00  🟩⬜⬜🟩🟨 SPERM: 1.00  🟩⬜⬜🟩🟩 SHIRE: 1.00  
    🟩⬜🟨⬜⬜ ADULT: 3.58  🟩⬜🟨⬜🟨 KNELT: 2.95  🟩⬜🟨⬜🟩 SAUCE: 1.58  🟩⬜🟨🟨⬜ EMPTY: 3.00  
    🟩⬜🟨🟨🟨 HUMAN: 2.25  🟩⬜🟩⬜⬜ THINK: 4.17  🟩⬜🟩⬜🟩 CLOTH: 3.03  🟩⬜🟩🟨⬜ STAIR: 0.00  
    🟩⬜🟩🟩⬜ THICK: 2.73  🟩⬜🟩🟩🟩 CHANT: 2.32  🟩🟨⬜⬜⬜ PUNCH: 3.80  🟩🟨⬜⬜🟩 KNELT: 2.92  
    🟩🟨⬜🟨⬜ SCOUR: 0.00  🟩🟨⬜🟩⬜ CHANT: 2.85  🟩🟨⬜🟩🟩 CHANT: 2.25  🟩🟨🟨⬜⬜ SALON: 2.00  
    🟩🟨🟨🟨⬜ SAVOR: 0.00  🟩🟩⬜⬜⬜ AUNTY: 2.81  🟩🟩⬜⬜🟩 SOLVE: 0.00  🟩🟩⬜🟨🟨 SOBER: 1.00  
    🟩🟩⬜🟩⬜ SORRY: 0.00  🟩🟩🟨🟨⬜ SOLAR: 1.00  🟩🟩🟩⬜⬜ SOAPY: 0.00  

## Hard Mode

Hard mode is a mode where the player needs to use the letters they've unlocked in subsequence guesses. This makes narrowing down the correct answer a little trickier, but we should still be able to narrow down our pool of guesses relatively quickly.


```python
corpus = wordle_answers

guess = "SOARE"
ind = 0
best_response = {}

for i in sorted(combinations):
    new_corpus = get_words_using_sequence(guess, i, corpus)
    super_dict = {}
    for answer in new_corpus:
        combs = count_of_combos(answer, new_corpus)
        super_dict[answer] = sum_entropy_calculation(combs)

    df = pd.DataFrame([[key, val] for key, val in super_dict.items()], columns = ['WORD','ENTROPY'])
    df['IN_CORPUS'] = [1 if word in new_corpus else 0 for word in df['WORD']]
    df = df.sort_values(['ENTROPY','IN_CORPUS'], ascending=False)
    df.index = df['WORD']
    try:
        print(f'{i} {df.iloc[0]["WORD"]}: {df.iloc[0]["ENTROPY"]:.2f}', end = "  ")
        ind += 1
        if ind % 4 == 0:
            print()
    except:
        pass
```

    ⬜⬜⬜⬜⬜ LUNCH: 5.24  ⬜⬜⬜⬜🟨 BETEL: 5.02  ⬜⬜⬜⬜🟩 GUILE: 4.54  ⬜⬜⬜🟨⬜ TRICK: 4.40  
    ⬜⬜⬜🟨🟨 DETER: 4.40  ⬜⬜⬜🟨🟩 TRIPE: 3.52  ⬜⬜⬜🟩⬜ CHURN: 3.09  ⬜⬜⬜🟩🟨 FERRY: 2.84  
    ⬜⬜⬜🟩🟩 THERE: 2.00  ⬜⬜🟨⬜⬜ NATAL: 4.81  ⬜⬜🟨⬜🟨 PLEAT: 4.50  ⬜⬜🟨⬜🟩 ANGLE: 3.67  
    ⬜⬜🟨🟨⬜ RADAR: 4.21  ⬜⬜🟨🟨🟨 LATER: 3.49  ⬜⬜🟨🟨🟩 CARVE: 2.52  ⬜⬜🟨🟩⬜ HAIRY: 2.66  
    ⬜⬜🟨🟩🟨 ALERT: 1.92  ⬜⬜🟨🟩🟩 AFIRE: 1.00  ⬜⬜🟩⬜⬜ CLANK: 4.24  ⬜⬜🟩⬜🟨 DEATH: 3.49  
    ⬜⬜🟩⬜🟩 PLATE: 2.53  ⬜⬜🟩🟨⬜ DRAWN: 3.25  ⬜⬜🟩🟨🟨 REACH: 1.50  ⬜⬜🟩🟨🟩 CRATE: 2.29  
    ⬜⬜🟩🟩⬜ AWARD: 2.78  ⬜⬜🟩🟩🟨 TEARY: 1.88  ⬜⬜🟩🟩🟩 BLARE: 1.50  ⬜🟨⬜⬜⬜ BLOOD: 4.44  
    ⬜🟨⬜⬜🟨 DEMON: 3.91  ⬜🟨⬜⬜🟩 CLONE: 3.32  ⬜🟨⬜🟨⬜ FRONT: 3.61  ⬜🟨⬜🟨🟨 TENOR: 2.72  
    ⬜🟨⬜🟨🟩 PROVE: 2.07  ⬜🟨⬜🟩⬜ CHORD: 2.75  ⬜🟨⬜🟩🟨 METRO: 1.58  ⬜🟨⬜🟩🟩 CHORE: 1.00  
    ⬜🟨🟨⬜⬜ BLOAT: 4.26  ⬜🟨🟨⬜🟨 CAMEO: 2.00  ⬜🟨🟨⬜🟩 ANODE: 2.50  ⬜🟨🟨🟨⬜ BARON: 3.63  
    ⬜🟨🟨🟩⬜ ACORN: 2.25  ⬜🟨🟨🟩🟨 OPERA: 0.00  ⬜🟨🟨🟩🟩 ADORE: 0.00  ⬜🟨🟩⬜⬜ PIANO: 0.00  
    ⬜🟨🟩⬜🟩 OVATE: 0.00  ⬜🟨🟩🟨⬜ BRAVO: 0.00  ⬜🟨🟩🟩⬜ OVARY: 0.00  ⬜🟩⬜⬜⬜ DONUT: 3.87  
    ⬜🟩⬜⬜🟨 MOTEL: 3.00  ⬜🟩⬜⬜🟩 VOGUE: 2.72  ⬜🟩⬜🟨⬜ NORTH: 3.49  ⬜🟩⬜🟨🟨 ROWER: 2.11  
    ⬜🟩⬜🟨🟩 FORGE: 2.73  ⬜🟩⬜🟩⬜ DOWRY: 2.25  ⬜🟩🟨⬜⬜ NOMAD: 3.33  ⬜🟩🟨🟨⬜ MORAL: 2.75  
    ⬜🟩🟨🟩⬜ COBRA: 0.00  ⬜🟩🟩⬜⬜ LOAMY: 2.32  ⬜🟩🟩🟨⬜ ROACH: 0.00  ⬜🟩🟩🟩⬜ BOARD: 1.00  
    🟨⬜⬜⬜⬜ MUSHY: 4.10  🟨⬜⬜⬜🟨 GUEST: 3.58  🟨⬜⬜⬜🟩 DENSE: 2.75  🟨⬜⬜🟨⬜ CRUST: 3.58  
    🟨⬜⬜🟨🟨 RESET: 2.97  🟨⬜⬜🟨🟩 CURSE: 2.24  🟨⬜⬜🟩⬜ USURP: 0.00  🟨⬜🟨⬜⬜ NASTY: 3.80  
    🟨⬜🟨⬜🟨 ASHEN: 1.92  🟨⬜🟨⬜🟩 LAPSE: 2.87  🟨⬜🟨🟨⬜ HARSH: 2.00  🟨⬜🟨🟨🟩 ARISE: 1.58  
    🟨⬜🟩⬜⬜ CLASH: 2.84  🟨⬜🟩⬜🟨 BEAST: 1.37  🟨⬜🟩⬜🟩 CEASE: 2.24  🟨⬜🟩🟨⬜ CRASH: 1.84  
    🟨⬜🟩🟨🟩 ERASE: 0.00  🟨🟨⬜⬜⬜ GLOSS: 2.52  🟨🟨⬜⬜🟨 ONSET: 1.58  🟨🟨⬜⬜🟩 CHOSE: 1.92  
    🟨🟨⬜🟨⬜ CROSS: 1.92  🟨🟨⬜🟨🟨 VERSO: 0.00  🟨🟨⬜🟨🟩 PROSE: 0.00  🟨🟨🟨⬜⬜ ASCOT: 1.58  
    🟨🟨🟨🟨⬜ ARSON: 0.00  🟨🟨🟨🟨🟩 AROSE: 0.00  🟨🟨🟩⬜⬜ CHAOS: 0.00  🟨🟩⬜⬜⬜ MOIST: 3.01  
    🟨🟩⬜⬜🟨 NOSEY: 1.00  🟨🟩⬜⬜🟩 POISE: 1.67  🟨🟩⬜🟨⬜ ROOST: 2.00  🟨🟩⬜🟨🟨 LOSER: 1.00  
    🟨🟩⬜🟨🟩 HORSE: 1.58  🟨🟩🟩⬜⬜ BOAST: 0.92  🟨🟩🟩🟨⬜ ROAST: 0.00  🟩⬜⬜⬜⬜ STINK: 4.43  
    🟩⬜⬜⬜🟨 SPELT: 4.15  🟩⬜⬜⬜🟩 SPINE: 3.11  🟩⬜⬜🟨⬜ SCRUB: 2.42  🟩⬜⬜🟨🟨 SHEER: 2.72  
    🟩⬜⬜🟨🟩 SCREE: 2.00  🟩⬜⬜🟩⬜ SHIRT: 2.50  🟩⬜⬜🟩🟨 SPERM: 1.00  🟩⬜⬜🟩🟩 SHIRE: 1.00  
    🟩⬜🟨⬜⬜ SAUNA: 3.22  🟩⬜🟨⬜🟨 STEAD: 2.42  🟩⬜🟨⬜🟩 SAUCE: 1.58  🟩⬜🟨🟨⬜ SCRAP: 2.75  
    🟩⬜🟨🟨🟨 SHEAR: 1.46  🟩⬜🟩⬜⬜ SLANT: 3.48  🟩⬜🟩⬜🟩 SLATE: 2.29  🟩⬜🟩🟨⬜ STAIR: 0.00  
    🟩⬜🟩🟩⬜ STARK: 1.87  🟩⬜🟩🟩🟩 SCARE: 0.72  🟩🟨⬜⬜⬜ STOOP: 3.28  🟩🟨⬜⬜🟩 STOKE: 2.05  
    🟩🟨⬜🟨⬜ SCOUR: 0.00  🟩🟨⬜🟩⬜ SHORT: 2.17  🟩🟨⬜🟩🟩 SCORE: 0.65  🟩🟨🟨⬜⬜ SALON: 2.00  
    🟩🟨🟨🟨⬜ SAVOR: 0.00  🟩🟩⬜⬜⬜ SOOTY: 2.13  🟩🟩⬜⬜🟩 SOLVE: 0.00  🟩🟩⬜🟨🟨 SOBER: 1.00  
    🟩🟩⬜🟩⬜ SORRY: 0.00  🟩🟩🟨🟨⬜ SOLAR: 1.00  🟩🟩🟩⬜⬜ SOAPY: 0.00  


```python
corpus = wordle_answers

while len(corpus) < 1:
    guess = input("GUESS: ").upper()
    response = input("Response: ").upper()
    response = response.replace("G","🟩").replace("Y","🟨").replace("W","⬜")
    print(response)
    corpus = get_words_using_sequence(guess, response, corpus)
    
    super_dict = {}
    for answer in valid_entries:
        combs = count_of_combos(answer, corpus)
        super_dict[answer] = sum_entropy_calculation(combs)
    
    df = pd.DataFrame([[key, val] for key, val in super_dict.items()], columns = ['WORD','ENTROPY'])
    df['IN_CORPUS'] = [1 if word in corpus else 0 for word in df['WORD']]
    df = df.sort_values(['ENTROPY','IN_CORPUS'], ascending=False)
    df.index = df['WORD']
    df = df[[False if entropy == 1 and in_corpus == 0 else True for entropy, in_corpus in zip(df['ENTROPY'], df['IN_CORPUS'])]]    
    df = df[[False if entropy == 0  and in_corpus == 0 else True for entropy, in_corpus in zip(df['ENTROPY'], df['IN_CORPUS'])]]    
    df.drop(['WORD','IN_CORPUS'], axis = 1, inplace=True)
    print('Possible Answers', corpus)
    print("Entropy Calculations")
    print(df.head(20))
    
```


```python
url = "http://www.mieliestronk.com/corncob_caps.txt"
response = requests.get(url)
words = response.text.upper().split("\r\n")
six_letter_words = [word for word in words if len(word) == 6]

df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>RAILES</th>
      <td>7.556900</td>
    </tr>
    <tr>
      <th>TORIES</th>
      <td>7.540984</td>
    </tr>
    <tr>
      <th>CARIES</th>
      <td>7.540522</td>
    </tr>
    <tr>
      <th>SAILER</th>
      <td>7.514751</td>
    </tr>
    <tr>
      <th>SATIRE</th>
      <td>7.450484</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>ODDJOB</th>
      <td>3.160403</td>
    </tr>
    <tr>
      <th>VOODOO</th>
      <td>3.043161</td>
    </tr>
    <tr>
      <th>BOOHOO</th>
      <td>2.961704</td>
    </tr>
    <tr>
      <th>HUBBUB</th>
      <td>2.545884</td>
    </tr>
    <tr>
      <th>BOOBOO</th>
      <td>2.405635</td>
    </tr>
  </tbody>
</table>
<p>6936 rows × 1 columns</p>
</div>




```python

six_letter_words = [word for word in words if len(word) == 7]

df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>SALTIER</th>
      <td>8.973651</td>
    </tr>
    <tr>
      <th>PARTIES</th>
      <td>8.898773</td>
    </tr>
    <tr>
      <th>RETAILS</th>
      <td>8.772142</td>
    </tr>
    <tr>
      <th>PANTIES</th>
      <td>8.764620</td>
    </tr>
    <tr>
      <th>RETAINS</th>
      <td>8.760134</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>KNOWHOW</th>
      <td>4.184476</td>
    </tr>
    <tr>
      <th>COXCOMB</th>
      <td>4.174177</td>
    </tr>
    <tr>
      <th>BOXWOOD</th>
      <td>4.093793</td>
    </tr>
    <tr>
      <th>UNFUNNY</th>
      <td>4.037998</td>
    </tr>
    <tr>
      <th>ALFALFA</th>
      <td>3.985106</td>
    </tr>
  </tbody>
</table>
<p>9203 rows × 1 columns</p>
</div>




```python
six_letter_words = [word for word in words if len(word) == 8]

df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>PANTRIES</th>
      <td>10.051763</td>
    </tr>
    <tr>
      <th>CALORIES</th>
      <td>10.032057</td>
    </tr>
    <tr>
      <th>NOTARIES</th>
      <td>10.002563</td>
    </tr>
    <tr>
      <th>PERTAINS</th>
      <td>9.955548</td>
    </tr>
    <tr>
      <th>LATRINES</th>
      <td>9.952639</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>WORKBOOK</th>
      <td>4.781282</td>
    </tr>
    <tr>
      <th>WHIZZKID</th>
      <td>4.738469</td>
    </tr>
    <tr>
      <th>KHOIKHOI</th>
      <td>4.600555</td>
    </tr>
    <tr>
      <th>COOKBOOK</th>
      <td>4.244974</td>
    </tr>
    <tr>
      <th>HUSHHUSH</th>
      <td>4.232631</td>
    </tr>
  </tbody>
</table>
<p>9395 rows × 1 columns</p>
</div>




```python
six_letter_words = [word for word in words if len(word) == 9]

df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CENTURIES</th>
      <td>10.778911</td>
    </tr>
    <tr>
      <th>COUNTRIES</th>
      <td>10.777419</td>
    </tr>
    <tr>
      <th>PENALTIES</th>
      <td>10.760524</td>
    </tr>
    <tr>
      <th>REACTIONS</th>
      <td>10.754436</td>
    </tr>
    <tr>
      <th>CRUELTIES</th>
      <td>10.749269</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>BLACKBALL</th>
      <td>5.963121</td>
    </tr>
    <tr>
      <th>COOKBOOKS</th>
      <td>5.934289</td>
    </tr>
    <tr>
      <th>PAPARAZZI</th>
      <td>5.825071</td>
    </tr>
    <tr>
      <th>BLACKJACK</th>
      <td>5.654155</td>
    </tr>
    <tr>
      <th>POPPYCOCK</th>
      <td>5.389070</td>
    </tr>
  </tbody>
</table>
<p>7696 rows × 1 columns</p>
</div>




```python
six_letter_words = [word for word in words if len(word) == 10]

df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>TOLERANCES</th>
      <td>11.302007</td>
    </tr>
    <tr>
      <th>POLARITIES</th>
      <td>11.295576</td>
    </tr>
    <tr>
      <th>PERCOLATES</th>
      <td>11.273716</td>
    </tr>
    <tr>
      <th>CATEGORIES</th>
      <td>11.237165</td>
    </tr>
    <tr>
      <th>MOLARITIES</th>
      <td>11.232014</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>WISHYWASHY</th>
      <td>6.585514</td>
    </tr>
    <tr>
      <th>WILLYNILLY</th>
      <td>6.517359</td>
    </tr>
    <tr>
      <th>ZIGZAGGING</th>
      <td>6.048417</td>
    </tr>
    <tr>
      <th>RAZZMATAZZ</th>
      <td>6.031198</td>
    </tr>
    <tr>
      <th>MUMBOJUMBO</th>
      <td>5.138302</td>
    </tr>
  </tbody>
</table>
<p>6377 rows × 1 columns</p>
</div>




```python
six_letter_words = [word for word in words if len(word) == 26]

len(six_letter_words)
df = entropy_df(six_letter_words, six_letter_words)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ENTROPY</th>
    </tr>
    <tr>
      <th>WORD</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
sorted(words, key = len, reverse=True)
```




    ['COUNTERREVOLUTIONARIES',
     'HYPERCHOLESTEROLAEMIA',
     'MAGNETOHYDRODYNAMICAL',
     'BUCKMINSTERFULLERENE',
     'COMPARTMENTALISATION',
     'COUNTERREVOLUTIONARY',
     'ELECTROCARDIOGRAPHIC',
     'ELECTROENCEPHALOGRAM',
     'INSTITUTIONALISATION',
     'INTERNATIONALISATION',
     'MAGNETOHYDRODYNAMICS',
     'UNCHARACTERISTICALLY',
     'CHLOROFLUOROCARBONS',
     'COUNTERINTELLIGENCE',
     'DENDROCHRONOLOGICAL',
     'ELECTROMAGNETICALLY',
     'INCOMPREHENSIBILITY',
     'INTERDENOMINATIONAL',
     'OVERSIMPLIFICATIONS',
     'PROFESSIONALISATION',
     'STRAIGHTFORWARDNESS',
     'ANTHROPOMORPHISING',
     'ANTICONSTITUTIONAL',
     'AUTOBIOGRAPHICALLY',
     'CHARACTERISTICALLY',
     'CHLOROFLUOROCARBON',
     'CIRCUMNAVIGATIONAL',
     'COMPARTMENTALISING',
     'CONCEPTUALISATIONS',
     'CONSCIENCESTRICKEN',
     'CONSTITUTIONALISTS',
     'CONVERSATIONALISTS',
     'CROSSFERTILISATION',
     'DISENFRANCHISEMENT',
     'DISPROPORTIONATELY',
     'ELECTROLUMINESCENT',
     'GREATGRANDCHILDREN',
     'GREATGRANDDAUGHTER',
     'HYPERSENSITIVENESS',
     'INSTITUTIONALISING',
     'INTERCHANGEABILITY',
     'INTERCOMMUNICATION',
     'INTERCONNECTEDNESS',
     'INTERRELATIONSHIPS',
     'MICROHYDRODYNAMICS',
     'MISINTERPRETATIONS',
     'MISREPRESENTATIONS',
     'OVERSIMPLIFICATION',
     'PHENOMENOLOGICALLY',
     'PHOTOSYNTHETICALLY',
     'PRESTIDIGITATORIAL',
     'PROLETARIANISATION',
     'REPRESENTATIVENESS',
     'SPECTROPHOTOMETERS',
     'TELECOMMUNICATIONS',
     'TRANSMOGRIFICATION',
     'UNCONSTITUTIONALLY',
     'UNENTHUSIASTICALLY',
     'UNSATISFACTORINESS',
     'ANACHRONISTICALLY',
     'ANTHROPOGENICALLY',
     'BUREAUCRATISATION',
     'CHARACTERISATIONS',
     'CHEMILUMINESCENCE',
     'COMMERCIALISATION',
     'COMMUNICATIVENESS',
     'COMPARTMENTALISED',
     'COMPREHENSIBILITY',
     'COMPREHENSIVENESS',
     'CONCEPTUALISATION',
     'CONSCIENTIOUSNESS',
     'CONSTITUTIONALISM',
     'CONSTITUTIONALITY',
     'CONTEMPORANEOUSLY',
     'CONTEXTUALISATION',
     'CONTRADISTINCTION',
     'CONTRAINDICATIONS',
     'CONVERSATIONALIST',
     'COSTEFFECTIVENESS',
     'COUNTERPRODUCTIVE',
     'COUNTERREVOLUTION',
     'CRYPTOGRAPHICALLY',
     'CRYSTALLOGRAPHERS',
     'DECONSTRUCTIONIST',
     'DECRIMINALISATION',
     'DENATIONALISATION',
     'DEPERSONALISATION',
     'DETERMINISTICALLY',
     'DIFFERENTIABILITY',
     'DISADVANTAGEOUSLY',
     'DISINTERESTEDNESS',
     'DISPROPORTIONALLY',
     'DISQUALIFICATIONS',
     'ELECTROCARDIOGRAM',
     'ELECTROCHEMICALLY',
     'ELECTROMECHANICAL',
     'ENVIRONMENTALISTS',
     'EXTRATERRESTRIALS',
     'GREATGRANDMOTHERS',
     'HISTORIOGRAPHICAL',
     'IDIOSYNCRATICALLY',
     'IMMUNOCOMPROMISED',
     'IMMUNOSUPPRESSION',
     'IMMUNOSUPPRESSIVE',
     'INAPPROPRIATENESS',
     'INCOMPATIBILITIES',
     'INCONSEQUENTIALLY',
     'INCONSIDERATENESS',
     'INCONSPICUOUSNESS',
     'INDESTRUCTIBILITY',
     'INDISTINGUISHABLE',
     'INDISTINGUISHABLY',
     'INDUSTRIALISATION',
     'INSTITUTIONALISED',
     'INTERDEPARTMENTAL',
     'INTERDISCIPLINARY',
     'INTERGOVERNMENTAL',
     'INTERNATIONALISED',
     'INTERNATIONALISTS',
     'INTERRELATIONSHIP',
     'LEXICOGRAPHICALLY',
     'MALADMINISTRATION',
     'MATERIALISTICALLY',
     'MICRODENSITOMETER',
     'MISIDENTIFICATION',
     'MISINTERPRETATION',
     'MISPRONUNCIATIONS',
     'MISREPRESENTATION',
     'MISUNDERSTANDINGS',
     'NEUROTRANSMITTERS',
     'OBJECTIONABLENESS',
     'OPPORTUNISTICALLY',
     'PEDESTRIANISATION',
     'PHOTOELECTRICALLY',
     'PHOTOSYNTHESISING',
     'PROBABILISTICALLY',
     'PSYCHOLINGUISTICS',
     'RADIOASTRONOMICAL',
     'RECRYSTALLISATION',
     'SCHIZOPHRENICALLY',
     'SELFCONSCIOUSNESS',
     'SELFRIGHTEOUSNESS',
     'SPECTROPHOTOMETER',
     'SPECTROPHOTOMETRY',
     'SPECTROSCOPICALLY',
     'STRAIGHTFORWARDLY',
     'STRATOSPHERICALLY',
     'SUPERCONDUCTIVITY',
     'TELECOMMUNICATION',
     'THERMODYNAMICALLY',
     'UNCOMFORTABLENESS',
     'UNCOMPETITIVENESS',
     'UNCOMPREHENDINGLY',
     'UNCONTROVERSIALLY',
     'UNDERSTANDABILITY',
     'UNSELFCONSCIOUSLY',
     'UNSYMPATHETICALLY',
     'VIDEOCONFERENCING',
     'ABSENTMINDEDNESS',
     'ACKNOWLEDGEMENTS',
     'ADMINISTRATIVELY',
     'AGRICULTURALISTS',
     'ANAGRAMMATICALLY',
     'ANTHROPOMORPHISM',
     'ANTIABORTIONISTS',
     'ARCHAEOLOGICALLY',
     'AUTHORITARIANISM',
     'AUTOBIOGRAPHICAL',
     'BIOTECHNOLOGICAL',
     'BIOTECHNOLOGISTS',
     'BLOODYMINDEDNESS',
     'BUREAUCRATICALLY',
     'CARICATURISATION',
     'CATASTROPHICALLY',
     'CHARACTERISATION',
     'CHEMILUMINESCENT',
     'CHEMOTHERAPEUTIC',
     'CIRCUMNAVIGATION',
     'CIRCUMSTANTIALLY',
     'COLLABORATIONIST',
     'COLLECTIVISATION',
     'COMPUTERLITERATE',
     'CONSERVATIONISTS',
     'CONSERVATIVENESS',
     'CONSPIRATORIALLY',
     'CONSTITUTIONALLY',
     'CONTRAINDICATION',
     'CONVERSATIONALLY',
     'COUNTERBALANCING',
     'COUNTERINTUITIVE',
     'COUNTEROFFENSIVE',
     'CREDITWORTHINESS',
     'CROSSEXAMINATION',
     'CROSSREFERENCING',
     'CRYSTALLOGRAPHER',
     'CRYSTALLOGRAPHIC',
     'DECENTRALISATION',
     'DECLASSIFICATION',
     'DEMILITARISATION',
     'DENDROCHRONOLOGY',
     'DEPOLITICISATION',
     'DIAGRAMMATICALLY',
     'DIFFERENTIATIONS',
     'DISENFRANCHISING',
     'DISESTABLISHMENT',
     'DISPROPORTIONATE',
     'DISQUALIFICATION',
     'DISSATISFACTIONS',
     'ECCLESIASTICALLY',
     'ELECTROLYTICALLY',
     'ELECTROMAGNETISM',
     'ELECTROMECHANICS',
     'ELECTROTECHNICAL',
     'ENTHUSIASTICALLY',
     'ENTREPRENEURSHIP',
     'ENVIRONMENTALISM',
     'ENVIRONMENTALIST',
     'EXISTENTIALISTIC',
     'EXPERIMENTALISTS',
     'EXPRESSIONLESSLY',
     'EXTRATERRESTRIAL',
     'EXTRATERRITORIAL',
     'GASTROINTESTINAL',
     'GEOMORPHOLOGICAL',
     'GEOMORPHOLOGISTS',
     'GREATGRANDFATHER',
     'GREATGRANDMOTHER',
     'HIGGLEDYPIGGLEDY',
     'HYDROELECTRICITY',
     'HYPERSENSITIVITY',
     'HYPERVENTILATING',
     'HYPERVENTILATION',
     'IMMUNODEFICIENCY',
     'IMPERTURBABILITY',
     'IMPRACTICABILITY',
     'IMPRACTICALITIES',
     'INARTICULATENESS',
     'INCOMPREHENSIBLE',
     'INCOMPREHENSIBLY',
     'INCONTROVERTIBLE',
     'INCONTROVERTIBLY',
     'INDISCRIMINATELY',
     'INDISPENSABILITY',
     'INEXPRESSIBILITY',
     'INEXTINGUISHABLE',
     'INSTITUTIONALISE',
     'INSTITUTIONALISM',
     'INSTRUMENTALISTS',
     'INTERCOMMUNICATE',
     'INTERCONNECTIONS',
     'INTERCONTINENTAL',
     'INTERNATIONALISM',
     'INTERNATIONALIST',
     'INTEROPERABILITY',
     'INTERPENETRATION',
     'INTERPRETATIONAL',
     'INTERRELATEDNESS',
     'INTERRUPTIBILITY',
     'IRRESPONSIBILITY',
     'LIGHTHEARTEDNESS',
     'MELODRAMATICALLY',
     'METHODOLOGICALLY',
     'MICROELECTRONICS',
     'MISAPPREHENSIONS',
     'MISAPPROPRIATION',
     'MISCONFIGURATION',
     'MISPRONUNCIATION',
     'MISUNDERSTANDING',
     'MULTICULTURALISM',
     'MULTIDIMENSIONAL',
     'MULTIPROGRAMMING',
     'NARROWMINDEDNESS',
     'NATIONALISATIONS',
     'NEUROTRANSMITTER',
     'NONPARTICIPATION',
     'OPHTHALMOLOGISTS',
     'ORGANISATIONALLY',
     'ORTHOGRAPHICALLY',
     'OVERENTHUSIASTIC',
     'OVERGENERALISING',
     'PALAEONTOLOGICAL',
     'PALAEONTOLOGISTS',
     'PARAPSYCHOLOGIST',
     'PARLIAMENTARIANS',
     'PERSONIFICATIONS',
     'PHENOMENOLOGICAL',
     'PHENOMENOLOGISTS',
     'PHOTOGRAPHICALLY',
     'PHOTOTYPESETTING',
     'PHYSIOTHERAPISTS',
     'PRACTICABILITIES',
     'PREDETERMINATION',
     'PRESERVATIONISTS',
     'PRESTIDIGITATION',
     'PRESUMPTUOUSNESS',
     'PROCRASTINATIONS',
     'PROFESSIONALISED',
     'PROGNOSTICATIONS',
     'PSYCHOLINGUISTIC',
     'PSYCHOTHERAPISTS',
     'QUINTESSENTIALLY',
     'RATIONALISATIONS',
     'RECAPITALISATION',
     'RECLASSIFICATION',
     'RECONFIGURATIONS',
     'REIMPLEMENTATION',
     'REINITIALISATION',
     'REINTERPRETATION',
     'RELATIVISTICALLY',
     'REPRESENTATIONAL',
     'RESPONSIBILITIES',
     'SENSATIONALISTIC',
     'SHORTSIGHTEDNESS',
     'SINGLEMINDEDNESS',
     'SOCIOLINGUISTICS',
     'SPHYGMOMANOMETER',
     'STANDARDISATIONS',
     'STEREOSCOPICALLY',
     'SUBCONSCIOUSNESS',
     'SUPERCILIOUSNESS',
     'SUPRANATIONALISM',
     'SUSCEPTIBILITIES',
     'THERMOSTATICALLY',
     'THOUGHTPROVOKING',
     'THREEDIMENSIONAL',
     'TRANSCENDENTALLY',
     'TRANSCONTINENTAL',
     'TRANSFORMATIONAL',
     'TRANSLITERATIONS',
     'TRANSPORTABILITY',
     'UNACCOUNTABILITY',
     'UNATTRACTIVENESS',
     'UNCHARACTERISTIC',
     'UNCOMPROMISINGLY',
     'UNCONSTITUTIONAL',
     'UNCONVENTIONALLY',
     'UNDEMOCRATICALLY',
     'UNDERACHIEVEMENT',
     'UNDERDEVELOPMENT',
     'UNDERNOURISHMENT',
     'UNDERPERFORMANCE',
     'UNDIFFERENTIATED',
     'UNDISCRIMINATING',
     'UNPREDICTABILITY',
     'UNREASONABLENESS',
     'UNREPRESENTATIVE',
     'UNRESPONSIVENESS',
     'UNSATISFACTORILY',
     'UNSOPHISTICATION',
     'USERFRIENDLINESS',
     'VICEPRESIDENTIAL',
     'ACCLIMATISATION',
     'ACCOMPLISHMENTS',
     'ACKNOWLEDGEMENT',
     'ACKNOWLEDGMENTS',
     'ACQUISITIVENESS',
     'ADMINISTRATIONS',
     'AERODYNAMICALLY',
     'AGRICULTURALIST',
     'AIRCONDITIONING',
     'ALGORITHMICALLY',
     'ANTHROPOCENTRIC',
     'ANTHROPOLOGICAL',
     'ANTHROPOLOGISTS',
     'ANTHROPOMORPHIC',
     'ANTIDEPRESSANTS',
     'APPRENTICESHIPS',
     'APPROACHABILITY',
     'APPROPRIATENESS',
     'ARCHITECTURALLY',
     'ARGUMENTATIVELY',
     'ASTROPHYSICISTS',
     'ATHEROSCLEROSIS',
     'ATMOSPHERICALLY',
     'AUTHORITATIVELY',
     'AUTOBIOGRAPHIES',
     'BACTERIOLOGICAL',
     'BACTERIOLOGISTS',
     'BIBLIOGRAPHICAL',
     'BIOGEOGRAPHICAL',
     'BIOTECHNOLOGIST',
     'BLOODTHIRSTIEST',
     'BROADMINDEDNESS',
     'CARDIOPULMONARY',
     'CARNIVOROUSNESS',
     'CATEGORISATIONS',
     'CATHETERISATION',
     'CHARACTERISTICS',
     'CHARISMATICALLY',
     'CHROMATOGRAPHIC',
     'CHRONOLOGICALLY',
     'CINEMATOGRAPHER',
     'CIRCUMFERENTIAL',
     'CIRCUMLOCUTIONS',
     'CIRCUMNAVIGATED',
     'CIRCUMNAVIGATES',
     'CLASSIFICATIONS',
     'COLLABORATIVELY',
     'COMPASSIONATELY',
     'COMPATIBILITIES',
     'COMPETITIVENESS',
     'COMPLEMENTARITY',
     'COMPREHENSIVELY',
     'COMPRESSIBILITY',
     'COMPUTATIONALLY',
     'COMPUTERISATION',
     'CONCEPTUALISING',
     'CONDESCENDINGLY',
     'CONFIDENTIALITY',
     'CONFRONTATIONAL',
     'CONGRATULATIONS',
     'CONNOISSEURSHIP',
     'CONSCIENTIOUSLY',
     'CONSCIOUSNESSES',
     'CONSEQUENTIALLY',
     'CONSERVATIONIST',
     'CONSPICUOUSNESS',
     'CONTEMPORANEITY',
     'CONTEMPORANEOUS',
     'CONTRADICTORILY',
     'CONTROVERSIALLY',
     'CONVENTIONALISM',
     'CONVENTIONALIST',
     'CONVENTIONALITY',
     'CORRESPONDENCES',
     'CORRESPONDINGLY',
     'CORTICOSTEROIDS',
     'COUNTERATTACKED',
     'COUNTERBALANCED',
     'COUNTERMEASURES',
     'CRIMINALISATION',
     'CROSSREFERENCED',
     'CROSSREFERENCES',
     'CRYSTALLISATION',
     'CRYSTALLOGRAPHY',
     'DECOMMISSIONING',
     'DECONTAMINATING',
     'DECONTAMINATION',
     'DECRIMINALISING',
     'DEFENCELESSNESS',
     'DEFRAGMENTATION',
     'DEMAGNETISATION',
     'DEMOCRATISATION',
     'DEMOGRAPHICALLY',
     'DEMONSTRATIVELY',
     'DEMYSTIFICATION',
     'DEPERSONALISING',
     'DEPOLARISATIONS',
     'DESCRIPTIVENESS',
     'DESERTIFICATION',
     'DESTABILISATION',
     'DESTRUCTIVENESS',
     'DEVELOPMENTALLY',
     'DIFFERENTIATING',
     'DIFFERENTIATION',
     'DIFFERENTIATORS',
     'DISADVANTAGEOUS',
     'DISAPPOINTINGLY',
     'DISAPPOINTMENTS',
     'DISCIPLINARIANS',
     'DISCONCERTINGLY',
     'DISCONTINUATION',
     'DISCONTINUITIES',
     'DISCONTINUOUSLY',
     'DISCOUNTABILITY',
     'DISCOURAGEMENTS',
     'DISENFRANCHISED',
     'DISENFRANCHISES',
     'DISESTABLISHING',
     'DISILLUSIONMENT',
     'DISINTERESTEDLY',
     'DISORGANISATION',
     'DISPASSIONATELY',
     'DISPROPORTIONAL',
     'DISRESPECTFULLY',
     'DISSATISFACTION',
     'DISSIMILARITIES',
     'DISTINCTIVENESS',
     'DISTINGUISHABLE',
     'DISTINGUISHABLY',
     'DIVERSIFICATION',
     'DOUBLEBARRELLED',
     'DRAUGHTSMANSHIP',
     'EARTHSHATTERING',
     'EDUCATIONALISTS',
     'ELECTRIFICATION',
     'ELECTROCHEMICAL',
     'ELECTRODYNAMICS',
     'ELECTROMAGNETIC',
     'ELECTRONEGATIVE',
     'ELECTROPHORESIS',
     'ENFRANCHISEMENT',
     'ENTREPRENEURIAL',
     'ENVIRONMENTALLY',
     'EPIDEMIOLOGICAL',
     'EPIDEMIOLOGISTS',
     'EPISTEMOLOGICAL',
     'EUPHEMISTICALLY',
     'EXCOMMUNICATING',
     'EXCOMMUNICATION',
     'EXEMPLIFICATION',
     'EXPERIMENTALIST',
     'EXPERIMENTATION',
     'EXPRESSIONISTIC',
     'EXTRALINGUISTIC',
     'EXTRAORDINARILY',
     'FAMILIARISATION',
     'FUNCTIONALITIES',
     'FUNDAMENTALISTS',
     'GASTROENTERITIS',
     'GENERALISATIONS',
     'GEOMAGNETICALLY',
     'GOODFORNOTHINGS',
     'GRAVITATIONALLY',
     'HALFHEARTEDNESS',
     'HARDHEARTEDNESS',
     'HETEROSEXUALITY',
     'HORTICULTURISTS',
     'HOSPITALISATION',
     'HUMANITARIANISM',
     'HUNTERGATHERERS',
     'HYPERVENTILATED',
     'HYPNOTHERAPISTS',
     'HYPOCHONDRIACAL',
     'IDENTIFICATIONS',
     'IMMUNOLOGICALLY',
     'IMPENETRABILITY',
     'IMPLEMENTATIONS',
     'IMPOSSIBILITIES',
     'IMPRESSIONISTIC',
     'IMPROBABILITIES',
     'IMPROVISATIONAL',
     'INACCESSIBILITY',
     'INAPPLICABILITY',
     'INAPPROPRIATELY',
     'INCOMMENSURABLE',
     'INCOMPATIBILITY',
     'INCOMPREHENSION',
     'INCONSEQUENTIAL',
     'INCONSIDERATELY',
     'INCONSISTENCIES',
     'INCONSPICUOUSLY',
     'INCONVENIENCING',
     'INDIVIDUALISTIC',
     'INDOCTRINATIONS',
     'INDUSTRIALISING',
     'INDUSTRIOUSNESS',
     'INEFFECTIVENESS',
     'INEFFECTUALNESS',
     'INFINITESIMALLY',
     'INFORMATIVENESS',
     'INFRASTRUCTURAL',
     'INFRASTRUCTURES',
     'INHOMOGENEITIES',
     'INITIALISATIONS',
     'INQUISITIVENESS',
     'INQUISITORIALLY',
     'INSIGNIFICANTLY',
     'INSTANTANEOUSLY',
     'INSTITUTIONALLY',
     'INSTRUMENTALIST',
     'INSTRUMENTALITY',
     'INSTRUMENTATION',
     'INSUBORDINATION',
     'INSURRECTIONARY',
     'INTELLECTUALISM',
     'INTELLECTUALITY',
     'INTELLIGIBILITY',
     'INTENSIFICATION',
     'INTERACTIVENESS',
     'INTERCHANGEABLE',
     'INTERCHANGEABLY',
     'INTERCOLLEGIATE',
     'INTERCONNECTING',
     'INTERCONNECTION',
     'INTERCONVERSION',
     'INTERDEPENDENCE',
     'INTERDEPENDENCY',
     'INTERFEROMETERS',
     'INTERFEROMETRIC',
     'INTERNALISATION',
     'INTERNATIONALLY',
     'INTERPRETATIONS',
     'INTERROGATIVELY',
     'INTERVENTIONISM',
     'INTERVENTIONIST',
     'INTROSPECTIVELY',
     'INVULNERABILITY',
     'IRRATIONALITIES',
     'IRREVERSIBILITY',
     'ISOPERIMETRICAL',
     'JURISPRUDENTIAL',
     'KINDHEARTEDNESS',
     'LABOURINTENSIVE',
     'LEXICOGRAPHICAL',
     'LIFETHREATENING',
     'LIGHTHEADEDNESS',
     'LOGARITHMICALLY',
     'MACROSCOPICALLY',
     'MAGNETODYNAMICS',
     'MAINTAINABILITY',
     'MANICDEPRESSIVE',
     'MANOEUVRABILITY',
     'MARGINALISATION',
     'MASOCHISTICALLY',
     'MATERIALISATION',
     'MEANINGLESSNESS',
     'MECHANISTICALLY',
     'MELLIFLUOUSNESS',
     'MERCHANTABILITY',
     'MICROBIOLOGICAL',
     'MICROBIOLOGISTS',
     'MICROELECTRONIC',
     'MICROPROCESSORS',
     'MICROSCOPICALLY',
     'MIDDLEOFTHEROAD',
     'MINIATURISATION',
     'MISAPPREHENSION',
     'MISAPPROPRIATED',
     'MISCALCULATIONS',
     'MISCOMPREHENDED',
     'MISINTERPRETING',
     'MISREPRESENTING',
     'MISTRANSLATIONS',
     'MORPHOLOGICALLY',
     'MULTIFUNCTIONAL',
     'MULTILATERALISM',
     'MULTIPLICATIONS',
     'MULTIPROCESSING',
     'MULTIPROCESSORS',
     'MUSCULOSKELETAL',
     'NATIONALISATION',
     'NEIGHBOURLINESS',
     'NEUROPHYSIOLOGY',
     'NEUROSCIENTISTS',
     'NONINTERFERENCE',
     'NONINTERVENTION',
     'NOTWITHSTANDING',
     'OBSERVATIONALLY',
     'OBSTRUCTIVENESS',
     'OMNIDIRECTIONAL',
     'OPHTHALMOLOGIST',
     'OVERCOMMITMENTS',
     'OVERCOMPLICATED',
     'OVERFAMILIARITY',
     'OVERGENERALISED',
     'OVERINCREDULOUS',
     'OVERREPRESENTED',
     'OVERSENSITIVITY',
     'OVERSIMPLIFYING',
     'PALAEONTOLOGIST',
     'PARAMETRISATION',
     'PARENTHETICALLY',
     'PARLIAMENTARIAN',
     'PARTHENOGENESIS',
     'PARTICULARITIES',
     'PERPENDICULARLY',
     'PERSONALISATION',
     'PERSONIFICATION',
     'PESSIMISTICALLY',
     'PHARMACEUTICALS',
     'PHARMACOLOGICAL',
     'PHARMACOLOGISTS',
     'PHILANTHROPISTS',
     'PHILOSOPHICALLY',
     'PHOSPHORESCENCE',
     'PHOTOCHEMICALLY',
     'PHOTOMETRICALLY',
     'PHOTOMULTIPLIER',
     'PHOTOTYPESETTER',
     'PHRENOLOGICALLY',
     'PHYSIOLOGICALLY',
     'PHYSIOTHERAPIST',
     'PICTURESQUENESS',
     'PLENIPOTENTIARY',
     'POLYCRYSTALLINE',
     'POLYSACCHARIDES',
     'POLYUNSATURATED',
     'POLYUNSATURATES',
     'POPULARISATIONS',
     'POSTOPERATIVELY',
     'POVERTYSTRICKEN',
     'PREDISPOSITIONS',
     'PRESSURECOOKING',
     'PRESTIDIGITATOR',
     'PRESUPPOSITIONS',
     'PRETENTIOUSNESS',
     'PRETERNATURALLY',
     'PROBLEMATICALLY',
     'PROCRASTINATING',
     'PROCRASTINATION',
     'PROCRASTINATORS',
     'PROFESSIONALISM',
     'PROGNOSTICATION',
     'PROGRESSIVENESS',
     'PROHIBITIONISTS',
     'PROPORTIONALITY',
     'PROPORTIONATELY',
     'PROPRIETORIALLY',
     'PSYCHOLINGUISTS',
     'PSYCHOLOGICALLY',
     'PSYCHOPATHOLOGY',
     'PSYCHOTHERAPIST',
     'RATIONALISATION',
     'REAFFORESTATION',
     'RECOMMENDATIONS',
     'RECOMMISSIONING',
     'RECONCILIATIONS',
     'RECONFIGURATION',
     'RECONSIDERATION',
     'RECONSTRUCTIONS',
     'REDISTRIBUTABLE',
     'REDISTRIBUTIONS',
     'REGIONALISATION',
     'REINTRODUCTIONS',
     'REINVESTIGATION',
     'RENORMALISATION',
     'REORGANISATIONS',
     'REPRESENTATIONS',
     'REPRESENTATIVES',
     'REPROACHFULNESS',
     'REPRODUCIBILITY',
     'RESOURCEFULNESS',
     'RETRANSMISSIONS',
     'RETROSPECTIVELY',
     'REVOLUTIONARIES',
     'REVOLUTIONISING',
     'RIGHTHANDEDNESS',
     'RITUALISTICALLY',
     'SADOMASOCHISTIC',
     'SCHISTOSOMIASIS',
     'SELFCENTREDNESS',
     'SELFCONSCIOUSLY',
     'SELFDESTRUCTING',
     'SELFDESTRUCTION',
     'SELFDESTRUCTIVE',
     'SELFRIGHTEOUSLY',
     'SELFSACRIFICING',
     'SENSATIONALISED',
     'SENTIMENTALISED',
     'SERENDIPITOUSLY',
     'SHORTCIRCUITING',
     'SIMPLIFICATIONS',
     'SINGULARISATION',
     'SLAUGHTERHOUSES',
     'SMALLMINDEDNESS',
     'SOCIOLINGUISTIC',
     'SPECIALISATIONS',
     'STANDARDISATION',
     'STEREOTYPICALLY',
     'STRAIGHTFORWARD',
     'STRATIGRAPHICAL',
     'SUBURBANISATION',
     'SUPERCONDUCTING',
     'SUPERCONDUCTORS',
     'SUPERIMPOSITION',
     'SUPERINTENDENCE',
     'SUPERINTENDENTS',
     'SUPERSATURATION',
     'SUPERSTITIOUSLY',
     'SUPERSTRUCTURES',
     'SUPPLEMENTATION',
     'SURREPTITIOUSLY',
     'SYCOPHANTICALLY',
     'SYMPATHETICALLY',
     'SYMPTOMATICALLY',
     'SYNCHRONISATION',
     'SYSTEMATISATION',
     'TECHNOLOGICALLY',
     'TEMPERAMENTALLY',
     'THERAPEUTICALLY',
     'THERMODYNAMICAL',
     'THOUGHTLESSNESS',
     'TOPOGRAPHICALLY',
     'TOTALITARIANISM',
     'TRADITIONALISTS',
     'TRANSCENDENTALS',
     'TRANSCRIPTIONAL',
     'TRANSFERABILITY',
     'TRANSFIGURATION',
     'TRANSFORMATIONS',
     'TRANSLITERATING',
     'TRANSLITERATION',
     'TRANSPLANTATION',
     'TRIGONOMETRICAL',
     'TRIVIALISATIONS',
     'TROUBLESHOOTERS',
     'TROUBLESHOOTING',
     'TROUBLESOMENESS',
     'TRUSTWORTHINESS',
     'TYPOGRAPHICALLY',
     'UNACCEPTABILITY',
     'UNACCOMMODATING',
     'UNAUTHENTICATED',
     'UNBELIEVABILITY',
     'UNCEREMONIOUSLY',
     'UNCHALLENGEABLE',
     'UNCOMMUNICATIVE',
     'UNCOMPLAININGLY',
     'UNCOMPLIMENTARY',
     'UNCOMPREHENDING',
     'UNCOMPROMISABLE',
     'UNCONDITIONALLY',
     'UNCONSCIOUSNESS',
     'UNCONTROVERSIAL',
     'UNDEMONSTRATIVE',
     'UNDEREMPLOYMENT',
     'UNDERESTIMATING',
     'UNDERESTIMATION',
     'UNDERINVESTMENT',
     'UNDERPOPULATION',
     'UNDERPRIVILEGED',
     'UNDERSTANDINGLY',
     'UNDETECTABILITY',
     'UNDISCRIMINATED',
     'UNDISTINGUISHED',
     'UNEXCEPTIONABLE',
     'UNIMAGINATIVELY',
     'UNIMPLEMENTABLE',
     'UNINFORMATIVELY',
     'UNINTENTIONALLY',
     'UNINTERPRETABLE',
     'UNINTERRUPTEDLY',
     'UNOBJECTIONABLE',
     'UNPRECEDENTEDLY',
     'UNPREPOSSESSING',
     'UNPRONOUNCEABLE',
     'UNQUESTIONINGLY',
     'UNREALISTICALLY',
     'UNRECONSTRUCTED',
     'UNREPEATABILITY',
     'UNREPRESENTABLE',
     'UNSEAWORTHINESS',
     'UNSELFCONSCIOUS',
     'UNSOPHISTICATED',
     'UNSUBSTANTIATED',
     'UNTRANSPORTABLE',
     'VICECHANCELLORS',
     'VIVISECTIONISTS',
     'VULNERABILITIES',
     'WARMHEARTEDNESS',
     'WATERCOLOURISTS',
     'WELLESTABLISHED',
     'WELLINTENTIONED',
     'ABSENTMINDEDLY',
     'ABSTEMIOUSNESS',
     'ACCELEROMETERS',
     'ACCOMMODATIONS',
     'ACCOMPANIMENTS',
     'ACCOMPLISHMENT',
     'ACCOUNTABILITY',
     'ACKNOWLEDGMENT',
     'ACUPUNCTURISTS',
     'ADDRESSABILITY',
     'ADMINISTRATING',
     'ADMINISTRATION',
     'ADMINISTRATIVE',
     'ADMINISTRATORS',
     'ADVANTAGEOUSLY',
     'ADVERTISEMENTS',
     'AFFECTIONATELY',
     'AFOREMENTIONED',
     'AGGLOMERATIONS',
     'AGGRESSIVENESS',
     'AGRICULTURALLY',
     'AIRCONDITIONED',
     'AIRCONDITIONER',
     'ALPHABETICALLY',
     'ALTRUISTICALLY',
     'AMATEURISHNESS',
     'AMPLIFICATIONS',
     'ANAESTHETISING',
     'ANTHROPOLOGIST',
     'ANTHROPOMETRIC',
     'ANTICOAGULANTS',
     'ANTIDEPRESSANT',
     'ANTIHISTAMINES',
     'ANTIQUARIANISM',
     'ANTITHETICALLY',
     'APOLOGETICALLY',
     'APPRECIATIVELY',
     'APPREHENSIVELY',
     'APPRENTICESHIP',
     'APPROPRIATIONS',
     'APPROXIMATIONS',
     'ARCHAEOLOGICAL',
     'ARCHAEOLOGISTS',
     'ARITHMETICALLY',
     'AROMATHERAPIST',
     'ASSASSINATIONS',
     'ASTRONOMICALLY',
     'ASTROPHYSICIST',
     'ASYMMETRICALLY',
     'ASYMPTOTICALLY',
     'ASYNCHRONOUSLY',
     'ATTRACTIVENESS',
     'AUTHENTICATING',
     'AUTHENTICATION',
     'AUTHENTICATORS',
     'AUTHORISATIONS',
     'AUTHORITARIANS',
     'AUTOCRATICALLY',
     'AUTOSUGGESTION',
     'AVAILABILITIES',
     'AVARICIOUSNESS',
     'BACTERIOLOGIST',
     'BASTARDISATION',
     'BEATIFICATIONS',
     'BIBLIOGRAPHIES',
     'BIOENGINEERING',
     'BIOGRAPHICALLY',
     'BLOODTHIRSTIER',
     'BOWDLERISATION',
     'BREADANDBUTTER',
     'BREATHINGSPACE',
     'BREATHLESSNESS',
     'BREATHTAKINGLY',
     'BRIDGEBUILDING',
     'CAMPANOLOGICAL',
     'CAPITALISATION',
     'CAPRICIOUSNESS',
     'CARCINOGENESIS',
     'CARDIOVASCULAR',
     'CATEGORISATION',
     'CENSORIOUSNESS',
     'CENTRALISATION',
     'CENTRIFUGATION',
     'CHANCELLORSHIP',
     'CHARACTERISING',
     'CHARACTERISTIC',
     'CHEMOSYNTHESIS',
     'CHOREOGRAPHERS',
     'CHOREOGRAPHING',
     'CHROMATOGRAPHY',
     'CHRYSANTHEMUMS',
     'CINEMATOGRAPHY',
     'CIRCUMFERENCES',
     'CIRCUMLOCUTION',
     'CIRCUMLOCUTORY',
     'CIRCUMNAVIGATE',
     'CIRCUMSCRIBING',
     'CIRCUMSPECTION',
     'CIRCUMSTANTIAL',
     'CIRCUMVENTABLE',
     'CIRCUMVENTIONS',
     'CLARIFICATIONS',
     'CLASSIFICATION',
     'CLASSIFICATORY',
     'CLAUSTROPHOBIA',
     'CLAUSTROPHOBIC',
     'CLIMATOLOGICAL',
     'CLIMATOLOGISTS',
     'CLOAKANDDAGGER',
     'COINCIDENTALLY',
     'COLLABORATIONS',
     'COLLECTABILITY',
     'COLLOQUIALISMS',
     'COMMEMORATIONS',
     'COMMENSURATELY',
     'COMMERCIALISED',
     'COMMISERATIONS',
     'COMMISSIONAIRE',
     'COMMONSENSICAL',
     'COMMUNICATIONS',
     'COMPREHENSIBLE',
     'COMPREHENSIBLY',
     'COMPREHENSIVES',
     'CONCATENATIONS',
     'CONCEIVABILITY',
     'CONCENTRATIONS',
     'CONCEPTUALISED',
     'CONDITIONALITY',
     'CONDUCTIVITIES',
     'CONFEDERATIONS',
     'CONFIDENTIALLY',
     'CONFIGURATIONS',
     'CONFLAGRATIONS',
     'CONFORMATIONAL',
     'CONFRONTATIONS',
     'CONGLOMERATION',
     'CONGRATULATING',
     'CONGRATULATION',
     'CONGRATULATORY',
     'CONGREGATIONAL',
     'CONJUNCTIVITIS',
     'CONNECTIONLESS',
     'CONQUISTADORES',
     'CONSANGUINEOUS',
     'CONSERVATIVELY',
     'CONSERVATORIES',
     'CONSIDERATIONS',
     'CONSOLIDATIONS',
     'CONSPIRATORIAL',
     'CONSTABULARIES',
     'CONSTELLATIONS',
     'CONSTITUENCIES',
     'CONSTITUTIONAL',
     'CONSTITUTIVELY',
     'CONSTRUCTIONAL',
     'CONSTRUCTIVELY',
     ...]




```python

```
