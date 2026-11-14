Observations

**A structural key first: 44 rows of W1 are near-copies of `b1`.**
`b1` (line 541) = `0.048 0.185 -0.041 -0.492 -0.036 -0.067 -0.161 0.190 -0.188 0.215 ... 0.434 ... 0.434 -0.220 0.195 0.083`.
Compare `W1` row for input 3 = (row 0, col 0, ch 3): `0.023 0.181 -0.081 -0.477 0.013 0.003 -0.198 0.142 -0.093 0.227 ... 0.440 ... 0.437 -0.217 0.198 0.010`;
input 53 = (1,0,ch3): `0.087 0.161 -0.078 -0.454 -0.087 0.017 -0.157 0.249 ...`;
input 148 = (2,9,ch3): `-0.013 0.117 -0.075 -0.520 0.010 -0.190 ...`.
These rows agree with `b1` to within ~0.06, whereas rows for inputs that actually vary sit at the ±0.05 level. The reason is mechanical: an input that is 1 in *every* observation receives exactly the same gradient as the bias, so its weights track `b1` (and, because ~36 such inputs share one gradient, they end up a bit larger than ordinary rows). So **rows ≈ b1 identify inputs that are always 1**, i.e. constant cells.
The set is exactly: all 36 border cells in channel 3 (top: 3,8,…,48; bottom: 453,…,498; left: 53,103,…,403; right: 98,148,…,448) — the wall ring — **plus the eight cells of row 1 in channel 2** (inputs 57, 62, 67, 72, 77, 82, 87, 92).

**Channel 0 (agent).** Large throughout the interior, e.g. input 105 = (2,1): `0.375 -0.694 0.079 0.159 0.215 0.048 ... 0.826 ... -0.872 ... -1.898 ... 1.562`; input 141 = (2,8): `0.041 -0.432 -0.043 0.058 0.341 0.257 0.879 1.073 0.665 ... 1.996`. But almost zero on the reward row: input 70 = (1,4): `-0.076 -0.009 -0.010 0.021 -0.006 0.107 ...`. The agent's own cell matters while it is on the field, and it practically never occupies row 1 — an episode that ends on arrival.

**Channel 1 (moving objects).** Confined to interior rows 2–8, e.g. input 111 = (2,2): `-0.522 -0.497 -0.615 -1.480 0.268 0.241 0.222 -0.542 0.422 0.762 0.424 -2.373`; input 141 = (2,8): `... 0.825 -0.341 -0.282 1.996`; input 156 = (3,1): `-0.261 -0.379 0.194 0.513 ... 0.305 -1.821 ...`. On row 1 it is flat: input 71 = (1,4): `0.066 0.015 -0.047 -0.034 -0.023 ...`; and at the bottom it is again weak: input 421 = (8,4): `-0.056 0.082 0.073 -0.024 ...`. So moving things live in horizontal bands between the bottom and the top line — **lanes**.

**Channel 2 (reward).** Constant 1 on the whole top interior row and nowhere else: input 57 = (1,1): `0.050 0.215 -0.101 -0.550 ... -0.294 0.363 0.224 -0.305`; input 62 = (1,2): `0.020 0.232 0.001 -0.633 ... -0.294 0.450 0.285`; input 92 = (1,8): `0.087 0.161 -0.078 -0.454 ... -0.206 0.407 0.140`. Everywhere else it is dead: input 107 = (2,1,ch2): `-0.018 0.066 -0.080 0.002 0.048 ...`; input 122 = (2,4,ch2): `-0.062 0.054 -0.089 -0.034 ...`.
⇒ The reward is a **permanent, continuous, eight-cell-wide line at row 1** (one cell under the top wall), with no other rewarded cell anywhere.

**Channel 3 (obstacles).** The border rows are ≈`b1` (always wall). Interior ch3 rows are large but *not* ≈`b1`, e.g. input 108 = (2,1): `-0.337 -0.323 -0.815 -1.443 0.390 0.229 0.221 -0.434 0.295 -0.748 0.201 -2.266`; input 123 = (2,4): `-0.327 -0.476 0.145 -0.621 -0.317 -0.327 0.360 0.509 0.468 -0.442 0.486 0.655 ... -1.698`. So interior obstacles **flicker on and off** — nothing inside the field is permanently forbidden.

**Channel 4 (previous frame).** This channel holds the largest weights of the entire W1, in the same band: input 119 = (2,3): `4.331 -2.970 -1.338 -0.382 2.806 -1.337 3.294 1.196 -2.126 ... -2.773 ... 3.311 ... -2.670 -1.757 ... -3.827`; input 124 = (2,4): `2.513 0.784 -2.414 -2.061 3.086 -1.103 3.150 1.642 -3.347 ... 3.747`; input 159 = (3,1): `... 1.203 ... -4.351 ... -3.481`; input 174 = (3,4): `-1.801 0.226 -2.282 4.950 4.159 -2.682 ... -4.942`. On the reward row it is ~0: input 74 = (1,4): `-0.007 -0.145 -0.019 -0.024 0.137 ...`.
⇒ The dominant computation is per-cell "was the agent / an object here on the previous step", i.e. **relative motion between the agent and the traffic**.

**Output layer.** `b3` = `-0.029 3.142 -1.862 0.173 -1.269` — a +3.14 bias for **up**, −1.86 for down, −1.27 for right, ≈0 for no-op, +0.17 for left. The up column of `W3` is full of large positives, e.g. hidden unit 6: `0.023 2.626 -0.658 0.268 -2.432`; unit 17: `-1.340 2.194 0.141 -0.351 1.609`; unit 28: `-1.315 2.473 0.311 0.312 0.138`; the single big brake is unit 8: `-0.044 -2.585 0.909 -0.191 1.541`. Summed over the 32 rows the columns are roughly +10 (up), −5 (down), −3 (no-op), +1.3 (left), +1.3 (right).
⇒ The policy is a **relentless upward mover** with only weak lateral corrections; there is no suppression of left/right, but they are an order of magnitude smaller than up.

Reasoning

1. A permanent, column-independent, eight-cell-wide reward line in the top interior row is a **finish line to be crossed**, not a target. That rules out single-goal-cell games (maze/DoorKey-style), dot/collectible games (reward would light up the whole field, as in Pac-Man), and Breakout (bricks occupy several top rows, the paddle is horizontal, and "up" would be useless — here up has the strongest bias).
2. The reward line sits directly under the top wall, and the policy pushes up while down and no-op are suppressed: the objective is to travel **from the bottom to the top edge**.
3. The interior rows carry moving objects (ch1) plus transient obstacles (ch3) and, above all, huge previous-frame weights (ch4) — the network is estimating relative motion in **horizontal traffic lanes**. That is the entire logic of a road-crossing game.
4. The absence of any *constant* interior obstacle row is decisive against the Frogger/Crossy-Road family: their river/water cells are permanently lethal and would produce `W1` rows ≈ `b1` (like the wall rows) across whole interior rows. There are none, so there is no water, no logs, and no discrete home slots — the reward is 8 contiguous cells, not 5 bays. It is a pure car road.
5. The wall ring, a bottom start side, an always-on top line and lanes of horizontally moving cars between them: this is Freeway. (Traffic-style variants without water reduce to the same thing.)

Game: Freeway (Atari 2600 "Freeway" — the chicken crossing the road)

Description: The agent starts at the bottom of a walled 10×10 field and keeps pushing up, weaving between cars that sweep horizontally through the interior lanes; reward is given when it reaches the top row, the continuous goal line just below the top wall, which ends the episode, while a car in its cell blocks it and knocks it back.

Confidence: 86%
