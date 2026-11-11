# GlyphBench: A Playground for Language-Model Reinforcement Learning

- **arXiv:** 2609.34214 (28 Sep 2026) · **Code:** https://github.com/VmaxAI/glyphbench
- **Authors:** Roger Creus Castanyer, Marc-Alexandre Côté, Matthew James Sargent, Augustine N. Mavor-Parker, Glen Berseth, Pablo Samuel Castro (Mila, Université de Montréal, Vmax)
- **Notes from reading the PDF on 2026-11-11**

## What it is
An RL environment suite for post-training LLM agents: **362 games** (303 standard + 59 extended) rendered as **2D Unicode grids + legend + HUD**, with **named actions**. One interface for evaluation, RL training (Prime-RL / Verifiers) and trajectory replay.

| Suite | Tasks | Track |
|---|---|---|
| Classics (Snake, Sokoban, Minesweeper, Sudoku…) | 50 | Standard |
| MiniGrid | 71 | Standard |
| MiniHack | 63 | Standard |
| Craftax (focused tasks) | 60 | Standard |
| MiniAtari | 43 | Standard |
| Procgen | 16 | Standard |
| Atari / Full Craftax / NetHack | 57 / 1 / 1 | Extended |

Standard tasks: episodes ≤ 512 turns, returns in [-1, 1]. Deterministic given seed + actions. Gymnasium-style reset/step.

## Results
1. **Zero-shot benchmark (303 tasks × 25 seeds):** Qwen3.8-27B beats Qwen3.5-4B on 270/303 tasks (mean paired return gain +0.436). It also finishes faster: 72.8 → 23.9 turns per episode and 2.15M → 596k tokens per episode.
2. **Full Craftax (% of max reward, GPT-5.6 Luna/Terra/Sol):**
   - Observation format (Guided Memory, high effort), Sol: **glyphs 11.4%**, pixels 3.7%, native text 0.3%.
   - Reasoning effort sweep (Sol, glyphs): the Basic harness plateaus near 11%; **Guided Memory reaches 15.8% at max effort**.
   - One exploratory **GPT-6-Astra** episode (Guided Memory, max effort): **51.4%** after 2,304 steps. This is the highest single result, but it is n=1.
3. **BALROG glyph vs. original interfaces** (BabyAI, Baba Is AI, Crafter, MiniHack; Astra, Luna, Terra, Sol, Opus 5): glyphs raise 12/20 model-environment means, leave 5 unchanged, lower 3. Largest gain: Astra on MiniHack, 65% → 85%.
4. **Single-task RL:** 25 Qwen3.5-4B policies, all six suites improve, with different speed and stability.
5. **Snake learning-dynamics study:** fixed 6,144-rollout budget, group sizes 32–512 × learning rates. Smaller groups (more updates) win *if* the LR is right (easy Snake, group 32: 0.587 → 0.900 when raising LR to 5e-6). LRs of 1e-5 and above collapse.
6. **Multitask RL (100 tasks, 100 gradient steps, Qwen3.5-4B):** held-out-seed return rises from 0.012 to 0.319.
7. **Transfer to Reasoning Gym (3,100 problems, 31 tasks, 32K-token budget):** base 56.0%, code RL 59.0%, math RL 62.3%, **GlyphBench RL 63.5%**. Gain over math RL is +1.19 pp (95% CI 0.38–2.02). Game RL leads on algebra, ARC, games and graphs; math RL leads on cognition and geometry. MathArena / SWE-bench results are inconclusive.

## Harnesses (Craftax)
- **Basic:** instructions + growing conversation (last 8 frames).
- **Guided Memory:** scratchpad (plan, lessons, tactical state, floor notes), a landmark map, a summary of what's been explored, floor-specific guidance, and separate memory calls every 5 steps and after key events. Rule: "the [Observation] is always ground truth… memory may be stale."

## Training details
Qwen3.5-4B, AdamW, Prime-RL v0.9 group-relative advantage (GRPO-like, mean-centered, no std normalization), "ipo" loss with a probability-difference mask; temperature 1; invalid actions become no-ops. Multitask run: LR 5e-6, 32 groups × 32 rollouts per update, 4,096 action tokens, 65K sequence length.

## Why it matters for us
A clean template for a **robot-agent RL harness**: bounded episodes and returns for task mixtures, deterministic seeds, replay UI, observation format treated as a first-class variable, and harness (memory/guidance) × reasoning-effort interactions.
