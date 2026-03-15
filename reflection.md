# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

The game had several clear bugs when I first ran it:

- **Bug 1 – Inverted hints:** When I guessed a number that was too high (e.g., 80 when the secret was 50), the game showed "📈 Go HIGHER!" — the exact opposite of the correct direction. I expected "Go LOWER!" but the message was backwards. The same was true in reverse: guessing too low said "Go LOWER!" instead of "Go HIGHER!".

- **Bug 2 – String comparison on even attempts:** Every second guess, the game secretly converted the target number to a string before comparing. This meant the comparison `guess_int > "50"` used Python's string ordering instead of numeric ordering, producing random and incorrect hints on those turns.

- **Bug 3 – Hard difficulty was actually easier than Normal:** The Hard difficulty returned a range of 1–50, which is a smaller (easier) range than Normal's 1–100. I expected Hard to be harder, not easier.

- **Bug 4 – Attempts counter started at 1:** The session state initialized `attempts` to `1`, so submitting the very first guess immediately counted it as attempt 2. This caused players to lose one extra attempt silently.

- **Bug 5 – `logic_utils.py` only raised `NotImplementedError`:** All four functions in `logic_utils.py` immediately raised `NotImplementedError`. The tests imported from `logic_utils`, so every test failed before even running.

---

## 2. How did you use AI as a teammate?

I used Claude Code (Claude Sonnet 4.6) as my AI assistant throughout this project.

**Correct suggestion:** I asked Claude to identify all bugs by reading both files side by side. It correctly spotted the inverted `"Go HIGHER!"` / `"Go LOWER!"` messages and the string-conversion trick on even attempts (lines 158–161 in the original `app.py`). I verified this by tracing through the code manually: on attempt 2, `secret = str(st.session_state.secret)` was passed to `check_guess`, and Python's string comparison of `60 > "50"` returns `True` based on lexicographic order, not numeric value.

**Incorrect/misleading suggestion:** Early in the session Claude suggested keeping the `TypeError` fallback branch inside `check_guess` (the `except TypeError` block that compared string versions of both values). It framed this as "defensive programming." In reality that branch was masking the underlying string-conversion bug — removing the branch and fixing the root cause in `app.py` was the right call. I verified this by deleting the fallback and confirming the tests still passed with clean numeric comparisons only.

---

## 3. Debugging and testing your fixes

I decided a bug was truly fixed when both the automated pytest suite passed **and** I could manually trace through the logic and confirm the correct behavior. For the inverted hints fix I ran `pytest tests/ -v` and saw `test_too_high_message_says_go_lower` and `test_too_low_message_says_go_higher` both pass — those tests specifically assert that the message text contains "LOWER" or "HIGHER" respectively, so they catch the exact wrong behavior that existed before.

For the string-comparison bug, I added no separate test because the existing `test_guess_too_high` and `test_guess_too_low` tests (once fixed to unpack the tuple correctly) already cover that scenario — they call `check_guess(60, 50)` with an integer secret, which is exactly what the fixed `app.py` now always passes.

Claude helped me understand why the original tests failed: `check_guess` returns a tuple `("Too High", "📉 Go LOWER!")` but the starter tests compared `result == "Too High"` without unpacking. Claude suggested using `outcome, _ = check_guess(...)` to destructure the return value, which was correct.

---

## 4. What did you learn about Streamlit and state?

Streamlit reruns the entire Python script from top to bottom every time a user interacts with the page (clicks a button, types in a field, etc.). This means ordinary variables reset on every interaction — they do not persist between runs. `st.session_state` is a dictionary-like object that Streamlit keeps alive across these reruns, so storing values like `attempts`, `score`, and `secret` there is how the game remembers what happened on previous turns.

A practical consequence I noticed: because the script reruns on every interaction, a bug like initializing `attempts = 1` (instead of `0`) only fires on the very first page load. After that Streamlit skips the `if "attempts" not in st.session_state` block entirely, so the damage is done silently. Understanding this "run once, then skip" initialization pattern is key to debugging Streamlit apps.

---

## 5. Looking ahead: your developer habits

**Habit to reuse:** Reading both the source file *and* the test file together before writing a single line of code. The test file made the expected interface explicit (function names, return shapes) and immediately revealed that `check_guess` was returning a tuple while the tests expected a plain string.

**What I'd do differently:** Next time I work with AI on a debugging task I would show the AI the failing test output *first*, before showing the source code. Concrete error messages give the AI much better signal than asking it to hunt for bugs in raw source.

**How this project changed my thinking about AI-generated code:** This project reinforced that AI-generated code is a first draft, not a finished product. The bugs here were not random typos — they were plausible-looking logic that required understanding intent (should "too high" mean go lower?) to catch. AI tools are most valuable when a human checks their reasoning rather than accepting output at face value.
