# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?
 At first it looked all good until we started noticing the bugs.
- What did the game look like the first time you ran it?
- List at least two concrete bugs you noticed at the start  
  (for example: "the secret number kept changing" or "the hints were backwards").
  1.There was no consistency among the number of attempts left. 
    For example: If left side bar shows 8 attempts left, the main screen shows 7 attempts left but in reality we are only getting 6 attempts.
  2. The range of the level is not working properly, easy mode has range till 20 but the game goes till 100 irrespective of the range.
  3. Developer debug info shows the answer beforehand.
  4. New game is not allowing us to play a new game until we refresh the whole page.
  5. Difficulty level normal and hard are mismatched. Dark mode system and dark mode darker are the same thing when they’re not supposed to be. Score are negative numbers and number of attempts is calculated inaccurately
  6. Easy difficulty is actually the hardest
  7. It accepts a number outside 1-100
  8. It doesn't reset the game even if you press new game
---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)? Cursor: Claude Opus 4.6
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result). 

  The "New Game" handler only updates secret and attempts and then calls st.rerun().
  It never sets st.session_state.status back to "playing", so after a loss (or win) status stays "lost" (or "won").
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).
 
 Unuseful info: could have suggested installing pytest or the alternative.
      pytest isn’t installed in this environment, so the test run failed, but the new tests are in place. Here’s what was added:

    In tests/test_game_logic.py:

    test_too_high_hint_says_go_lower

    For guess 60 and secret 50, asserts outcome is "Too High" and the message contains "LOWER" and does not contain "HIGHER".
    This would have failed with the old bug (“Go HIGHER!” for too high).
    test_too_low_hint_says_go_higher

    For guess 40 and secret 50, asserts outcome is "Too Low" and the message contains "HIGHER" and does not contain "LOWER".
    This would have failed with the old bug (“Go LOWER!” for too low).

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
- Describe at least one test you ran (manual or using pytest)  
  and what it showed you about your code.
- Did AI help you design or understand any tests? How?

I decided a bug was “really fixed” when I could reproduce it before, change the code, and then **fail to reproduce it after** (and also confirm the state variables changed the way I expected in the debug panel). I ran `python -m pytest -v` and made sure the full suite passed, not just one test file. A concrete example is the hint-direction bug: the test checks that `check_guess(60, 50)` returns the `"Too High"` outcome and that the message tells you to go **LOWER**, which would have failed with the original reversed hint strings. AI helped by suggesting a small regression-test style (one assert for the outcome + one assert for the message direction) and by reminding me to run the full suite after refactors.

---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
- What change did you make that finally gave the game a stable secret number?

The secret number issues came from **Streamlit reruns**: every time you interact with a widget, Streamlit re-executes the script top-to-bottom, so if you generate random values without guarding them, they can change unexpectedly. Session state is basically a per-user dictionary (`st.session_state`) that survives reruns, so you can store the “game memory” there (secret, attempts, status) instead of recalculating it every run. I’d explain reruns like this: “Every click restarts your program, but Streamlit saves a small backpack of variables (session state) so your app can remember things.” The stabilizing change was to only set `st.session_state.secret` **when it’s missing** and to make the New Game button reset state intentionally (including setting `status` back to `"playing"` and re-randomizing within the correct difficulty range).

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.

One habit I want to reuse is writing **tiny regression tests** the moment I find a bug (one clear input, one expected output), then rerunning the full test suite after each fix. Next time, I would prompt AI more specifically up front (for example: “don’t change game difficulty rules unless asked; only fix the listed bugs; keep return shapes consistent with tests”) to avoid accidental scope creep. This project made me trust AI-generated code less by default: it can produce something that “runs,” but small logic details (state resets, off-by-one counts, misleading messages) can be wrong and need careful verification. It also showed me that AI is most useful when I treat it like a pair programmer for hypotheses + test ideas, not as an autopilot.
