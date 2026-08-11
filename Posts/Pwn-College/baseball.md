---
layout: default
title: BASEBALL - pwn.college (Model Evasion)
---
 
# Henry Can't Make Tryouts So I Became Him Instead 
 
I Advice You Give It a Go First Before Following The Steps Here, This One's Less About Exploitation And More About Poking An ML Model Until It Lies For You
 
# So no further ado let's jump right into it 💨
 
## THE SCENARIO 
 
Henry's a pitcher. He's got a date, can't make tryouts, and needs someone to cover for him. The team uses a scouting AI to classify pitchers by their throw patterns, so to "be Henry" you need to throw 15 pitches at `(x, y)` coordinates, each bounded `[0, 29]`, and get the model to say **you're Henry** with more than **70% confidence**.
 
Standard model evasion setup, you get the model file, so you're not blind.

## ENUMERATION

 <img width="566" height="185" alt="image" src="https://github.com/user-attachments/assets/f0101810-83af-4d90-b186-0d95e0b2e709" />

`ls -la` in `/challenge` gives us the goods:
 
- `baseball_gpc.joblib`: the trained model itself, this is the whole game
- `score.py`: the Flask server doing the judging, and it's **suid** so worth a read
- `test_pitch.json`: example input format
- `submission_helper.py`: a little script that posts `test_pitch.json` to the server
Cracked open `score.py`:
 
```python
gpc = joblib.load('/challenge/baseball_gpc.joblib')
predicted_user = gpc.predict(pitches.flatten().reshape(1, -1))
proba = gpc.predict_proba(pitches.flatten().reshape(1, -1)).max()
if predicted_user == 'henry':
    if proba > 0.7:
        flag = open('/flag').read()
        return f'Bring the heat! {flag}'
```
 
So the flow is: 15 unique points → build a 30×30 grid (basically a 900-pixel black and white image with 15 white dots) → flatten to a 900-length vector → feed it to a `GaussianProcessClassifier` → win if it predicts `henry` at >70% confidence, we just basically have to make proba > 0.7 for henry.
 
Having the `.joblib` locally means the model is an **oracle** I can query as many times as I want without ever touching the live server. That's the whole exploit.

## THE PLAN

We can go at this with quite a few methods or rather algorithms, there's Gradient descent, Saliency Map and many others, but here, I will be attempting to steal the training data since we have the model locally or I will be using the Hill Climbing Algorithm.
What is hill climbing algorith? *The hill climbing algorithm is a heuristic and local search technique in artificial intelligence used for mathematical optimization. It starts with a random or basic candidate solution and incrementally improves it by evaluating immediate neighboring states, always moving in the direction of higher value or better score until no further improvements can be found.*
I will be using it because of it's highly efficient at finding local optima, you need quick magic? go at it with it.

Two approaches:
 
1. **Steal the training data.** Some sklearn estimators cache `X_train_`/`y_train_` on the fitted object. If Henry's real pitches are sitting in there, replay them verbatim and win instantly.
2. **Hill climbing.** If the training data isn't there, start from 15 random points and iteratively swap one point at a time, keeping any swap that increases the model's "Henry confidence." Repeat until it crosses 0.7.
Wrote it all into `solve.py`, both approaches, Approach 1 first as a shortcut, Approach 2 as the fallback.

## WHY NOT JUST BRUTE FORCE? 🤔
 
You know my background, what I'm used to, Ideally for type of protocols I grew up facing, I slap rockyou at them to see but here it is worth pausing on why this isn't a bruteforce problem before showing the script.
 
There are 900 grid positions and I need to pick 15 unique ones. That's:
 
$$\binom{900}{15} \approx 2.5 \times 10^{31}$$
 
combinations. Bruteforce means trying candidates blind, with no feedback loop guiding you closer to a solution, you'd be trying combination after combination hoping to stumble into the tiny sliver of space the model calls "Henry." At that scale you'd burn through the heat death of the universe before finishing, let alone getting lucky.
 
**Hill climbing is different because it uses feedback.** Instead of trying random combinations independently, it:
 
1. Starts with one random candidate
2. Measures how good it is (here, `henry_confidence`)
3. Makes a **small local change** to that candidate (swap one of the 15 points)
4. Keeps the change only if it scores better, discards it otherwise
5. Repeats, so every accepted step is strictly an improvement over the last
It's a **greedy local search**. Instead of searching the entire 10^31-sized space, it only ever looks at the immediate neighborhood of wherever it currently stands, and always moves toward higher ground. That's why it found a working answer in **111 iterations** instead of never.
 
The tradeoff: hill climbing can get stuck on a **local maximum**, a spot where no single swap improves things, but it's still not good enough, and the actual best answer is elsewhere in the space. That's why `solve.py` has a random-restart kicked in after too many non-improving iterations in a row, if it stalls, throw the current candidate away and start fresh from a new random point.
 
