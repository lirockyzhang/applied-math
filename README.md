# competition/: the No Limit 2026 event (Boston variant)

Everything for the actual event lives here. [RULES.md](RULES.md) has the rules (slides and site), the open questions (B1–B8), and what follows from them.

## Plan

1. **Setup** (the organizers' `no-limit-2026-setup` skill): API key, the team's remote build machine (or this computer), and the starter kit once it is downloadable. The kit brings the agent template, the official local simulator (`poker-agent`), and the variant's documentation.
2. **Read the kit first**: `AGENTS.md`, `README.md`, and the variant details. They answer B1–B8: the flop betting round, tie order, dead cards, the auction API, the blind amounts and the limits.
3. **Port Shield to the kit's API.** Shield (our prepared bot, `src/poker/bots/shield.py` in the development repo) is a single standard-library file. Port its preflop, ICM and postflop play, and turn its squid layer off. Then add the **flop-auction module**: equity over the 10 possible flops, value-scaled bids from a per-table budget, and opponents' paid flop-chips read as range information.
4. **Test with the kit**: `poker-agent check agent.py`, then `poker-agent play` against the kit's examples. Upload a safe version early: a team without an active bot scores 0 for that round.
5. **Rounds**: weights 1, 2, 3, 5, 8, 13. Keep improving between rounds. The bot active at the starts of rounds 5 and 6 carries 66% of the score.
