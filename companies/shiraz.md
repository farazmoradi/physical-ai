# Shiraz AI

- **Site:** https://shiraz.ai/
- **HQ:** San Francisco
- **Founded:** 2026 · Y Combinator **S26**
- **Team (2):** Navid Aghasadeghi (CEO), Mojtaba Mozaffar (CTO)
- **Funding:** YC. PatentVC and AITFund say on LinkedIn that they invested (no amounts). No public round on record.
- **Category:** Full-stack robots for manufacturing (hardware + foundation model + deployment)
- **Last researched:** 2026-11-06

## What they do in physical AI

Tagline: **"Robots that learn on the job."**

Shiraz builds a **robot foundation model that learns a new manufacturing task from a single human demonstration**, and delivers it **working on your line: hardware, model, and deployment.**

- **First market:** contract manufacturing, where production lines change every week. They argue both traditional automation (hand-programmed, costly to change) and today's general robot models (VLAs) break down in that setting.
- The company says its robots are **already running at customer sites**.

## How it works (deeper dive, 2026-11-08)

1. **Show it once, on a phone.** A factory worker films themselves doing the task one time with a phone. No teleoperation rig, no data-collection campaign.
2. **The model watches the video as context.** Shiraz's robot foundation model takes the human video as a prompt (**in-context learning**) and works out how to do the same task with *its own* robot body. Going from human hands to robot grippers is the **"embodiment gap"** they say they bridge.
3. **No retraining.** No overnight fine-tuning and no engineer visit. The model adapts on the spot, including to new variations of the task.
4. **The robot runs the task on the line.** When the product changes next week, film the new task and repeat.

**Why they think this matters:** In contract manufacturing, lines change weekly. Classic automation is hand-programmed and only pays off for long, unchanging runs. Today's vision-language-action models (VLAs) usually need lots of demonstrations plus fine-tuning for each new task. Shiraz pitches **in-context learning as the missing piece in current VLA approaches**.

**What they sell:** a turnkey package of robot hardware + foundation model + on-site deployment. The specific robot or arm isn't disclosed publicly.

**Traction claims:** the website says systems are "operational at customer sites", and a LinkedIn post says they're deploying to first customers. The YC profile describes onboarding *pilot partners*. No customers are named.

**Open questions:** which hardware they use; how reliable one-shot learning is (success rate, cycle time compared with a human); what kinds of tasks (pick-and-place, kitting, packing, assembly?); pricing (robot sale versus robots-as-a-service).

Additional sources: https://yclist.com/s26/shiraz-ai/ · https://www.extruct.ai/data-room/ycombinator-companies-s26/ · https://x.com/realnavida · https://streamlinefeed.co.ke/news/yc-summer-2026-ai-factories-health-schools-markets

## Team background
- **Navid Aghasadeghi (CEO):** ex-Senior Staff Robotics Engineer at Boston Dynamics, on the core team that took Spot from R&D to commercial deployment and led industrial manipulation. Ex-Principal Robotics Engineer at Rethink Robotics (Sawyer controls, force control). Founded GenZArt (consumer AI app). PhD ECE, UIUC.
- **Mojtaba Mozaffar (CTO):** whole-body control and bimanual manipulation at Amazon Robotics. Former research professor at Northwestern. PhD ME/Robotics, Northwestern.

## In plain words
A factory shows the robot a job once ("take this part, put it there"). The robot learns it and starts doing it. When the factory switches to a new product next week, someone shows it the new job, with no weeks of reprogramming. Shiraz brings the robot, the brain and the setup.

## Take
The only company of the five that sells **working robots to end users** rather than tools, data or models to robot builders. The bet: high-mix, low-volume factories are an underserved market because normal automation only pays off for long, unchanging runs. Main risks: one-demo learning has to be reliable enough for production, and full-stack (hardware + deployment) businesses scale slowly.

## Caveats
Self-reported (site, YC profile, LinkedIn). No independent coverage of customers or funding found. The site itself was blocked from the research environment; details come from search results.

## Sources
- https://shiraz.ai/
- https://www.ycombinator.com/companies/shiraz-ai
- https://www.ycombinator.com/launches/Si7-shiraz-ai-robots-that-learn-on-the-job
- https://www.oneroadmap.io/funded-companies/shiraz-ai
- https://x.com/MozaffarMojtaba
