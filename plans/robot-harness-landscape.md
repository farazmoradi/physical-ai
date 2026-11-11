# Robot harnesses on GitHub: landscape (2026-11-11)

Star counts and last-push dates come from a GitHub search on 2026-11-11. Descriptions come from each repo's README (self-reported; not tested by me).

"Harness" means two different things in robotics:
- **Eval harness:** run a robot *policy* (VLA, imitation policy, scripted) against tasks and score it. Like lm-eval-harness / Inspect, for robots.
- **Agent harness:** let an *LLM/VLM agent* drive a robot through tools (MCP, code, skills). Like Claude Code, for robots.
- The interesting new zone is **both at once**: a harness that evaluates *and* trains (RL) agents on robot tasks.

## 1. Eval harnesses (policy → benchmark → score)

| Repo | ★ | What it is | Notes |
|---|---|---|---|
| [allenai/vla-evaluation-harness](https://github.com/allenai/vla-evaluation-harness) | 678 | "One framework to evaluate any VLA on any sim benchmark." 18 benchmarks × 13 model servers (LIBERO, CALVIN, SimplerEnv, ManiSkill, RoboCasa…) | Benchmarks in Docker, models as uv-script servers. Batch-parallel (2,000 LIBERO episodes in 18 min on 1 H100). LeRobot bridge (π0/π0.5, GR00T…). Public leaderboard. Agent skills for adding benchmarks. **The closest thing to lm-eval-harness for robots.** |
| [NVlabs/RoboLab](https://github.com/NVlabs/RoboLab) | 613 | NVIDIA task-based benchmark on Isaac Lab: 120+ manipulation tasks with automatic success detection | Bring-your-own robot; server-client policy interface; results dashboard with videos; **Claude Code skills that generate new scenes and tasks from natural language**. Needs NVIDIA GPU. |
| [simpler-env/SimplerEnv](https://github.com/simpler-env/SimplerEnv) | 1,180 | Sim evals built to *correlate with real-robot* results (RT-1, Octo, OpenVLA on Google robot / WidowX) | De-facto standard sim proxy for real performance |
| [isaac-sim/IsaacLab-Arena](https://github.com/isaac-sim/IsaacLab-Arena) | 653 | NVIDIA framework for composing large-scale sim evaluations on Isaac Lab | [LightwheelAI/LW-BenchHub](https://github.com/LightwheelAI/LW-BenchHub) (207★) builds a benchmark hub on it |
| [robo-arena/roboarena](https://github.com/robo-arena/roboarena) | 116 | **Distributed real-robot** benchmarking: serve your policy remotely, labs around the world evaluate it on real DROID robots | Policy-server interface; pairs with openpi |
| [InternRobotics/InternManip](https://github.com/InternRobotics/InternManip) | 176 | All-in-one manipulation train + eval suite | |
| Smaller, recent: [praxis-eval](https://github.com/Chaoqi-LIU/praxis-eval), [ManiLoop](https://github.com/Daniel-rmc/ManiLoop), [FluxThemis](https://github.com/FluxVLA/FluxThemis), [VLA-Arena](https://github.com/PKU-Alignment/VLA-Arena) (222★), [VLABench](https://github.com/OpenMOSS/VLABench) (482★) | | Cross-benchmark VLA evaluation | Signals a crowded but unconsolidated space |

## 2. Eval harnesses for real robots + agents

| Repo | ★ | What it is | Notes |
|---|---|---|---|
| [robocurve/inspect-robots](https://github.com/robocurve/inspect-robots) | 754 | **"Inspect AI for robotics."** Define a benchmark once, then run any policy (VLA *or* LLM agent) on any embodiment (real arm, humanoid, sim) | Auditable logs (grader scores, LLM transcript, config), Rerun live viewer, plugin per rig (e.g. YAM arms + MolmoAct2). LLM-agent plugin drives the arm via guard-checked tool calls. Core is numpy-only, so it runs on a Mac for sim/API work; real-robot control is Linux-only. Early, API may change. |
| [Noietch/EVA-CLIENT](https://github.com/Noietch/EVA-CLIENT) | 334 | One tool for **deploy / evaluate / collect data / RL** on real robots | Policy backends: openpi, GR00T, StarVLA. 8 robots. Multi-checkpoint eval sweeps with per-trial video + milestone scores. Human-in-the-loop takeover during rollouts. Exports LeRobot / MCAP / HDF5. |
| [Ice-Citron/Matrix-Lab](https://github.com/Ice-Citron/Matrix-Lab) | 3 | **Coverage-driven edge-case hunting**: pushes a policy from 90% to 99.99% reliability by searching for its failure cases in sim (MuJoCo, ManiSkill) | Tiny, but the thesis ("the remaining nines") is exactly the factory-reliability problem |

## 3. Agent harnesses (an LLM/VLM drives the robot)

| Repo | ★ | What it is | Notes |
|---|---|---|---|
| [dimensionalOS/dimos](https://github.com/dimensionalOS/dimos) | 4,740 | "Agentic operating system for physical space": runtime + SDK for navigation, manipulation, perception on humanoids, quadrupeds, drones | Agent CLI + MCP; blueprints; remote teleop; replay demos |
| [OpenMind/OM1](https://github.com/OpenMind/OM1) | 2,965 | Modular AI hardware-abstraction layer for robots | |
| [robotmcp/ros-mcp-server](https://github.com/robotmcp/ros-mcp-server) | 1,521 | Connects Claude/GPT/Gemini to **any ROS 1/2 robot** via MCP + rosbridge, with no robot code changes | Demos: Claude diagnosing an industrial end-effector from manuals; Go2 in Isaac Sim. Related: [wise-vision/ros2_mcp](https://github.com/wise-vision/ros2_mcp), [nav2_mcp_server](https://github.com/ajtudela/nav2_mcp_server) |
| [RobotecAI/rai](https://github.com/RobotecAI/rai) | 615 | Vendor-agnostic agentic framework on ROS 2: multi-agent, speech, perception, sim connectors | Includes **rai_bench** for benchmarking agents/models/tools |
| [rokbenko/quackd](https://github.com/rokbenko/quackd) | 294 | "A command line for the robots you own": each robot joins via a manifest adapter; an LLM plans across a *flock* | Runs on SO-101 via LeRobot, Pollen Microduck, Open Duck Mini; sims included |
| [omnilink-tech/omnisim](https://github.com/omnilink-tech/omnisim) | 192 | "Robotics workshop for agents": high-fidelity sim, digital twins, even CNC/laser/3D-printer twins, all driven by talking to an agent | |
| [NVIDIA/skills](https://github.com/NVIDIA/skills) | 3,933 | Agent Skills for NVIDIA physical-AI products, for Claude Code, Codex etc. | Also [robotics-agent-skills](https://github.com/arpitg1304/robotics-agent-skills) (401★), [ros2-engineering-skills](https://github.com/dbwls99706/ros2-engineering-skills) (237★) |
| [waltstephen/CaPtain-Harness](https://github.com/waltstephen/CaPtain-Harness) | 5 | **Code-as-policy harness**: a VLM controls the robot by writing Python against 22 low-level primitives, with text "receipts", hidden ground-truth state, scored on final state | Same API across LIBERO, RoboTwin 2.0, SimplerEnv, CaP-Bench **and real arms**. Built for **training**: server-side budgets, replayable transcripts, RL hooks. Used to post-train **CaPtain-4B (Qwen, SFT + RL)**. Tiny star count, very relevant design. |

## 4. Foundations everything plugs into
[huggingface/lerobot](https://github.com/huggingface/lerobot) (28.8k★, datasets, policies, SO-101) · [Physical-Intelligence/openpi](https://github.com/Physical-Intelligence/openpi) (14.6k★) · [NVIDIA/Isaac-GR00T](https://github.com/NVIDIA/Isaac-GR00T) (8.4k★) · [isaac-sim/IsaacLab](https://github.com/isaac-sim/IsaacLab) (8.6k★) · [robosuite](https://github.com/ARISE-Initiative/robosuite) (2.7k★) · [LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO) (2.5k★) · [RLBench](https://github.com/stepjam/RLBench) (1.8k★)

## 5. Patterns worth copying
- **Policy as a server, benchmark as a client** (vla-eval-harness, RoboLab, RoboArena, inspect-robots). Swap either side freely.
- **Benchmarks in Docker**, models as self-contained scripts, to escape dependency hell.
- **Hidden ground truth, scored on final state** (CaPtain). Prevents the agent "seeing" the answer and makes RL rewards clean.
- **Auditable logs + replay** (Rerun `.rrd`, transcripts, per-trial video).
- **Agent skills / natural-language task generation** (RoboLab, vla-eval-harness, NVIDIA skills).
- **Coverage-driven failure search** (Matrix Lab) instead of fixed episode counts.

## 6. Gaps = where we could build
1. **One-shot / demo-conditioned evaluation.** No harness above treats "here is one demo video, now do it under these variations" as a first-class protocol. A plugin for inspect-robots or vla-eval-harness that does this (demo + perturbation sweep + success scoring, sim and SO-101) is directly useful to Shiraz-type companies.
2. **Harness for RL post-training of robot agents.** CaPtain shows it works (4B model, SFT + RL) but it's a research repo. A clean "gym for robot agents" with verifiable rewards, budgets and transcripts maps directly onto LLM post-training skills (GRPO, reward design, eval gating).
3. **Reliability harness for factory tasks:** coverage-driven stress tests and regression suites built from real failures, the "release-gating eval framework" idea applied to robots.

## Suggested first step
Install **inspect-robots** (works on a Mac for sim) and **vla-evaluation-harness** (rented GPU), run one LIBERO / SimplerEnv eval end-to-end, read **CaPtain-Harness**'s design, then prototype gap #1 or #2 on MuJoCo + an SO-101.