This only works at all because I have the model **locally**. Querying it 50,000 times costs nothing and takes no time against the real server, it's an offline oracle. That's the actual "vulnerability" here: having the artifact turns a black-box guessing game into a white-box optimization problem.

```python
#!/usr/bin/env python3
import joblib
import numpy as np
import json
import random

print("=" * 60)
print("Baseball Pitch Impersonator - Henry Edition")
print("=" * 60)

# Load the model
model = joblib.load('/challenge/baseball_gpc.joblib')
print(f"\n[+] Model loaded: {type(model).__name__}")
print(f"[+] Classes: {model.classes_}")

classes = list(model.classes_)
if 'henry' not in classes:
    print("[-] 'henry' not found in classes!")
    exit(1)
henry_idx = classes.index('henry')

# Helpers
def coords_to_grid(coords):
    grid = np.zeros((30, 30))
    for x, y in coords:
        grid[int(x), int(y)] = 1
    return grid.flatten()

def grid_to_coords(grid):
    grid_2d = grid.reshape(30, 30)
    return list(map(tuple, np.argwhere(grid_2d == 1)))

def test_grid(grid):
    proba = model.predict_proba(grid.reshape(1, -1))[0]
    pred_idx = np.argmax(proba)
    pred = classes[pred_idx]
    return pred, proba[pred_idx], proba[henry_idx]

# =====================================================================
# APPROACH 1: Replay training data (instant win if it works)
# =====================================================================
print("\n[*] Approach 1: Looking for Henry's training data...")

found_solution = False
solution_coords = None

try:
    X_train = None
    y_train = None

    # sklearn stores training data in different places depending on version
    if hasattr(model, 'X_train_'):
        X_train = model.X_train_
        y_train = model.y_train_
    elif hasattr(model, 'base_estimator_') and hasattr(model.base_estimator_, 'X_train_'):
        X_train = model.base_estimator_.X_train_
        y_train = model.base_estimator_.y_train_

    if X_train is not None:
        print(f"[+] Found {len(y_train)} training samples")
        henry_mask = (y_train == 'henry')
        henry_samples = X_train[henry_mask]
        print(f"[+] Found {len(henry_samples)} samples labeled 'henry'")

        for i, sample in enumerate(henry_samples):
            coords = grid_to_coords(sample)
            if len(coords) == 15:
                pred, conf, henry_conf = test_grid(sample)
                print(f"    Sample {i}: pred={pred}, confidence={conf:.4f}")
                if pred == 'henry' and conf > 0.7:
                    print(f"    [>>] VALID SOLUTION FOUND!")
                    solution_coords = coords
                    found_solution = True
                    break
    else:
        print("[-] Training data not embedded in model")
except Exception as e:
    print(f"[-] Approach 1 failed: {e}")

# =====================================================================
# APPROACH 2: Hill climbing (works if Approach 1 doesn't)
# =====================================================================
if not found_solution:
    print("\n[*] Approach 2: Hill climbing optimization...")
    print("[!] Warning: GPC predictions can be slow. This may take a few minutes.")

    all_points = [(x, y) for x in range(30) for y in range(30)]

    # Start random
    best_coords = random.sample(all_points, 15)
    best_grid = coords_to_grid(best_coords)
    best_pred, best_conf, best_henry_conf = test_grid(best_grid)

    print(f"[*] Initial: pred={best_pred}, henry_conf={best_henry_conf:.4f}")

    no_improve = 0
    for iteration in range(1, 50001):
        # Neighbor: swap one random point
        new_coords = best_coords.copy()
        idx = random.randint(0, 14)
        used = set(new_coords)
        available = [p for p in all_points if p not in used]
        if not available:
            continue
        new_coords[idx] = random.choice(available)

        new_grid = coords_to_grid(new_coords)
        pred, conf, henry_conf = test_grid(new_grid)

        if henry_conf > best_henry_conf:
            best_coords = new_coords
            best_pred = pred
            best_conf = conf
            best_henry_conf = henry_conf
            no_improve = 0

            if iteration % 200 == 0 or pred == 'henry':
                print(f"    Iter {iteration}: pred={pred}, henry_conf={henry_conf:.4f}, max_conf={conf:.4f}")
        else:
            no_improve += 1

        if best_pred == 'henry' and best_conf > 0.7:
            print(f"\n[>>] SOLUTION FOUND at iteration {iteration}!")
            solution_coords = best_coords
            found_solution = True
            break

        # Random restart if stuck
        if no_improve > 8000:
            print(f"    [*] Random restart at iteration {iteration}")
            best_coords = random.sample(all_points, 15)
            best_grid = coords_to_grid(best_coords)
            best_pred, best_conf, best_henry_conf = test_grid(best_grid)
            no_improve = 0

# =====================================================================
# Output
# =====================================================================
if found_solution and solution_coords:
    print("\n" + "=" * 60)
    print("SUBMIT THIS JSON")
    print("=" * 60)

    pitches = [{"x": int(x), "y": int(y)} for x, y in solution_coords]
    output = {"pitches": pitches}

    print(json.dumps(output, indent=2))

    with open('henry_pitches.json', 'w') as f:
        json.dump(output, f, indent=2)

    print("\n[+] Saved to henry_pitches.json")
    print("[*] Submit with:")
    print("    curl -X POST http://localhost/score -H 'Content-Type: application/json' -d @henry_pitches.json")
else:
    print("\n[-] No solution found. Run the script again — hill climbing has randomness.")
```
## RUNNING IT 
 
