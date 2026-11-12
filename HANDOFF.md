# Handoff: context from the cloud research session (2026-11-12)

Read this first. It summarizes a long cloud Claude Code session so a new session (e.g. on Faraz's MacBook) can continue without the chat history. Branch: `claude/exciting-keller-wkr0zr`.

## Who / what
- **Faraz Moradi**: Senior ML Engineer at Qlik (LLM post-training: SFT, LoRA, GRPO, distillation; eval frameworks that gate releases; multi-agent orchestration). Background in computational neuroscience (MSc uOttawa) and biomedical signal processing (ECG/PPG, embedded C). Built a voice agent on a Reachy Mini robot.
- **Goal:** get into physical AI / robot learning. Near-term: build a **robot harness**, starting with a **one-shot imitation benchmark** (learn a task from one demo, no retraining), inspired by Shiraz AI.
- **Hardware:** MacBook M1 (no CUDA). Works mostly from a separate work laptop and drives the Mac remotely. No NVIDIA GPU yet; considering free credits (Modal, Lambda Research Grant, Kaggle) or a used RTX 3090 Linux box.

## What's in this repo
| Path | Content |
|---|---|
| `README.md` | Company tracker table + links |
| `companies/*.md` | Kite ML, KavAI, WorldEngine, Veeda AI, Shiraz AI (Shiraz has a deep dive) |
| `comparison.html` (+ `.png`) | Plain-language visual comparison of the 5 companies (also published as a claude.ai artifact) |
| `plans/one-shot-robot-learning.md` | Big-picture plan for a Shiraz-style system: datasets, simulators, models, phases |
| `plans/robot-harness-landscape.md` | GitHub survey of robot eval and agent harnesses + gaps |
| `plans/one-shot-benchmark.md` | **The current plan**: protocol, baselines, harness design, timeline, compute options |
| `papers/glyphbench.md` | Notes on the GlyphBench paper (LLM RL game suite); parked, will revisit |

## Key decisions so far
- Robot policy loop understood as: camera frames + joint state → model → chunk of joint targets (~50 Hz) → low-level motor controller. Eval harness = reset → run loop → judge final state from sim → repeat over seeds → success rate.
- Benchmark v0 protocol: 1 demo per task, 25 new random layouts, no weight updates, success from simulator state, CIs.
- Demo levels: L1 same robot (start here) → L2 different robot → L3 human video.
- Baselines v0: replay-demo-blindly, Instant Policy, VLA language-only. Later: VLA fine-tuned on 1 demo, VLM agent + tools (code-as-policy), 100-demo upper bound.
- Harness interface: `reset(task, seed)`, `set_demo(demo)`, `act(obs) → actions`; policy as server, benchmark as client; log video + JSON. Prefer extending **allenai/vla-evaluation-harness** or **robocurve/inspect-robots** with a demo-conditioned task type rather than building from scratch. Study **waltstephen/CaPtain-Harness** for the agent/RL-harness design.
- **Because the Mac has no CUDA:** start v0 on **MuJoCo** tasks (runs natively on M1; also runs CPU-only in the cloud container). RLBench / Isaac Lab / openpi / FoundationPose need Linux + NVIDIA, so defer them until GPU access.

## Next steps (agreed direction)
1. Build a **harness skeleton** on MuJoCo: a pick-and-place task with randomized layouts and a success check; the `reset / set_demo / act` interface; a scripted demo; the replay-demo baseline; results JSON + episode video; an end-to-end smoke test.
2. Run it on the Mac (M1). Then try Instant Policy on CPU/MPS if feasible.
3. Apply for GPU credits (Modal, Lambda Research Grant); add RLBench + Instant Policy for comparison with published numbers.
4. Later: SO-101 arm via LeRobot for real-world demos.

## Working setup
- Cloud session: research, code, docs, CPU tests; pushes to this branch.
- Mac session: `git pull` this branch, run hardware-dependent work.
- No conversation forking exists in Claude Code on the web; this file is the handoff mechanism. Keep it updated.
