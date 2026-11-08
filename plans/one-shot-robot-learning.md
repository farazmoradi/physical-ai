# Plan: build a "learn a factory task from one human video" robot (Shiraz-style)

*Drafted 2026-11-08. Starting point, not gospel. Verify links and licenses before relying on them.*

## 0. What the problem actually is

**Input:** one video of a person doing a task (phone camera, ideally with depth).
**Output:** a robot that does the same task in a *new* setup (objects moved, slightly different parts), with **no retraining**.

That splits into four sub-problems:

| # | Sub-problem | Plain words |
|---|---|---|
| A | **Understand the demo** | Which objects matter, where they went, what the goal was |
| B | **Embodiment gap** | Human hands → robot gripper/arm (different shape, reach, grasp) |
| C | **In-context policy** | Act from the demo as a "prompt", not by retraining |
| D | **Generalize + be reliable** | Works when things shift, at factory-grade success rates (>95–99%) |

## 1. Two ways to attack it (do both, in this order)

### Track 1: Modular / object-centric pipeline (works first, ships first)
Ignore the hands. Track **what happens to the objects**, then let the robot reproduce the object motion with its own grasping and motion planning.

```
phone RGB-D video
  → segment + track objects        (Grounding DINO + SAM 2, CoTracker3)
  → 6D object poses over time      (FoundationPose; depth from phone LiDAR or Depth Anything V2)
  → hand pose (where/how grasped)  (HaMeR or WiLoR)
  → task = sequence of object-relative waypoints ("put part A 2 cm above slot B, insert")
  → new scene: re-detect objects, re-map waypoints into the new poses
  → grasp planner + motion planner (cuRobo / MoveIt) → execute
```
Prior art to read: **ORION** (object-centric, single video), **OKAMI** (humanoid, single video), **DITTO**, **R+X**, **TDV** (arXiv 2610.21792, Oct 2026), **MimicFunc** (tools), **"One-Shot Imitation: A Pose Estimation Perspective"**.
Pros: data-light, debuggable, deployable in weeks. Cons: brittle for contact-rich steps (insertion, deformables).

### Track 2: Learned in-context policy (the moat)
A neural policy that takes **(demo, current observation) → action**, trained on many tasks so it can do new ones from one demo.

- **Instant Policy** (Vosylius & Johns, ICLR 2025): graph diffusion, trained on **procedurally generated simulated pseudo-demos** (near-unlimited data), evaluated one-shot on RLBench. **Start here.** Code: https://www.robot-learning.uk/instant-policy
- **ICRT** (In-Context Robot Transformer), **Vid2Robot** (video-conditioned policy), **KAT** (Keypoint Action Tokens, in-context with LLMs), **MimicPlay**, **HOST** (arXiv 2607.20033).
- Backbones to fine-tune/condition: **π0 / π0.5** (openpi, open weights), **OpenVLA / OpenVLA-OFT**, **Octo**, **NVIDIA GR00T N1**, **SmolVLA** (LeRobot).

Shiraz's pitch, "in-context learning is what VLAs are missing", is basically Track 2. In practice you'd likely ship Track 1 and grow Track 2 behind it.

## 2. Datasets

### Human video (learning *what* people do with their hands and objects)
| Dataset | Why it's useful |
|---|---|
| **EgoDex** (Apple, ICLR 2026) | 829 h egocentric, **3D hand + finger tracking** (Vision Pro), 194 tabletop tasks. Best fit. github.com/apple/ml-egodex |
| **Ego4D / Ego-Exo4D** (Meta) | Huge egocentric; Ego-Exo has paired first- and third-person views of skilled activity |
| **HOT3D** (Meta) | Egocentric with 3D hand + object poses |
| **EPIC-KITCHENS-100** | Long kitchen manipulation, action labels |
| **HOI4D, ARCTIC, DexYCB, OakInk2** | Hand-object 3D pose / contact. Good for training the "understand the demo" perception |
| **Something-Something v2** | Short object-interaction clips with labels |

### Robot data (learning *how a robot* does it)
| Dataset | Why |
|---|---|
| **Open X-Embodiment** | 1M+ episodes, 20+ robots. Standard pretraining mix |
| **DROID** | Large, diverse Franka teleop "in the wild" |
| **BridgeData V2** | WidowX tabletop, widely benchmarked |
| **RH20T** | **Paired human + robot demos** of the same tasks. Directly relevant to the embodiment gap |
| **EgoMimic** data | Egocentric human + robot co-training |
| **AgiBot World** | Large real-robot dataset, many tasks |
| **LeRobot Hub** (Hugging Face) | Hundreds of SO-100/101, Koch and ALOHA datasets in one format |

### Manufacturing-flavoured
Public data is thin, and that is part of the opportunity. Use **NVIDIA Factory / AutoMate / IndustReal** assets (sim assembly: pegs, gears, nuts), **NIST assembly task boards** (a physical benchmark you can buy or 3D-print), and record your own.

## 3. Simulators and benchmarks for testing one-shot learning

| Sim / benchmark | Use it for |
|---|---|
| **RLBench** (CoppeliaSim) | **The standard one-/few-shot imitation benchmark** (100+ tasks; Instant Policy reports here). Your first leaderboard |
| **ManiSkill3** (SAPIEN, GPU-parallel) | Fast, many robots, demos included. Test **cross-embodiment** (demo by robot A, executed by robot B) |
| **robosuite + robomimic + MimicGen** | MimicGen turns ~10 demos into thousands. Great for generating training tasks |
| **LIBERO** | 130 tasks; test generalizing to *unseen* tasks (LIBERO-90 → LIBERO-10) |
| **Meta-World** (ML10/ML45) | Classic few-shot / meta-learning splits |
| **Isaac Lab** (NVIDIA) + Factory/AutoMate | Contact-rich **assembly** (insertion, gears). Closest to factory work |
| **RoboCasa** | Many objects/scenes; procedural variation |
| **SimplerEnv** | Sim evals that correlate with real Bridge/RT-1 robots |
| **MuJoCo (+ Menagerie, MJX), Genesis** | Build your own custom factory cells quickly |

