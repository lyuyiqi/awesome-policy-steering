# Rudder

A curated list of papers on **inference-time steering of Vision-Language-Action (VLA) models**.

Steering here means: changing a (typically pretrained or frozen) VLA's behavior at deployment through prompts, context, internal-feature intervention, observation transforms, action re-ranking, denoising / flow guidance, latent-noise control, or memory — without standard end-to-end retraining.

A paper is included only if it demonstrates its steering mechanism on a VLA (vision-language pretrained backbone driving action prediction — e.g. OpenVLA, Octo, RT-1-X, π₀, π₀.₅, π0.7) or is specifically designed for VLA architectures. Methods that steer only generic diffusion policies or visuomotor policies, even when conceptually similar, are listed in [What's Out of Scope](#whats-out-of-scope).

Papers are split by **where the steering signal acts**:

- **Steering the VLM (semantic / representation layer):** intervenes on prompts, context, the language–vision tower, internal features, attention, or the observation input. These methods reshape *what the model perceives or reasons about* before any action is committed.
- **Steering the Action Expert (sampling / generation layer):** intervenes on action proposals, candidate ranking, denoising / flow dynamics, latent noise, or output decoding. These methods reshape *how actions are sampled* given a fixed semantic interpretation.

A few works span both layers; they appear under their primary mechanism with a note.

## Contents

- [Steering the VLM](#steering-the-vlm-semantic--representation-layer)
- [Steering the Action Expert](#steering-the-action-expert-sampling--generation-layer)
- [Native Steerability via Training-Time Design](#native-steerability-via-training-time-design)
- [What's Out of Scope](#whats-out-of-scope)

---

## Steering the VLM (semantic / representation layer)

### Internal feature / activation steering

- [**Mechanistic Interpretability for Steering Vision-Language-Action Models**](https://arxiv.org/abs/2509.00328) (Häon et al., CoRL 2025) — Projects FFN activations onto the token embedding basis to find sparse semantic directions (speed, direction, grasp) causally linked to action selection; activation steering at inference with no fine-tuning. Demonstrated on π₀ and OpenVLA, LIBERO and a UR5.
- [**Observing and Controlling Features in Vision-Language-Action Models**](https://arxiv.org/abs/2604.17880) (Buurmeijer et al., 2026) — Formalizes feature-observability and feature-controllability for VLAs; uses linear classifiers to observe features and minimal linear interventions (grounded in optimal control) to steer π₀.₅ and OpenVLA outputs while preserving closed-loop behavior.
- [**DeLock: Breaking Lock-In — Preserving Steerability under Low-Data VLA Post-Training**](https://arxiv.org/abs/2602.10556) (Huang et al., 2026) — Identifies "lock-in": low-data SFT destroys the VLA's instruction-following steerability. Mitigation combines preserved visual grounding during training with test-time contrastive prompt guidance.

### Observation interventions

- [**BYOVLA: Run-time Observation Interventions Make Vision-Language-Action Models More Visually Robust**](https://arxiv.org/abs/2410.01971) (Hancock et al., ICRA 2025) — VLM identifies task-irrelevant regions; sensitivity probes find regions the frozen VLA depends on; minimal image edits to the intersection. Black-box, recovers near-nominal performance under distractors on OpenVLA.

### Decoding-time intervention on the VLM head

- [**PCD: Policy Contrastive Decoding for Robotic Foundation Models**](https://arxiv.org/abs/2505.13255) (Wu et al., ICLR 2026) — Contrasts action distributions from original observation vs. object-masked observation to suppress reliance on spurious visual cues. Training-free plug-in for autoregressive OpenVLA and (via KDE) diffusion-based Octo and π₀; +50.6% / +29.7% / +8.9% in simulation, +108% on real for π₀.

### Foresight / subgoal conditioning at inference

- [**ForeAct: Foresight-Guided Action Steering**](https://arxiv.org/abs/2602.12322) (2026) — Fast (0.33s) foresight image generator plus a VLM subtask describer steers frozen π₀ at every step. +40.9% over the base VLA.

### Retrieval-augmented prompting at inference

- [**MAP-VLA: Memory-Augmented Prompting for VLA**](https://arxiv.org/abs/2511.09516) — Retrieves demonstration-derived prompt tokens at inference — effectively inference-time prompt tuning of a frozen VLA.

---

## Steering the Action Expert (sampling / generation layer)

### VLM / programmatic-reward guidance in denoising

- [**VLS: Steering Pretrained Robot Policies via Vision-Language Models**](https://arxiv.org/abs/2602.03973) (Liu et al., 2026) — VLM grounds OOD observation–language pairs into keypoints and stage-wise differentiable programmatic rewards; injects gradients into denoising plus Feynman-Kac particle resampling with RBF repulsion. +31% CALVIN, +13% LIBERO-PRO; Franka real-robot deployment on VLA backbones.
- [**VLA-Pilot: Plug-and-Play Inference-Time VLA Policy Steering via Embodied Evolutionary Diffusion**](https://arxiv.org/abs/2511.14178) (Li et al., 2025) — Embodied Policy Steering Chain-of-Thought (MLLM as open-world verifier) + Evolutionary Diffusion (mutation–selection in the VLA's noise space) + iterative refinement. Zero finetuning, cross-embodiment; ~+31% over base VLAs.
- [**ProgressVLA: Progress-Guided Diffusion Policy for Vision-Language Robotic Manipulation**](https://arxiv.org/abs/2601.20239) (Yan et al., 2026) — Pretrained progress estimator + inverse-dynamics world model + classifier-style progress guidance in the VLA's latent action space. Also distills guided targets back into the denoiser, transitioning external guidance into internal capability.
- [**TAG: Target-Aware Guidance for Vision-Language-Action Models**](https://arxiv.org/abs/2602.22056) (2026) — Classifier-free-guidance-style dual branches on original obs and an object-erased counterfactual obs; the residual steers the VLA toward target evidence. π₀.₅: 95.2 → 97.9 on LIBERO; reduces near-miss / wrong-object errors on LIBERO-Plus and VLABench.

### Verifier-based selection (Best-of-N / re-ranking)

- [**V-GPS: Steering Your Generalists — Improving Robotic Foundation Models via Value Guidance**](https://arxiv.org/abs/2410.13816) (Nakamoto et al., CoRL 2024) — Language-conditioned Cal-QL value function re-ranks actions from any generalist VLA (Octo, RT-1-X, OpenVLA, etc.); 5 policies × 12 tasks, ~1.28–1.59× single-step overhead. Canonical advantage-weighted regression at inference.
- [**RoVer: Robot Reward Model as Test-Time Verifier for Vision-Language-Action Models**](https://arxiv.org/abs/2510.10975) (Dai et al., 2025) — Process Reward Model returns scalar score *and* an action-space direction; caches perception features across candidates for efficient test-time scaling of VLAs.
- [**Do What You Say: Steering Vision-Language-Action Models via Runtime Reasoning-Action Alignment**](https://arxiv.org/abs/2509.18130) (Wu et al., 2025) — VLM-generated chain-of-thought acts as both target plan and verifier, filtering action trajectories inconsistent with the spoken plan.
- [**When to Act, Ask, or Learn: Uncertainty-Aware Policy Steering**](https://arxiv.org/abs/2602.22474) (Yuan, Wu, Bajcsy, 2026) — Conformal-prediction calibration of a VLM verifier on top of a base VLA; decides between executing, asking for clarification, or requesting an intervention. Addresses the miscalibration that breaks naive VLM-as-verifier setups.
- [**VGAS: Value-Guided Action-Chunk Selection for Few-Shot Vision-Language-Action Adaptation**](https://arxiv.org/abs/2602.07399) (Xu et al., 2026) — Inference-time best-of-N over action chunks using a geometrically grounded Q-Chunk-Former critic, with explicit geometric regularization for near-miss disambiguation.
- [**SITCOM: Scaling Inference-Time COMpute for VLAs**](https://arxiv.org/abs/2510.04041) (Saxena, Shah et al., 2025) — Endows any pretrained VLA with MPC-style model-based rollouts and reward-based trajectory ranking at test time.

### Latent-noise / initial-noise steering

- [**DSRL: Steering Your Diffusion Policy with Latent Space Reinforcement Learning**](https://arxiv.org/abs/2506.15799) (Wagenmaker, Nakamoto et al., CoRL 2025) — Keeps the base diffusion policy frozen and learns an RL policy over its initial-noise latent. Black-box, low-dimensional, no backprop through denoising. Demonstrated on π₀ on real WidowX and Aloha hardware.
- [**USR: Unified Steering and Residual Refinement**](https://openreview.net/forum?id=DbBD2aT1OG) — Combines DSRL-style noise steering with a residual action correction, addressing the mode confinement of pure latent steering. Targets diffusion/flow VLAs.
- [**OptimusVLA: Task Prior and Local Consistency Memory**](https://arxiv.org/abs/2602.20200) (2026) — Replaces isotropic initial noise with a retrieved task-prior plus a local-consistency memory term — memory-augmented noise-space initialization for VLA action experts.

### Tree search / planning over actions

- [**FORGE-Tree: Monte Carlo Tree Diffusion for Long-Horizon VLA**](https://arxiv.org/abs/2510.21744) (2025) — MCTD over long-horizon VLAs; partially denoises trajectory segments using frozen OpenVLA / Octo encoders. +13.4–17.2 pp on LIBERO.
- [**Value Vision-Language-Action Planning & Search**](https://arxiv.org/abs/2601.00969) (Neary et al., 2026) — Extends test-time-compute techniques (CoT prompting, self-consistency, MCTS) from LLMs to VLAs to address reactive-execution failures.

---

## Native Steerability via Training-Time Design

Not strictly inference-time, but the resulting VLA has a steering interface that operates at deployment without further training.

- [**π0.7: A Steerable Generalist Robotic Foundation Model with Emergent Capabilities**](https://arxiv.org/abs/2604.15483) (Physical Intelligence, 2026) — Trained with diverse multimodal prompt conditioning (language commands, strategy metadata, subgoal images). Classifier-free guidance on metadata at inference allows precise steering toward different strategies on the same task — the cleanest existing example of a trained-in conditioning channel steerable at inference.
- [**Hume: Introducing System-2 Thinking in Visual-Language-Action Model**](https://arxiv.org/abs/2505.21432) (Song et al., 2025) — Dual-system VLA: System-2 augments the backbone with a value-query head and selects action chunks via value-guided best-of-N; System-1 does fast cascaded denoising of the chosen chunk. +4.4 / +25.9 / +12.9% on LIBERO / Simpler / real.
- [**LAP: Language-Action Pre-Training Enables Zero-shot Cross-Embodiment Transfer**](https://arxiv.org/abs/2604.23121) (Zha et al., 2026) — Represents low-level robot actions directly in natural language, aligning action supervision with the VLM's I/O distribution. ~2× zero-shot success over prior VLAs on unseen embodiments — a representation choice that makes the model steerable by language alone.
- [**ST-π: Spatiotemporal VLA with Structured Action Expert**](https://arxiv.org/abs/2601.04052) (2026) — Spatiotemporal VLM paired with a structured action expert; chunk-level prompts carry explicit semantic / spatial / temporal attributes that can be set at inference. LIBERO avg 97.4, STAR 80.1.
- [**RSS: Stable Language Guidance for VLAs**](https://openreview.net/forum?id=YvsUD8C9QS) (2026) — Monte-Carlo syntactic integration plus residual affordance steering for robustness to instruction-blindness. Trained-in interface, exercised at inference.
- [**ATE: Align-Then-Steer Across Embodiments**](https://openreview.net/forum?id=T3i7Ifeatk) (ICLR 2026 Poster) — VAE plus reverse-KL unifies the action latent space across embodiments, then steers diffusion / flow VLAs across embodiments at inference. +9.8% sim, +32% real cross-embodiment.

---

## What's Out of Scope

To keep the list focused on *VLA* steering, the following are excluded even though they often appear in adjacent reviews:

- **Steering methods demonstrated only on generic diffusion policies or visuomotor policies, not on a VLA backbone.** Examples: *FOREWARN* (foresight–forethought on diffusion policies), *UF-OPS* (verifier on deployed rollouts), *VGD* (value-guided denoising), *DynaGuide* (latent-dynamics classifier guidance on CALVIN diffusion policies), *LPB* / *LatentCBF* (latent barrier functions on visuomotor policies), *GPC* (generative predictive control), *TDP* (tree-guided diffusion planner), *TouchGuide* (tactile guidance of visuomotor policies), *ADPro* (manifold-projected priors), *PPGuide*.
- **Human-in-the-loop steering on non-VLA generative policies.** *ITPS* and *Yell At Your Robot* are foundational HITL steering work but were demonstrated on diffusion / hierarchical policies rather than on a VLA; their techniques are commonly imported into VLA settings but the original works are out of scope here.
- **Subgoal / trajectory-conditioning methods on non-VLA goal-conditioned policies**, e.g. *SuSIE* and *RT-Trajectory*, where the steered policy is not a VLA.
- **Foundational VLA architectures without a dedicated steering mechanism**: *RT-2*, *OpenVLA*, *Octo*, *π₀*, *π₀.₅*, *RDT-1B*. These are the *targets* of steering, not steering methods themselves.
- **Trained-in reasoning or memory architectures whose "steering" is a model capability rather than an external intervention**: *CoT-VLA* and *ECoT* (chain-of-thought baked into the VLA via training), *MemoryVLA* and *ReMem-VLA* (memory-augmented VLA architectures requiring retraining). The runtime behavior is shaped by training, not by an inference-time controller acting on a frozen model.
- **Online RL / fine-tuning methods that modify policy weights** during deployment, e.g. *Policy Decorator*, *FlowCorrect*, *VLA-RL*, *RobustVLA* — these cross the line from steering into adaptation.
- **Generic CV diffusion / flow-matching guidance** (CFG, autoguidance, DPS, MPGD, DOODL, SVDD, FK Steering, ControlNet, Composable Diffusion, Particle Guidance, Restart Sampling, …) — important conceptual background, but not VLA work.

If you think a paper here belongs in or out of either category, open an issue or PR.

## Contributing

Pull requests welcome. When adding a paper:

1. Verify the paper genuinely steers a VLA (vision-language pretrained backbone driving action prediction), not just a generic diffusion policy or visuomotor model.
2. Place it under VLM-layer or action-expert-layer based on where the steering signal acts.
3. Use the title as the linked text, pointing to the arXiv abstract page (or official venue page if no arXiv version exists).
4. Provide a one- to two-sentence summary noting the steering mechanism and a headline result.

## License

This list is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
