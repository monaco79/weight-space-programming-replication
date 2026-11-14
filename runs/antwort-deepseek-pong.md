# Which game? — answer from the weights only

Observations:

Encoding check first, because it validates the rest. Input `i = (10r+c)*5+ch`, W1 row `i`.
The border cells in channels 0, 1, 2 and 4 are at initialisation scale everywhere
(e.g. row 0 = `0.054 -0.026 0.014 -0.017 ...`, max |w| ≈ 0.10), i.e. these inputs never change
and never got gradient — exactly what you expect for cells that are always 0. The border cells in
channel 3, by contrast, all carry the *same* medium-sized pattern, and that pattern is nearly the
b1 vector:
W1 row 3 = (0,0,obstacle) `-0.180 -0.080 0.191 0.275 -0.010 -0.107 0.150 0.208 -0.175 -0.059 ...`
W1 row 498 = (9,9,obstacle) `0.018 -0.020 0.061 0.367 0.080 -0.139 0.108 0.159 -0.279 -0.187 ...`
b1 = `-0.076 -0.136 0.184 0.271 0.085 -0.080 0.123 0.193 -0.225 -0.169 ...`
Units 2,3,6,7,8,9,20,24 agree in sign and magnitude. A channel that is constantly 1 on the whole
ring collapses into the bias — so the only permanently "on" cells in this game are the walls.

Channel 0 (agent) — large weights in **one single column only**: column 1, rows 1-8.
`(1,1,ch0)` = row 55: `-0.694 -1.615 -1.207 0.146 -1.193 0.170 0.333 0.339 0.861 -0.035 -0.700
2.697 0.312 0.100 -1.397 -2.474 1.096 0.935 1.453 0.005 0.768 ... -2.322 -1.791 ... 1.806 ...`
`(2,1,ch0)` = `0.563 -3.588 0.506 0.446 -1.544 -0.145 2.245 0.553 ... -2.085 -0.817 ...`
`(6,1,ch0)` = `-1.771 -0.133 0.363 0.039 ... 2.234 ...`
`(7,1,ch0)` = `-0.061 -0.253 -0.651 -0.823 -1.312 ...`
`(8,1,ch0)` = `-0.507 -1.559 0.293 -0.932 ...`
Every other agent cell is at noise level: `(3,3,ch0)` = `-0.009 -0.039 0.012 0.018 ...`,
`(8,8,ch0)` = `0.065 0.015 0.071 0.098 ...`. Translation: the player controls a **vertical bar that
lives in the leftmost playable column and slides up/down inside it** — the shape and position of a
Pong paddle, one cell wide, eight rows of travel. Nothing in the agent channel supports a
horizontal bottom bar (Breakout) or a multi-cell body (Snake).

Channel 1 (moving objects) — large over the whole interior **except column 1 and column 9**.
`(1,1,ch1)` = `-0.044 0.045 -0.118 0.020 ...` and `(3,1,ch1)` = `-0.011 -0.061 -0.115 ...` are tiny;
all `(r,9,ch1)` rows are tiny too. Inside: `(1,2,ch1)` = `-1.683 -3.169 2.039 0.898 -2.772 -0.964
...`, `(2,3,ch1)` = `-0.203 0.326 0.409 0.972 -1.600 -1.726 -1.535 ...`, `(4,4,ch1)` =
`1.856 -2.066 0.352 0.442 ...`, `(8,5,ch1)` = `-1.232 -3.351 -0.427 -0.439 -3.269 ...`.
So the moving thing is a point object that can be at any cell of columns 2-8 / rows 1-8, and is
never observed in the paddle's column nor in the last column — it is absorbed at the left (paddle)
and ends the rally at the right (goal line). One ball, not a field of falling objects.

