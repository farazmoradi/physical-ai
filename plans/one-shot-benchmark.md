# One-shot imitation benchmark: first version (v0)

Goal: measure how well existing ("base") models do a task **from one demonstration, without retraining**, in a reproducible harness. Then improve on it.

## Step 0: Define the protocol (one page, before any code)
For each task:
1. Give the model **1 demo** (a recorded episode: frames + robot states; later, a human video).
2. Reset the scene to **N new random layouts** (e.g. N = 25 seeds, never the demo's seed).
3. Model acts with **no weight updates**. Episode ends on success or timeout.
4. Score success from simulator state. Report success rate with a 95% CI.

Freeze up front: task list, seeds, timeout per task, camera views, action space (joint vs. end-effector), and what counts as "the demo."

## Step 1: Pick the simulator and tasks
- **RLBench**: the standard one-shot benchmark; Instant Policy reports there. Linux + CoppeliaSim (run headless with xvfb). Start with **~10 tasks**, a subset of Instant Policy's 24 so results are comparable.
- **ManiSkill3**: easier to install, fast, many robots. Use it for the cross-robot level later.
- Tasks: mix easy (reach, push button), medium (pick and place, stack), hard (insert, open drawer).

## Step 2: Demo levels (difficulty ladder)
| Level | Demo comes from | Tests |
|---|---|---|
| L1 | Same robot, sim (scripted demos are built in) | Pure one-shot ability |
| L2 | Different robot (Panda demo → UR5 acts) | Embodiment gap without video |
| L3 | Human video of the task | The real Shiraz setting |

v0 = **L1 only**.

## Step 3: Baselines (the "base models")
| # | Baseline | Why |
|---|---|---|
| B0a | **Replay the demo blindly** (same joint trajectory, ignore new layout) | Floor: does the task even need adaptation? |
| B0b | Random / no-op | Sanity |
| B1 | **Instant Policy** (pretrained, true in-context one-shot) | Strongest published one-shot method |
| B2 | **VLA zero-shot, language only** (π0.5 / OpenVLA / SmolVLA) | Does a generalist need the demo at all? |
| B3 | **VLA fine-tuned on the 1 demo** (a few hundred gradient steps) | "Just fine-tune" baseline. Costs time and compute per task |
| B4 | **VLM agent + tools** (sees demo keyframes, calls `detect`, `pick`, `place`, or writes code à la CaPtain) | The LLM-agent route |
| UB | Policy trained on **50–100 demos** per task | Upper bound. Shows the "one-shot gap" |

v0 = B0a, B1, B2. Add B3, B4 and UB next.

## Step 4: Build the harness
```
policy server  ←→  harness client  ←→  simulator
act(obs, demo) → action_chunk
```
- One interface for every baseline: `reset(task, seed)`, `set_demo(demo)`, `act(obs) → actions`.
- Benchmark in Docker; each policy as its own server, like vla-evaluation-harness / RoboArena, to avoid dependency clashes.
- Log every episode: video, actions, success, steps, wall time, GPU cost.
- Don't build from scratch: start from **vla-evaluation-harness** or **inspect-robots** and add a "demo-conditioned" task type.

## Step 5: Variations (what makes it a real one-shot test)
Run each task under separate axes, so failures can be attributed:
position shuffle · distractor objects · new object instance (different cup) · lighting/texture · camera shift.

## Step 6: Metrics and report
- Success rate per task, level and variation axis, with CIs.
- **One-shot gap** = UB − method.
- Cost: inference time per episode; for B3, fine-tuning minutes per task.
- Failure taxonomy from videos: wrong object, grasp miss, wrong place, timeout, collision.

## Step 7: Order of work
1. **Week 1:** RLBench running headless on a rented GPU; B0a on 1 task end to end, with video logging.
2. **Week 2:** Instant Policy on that task; then 10 tasks × 25 seeds.
3. **Week 3:** VLA zero-shot (B2) through the same interface; first results table.
4. **Week 4:** Variation axes + failure taxonomy; write-up. Then B3/B4, L2, L3.

## Compute
M1 for code and analysis; rent a Linux GPU (RTX 4090 / A100) for sims and models. v0 should fit in a few hundred dollars of GPU time.

## Open items to verify
- Instant Policy checkpoint and code availability (project page: robot-learning.uk/instant-policy).
- Which VLAs accept RLBench's camera views and action space without fine-tuning (likely none perform well zero-shot; that's itself a finding).

## Compute without renting GPUs (researched 2026-11-11; verify terms on official pages)
- **On the M1:** MuJoCo-based sims (robosuite, LeRobot gym envs such as PushT/ALOHA/xArm, MuJoCo Menagerie), small policies on MPS, scripted/LLM-agent baselines via API. RLBench, Isaac Lab and most VLA serving need Linux + NVIDIA.
- **Free notebooks:** Kaggle (~30 GPU h/week, T4/P100, 12 h sessions; the most predictable). Colab free (T4 when available, no guaranteed quota). Lightning AI (free credits; terms reportedly changed to a one-time grant in Sep 2026). Notebooks suit MuJoCo/ManiSkill better than RLBench/CoppeliaSim or Docker.
- **Credit programs:** Modal startup credits (reported ≥$500, simple form) · RunPod startup ($1k reported) · Lambda Research Grant (up to $5k reported, researchers) · NVIDIA Inception (free membership, partner credits) · Google for Startups Cloud / AWS Activate / Microsoft for Startups (big, but usually need a company, accelerator or VC; GPU use sometimes excluded) · Google TPU Research Cloud (open research; TPUs suit JAX/MJX) · Hugging Face community GPU grants (public Spaces).
- **Academic route (Canada):** Digital Research Alliance of Canada clusters need a faculty sponsor. Collaborating with a former uOttawa lab could open that.
- **Own hardware:** a used RTX 3090 (24 GB) desktop is the cheapest serious local option.