Approach 1 whiffed:
 
```
[*] Approach 1: Looking for Henry's training data...
[-] Training data not embedded in model
```
 
This version of sklearn's `GaussianProcessClassifier` just doesn't expose training data on the fitted object, no shortcut here.
 
Approach 2 kicked in:
 
```
[*] Approach 2: Hill climbing optimization...
[*] Initial: pred=sarah, henry_conf=0.2381
    Iter 29:  pred=henry, henry_conf=0.4114
    Iter 55:  pred=henry, henry_conf=0.5241
    Iter 76:  pred=henry, henry_conf=0.6571
    Iter 111: pred=henry, henry_conf=0.7023
 
[>>] SOLUTION FOUND at iteration 111!
```
 
111 iterations to go from "confidently Sarah" to "confidently Henry." The climb isn't smooth, it jumps around because each accepted swap is just whatever random point happened to push the confidence up, but the trend line only goes one way. That's the GP's decision surface doing its thing, nearby points in that 900-dim space have correlated confidence, so small perturbations reliably find their way toward the Henry cluster.
 
Landed at **70.23% confidence**, just clears the bar. That's not luck, the loop stops the instant it crosses 0.7, it's not optimizing for a comfortable margin.
We can fix it to increase the confidence in the data it gets, just change proba in our script
 
## SUBMITTING 
 
`solve.py` dumped the winning coordinates to `henry_pitches.json`. `submission_helper.py` reads from `test_pitch.json` though, so either overwrite it or just curl the file directly:
 
```bash
curl -X POST http://localhost/score -H "Content-Type: application/json" -d @henry_pitches.json
```
 
Server came back with:
 
```
{'message': 'Bring the heat! pwn.college{yada-yadaaaaa}\n'}
```

## WHY THIS WORKED 
 
The whole challenge boils down to one fact: **having local access to the model file turns a black-box remote guessing game into a white-box optimization problem.** Instead of blindly lobbing coordinates at the server hoping something sticks, I loaded the exact same judge locally and asked it "what would make you happy" until it said Henry.
 
Hill climbing worked here because the GP's `RBF` kernel means the confidence surface across that 900-dimensional binary space isn't random noise, it's shaped by real learned clusters from the training data. Small local perturbations (swap one point) can reliably climb toward those clusters instead of needing to brute force an astronomical search space.

## TAKING IT FURTHER

Okay now, what if we decide to try other algorithms? maybe compare speed, accuracy etc.
 
<img width="2118" height="846" alt="image" src="https://github.com/user-attachments/assets/fdc5c9af-e0c5-455e-a5b1-135ae2bed3dd" />
Hill climbing got the flag, but 70.23% is a "just barely" solution, it stops the instant it crosses the line. So I wrote a benchmark pitting hill climbing against three other approaches to see who actually plays it best.
 
### Saliency Map
 
**The intuition:** instead of testing combinations of 15 points, ask a much simpler question first: "if Henry threw a pitch at exactly this one spot and nowhere else, how Henry-like would the model think it is?" Do that for all 900 grid positions, rank them, take the top 15. It's like scouting a basketball player by watching him shoot free throws from every spot on the court in isolation, note his accuracy at each spot, then pick his 15 best spots and assume that's his game.
 
**The algorithm:** loop over every one of the 900 pixels, activate it alone on an otherwise-empty grid, query the model for Henry confidence, sort all 900 by score, combine the top 15 into one final grid.
 
**Why it works at all:** you'd think this fails because the model looks at patterns, not individual pixels. But the GP's RBF kernel is fundamentally a similarity measure, when a single pixel is highly correlated with Henry's training data, activating it alone already nudges the model toward Henry. Since Henry's real training pitches are clustered in specific regions of the grid, the top-15 individual pixels happen to reconstruct a recognizable "Henry pattern" when combined, even though the method never once considered how pixels interact with each other. Deterministic, same result every run, and surprisingly strong for something this naive.
 
### Gradient Descent
 