### How to test "from a *human* video" in sim (the tricky part)
Sim demos are usually robot demos, so build the test in three levels:
1. **Robot → same robot (one-shot):** RLBench one-shot protocol. Tests sub-problem C.
2. **Robot A → robot B:** demo by a Panda, executed by a UR5/xArm in ManiSkill3/robosuite. Tests the embodiment gap without real video.
3. **Human → robot:** render a MANO hand doing the task in sim (as DexMV/ARCTIC-style pipelines do), **or** build a physical mock-up of a sim task, film yourself with a phone, and feed that video to the sim robot. Then move to real hardware.

## 4. Hardware to start with
- **Cheap dev arm:** SO-101 (LeRobot, a few hundred $), or Koch. Great for iteration.
- **"Real" arm for factory credibility:** UFactory xArm 6/7 or Lite6, Franka FR3, or UR3e/UR5e (used ones are cheaper).
- **Gripper:** parallel jaw (Robotiq 2F-85 or similar). Skip dexterous hands at first.
- **Cameras:** 1–2 RealSense D405/D435 or ZED on the robot; an **iPhone Pro with LiDAR** (Record3D app) for recording demos as RGB-D.
- **Compute:** one RTX 4090/5090 workstation, plus cloud GPUs for training.

## 5. Phased plan

### Phase 0 (weeks 0–2): Pick the wedge
- Choose **one task family**: e.g. bin-to-tray **pick-and-place / kitting**, machine tending, or simple insertion. Talk to 5–10 local contract manufacturers. What changes weekly? What do they pay a person to do?
- Define metrics: **success rate from 1 demo on new layouts**, time from demo to running, cycle time vs. a human, interventions per hour.

### Phase 1 (weeks 2–6): Sim harness + baselines
- Set up RLBench + ManiSkill3 with the three-level eval above.
- Reproduce **Instant Policy** one-shot numbers. Add a **fine-tuned VLA baseline** (π0 or SmolVLA, fine-tuned on N demos) to show the value of no fine-tuning.
- Deliverable: a dashboard of one-shot success across 20–30 tasks.

### Phase 2 (weeks 4–10): Track 1 on a real robot
- Build the phone-video → object-poses → waypoints → planner pipeline on an SO-101 / xArm.
- 10 tabletop tasks, 20 trials each with shuffled layouts. Record every failure and why.
- Deliverable: a 2-minute video of "film once, robot does it" on 3 tasks. **This is your pitch.**

### Phase 3 (months 3–6): Track 2, the learned in-context policy
- Pretrain on sim pseudo-demos (Instant-Policy style, plus MimicGen for variety). Co-train on robot data (Open X, DROID, RH20T) and human-hand data (EgoDex, HOT3D) for the human→robot mapping.
- Condition on the demo (video or the extracted object-trajectory graph from Track 1). Use Track 1's perception as the interface.
- Compare against Track 1 on the same real tasks; use it where Track 1 fails (contact, deformables).

### Phase 4 (months 6–12): Pilot
- One paying or free pilot at a small manufacturer, with one cell and one task family.
- Log every failure. That data feeds the next training round (WorldEngine-style flywheel), and you build a dataset nobody else has.
- Decide the business model: sell cells, or robots-as-a-service per hour.

## 6. Prior art and competitors to study
- **Shiraz AI** (YC S26): the exact pitch.
- **Micropsi Industries (MIRAI)**: industrial robot skills from human demos; has existed for years. Learn from what they ship.
- Bigger robot-brain players moving into factories: Physical Intelligence, Skild, NVIDIA GR00T ecosystem.
- Survey to read first: "Learning by Watching: A Review of Video-based Learning Approaches for Robot Manipulation" (arXiv 2402.07127).

## 7. Biggest risks
1. **Reliability:** research demos hit ~60–90%; factories need 99%+. Track 1 + good engineering is how you close the gap.
2. **Ambiguous demos:** one video doesn't say what *matters* (does orientation matter? tolerance?). Plan a 10-second "confirm" UI for the worker.
3. **Hardware and integration:** fixtures, safety, PLC integration eat time. Budget for them.
4. **Data moat:** public data has almost no factory tasks. Your pilots *are* your dataset.

## Sources
- Instant Policy: https://arxiv.org/abs/2411.12633 · https://www.robot-learning.uk/instant-policy
- EgoDex: https://arxiv.org/abs/2505.11709 · https://github.com/apple/ml-egodex
- OKAMI: https://arxiv.org/abs/2410.11792
- MimicFunc: https://arxiv.org/abs/2508.13534
- One-Shot Imitation under Mismatched Execution: https://arxiv.org/abs/2409.06615
- One-Shot Imitation, Pose Estimation Perspective: https://arxiv.org/abs/2310.12077
- TDV (one-shot human video, Oct 2026): https://arxiv.org/abs/2610.21792
- HOST: https://arxiv.org/abs/2607.20033
- Single demo → general contact-rich policy: https://arxiv.org/abs/2605.17601
- Video-based learning survey: https://arxiv.org/abs/2402.07127
- Other datasets, sims and models above are from general knowledge; check their current licenses and versions.
