# Rudder

A curated list of papers on **inference-time steering of Vision-Language-Action (VLA) models**.

Steering here means: changing a (typically pretrained or frozen) VLA's behavior at deployment through prompts, context, internal-feature intervention, observation transforms, action re-ranking, denoising/flow guidance, or online residuals — without standard end-to-end retraining.

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

### Observation interventions

- [**BYOVLA: Run-time Observation Interventions Make Vision-Language-Action Models More Visually Robust**](https://arxiv.org/abs/2410.01971) (Hancock et al., ICRA 2025) — VLM identifies task-irrelevant regions; sensitivity probes find regions the frozen VLA depends on; minimal image edits to the intersection. Black-box, recovers near-nominal performance under distractors.

### Internal feature / activation steering

- [**Mechanistic Interpretability for Steering Vision-Language-Action Models**](https://arxiv.org/abs/2509.00328) (Häon et al., CoRL 2025) — Projects FFN activations onto the token embedding basis to find sparse semantic directions (speed, direction, etc.) causally linked to action selection; activation steering at inference, no fine-tuning. Demonstrated on π₀ and OpenVLA, LIBERO and a UR5.
- [**Observing and Controlling Features in Vision-Language-Action Models**](https://arxiv.org/abs/2603.05487) (Buurmeijer et al., 2026) — Formalizes feature-observability and feature-controllability for VLAs; uses a linear classifier to observe features and minimal linear interventions (grounded in optimal control) to steer π₀.₅ and OpenVLA outputs while preserving closed-loop behavior.

### Decoding-time intervention on the VLM head

- [**PCD: Policy Contrastive Decoding for Robotic Foundation Models**](https://arxiv.org/abs/2505.13255) (Wu et al., ICLR 2026) — Contrasts action distributions from original observation vs. object-masked observation to suppress reliance on spurious visual cues. Training-free plug-in; works on autoregressive OpenVLA and on diffusion-based Octo and π₀ (via KDE-based probabilistic modeling). +50.6% / +29.7% / +8.9% in simulation; +108% on real for π₀.

### Human-interaction steering on top of generative policies

- [**ITPS: Inference-Time Policy Steering through Human Interactions**](https://arxiv.org/abs/2411.16627) (Wang et al., ICRA 2025) — Six families of steering from human input (point goal, sketch, physical correction); the analysis identifies stochastic sampling as the best alignment-vs-OOD trade-off. Targets generative robot policies (diffusion); commonly imported into VLA settings.
- [**Yell At Your Robot: Improving On-the-Fly from Language Corrections**](https://arxiv.org/abs/2403.12910) (Shi et al., 2024) — Real-time spoken language corrections fed as new instructions to a hierarchical language-conditioned policy, blurring the line between high-level steering and online fine-tuning.

---

## Steering the Action Expert (sampling / generation layer)

### Verifier-based selection (Best-of-N / re-ranking)

- [**V-GPS: Steering Your Generalists — Improving Robotic Foundation Models via Value Guidance**](https://arxiv.org/abs/2410.13816) (Nakamoto et al., CoRL 2024) — Language-conditioned Cal-QL value function re-ranks actions from any generalist VLA (Octo, RT-1-X, OpenVLA, etc.); 5 policies × 12 tasks. ~1.28–1.59× single-step overhead.
- [**RoVer: Robot Reward Model as Test-Time Verifier for Vision-Language-Action Models**](https://arxiv.org/abs/2510.10975) (Dai et al., 2025) — Process Reward Model returns scalar score *and* an action-space direction for candidate expansion. Caches perception features across candidates for test-time scaling.
- [**FOREWARN: From Foresight to Forethought — VLM-In-the-Loop Policy Steering via Latent Alignment**](https://arxiv.org/abs/2502.01828) (Wu et al., RSS 2025) — Decouples *foresight* (latent dynamics predicts future latent obs) from *forethought* (small VLM scores a textual "behavior narration" of the latent). Outstanding paper at the ICLR 2025 World Model Workshop.
- [**UF-OPS: Update-Free On-Policy Steering via Verifiers**](https://arxiv.org/abs/2603.10282) (Attarian et al., 2026) — Verifiers trained from the policy's *own* deployed rollouts (successes + failures) steer black-box diffusion policies. +49% average over 5 real tasks.
- [**When to Act, Ask, or Learn: Uncertainty-Aware Policy Steering**](https://arxiv.org/abs/2602.22474) (Yuan, Wu, Bajcsy, 2026) — Conformal-prediction calibration of VLM + base policy; decides between executing, asking for clarification, or requesting an intervention.
- [**VGAS: Value-Guided Action-Chunk Selection for Few-Shot Vision-Language-Action Adaptation**](https://arxiv.org/abs/2602.07399) (Xu et al., 2026) — Inference-time best-of-N over action chunks using a geometrically grounded Q-Chunk-Former critic, with explicit geometric regularization for near-miss disambiguation.
- [**SITCOM: Scaling Inference-Time COMpute for VLAs**](https://arxiv.org/abs/2510.04041) (Saxena, Shah et al., 2025) — Endows any pretrained VLA with MPC-style model-based rollouts and reward-based trajectory ranking.

### Value / Q-function guidance in denoising (latent-noise steering)

- [**DSRL: Steering Your Diffusion Policy with Latent Space Reinforcement Learning**](https://arxiv.org/abs/2506.15799) (Wagenmaker, Nakamoto et al., CoRL 2025) — Keeps the base diffusion policy frozen; learns an RL policy over its initial-noise latent. Black-box, low-dimensional noise space, no backprop through denoising. Demonstrated on π₀ on real WidowX and Aloha hardware.

### World-model / dynamics-guided denoising

- [**DynaGuide: Steering Diffusion Policies with Active Dynamic Guidance**](https://arxiv.org/abs/2506.13922) (Du & Song, NeurIPS 2025) — External latent dynamics model on top of DINOv2 features supplies a classifier-guidance signal during action denoising. 70% steering success on articulated CALVIN; 5.4× over goal-conditioning under low-quality objectives.

### VLM / programmatic-reward guidance in denoising

- [**VLS: Steering Pretrained Robot Policies via Vision-Language Models**](https://arxiv.org/abs/2602.03973) (Liu et al., 2026) — VLM grounds OOD observation–language pairs into keypoints + stage-wise differentiable programmatic rewards; injects gradients into denoising plus Feynman-Kac particle resampling with RBF repulsion. +31% CALVIN, +13% LIBERO-PRO; Franka real-robot deployment.
- [**VLA-Pilot: Plug-and-Play Inference-Time VLA Policy Steering via Embodied Evolutionary Diffusion**](https://arxiv.org/abs/2511.14178) (Li et al., 2025) — Embodied Policy Steering Chain-of-Thought (MLLM as open-world verifier) plus Evolutionary Diffusion (mutation–selection in noise space) plus iterative refinement. Zero finetuning, cross-embodiment, 6 real-world tasks.
- [**ProgressVLA: Progress-Guided Diffusion Policy for Vision-Language Robotic Manipulation**](https://arxiv.org/abs/2603.27670) (Yan et al., 2026) — Pretrained progress estimator + inverse-dynamics world model + classifier-style progress guidance in latent action space. Also distills guided targets back into the denoiser, transitioning external guidance into internal capability.

### Tree search over denoising

- [**Value Vision-Language-Action Planning & Search**](https://arxiv.org/abs/2601.00969) (Neary et al., 2026) — Extends test-time-compute techniques (CoT prompting, self-consistency, MCTS) from LLMs to VLAs to address reactive-execution failures.

---

## Native Steerability via Training-Time Design

Not strictly inference-time, but the resulting model has a steering interface that operates at deployment without further training.

- [**π0.7: A Steerable Generalist Robotic Foundation Model with Emergent Capabilities**](https://arxiv.org/abs/2604.15483) (Physical Intelligence, 2026) — Trained with diverse multimodal prompt conditioning (language commands, strategy metadata, subgoal images). Classifier-free guidance on metadata at inference allows precise steering toward different strategies on the same task. The cleanest existing example of a trained-in conditioning channel that is steerable at inference.
- [**Hume: Introducing System-2 Thinking in Visual-Language-Action Model**](https://arxiv.org/abs/2505.21432) (Song et al., 2025) — Dual-system VLA: System-2 augments a VLA backbone with a value-query head and selects action chunks via value-guided best-of-N; System-1 does fast cascaded denoising of the chosen chunk. Steering is value-guided thinking baked into the architecture.
- [**LAP: Language-Action Pre-Training Enables Zero-shot Cross-Embodiment Transfer**](https://arxiv.org/abs/2602.10556) (Zha et al., 2026) — Represents low-level robot actions directly in natural language, aligning action supervision with the VLM's I/O distribution. ~2× zero-shot success over prior VLAs on unseen embodiments — a representation choice that makes the model steerable by language alone.

---

## What's Out of Scope

To keep the list focused, I excluded several papers that frequently show up in adjacent reviews:

- **Pure diffusion-policy steering on non-VLA backbones** (e.g. value-guided denoising on Robomimic Diffusion Policy without VLA application).
- **Generic CV diffusion / flow-matching guidance** (classifier-free guidance, MPGD, DPS, ControlNet, Reduce-Reuse-Recycle, etc.) — important conceptual background, but not VLA work.
- **Foundational VLA architectures without a dedicated steering mechanism** (RT-2, OpenVLA, RDT-1B, π₀, Octo).
- **Subgoal/trajectory-conditioning methods on non-VLA goal-conditioned policies** (SuSIE, RT-Trajectory, ECoT) where the steered policy isn't a VLA.
- **Online RL / fine-tuning methods that modify policy weights** (Policy Decorator, VLA-RL, RobustVLA, etc.) — these cross the line from steering into adaptation.

If you think a paper here belongs in or out of either category, open an issue or PR.

## Contributing

Pull requests welcome. When adding a paper:

1. Verify the paper genuinely steers a VLA (not just a generic diffusion policy or visuomotor model).
2. Place it under VLM-layer or action-expert-layer based on where the steering signal acts.
3. Use the title as the linked text, pointing to the arXiv abstract page.
4. Provide a one- to two-sentence summary noting the steering mechanism and a headline result.

## License

This list is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