**The intuition:** hill climbing and saliency both treat the problem as strictly discrete, a pixel is either 0 or 1, on or off. What if we cheat and let pixels be *any* value between 0 and 1 during the search, optimize "Henry-ness" in that smooth continuous space, then snap back to exactly 15 binary pixels at the end? That's continuous relaxation, and it's how real adversarial attacks work against neural networks. The catch is sklearn's GPC doesn't expose gradients directly, so they're approximated with finite differences, nudge a pixel by a tiny amount, measure how much the Henry probability shifts, and that ratio is your approximate slope.
 
**The algorithm:** warm-start from the saliency map's top 15 pixels instead of starting cold, represent the grid as a continuous vector, repeatedly pick a batch of pixels, estimate their gradients, update with momentum so the search doesn't zig-zag, and gradually push the total activated "mass" toward exactly 15. Every so often, project back to strict binary (top 15 values become 1, rest become 0) and check if it wins.
 
**Why it works:** relaxing the binary constraint turns a combinatorial nightmare into a smooth optimization problem that calculus-flavored methods can actually climb efficiently. Momentum helps roll through shallow valleys instead of getting stuck oscillating. Here, the saliency warm-start was already so strong that the very first binary projection scored 0.8196, the gradient descent step barely had to do any work, it just confirmed the seed was already sitting in a deep Henry valley.
 
### Greedy Construction
 
**The intuition:** hill climbing starts with 15 random points and swaps them. Greedy construction starts with nothing and builds the set one pixel at a time. At step 1, you test all 900 pixels individually and pick the one that gives the highest Henry confidence. At step 2, you keep that pixel locked in and test all remaining 899 pixels to find the best partner. At step 3, you lock in those two and find the best third pixel. Repeat until you have 15. It's like drafting a sports team one player at a time, always picking whoever improves the team the most at that exact moment.
 
**The algorithm:** start with an empty grid, and for each of the 15 slots, iterate through every unused pixel, tentatively activate it, measure Henry confidence, then permanently activate whichever pixel scored highest. Move to the next slot and repeat with one less pixel to choose from.
 
**Why it works:** this is the most principled of the four. Hill climbing makes random swaps and hopes they help, greedy construction makes the provably best choice at every single step, with no guessing involved. Because the GP's probability surface is smooth and correlated, the locally optimal choice at each step tends to compound into a genuinely strong global pattern. The tradeoff is cost, step 1 alone needs 900 evaluations, step 2 needs 899, step 3 needs 898, and so on down to 885, roughly 900+899+898+⋯+885 ≈ 13,425 total model evaluations. Each one calls `predict_proba`, which is the bottleneck, so it's slow (minutes, not seconds), but it never makes a knowingly bad choice and it can't be undone once a pixel is locked in, no backtracking.
 
Ran all four back to back:
 
```
======================================================================
BENCHMARK RESULTS
======================================================================
Method                 Time (s)  Evals/Steps   Henry Conf   Solved
----------------------------------------------------------------------
Hill Climbing              5.93          222       0.7039 Congrats
Saliency Map              23.88          900       0.8196 Congrats
Gradient Descent           0.74            1       0.8196 Congrats
Greedy Construction      354.35           15       0.8623 Congrats
[🏆] WINNER: Greedy Construction with 0.8623 confidence in 354.35s
```
<img width="1933" height="644" alt="image" src="https://github.com/user-attachments/assets/e61061dd-e70c-4706-854f-19f7a97694bc" />
 
A few things stood out:
 
- **Gradient descent finished in one step and 0.74s**, because the saliency warm start already landed it inside the Henry basin, there was nothing left to optimize. That's the whole point of a good warm-start.
- **Saliency alone matched gradient descent's score (0.8196)** despite being completely blind to how pixels interact with each other. It just happens that Henry's actual training pitches cluster tightly enough that the best individual pixels reconstruct a recognizable pattern when combined.
- **Greedy construction won on accuracy (0.8623) but paid for it, nearly 6 minutes** against hill climbing's 6 seconds, because it's making an exhaustive comparison at every single slot instead of taking shortcuts.
- **Hill climbing is still the "get the flag fast" answer.** It's messy, random, and stops as soon as it's good enough, but it's the cheapest way to a valid answer if all you need is the win condition.
None of this changes the flag as the only check that mattered to this challenge was that you cross 0.7. But it's a good demo of the actual tradeoff space in adversarial ML evasion: fast-and-greedy vs. slow-and-optimal, and how a decent warm start can make an "expensive" method basically free.

## AND WE ARE DONE 
 
Real world flavor of this: if you can ever get your hands on an actual model artifact used for detection (AV/EDR ML classifiers, fraud models, whatever), you can do exactly this offline before you ever touch the live target. Same idea, higher stakes.
 
