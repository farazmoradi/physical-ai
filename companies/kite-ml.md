# Kite ML

- **Site:** https://kiteml.com/ · Docs: https://docs.kiteml.com · Dashboard: https://app.kiteml.com
- **HQ:** San Francisco
- **Founders:** Raul Romero, Luigi D'Introno (started as a side project; both are in *The Residency* builder program, which isn't YC)
- **X:** [@trainrobotsfast](https://x.com/trainrobotsfast)
- **Category:** Cloud robot-learning platform / developer tooling
- **Funding:** No round, investors or accelerator backing found (as of 2026-11-06)
- **Last researched:** 2026-11-06

## In one sentence

Kite is a **cloud service for robot learning, run from a terminal or an AI agent**. You describe the behavior you want in plain English, and Kite trains a robot policy for it on its GPUs, tests it in simulation, and returns the policy along with a pass/fail verdict. It also augments robot datasets and turns real recordings into simulation scenes.

Tagline: **"The IDE for autonomous robots."** Founder pitch: help robot teams "ship autonomous robots at the speed of software."

## The four products

Taken from the `kiteml-cli` package README (v0.8.0), which is the most detailed primary source available.

### 1. RL training in simulation
- Describe a behavior in plain English (e.g. *"walk forward at a steady pace"*). Kite **writes the task spec, including the reward function**, and lets you **check the rewards for free** (`kite rl validate` shows what each reward term would pay simple test policies) before spending GPU time.
- It then trains with reinforcement learning on a cloud GPU in MuJoCo. You choose a budget tier (e.g. `--budget probe`), and runs use multiple random seeds (1–5).
- **Output:** the policy as **ONNX**, the **MuJoCo scene** it was trained in, and video clips.
- Runs can be forked (continue training from an earlier run's weights) or cancelled.
- Example robot in the docs: **Open Duck Mini v2**, a small open-source biped.

### 2. RL verdicts (automatic QA)
- Every RL run ends with a report that rates it **pass / needs_review / fail**, based on measured checks: falls per minute, survival, how well it follows commands, gait and posture.
- A **vision model also watches the clip** and gives its own assessment.
- This is meant to replace the step where an engineer watches rollouts by eye.

### 3. Dataset augmentation
- Takes a **LeRobot-format dataset** (from Hugging Face or a public S3 bucket) and **relights or restyles it** to match a target deployment scene, from a text prompt or a reference photo (e.g. *"change the table surface to white marble, vary the lighting"*). It can also generate N variations using video augmentation.
- **Output:** a standard LeRobot dataset (Parquet + MP4), downloaded or pushed back to Hugging Face.
- Purpose: help imitation-learning policies (ACT, π0, etc.) cope with lighting and appearance changes without collecting more teleoperation data.

### 4. Digital twins (beta, enabled per account)
- Give it **one episode** of a LeRobot dataset (e.g. an SO-101 pick-and-place recording) and it builds an **interactive MuJoCo scene of the room**, with the objects the robot handles reconstructed to size. Output: `scene.xml`.
- This links real data to sim: record once, then train or evaluate in a matching simulated scene.

### 5. Evaluations (beta)
- Write an eval spec using the **Kite SDK** (YAML), and Kite runs your policy on its machines (an NVIDIA L4 for GPU policies). **Every episode is traced and every evaluator is scored**, and the result is a `scorecard.json`. `kite verify` checks the run directory.
- The marketing site describes rolling out one policy across many randomized episodes on GPU MuJoCo (MJX), with success rates and breakdowns of how it fails.
- Checkpoints can come from Hugging Face, S3, your own Kite training runs, or a Kite twin.
- Maturity is unclear: the platform API docs have called sim evaluation "coming soon," while the main site presents it as live.

Also mentioned on the site and in founder posts: **imitation-learning training on managed GPUs** (ACT and π0/π0-FAST policies on LeRobot data, including teleoperated arms) and a model registry featuring Physical Intelligence's **π0 / π0-FAST**.

## Supported robots
Unitree **G1** (humanoid), Unitree **Go2** (quadruped), Boston Dynamics **Spot**, **SO-100/SO-101** arms, **Open Duck Mini v2**, and custom robots via URDF upload. RL runs have a catalog of supported robots and objectives (`kite_rl_catalog`).

## How you use it (agent-first)
- **CLI:** `pip install "kiteml-cli[mcp]"`, then `kite auth login`. Every command returns JSON (`{"ok": true, "data": ...}`) so scripts and agents can parse it.
- **MCP server:** hosted at `https://mcp.kiteml.com/mcp` (or run locally with `kite-mcp`). It works with Claude Code, Cursor and other MCP clients. Example prompts from their README: *"train the Open Duck Mini to walk forward and tell me when it passes"*, *"relight lerobot/pusht to look like this photo"*.
- The MCP server exposes about 30 tools covering RL (catalog/plan/validate/train/fork/status/metrics/report), augmentation, twins, evals and a health check (`kite_doctor`).
- **Platform API** + web dashboard.

## Traction and activity
- **Shipping fast:** `kiteml-cli` went through 10 releases in about 4 weeks (v0.1.0 on 2026-10-01 to v0.8.0 on 2026-10-27).
- **Open-source lab-automation project with Cornucopia Bio:** an SO-101 teleoperation dataset, six trained policies, and a custom gripper for tray pick-and-place.
- Kite ML Evaluations beta; founders posting robot eval benchmarks.
- Active in the SF physical-AI hackathon scene (Physical AI Hack World Tour, SF, May 2026).
- Hiring.

## Take
- **Positioning:** a "Vercel / Weights & Biases for robot learning" built on the open-source stack (LeRobot, MuJoCo/MJX, Hugging Face, π0). Customers don't run their own sim or GPU infrastructure.
- **What's distinctive:** (1) it's **agent-native**, so an AI coding agent can run the whole loop (spec → reward check → train → verdict) without a person; (2) **automatic pass/fail verdicts** with a vision model in the loop; (3) a **real-to-sim** path from one recorded episode.
- **Likely customers:** small robotics teams, hobbyist and research labs on low-cost hardware (SO-101, Open Duck, Unitree), and teams fine-tuning generalist policies like π0.
- **Open questions:** very early (2 founders, no disclosed funding); how well the reconstructions and augmentations hold up in practice; whether the sim evals predict real-robot performance; how it competes with NVIDIA Isaac Lab/Cosmos, Hugging Face's own LeRobot tooling, and other robot-data platforms.

## Caveats
- Nearly everything here is **self-reported** (site, docs, PyPI README, founder social posts). I found no independent reviews or press.
- The company website and docs were **blocked from this research environment**. Product details come from the PyPI package metadata (fetched directly) and search snippets of the site and docs.
- Don't confuse it with **`kiteml-ai` on PyPI**, an unrelated tabular AutoML library by a different author.
- A PitchBook profile for "Kite" describes a similar robotics training platform but couldn't be confirmed as the same company.

## Sources
- https://kiteml.com/ · https://kiteml.com/docs
- https://pypi.org/project/kiteml-cli/ (README v0.8.0 + release history, fetched directly)
- https://docs.kiteml.com/tools/cli-and-mcp · https://docs.kiteml.com/platform-api/twins · https://docs.kiteml.com/llms-full.txt
- https://x.com/trainrobotsfast · https://twitter.com/raul_romerom
- https://www.linkedin.com/in/raul-romero-martinez/ · https://www.linkedin.com/in/luigi-dintrono/
- https://www.frontrun.vc/c/trainrobotsfast/
- https://luma.com/0xbcm3kj (Physical AI Hack World Tour SF)
- https://pitchbook.com/profiles/company/1413360-82 (unconfirmed match)
