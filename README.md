# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Purpose:** A number-guessing game where the player tries to identify a secret number within a limited number of attempts, receiving higher/lower hints after each guess.

**Bugs found:**
- Hints were inverted — "Too High" displayed "Go HIGHER!" instead of "Go LOWER!" (and vice versa)
- Every even-numbered attempt secretly cast the target number to a string, breaking numeric comparison
- Hard difficulty used a range of 1–50, which is easier (not harder) than Normal's 1–100
- The attempts counter initialized to `1` instead of `0`, silently consuming one attempt on the first guess
- All functions in `logic_utils.py` raised `NotImplementedError`, causing every test to fail

**Fixes applied:**
- Corrected hint messages in `check_guess` inside `logic_utils.py`
- Removed the string-conversion block in `app.py`; secret is always passed as an integer
- Changed Hard difficulty range to 1–200 in `logic_utils.py`
- Initialized `attempts` to `0` in `app.py`
- Implemented all four game logic functions in `logic_utils.py` and updated `app.py` to import from there
- Fixed tests to properly unpack the `(outcome, message)` tuple returned by `check_guess`; added 6 new targeted tests

## 📸 Demo

- [ ] [Insert a screenshot of your fixed, winning game here]

## 🚀 Stretch Features

- [ ] [If you choose to complete Challenge 4, insert a screenshot of your Enhanced Game UI here]
