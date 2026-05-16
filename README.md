# Awesome VLA Steering

A curated list of papers on **inference-time steering of Vision-Language-Action (VLA) models** and closely related work on diffusion/flow-matching guidance, action experts, and steerable policies.

Steering here means: changing the behavior of a (typically frozen) base policy at deployment through context, sampling, external verifiers, world models, constraint projection, lightweight modules, observation transforms, or online residuals — without standard retraining of the main model.

## Contents

- [1. Verifier-Based Selection (Best-of-N / Re-ranking)](#1-verifier-based-selection-best-of-n--re-ranking)
- [2. Denoising-Loop Guidance (Internal Sampling Steering)](#2-denoising-loop-guidance-internal-sampling-steering)
  - [2.1 Value / Q-Function Guidance](#21-value--q-function-guidance)
  - [2.2 World-Model / Dynamics Guidance](#22-world-model--dynamics-guidance)
  - [2.3 VLM / Programmatic-Reward Guidance](#23-vlm--programmatic-reward-guidance)
  - [2.4 Tree Search and SMC](#24-tree-search-and-smc)
- [3. Human-in-the-Loop Steering](#3-human-in-the-loop-steering)
- [4. Observation / Representation / Decoding Interventions](#4-observation--representation--decoding-interventions)
- [5. Goal / Subgoal / Trajectory Conditioning](#5-goal--subgoal--trajectory-conditioning)
- [6. Memory-Augmented Inference](#6-memory-augmented-inference)
- [7. Test-Time Initialization](#7-test-time-initialization)
- [8. Online Residual / Latent RL](#8-online-residual--latent-rl)
- [9. Native Steerability via Training-Time Design](#9-native-steerability-via-training-time-design)
- [10. Related: CV Diffusion / Flow-Matching Guidance](#10-related-cv-diffusion--flow-matching-guidance)
- [Unifying Theoretical Lens](#unifying-theoretical-lens)

---

## 1. Verifier-Based Selection (Best-of-N / Re-ranking)

The base policy emits N candidates; an external scorer picks the best. The policy stays frozen and is treated as a black box.

- [**V-GPS: Steering Your Generalists — Improving Robotic Foundation Models via Value Guidance**](https://arxiv.org/abs/2410.13816) (Nakamoto et al., CoRL 2024) — Offline Cal-QL value function re-ranks actions from any generalist VLA (Octo, RT-1-X, etc.). 5 policies × 12 tasks; ~1.28–1.59× single-step overhead. The canonical instance of advantage-weighted regression executed at inference time.
- [**RoVer: Robot Process Reward Model**](https://arxiv.org/abs/2510.10975) (Dai et al.) — PRM returns both a scalar score *and* a direction in action space for zeroth-order gradient guidance. Caches perception features across candidates.
- [**FOREWARN: From Foresight to Forethought — VLM-In-the-Loop Policy Steering via Latent Alignment**](https://arxiv.org/abs/2502.01828) (Wu et al., RSS 2025) — Decouples *foresight* (latent dynamics model predicts future latent observations) from *forethought* (small VLM scores a textual "behavior narration" of the latent). Solves the problem that VLMs can't natively reason about low-level actions. ~3.7s/decision vs. VLM-Act's 22s.
- [**UF-OPS: Update-Free On-Policy Steering via Verifiers**](https://arxiv.org/abs/2603.10282) (Attarian et al.) — Trains small verifiers from the policy's *own* deployed rollouts (successes + failures). Self-bootstrapping verifier; +49% on real tasks.
- [**Do What You Say: Steering Vision-Language-Action Models via Runtime Reasoning-Action Alignment**](https://arxiv.org/abs/2509.18130) (Wu 2025) — VLM-generated CoT acts as both target plan and verifier; filters action trajectories inconsistent with the spoken plan.
- [**PPGuide: Performance-Predictor Guidance**](https://arxiv.org/abs/2603.10980) — Binary success classifier provides gradient signal to push samples away from failure modes.
- [**When to Act, Ask, or Learn**](https://arxiv.org/abs/2602.22474) (Yuan, Wu, Bajcsy) — Important caveat: VLM verifiers are typically miscalibrated; proposes uncertainty-aware decisions between acting, asking for help, or collecting more data.

---

## 2. Denoising-Loop Guidance (Internal Sampling Steering)

Modifies the diffusion/flow sampling dynamics directly rather than selecting after the fact.

### 2.1 Value / Q-Function Guidance

- [**VGD: Steering Diffusion Policies with Value-Guided Denoising**](https://openreview.net/forum?id=wrcTncImde) (NeurIPS 2025 workshop) — At each DDIM step, computes one-step clean-action estimate x̂₀ and adds ∇ₐQ(s, x̂) to predicted noise. Avoids backprop through the diffusion chain.
- [**DSRL: Steering Your Diffusion Policy with Latent Space Reinforcement Learning**](https://arxiv.org/abs/2506.15799) (Wagenmaker, Nakamoto et al., CoRL 2025) — Most theoretically grounded latent steering. Keeps BC policy frozen, learns an RL policy π_w(w|s) over the *initial noise*. Black-box, low-dimensional noise space, no unstable backward pass. Extended to π₀ on WidowX/Aloha.
- [**USR: Unified Steering and Residual Refinement**](https://openreview.net/forum?id=DbBD2aT1OG) — Combines DSRL-style noise steering with a residual action correction, since pure latent steering is mode-confined.

### 2.2 World-Model / Dynamics Guidance

- [**DynaGuide: Steering Diffusion Policies with Active Dynamic Guidance**](https://arxiv.org/abs/2506.13922) (Du & Song, NeurIPS 2025) — Separately trained latent dynamics model predicts future DINOv2 embeddings; log-sum-exp over guidance images gives a differentiable classifier-guidance signal. 70% steering success on CALVIN; 5.4× over goal-conditioning under weak goal descriptions.
- [**LPB: Latent Policy Barrier**](https://arxiv.org/abs/2508.05941) — Latent dynamics defines a barrier function approximating expert support; rejects/projects candidates leaving the support. Essentially a learned-latent CBF.
- [**LatentCBF: Latent Control Barrier Functions for Visuomotor Policies**](https://arxiv.org/abs/2511.18606) — Explicit, smooth, differentiable latent CBF; more permissive than least-restrictive filters.
- [**GPC: Generative Predictive Control**](https://arxiv.org/abs/2502.00622) — Two modes: GPC-RANK (best-of-N in world model) and GPC-OPT (gradient refinement). MPC-with-foresight wrapper.

### 2.3 VLM / Programmatic-Reward Guidance

- [**VLS: Steering Pretrained Robot Policies via Vision-Language Models**](https://arxiv.org/abs/2602.03973) (Liu et al., 2026) — VLM grounds OOD obs+language into keypoints + stage-wise differentiable programmatic rewards; injects gradients into denoising + Feynman-Kac particle resampling with RBF repulsion. +31% CALVIN, +13% LIBERO-PRO; verified on Franka.
- [**VLA-Pilot: Plug-and-Play Inference-Time VLA Policy Steering via Embodied Evolutionary Diffusion**](https://arxiv.org/abs/2511.14178) (Li et al.) — "Embodied CoT + Evolutionary Diffusion + Iterative Refinement." Zero-finetuning, cross-embodiment. MLLM generates interpretable steering objectives; evolutionary mutation in noise space (Restart-sampling cousin). ~+31% over base VLAs; approaches 50-demo finetuning.
- [**ProgressVLA: Progress-Guided Diffusion Policy for Vision-Language Robotic Manipulation**](https://arxiv.org/abs/2601.20239) (Yan et al., 2026) — Progress estimator + inverse-dynamics world model + classifier guidance, **and** distills guided targets back into the denoiser. Transitional method bridging external guidance and internal action-expert capability.
- [**TouchGuide: Tactile-Guided Steering of Pretrained Visuomotor Policies**](https://arxiv.org/abs/2603.24584) — Tactile contact physics as classifier guidance for diffusion/flow policies. Visual coarse action → tactile-feasibility refinement.
- [**TAG: Target-Aware Guidance for Vision-Language-Action Models**](https://arxiv.org/abs/2602.22056) — CFG-style: dual branches on original obs and object-erased counterfactual obs; residual guides toward target evidence. π0.5: 95.2→97.9 on LIBERO; reduces near-miss/wrong-object on LIBERO-Plus and VLABench.

### 2.4 Tree Search and SMC

- [**TDP: Tree-Guided Diffusion Planner**](https://arxiv.org/abs/2508.21800) — Parent trajectories sampled with particle guidance (diversity); child sub-trajectories with gradient-guided denoising (exploitation). Bi-level explore/exploit.
- [**FORGE-Tree: Monte Carlo Tree Diffusion for Long-Horizon VLA**](https://arxiv.org/abs/2510.21744) — MCTD over long-horizon VLAs; partially denoises trajectory segments using frozen OpenVLA/Octo encoder. +13.4–17.2 pp on LIBERO.

---

## 3. Human-in-the-Loop Steering

- [**ITPS: Inference-Time Policy Steering through Human Interactions**](https://arxiv.org/abs/2411.16627) (Wang et al., ICRA 2025) — Foundational paper. Six families: post-hoc perturbation, ranking, initialization, gradient-guided sampling, **stochastic sampling** (the winner — DDPM resampling from a conditional kernel), and biased prior. Stochastic sampling dominates the alignment-vs-OOD trade-off.
- [**Yell at Your Robot: Improving On-the-Fly from Language Corrections**](https://arxiv.org/abs/2403.12910) (Shi et al.) — Real-time language corrections to a hierarchical policy; corrections also stored for offline updates.
- [**Steering Robots with Inference-Time Interactions**](https://arxiv.org/abs/2506.14287) (Wang) — Systematizes ITPS + PoCo-style composition + TAMI hard-mode classifiers.

---

## 4. Observation / Representation / Decoding Interventions

Lightweight interventions at input, internal features, or decoding — typically the cheapest engineering path.

- [**BYOVLA: Run-time Observation Interventions Make Vision-Language-Action Models More Visually Robust**](https://arxiv.org/abs/2410.01971) (Hancock et al., 2024) — VLM identifies task-irrelevant regions; sensitivity probes find regions the frozen VLA depends on; minimal image edits to the intersection. Black-box, recovers near-nominal performance under distractors.
- [**PCD: Policy Contrastive Decoding for Robotic Foundation Models**](https://arxiv.org/abs/2505.13255) (Wu et al., ICLR 2026) — Action distribution from original image vs. object-masked image; contrastive decoding suppresses spurious visual cues. +50.6% / 29.7% / 8.9% on OpenVLA / Octo / π₀.
- [**Observing and Controlling Features in Vision-Language-Action Models**](https://arxiv.org/abs/2604.17880) (Buurmeijer et al., 2026) — Linear observer/controller on internal features of OpenVLA and π-class models; real-time steering via minimal linear intervention. Low latency, single-GPU friendly.
- [**Mechanistic Interpretability for Steering Vision-Language-Action Models**](https://arxiv.org/abs/2509.00328) (CoRL 2025) — Demonstrates steerable neurons for velocity / direction / grasp in Pi0 and OpenVLA; zero-shot behavior control via direct activation.
- [**DeLock: Breaking Lock-In — Preserving Steerability under Low-Data VLA Post-Training**](https://arxiv.org/abs/2602.10556) (Huang et al., 2026) — Identifies "lock-in": low-data SFT destroys steerability. Mitigation: preserve visual grounding during training + test-time contrastive prompt guidance.

---

## 5. Goal / Subgoal / Trajectory Conditioning

- [**SuSIE: Subgoal Synthesis via Image Editing**](https://arxiv.org/abs/2310.10639) (Black, Nakamoto et al.) — InstructPix2Pix edits current obs into a subgoal image; goal-conditioned low-level policy follows it. Frozen at deployment.
- [**RT-Trajectory: Robotic Task Generalization via Hindsight Trajectory Sketches**](https://arxiv.org/abs/2311.01977) (DeepMind, ICLR 2024) — Conditions RT-1 on 2D/2.5D trajectory sketches from humans, generators, or planners. "Sketch as prompt."
- [**CoT-VLA: Visual Chain-of-Thought Reasoning for VLA Models**](https://arxiv.org/abs/2503.22020) (Zhao et al.) — Autoregressively generates a future image as visual chain-of-thought, then attends to it for action prediction.
- [**ECoT: Embodied Chain-of-Thought**](https://arxiv.org/abs/2407.08693) — Language analogue: plan / subtask / box / EE-position reasoning text before action.
- [**ForeAct: Foresight-Guided Action Steering**](https://arxiv.org/abs/2602.12322) — Fast (0.33s on H100) foresight image generator + VLM subtask describer steers frozen π₀ at every step. +40.9% over baseline.

---

## 6. Memory-Augmented Inference

- [**MemoryVLA: Perceptual-Cognitive Memory for VLA Models**](https://arxiv.org/abs/2508.19236) — Perceptual + cognitive memory bank.
- [**ReMem-VLA: Recurrent Dual-Level Memory for VLA**](https://arxiv.org/abs/2603.12942) — Recurrent dual-level memory queries.
- [**MAP-VLA: Memory-Augmented Prompting for VLA**](https://arxiv.org/abs/2511.09516) — Retrieval of demonstration prompts; essentially inference-time prompt tuning.
- [**OptimusVLA: Task Prior and Local Consistency Memory**](https://arxiv.org/abs/2602.20200) — Task-level prior memory + local consistency; replaces isotropic noise with a retrieved task-prior. Memory-augmented noise-space initialization.

---

## 7. Test-Time Initialization

- [**ADPro: Action-Diffusion with Manifold-Projected Priors**](https://arxiv.org/abs/2508.06266) — Replaces isotropic noise with a manifold-projected, task-aware prior. Robotics analogue of CV's MPGD.

---

## 8. Online Residual / Latent RL

Base policy untouched but small modules trained online — the boundary of "training-free."

- [**Policy Decorator: Model-Agnostic Online Refinement for Large Policy Models**](https://arxiv.org/abs/2412.13630) — Trains a model-agnostic residual policy via online RL on top of frozen large BC policies; bounded exploration. 8 tasks across ManiSkill / Adroit.
- [**FlowCorrect: Online Correction of Pretrained Flow Policies**](https://arxiv.org/abs/2605.11809) — Frozen flow policy + lightweight LoRA/gating correction module trained from small amounts of human relative corrections.

---

## 9. Native Steerability via Training-Time Design

Not strictly inference-time, but the resulting interface is steered at inference without retraining.

- [**π0.7: A Steerable Generalist Robotic Foundation Model**](https://arxiv.org/abs/2604.15483) (Physical Intelligence, 2026) — Language + subgoal images + episode metadata + control-mode prompt; classifier-free guidance applied to metadata. Cleanest existing example of a trained-in conditioning channel steered at inference. ~120ms main inference, ~1.25s async subgoal generation.
- [**Hume: Introducing System-2 Thinking in Visual-Language-Action Model**](https://arxiv.org/abs/2505.21432) (Song et al., 2025) — Dual-system: slow value-guided System-2 selects long-horizon action chunks via best-of-N; fast System-1 does cascaded denoising control asynchronously. +4.4 / +25.9 / +12.9% on LIBERO / Simpler / real.
- [**RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control**](https://arxiv.org/abs/2307.15818) (CoRL 2023) — Actions as text tokens, co-trained with web-scale VL/VQA data. Foundational for "action-as-language."
- [**LAP: Language-Action Pretraining**](https://arxiv.org/abs/2604.23121) (2026) — Low-level actions expressed in natural language to align with the pretrained VLM I/O distribution. ~2× over prior VLAs zero-shot on new embodiments.
- [**ST-π: Spatiotemporal VLA with Structured Action Expert**](https://arxiv.org/abs/2601.04052) (2026) — Spatiotemporal VLM + structured action expert; chunk-level prompts with explicit semantic / spatial / temporal attributes. LIBERO avg 97.4, STAR 80.1.
- [**RSS: Stable Language Guidance for VLAs**](https://openreview.net/forum?id=YvsUD8C9QS) (2026) — Monte-Carlo syntactic integration + residual affordance steering for instruction-blindness robustness.
- [**ATE: Align-Then-Steer Across Embodiments**](https://openreview.net/forum?id=T3i7Ifeatk) (ICLR 2026 Poster) — VAE + reverse-KL unifies action latent space, then steers diffusion/flow VLAs across embodiments. +9.8% sim, +32% real cross-embodiment.

---

## 10. Related: CV Diffusion / Flow-Matching Guidance

Background machinery — many VLA steering methods are robotics imports of these.

### Classifier and Classifier-Free Guidance

- [**Diffusion Models Beat GANs on Image Synthesis**](https://arxiv.org/abs/2105.05233) (Dhariwal & Nichol, 2021) — Classifier guidance.
- [**Classifier-Free Diffusion Guidance**](https://arxiv.org/abs/2207.12598) (Ho & Salimans, 2022) — The CFG that everyone uses.
- [**Autoguidance: Guiding a Diffusion Model with a Bad Version of Itself**](https://arxiv.org/abs/2406.02507) (Karras et al., NeurIPS 2024) — Replaces unconditional model with a degraded conditional one. Record FIDs.
- [**In-Situ Autoguidance**](https://arxiv.org/abs/2510.17136) — Achieves autoguidance without an auxiliary model via stochastic perturbation.
- [**PAG: Perturbed-Attention Guidance**](https://arxiv.org/abs/2403.17377) — Training-free, conditioning-free quality booster by perturbing self-attention.

### Reward-Guided Sampling

- [**DPS: Diffusion Posterior Sampling**](https://arxiv.org/abs/2209.14687) (Chung et al., 2022) — Foundation of most training-free guidance.
- [**Universal Guidance for Diffusion Models**](https://arxiv.org/abs/2302.07121) (Bansal, Goldstein et al.) — Any off-the-shelf classifier + self-recurrence.
- [**MPGD: Manifold Preserving Guided Diffusion**](https://arxiv.org/abs/2311.16424) (He, Ermon et al., ICLR 2024) — Projects guidance onto data-manifold tangent space; 3.8× faster.
- [**DOODL: Direct Optimization of Diffusion Latents**](https://arxiv.org/abs/2303.13703) (Wallace, Ermon, Naik, ICCV 2023) — Uses EDICT for invertible diffusion ODE; exact gradients w.r.t. initial noise.
- [**ReNO: Reward-Based Noise Optimization**](https://arxiv.org/abs/2406.04312) (Eyring et al., NeurIPS 2024) — 100× faster than DOODL on one-step models; combines 4 rewards to avoid hacking.
- [**SVDD: Soft Value-Based Decoding in Diffusion**](https://arxiv.org/abs/2408.08252) (Li et al.) — SMC framework with intermediate-step potentials.
- [**FK Steering: Feynman-Kac Steering of Diffusion Models**](https://arxiv.org/abs/2501.06848) (Singhal et al.) — Subsumes TDS and SVDD.
- [**Inference-Time Scaling for Diffusion Models**](https://arxiv.org/abs/2501.09732) (Ma et al., 2025) — Design-space paper: random search over noise, zero-order search, search over paths.

### Compositional / Energy-Based

- [**Composable Diffusion Models**](https://arxiv.org/abs/2206.01714) (Liu, Du et al., ECCV 2022) — Treats multiple conditions as score functions, adds them.
- [**Reduce, Reuse, Recycle: Compositional Generation with Energy-Based Diffusion Models and MCMC**](https://arxiv.org/abs/2302.11552) (Du, NeurIPS 2023) — Corrects ill-conditioned product via Langevin/HMC MCMC at intermediate noise.

### Editing and Inversion

- [**SDEdit: Image Synthesis and Editing with Stochastic Differential Equations**](https://arxiv.org/abs/2108.01073) (Meng et al., 2022) — Add noise to a real input, re-denoise with new condition.
- [**Prompt-to-Prompt Image Editing with Cross-Attention Control**](https://arxiv.org/abs/2208.01626) (Hertz et al., 2022) — Edit images via cross-attention map control.

### Particle and Restart Methods

- [**Particle Guidance**](https://arxiv.org/abs/2310.13102) (Corso et al., ICLR 2024) — Non-IID sampling with joint particle potential for diversity.
- [**Restart Sampling for Improving Generative Processes**](https://arxiv.org/abs/2306.14878) (Xu, Jaakkola, NeurIPS 2023) — Alternates ODE-deterministic denoising with forward noise injections.

### Control and Adapter Methods

- [**ControlNet: Adding Conditional Control to Text-to-Image Diffusion Models**](https://arxiv.org/abs/2302.05543) (Zhang et al.) — Locked backbone + zero-conv side branch.
- [**T2I-Adapter: Lightweight Adapters for Diffusion Models**](https://arxiv.org/abs/2302.08453) (Mou et al.) — Small adapters provide structural control.

### Flow Matching

- [**Flow Matching for Generative Modeling**](https://arxiv.org/abs/2210.02747) (Lipman et al.) — Foundational flow matching paper.
- [**Guided Flows for Generative Modeling and Decision Making**](https://arxiv.org/abs/2311.13443) (Zheng et al., 2023) — CFG for flow matching; decision-making bridge.
- [**CFG-Zero***](https://arxiv.org/abs/2503.18886) (Fan et al.) — Early zeroing + optimized guidance scale for flow-matching CFG.
- [**Improving CFG of Flow Matching via Manifold Projection**](https://arxiv.org/abs/2510.05529) (Cai et al.) — Manifold projection + Anderson acceleration for stable flow CFG.

---

## Unifying Theoretical Lens

Almost every method above approximates the same **reward-tilted posterior**:

$$p^*(a \mid s) \propto p_\theta(a \mid s) \cdot \exp(\beta \cdot r(a, s))$$

This is exactly Levine's control-as-inference soft-optimal policy. Methods differ along three orthogonal axes:

| Axis | Options |
|------|---------|
| **What r is** | offline value (V-GPS, VGD); PRM (RoVer); world-model latent (DynaGuide, LPB, FOREWARN); programmatic VLM reward (VLS); VLM scalar (VLA-Pilot); human (ITPS); progress (ProgressVLA); tactile (TouchGuide); self-consistency (UF-OPS) |
| **How the signal acts** | re-ranking (V-GPS, FOREWARN); gradient classifier guidance (DynaGuide, VLS, VGD, PPGuide); SMC / particle resampling (VLS); evolutionary mutation (VLA-Pilot); RL on noise (DSRL, USR); residual (Policy Decorator); MCTS over denoising (FORGE-Tree); tree search over noise (TDP) |
| **Where it's enforced** | t = 0 only (post-hoc selection); every t (continuous guidance); chosen t's (FK resampling); t = T (noise-space optimization) |

---

## Contributing

Pull requests welcome. When adding a paper, please:

1. Place it in the most specific section that applies.
2. Use the title as the linked text, with the link pointing to the arXiv abstract page (or official venue page if no arXiv version exists).
3. Add a one- to two-sentence summary noting the steering mechanism and the headline result.

## License

This list is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