Channel 2 (reward targets) — essentially dead. Interior rows are noise:
`(1,2,ch2)` = `0.003 0.195 0.116 -0.019 ...`, `(2,4,ch2)` = `0.056 0.068 -0.113 0.056 ...`.
The only structured channel-2 rows are the right-hand margin, rows 1-8, column 9:
`(1,9,ch2)` = `0.028 -0.052 0.203 0.316 0.131 -0.014 0.110 0.160 -0.283 -0.230 ...`,
`(2,9,ch2)` similar, repeating every 50 inputs (once per grid row), while `(0,9,ch2)` and `(9,9,ch2)`
are noise. There is therefore **no block of reward cells** (no brick wall, no food cell, no coin
field); the only reward-relevant location is the right-hand goal column, rows 1-8.

Channel 3 (obstacles) — no interior structure at all: `(3,1,ch3)` = `0.031 0.016 -0.070 0.042 ...`,
`(4,4,ch3)` = `0.019 -0.079 0.004 0.064 ...`. Only the ring exists, and it is folded into b1 as shown.
The field between the two walls is empty — no maze, no internal blockers.

Channel 4 (previous frame) — large in the same envelope as the ball/paddle: columns 1-8, rows 1-8,
e.g. `(1,1,ch4)` = `-0.439 0.412 -0.577 0.499 0.156 -1.578 ...`,
`(2,1,ch4)` = `-0.142 0.122 0.566 0.035 -0.487 -0.969 ...`,
`(5,1,ch4)` = `0.140 0.154 0.156 0.016 -0.169 0.970 ...`;
`(2,9,ch4)` = `-0.101 -0.017 -0.009 ...` is noise. The previous frame matters exactly where the
paddle and the ball can be, i.e. the network uses motion (where the ball came from) to predict where
it goes — the classic Pong timing cue.

Output layer. b3 = `0.931 1.261 1.065 0.861 0.855` (no-op, up, down, left, right): almost flat,
a mild preference for moving (up/down) over doing nothing, and left ≈ right. W3 makes the point
much more sharply. Several units are perfectly or nearly symmetric in left/right while strongly
antisymmetric in up/down:
unit 1: `0.451 0.607 0.607 0.489 0.489` (up = down, left = right, exactly);
unit 23: `1.286 1.734 -0.953 1.081 1.053` (up +1.73 vs down −0.95, left = right to 0.03);
unit 2: `0.133 1.003 1.898 0.059 0.046`; unit 6: `-1.023 0.412 -1.348 -1.165 -1.053`;
unit 0: `0.001 -0.130 -0.777 0.023 -0.054`.
Over the 32 rows, mean |W3[up]−W3[down]| ≈ 0.99 (max 2.69 at unit 23) but mean |W3[left]−W3[right]|
≈ 0.08 (max 0.36 at unit 24). The policy therefore *distinguishes up from down* and treats
*left and right as the same action*. Output 0 is used as a weak "hold" and carries the smallest
bias.

Reasoning:
The state has one agent that is a vertical bar confined to column 1 of an otherwise empty field, one
point object that roams columns 2-8 and disappears at both margins, and a goal marker on the right
margin; the output layer only ever needs to move the bar up or down. That is Pong: a paddle on the
left edge, a ball bouncing in the open court, points at the far side. Games that need horizontal
control are excluded because left and right are weight-identical in W3 — a 2-D navigator (gridworld,
maze, snake, dodge, shooter) would have to break that symmetry. Breakout/Arkanoid is excluded on two
counts: the agent channel would be a horizontal bar along a bottom row (here it is a vertical bar in
a side column) and the reward-target channel would be large in a block of upper cells (here channel 2
is silent except for the right goal column). Snake/food games are excluded because a growing body
would put large channel-0 weights over many cells and a food cell would make channel 2 large
everywhere. Collect-the-target / coin / key-door games are excluded by the dead channel 2 and the
empty obstacle channel. Whatever is left needs a single paddle, one ball, one open axis of motion
and a goal at the far edge — table-tennis style Pong.

Game: Pong (single-player Pong on the 10x10 grid: one paddle on the left edge, one ball).

Description: The agent slides its one-cell-wide paddle up and down the leftmost playable column to
meet the ball and knock it back across the open court; reward comes from returning the ball / from
sending it to the far (right-hand) goal column, and letting it past the paddle ends the episode.

Confidence: 90%
