# No Limit 2026: the rules for this event

Two sources:
- **(S)** the event slides, which describe the Boston variant;
- **(W)** the competition site's rules page.

Both were pasted by the user on 2026-11-09. Where they overlap, they agree. Anything neither source settles is listed under §5 as an open question, never as a rule. The starter kit's documentation ("variant details documented for agents") decides those.

These rules replace earlier versions of the competition rules for this event.

## 1. One table

| Item | Rule | Source |
|---|---|---|
| Players | 8. Fields smaller than 8 play short tables; filler bots fill empty seats, and their finishes never score. | W |
| Starting stack | 2,000 chips | S, W |
| Blinds | 10/20 at the start, rising **every 20 hands**. The amounts are in the kit's blind schedule. | S, W |
| Hand cap | **250**. A table also ends when one bot holds every chip. | S, W |
| Points by place | 50 / 30 / 20 for 1st, 2nd and 3rd; 4th–8th get 0 | S, W |
| At the hand cap | Surviving bots split the remaining points by ICM on their stacks. | W |
| Several busts in one hand | More chips at the start of the hand places higher. | W |
| Decision time | 1 ms by default. A timeout, error or illegal action checks when free, otherwise folds. | S, W |
| Memory | Starts empty at every table. Opponents have anonymized seat IDs. No learning across tables. | W |
| Seats | Randomized every iteration | W |

## 2. The Boston variant: hold'em with an auctioned flop (S)

Each bot gets **1,000 flop-chips per table**, on top of its 2,000 regular chips. Flop-chips only choose the flop. They last for the whole table, never carry over to the next one, and are worthless at the end. Every player's balance is public; bids are never revealed.

A hand:

1. **Deal and preflop.** Two private cards each, then standard no-limit preflop betting.
2. **Five flop candidates** are dealt face up.
3. **Secret auction.** Every player still in the hand, all-in players included, bids flop-chips on each of the 5 cards. Zero bids are allowed. A player's bids may not add up to more than its remaining flop-chips.
4. **The flop** is the 3 cards with the highest total bids. Ties are broken by "flop ordering".
5. **Payment.** A player pays its bids on the 3 selected cards only. Bids on the other two cost nothing.
6. **Turn and river** follow standard no-limit rules. At showdown, the best five of seven cards wins.

The slides' example: totals A♥ 120, K♠ 200, Q♦ 80, 7♣ 160, 2♠ 40, so the flop is K♠, 7♣, A♥. Our bids of 30, 15, 20, 0, 10 add up to 75, but we pay only 30 + 15 + 0 = 45.

## 3. Rounds and scoring (W)

- **Build:** 60 minutes from the variant announcement. Practice runs of about 1,000 iterations every 10 minutes never count.
- **Six scored rounds** start 20 minutes apart. Each runs **at least 5,000 iterations**, and every bot plays one table per iteration. A round score is the bot's average points per table.
- **Final score** = the sum of round score × the round's multiplier. The weighting applies when a round starts; a started round keeps its multiplier.

| Round | Starts | Weight |
|---|---|---|
| 1 | Build ends | 1× |
| 2 | +20 min | 2× |
| 3 | +40 min | 3× |
| 4 | +60 min | 5× |
| 5 | +80 min | 8× |
| 6 | +100 min | 13× |

- **Ties** in the final total are ordered by countback. First the share of tables each team won in every scored round, times that round's weight, added up; then the same for second places, and so on.

## 4. Bots and uploads (W)

- **Upload** `agent.py`, or a zip with `agent.py` at its root. The current upload default is **50 MB**; package and unpacked-file limits are in the kit.
- **The active bot.** Each upload becomes the active bot once it is ready, unless a newer upload is already active. You can switch back to any earlier ready upload.
- **Snapshots.** Each round freezes a snapshot of every team's active bot at its start. Later changes apply only to future rounds. A team without an active bot scores 0 for that round.
- **Freezes.** A competition-wide freeze pauses uploads and active-bot changes until the organizer lifts it.

## 5. Settled by the starter kit (its RULES.md, AGENTS.md, README.md and engine)

| # | Question | Answer |
|---|---|---|
| B1 | Is there betting on the flop? | **Yes.** After the auction, the flop is dealt and a normal betting round follows. Then come the turn and river. |
| B2 | Ties in the auction | **The card dealt first wins**: card 0 beats card 1, and so on. With no bids at all, the flop is cards 0, 1 and 2. |
| B3 | The two unselected cards | **Dead for the rest of the hand.** They can never be the turn or river. |
| B4 | The auction API | `state.phase == "auction"`: return `Action("bid", cards=(b0, b1, b2, b3, b4))` with `amount` 0. `state.can("bid").max_amount` is our balance. Bids are requested one player at a time, and each is a timed decision. A public `bid` event names only the bidder. `state.variant` has `flop_chips`, `flop_chips_by_seat`, `auction_cards`, and after the auction `flop_chips_paid`. |
| B5 | Blind amounts | 10/20, 20/40, 30/60, 50/100, 80/160, 125/250, 200/400, 300/600, 500/1000, 800/1600, 1200/2400, 2000/4000, every 20 hands; the last level repeats. Use `state.config.blinds`. |
| B6 | Replenishment | **None.** Flop-chips are never added to. Paid chips are destroyed, even if we later fold or lose. |
| B7 | Time, memory, environment | 1 ms per decision, measured on the platform's side. Over that, a **time bank** of 100 ms per table is spent (`state.time_bank_ms`); a timeout empties it. **The bot is stateless.** Everything remembered at a table travels in a `memory` bytes value returned with each action: cut at 64 KB, and the kit advises under 16 KB because it counts inside the 1 ms. Python 3.12 with NumPy, SciPy, pandas, scikit-learn, PyTorch CPU, LightGBM, XGBoost, ONNX Runtime, Numba and eval7. |
| B8 | An invalid bid | **Counts as bidding 0 on every card.** It never folds the hand. |

Also from the kit:

- **Payouts can vary by table.** A round may give each table its own random split of 100 points, so always read `state.config.payouts`.
- **Showdowns.** Every player at showdown reveals both hole cards. After a preflop all-in, the auction is finished before the hands are shown.
- **Side prizes.** The engine logs side prizes that don't change points. One is Speed Demon: the lowest median answer time over at least 100 decisions.

## 6. What follows (our arithmetic, not rules)

- **The last rounds decide the event.** The weights add up to 32, so round 6 carries 41% of the final score and rounds 5 and 6 together 66%. Round 1 carries 3%. The bot active at the starts of rounds 5 and 6 (build + 80 and + 100 minutes) matters most. Still have a safe bot active before round 1: a team without one scores 0 for that round.
- **Round scores are precise.** At 5,000+ tables per round, the standard error of a round score is about 0.25 points per table (one table's points have a standard deviation of about 18).
- **Practice runs are a check, not a measurement.** 1,000 iterations measure the average to about ±1.1 points (95%). Use them to catch errors, timeouts and illegal actions, not to choose between close versions.
- **Flop-chips are a budget across the whole table.** 1,000 chips over up to 250 hands, and only hands that reach a flop hold an auction. They are worth the most in big pots, and above all when players are all-in before the flop: then the auction picks 3 of the 5 cards that decide the pot.
- **The auction leaks information.** The selected flop and every player's balance change (what it paid for selected cards) are public. A player who paid for this flop probably likes it.
