# Awesome VLA Steering

A curated list of papers on **inference-time steering of Vision-Language-Action (VLA) models**.

Steering here means: changing a (typically pretrained or frozen) VLA's behavior at deployment through prompts, context, internal-feature intervention, observation transforms, action re-ranking, denoising / flow guidance, latent-noise control, or memory — without standard end-to-end retraining.

A paper is included only if it demonstrates its steering mechanism on a VLA (vision-language pretrained backbone driving action prediction — e.g. OpenVLA, Octo, RT-1-X, π₀, π₀.₅, π0.7) or is specifically designed for VLA architectures. Methods that steer only generic diffusion policies or visuomotor policies appear under [Related Work](#related-work-steering-non-vla-policies); trained-in VLA reasoning / memory, online VLA adaptation, and generic CV diffusion guidance background are collected in [BACKGROUND.md](./BACKGROUND.md).

Papers are split by **where the steering signal acts**:

- **Steering the VLM (semantic / representation layer):** intervenes on prompts, context, the language–vision tower, internal features, attention, or the observation input. These methods reshape *what the model perceives or reasons about* before any action is committed.
- **Steering the Action Expert (sampling / generation layer):** intervenes on action proposals, candidate ranking, denoising / flow dynamics, latent noise, or output decoding. These methods reshape *how actions are sampled* given a fixed semantic interpretation.

A few works span both layers; they appear under their primary mechanism with a note.

## Contents

- [Steering the VLM](#steering-the-vlm-semantic--representation-layer)
- [Steering the Action Expert](#steering-the-action-expert-sampling--generation-layer)
- [Native Steerability via Training-Time Design](#native-steerability-via-training-time-design)
- [Related Work: Steering Non-VLA Policies](#related-work-steering-non-vla-policies)
- [Background & related work in a separate file →](./BACKGROUND.md)

---

## Steering the VLM (semantic / representation layer)

### Internal feature / activation steering

- [**Mechanistic Interpretability for Steering Vision-Language-Action Models**](https://arxiv.org/abs/2509.00328) (Häon et al., CoRL 2025) — Projects FFN activations onto the token embedding basis to find sparse semantic directions (speed, direction, grasp) causally linked to action selection; activation steering at inference with no fine-tuning. Demonstrated on π₀ and OpenVLA, LIBERO and a UR5.
- [**Observing and Controlling Features in Vision-Language-Action Models**](https://arxiv.org/abs/2604.17880) (Buurmeijer et al., 2026) — Formalizes feature-observability and feature-controllability for VLAs; uses linear classifiers to observe features and minimal linear interventions (grounded in optimal control) to steer π₀.₅ and OpenVLA outputs while preserving closed-loop behavior.
- [**DeLock: Breaking Lock-In — Preserving Steerability under Low-Data VLA Post-Training**](https://arxiv.org/abs/2602.10556) (Huang et al., 2026) — Identifies "lock-in": low-data SFT destroys the VLA's instruction-following steerability. Mitigation combines preserved visual grounding during training with test-time contrastive prompt guidance.

<!-- ### Observation interventions

- [**BYOVLA: Run-time Observation Interventions Make Vision-Language-Action Models More Visually Robust**](https://arxiv.org/abs/2410.01971) (Hancock et al., ICRA 2025) — VLM identifies task-irrelevant regions; sensitivity probes find regions the frozen VLA depends on; minimal image edits to the intersection. Black-box, recovers near-nominal performance under distractors on OpenVLA. -->

<!-- ### Decoding-time intervention on the VLM head

- [**PCD: Policy Contrastive Decoding for Robotic Foundation Models**](https://arxiv.org/abs/2505.13255) (Wu et al., ICLR 2026) — Contrasts action distributions from original observation vs. object-masked observation to suppress reliance on spurious visual cues. Training-free plug-in for autoregressive OpenVLA and (via KDE) diffusion-based Octo and π₀; +50.6% / +29.7% / +8.9% in simulation, +108% on real for π₀. -->

### Foresight / subgoal conditioning at inference

- [**ForeAct: Foresight-Guided Action Steering**](https://arxiv.org/abs/2602.12322) (2026) — Fast (0.33s) foresight image generator plus a VLM subtask describer steers frozen π₀ at every step. +40.9% over the base VLA.

<!-- ### Retrieval-augmented prompting at inference

- [**MAP-VLA: Memory-Augmented Prompting for VLA**](https://arxiv.org/abs/2511.09516) — Retrieves demonstration-derived prompt tokens at inference — effectively inference-time prompt tuning of a frozen VLA. -->

---

## Steering the Action Expert (sampling / generation layer)

### VLM / programmatic-reward guidance in denoising

- [**VLS: Steering Pretrained Robot Policies via Vision-Language Models**](https://arxiv.org/abs/2602.03973) (Liu et al., 2026) — VLM grounds OOD observation–language pairs into keypoints and stage-wise differentiable programmatic rewards; injects gradients into denoising plus Feynman-Kac particle resampling with RBF repulsion. +31% CALVIN, +13% LIBERO-PRO; Franka real-robot deployment on VLA backbones.
- [**VLA-Pilot: Plug-and-Play Inference-Time VLA Policy Steering via Embodied Evolutionary Diffusion**](https://arxiv.org/abs/2511.14178) (Li et al., 2025) — Embodied Policy Steering Chain-of-Thought (MLLM as open-world verifier) + Evolutionary Diffusion (mutation–selection in the VLA's noise space) + iterative refinement. Zero finetuning, cross-embodiment; ~+31% over base VLAs.
- [**ProgressVLA: Progress-Guided Diffusion Policy for Vision-Language Robotic Manipulation**](https://arxiv.org/abs/2601.20239) (Yan et al., 2026) — Pretrained progress estimator + inverse-dynamics world model + classifier-style progress guidance in the VLA's latent action space. Also distills guided targets back into the denoiser, transitioning external guidance into internal capability.
<!-- - [**TAG: Target-Aware Guidance for Vision-Language-Action Models**](https://arxiv.org/abs/2602.22056) (2026) — Classifier-free-guidance-style dual branches on original obs and an object-erased counterfactual obs; the residual steers the VLA toward target evidence. π₀.₅: 95.2 → 97.9 on LIBERO; reduces near-miss / wrong-object errors on LIBERO-Plus and VLABench. -->

### Verifier-based selection (Best-of-N / re-ranking)

- [**V-GPS: Steering Your Generalists — Improving Robotic Foundation Models via Value Guidance**](https://arxiv.org/abs/2410.13816) (Nakamoto et al., CoRL 2024) — Language-conditioned Cal-QL value function re-ranks actions from any generalist VLA (Octo, RT-1-X, OpenVLA, etc.); 5 policies × 12 tasks, ~1.28–1.59× single-step overhead. Canonical advantage-weighted regression at inference.
- [**RoVer: Robot Reward Model as Test-Time Verifier for Vision-Language-Action Models**](https://arxiv.org/abs/2510.10975) (Dai et al., 2025) — Process Reward Model returns scalar score *and* an action-space direction; caches perception features across candidates for efficient test-time scaling of VLAs.
- [**Do What You Say: Steering Vision-Language-Action Models via Runtime Reasoning-Action Alignment**](https://arxiv.org/abs/2510.16281) (Wu et al., 2025) — VLM-generated chain-of-thought acts as both target plan and verifier, filtering action trajectories inconsistent with the spoken plan.
<!-- - [**When to Act, Ask, or Learn: Uncertainty-Aware Policy Steering**](https://arxiv.org/abs/2602.22474) (Yuan, Wu, Bajcsy, 2026) — Conformal-prediction calibration of a VLM verifier on top of a base VLA; decides between executing, asking for clarification, or requesting an intervention. Addresses the miscalibration that breaks naive VLM-as-verifier setups. -->
<!-- - [**VGAS: Value-Guided Action-Chunk Selection for Few-Shot Vision-Language-Action Adaptation**](https://arxiv.org/abs/2602.07399) (Xu et al., 2026) — Inference-time best-of-N over action chunks using a geometrically grounded Q-Chunk-Former critic, with explicit geometric regularization for near-miss disambiguation. -->
- [**SITCOM: Scaling Inference-Time COMpute for VLAs**](https://arxiv.org/abs/2510.04041) (Saxena, Shah et al., 2025) — Endows any pretrained VLA with MPC-style model-based rollouts and reward-based trajectory ranking at test time.

### Latent-noise / initial-noise steering

<!-- - [**DSRL: Steering Your Diffusion Policy with Latent Space Reinforcement Learning**](https://arxiv.org/abs/2506.15799) (Wagenmaker, Nakamoto et al., CoRL 2025) — Keeps the base diffusion policy frozen and learns an RL policy over its initial-noise latent. Black-box, low-dimensional, no backprop through denoising. Demonstrated on π₀ on real WidowX and Aloha hardware. -->
- [**USR: Unified Steering and Residual Refinement**](https://openreview.net/forum?id=DbBD2aT1OG) — Combines DSRL-style noise steering with a residual action correction, addressing the mode confinement of pure latent steering. Targets diffusion/flow VLAs.
<!-- - [**OptimusVLA: Task Prior and Local Consistency Memory**](https://arxiv.org/abs/2602.20200) (2026) — Replaces isotropic initial noise with a retrieved task-prior plus a local-consistency memory term — memory-augmented noise-space initialization for VLA action experts. -->

<!-- ### Tree search / planning over actions

- [**FORGE-Tree: Monte Carlo Tree Diffusion for Long-Horizon VLA**](https://arxiv.org/abs/2510.21744) (2025) — MCTD over long-horizon VLAs; partially denoises trajectory segments using frozen OpenVLA / Octo encoders. +13.4–17.2 pp on LIBERO.
- [**Value Vision-Language-Action Planning & Search**](https://arxiv.org/abs/2601.00969) (Neary et al., 2026) — Extends test-time-compute techniques (CoT prompting, self-consistency, MCTS) from LLMs to VLAs to address reactive-execution failures. -->

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

## Related Work: Steering Non-VLA Policies

These methods are conceptually adjacent to VLA steering — many use the same machinery (verifier re-ranking, classifier guidance, latent barriers, HITL, subgoal conditioning) — but were demonstrated on generic diffusion policies, visuomotor policies, or non-VLA goal-conditioned policies. Listed here as a reference index; they are *not* the focus of this list.

### Diffusion / visuomotor policy steering (not VLA)

- [**FOREWARN: From Foresight to Forethought — VLM-In-the-Loop Policy Steering via Latent Alignment**](https://arxiv.org/abs/2502.01828) (Wu et al., RSS 2025) — Decouples *foresight* (latent dynamics predicts future latent obs) from *forethought* (small VLM scores a textual "behavior narration" of the latent). ~3.7s/decision vs. VLM-Act's 22s. Demonstrated on diffusion policies.
- [**UF-OPS: Update-Free On-Policy Steering via Verifiers**](https://arxiv.org/abs/2603.10282) (Attarian et al., 2026) — Verifiers trained from the policy's own deployed rollouts (successes + failures) steer black-box diffusion policies. +49% average over 5 real tasks.
- [**VGD: Steering Diffusion Policies with Value-Guided Denoising**](https://openreview.net/forum?id=wrcTncImde) (NeurIPS 2025 workshop) — At each DDIM step, computes a one-step clean-action estimate x̂₀ and adds ∇ₐQ(s, x̂) to the predicted noise. Avoids backprop through the diffusion chain.
- [**PPGuide: Performance-Predictor Guidance**](https://arxiv.org/abs/2603.10980) — Binary success classifier provides gradient signal to push samples away from failure modes.
- [**DynaGuide: Steering Diffusion Policies with Active Dynamic Guidance**](https://arxiv.org/abs/2506.13922) (Du & Song, NeurIPS 2025) — Latent dynamics model over DINOv2 features supplies a log-sum-exp classifier-guidance signal. 70% steering success on CALVIN; 5.4× over goal-conditioning under weak goal descriptions.
- [**LPB: Latent Policy Barrier**](https://arxiv.org/abs/2508.05941) — Latent dynamics defines a barrier function approximating expert support; rejects / projects candidates leaving the support. Essentially a learned-latent CBF.
- [**LatentCBF: Latent Control Barrier Functions for Visuomotor Policies**](https://arxiv.org/abs/2511.18606) — Explicit, smooth, differentiable latent CBF; more permissive than least-restrictive filters.
- [**GPC: Generative Predictive Control**](https://arxiv.org/abs/2502.00622) — Two modes: GPC-RANK (best-of-N in world model) and GPC-OPT (gradient refinement). MPC-with-foresight wrapper.
- [**TouchGuide: Tactile-Guided Steering of Pretrained Visuomotor Policies**](https://arxiv.org/abs/2603.24584) — Tactile contact physics as classifier guidance for diffusion / flow policies. Visual coarse action → tactile-feasibility refinement.
- [**TDP: Tree-Guided Diffusion Planner**](https://arxiv.org/abs/2508.21800) — Parent trajectories sampled with particle guidance (diversity); child sub-trajectories with gradient-guided denoising (exploitation). Bi-level explore / exploit.
- [**ADPro: Action-Diffusion with Manifold-Projected Priors**](https://arxiv.org/abs/2508.06266) — Replaces isotropic noise with a manifold-projected, task-aware prior. Robotics analogue of CV's MPGD.

### HITL steering on non-VLA generative policies

- [**ITPS: Inference-Time Policy Steering through Human Interactions**](https://arxiv.org/abs/2411.16627) (Wang et al., ICRA 2025) — Foundational HITL paper. Six families: post-hoc perturbation, ranking, initialization, gradient-guided sampling, **stochastic sampling** (the winner — DDPM resampling from a conditional kernel), and biased prior. Stochastic sampling dominates the alignment-vs-OOD trade-off. Targets diffusion policies.
- [**Yell at Your Robot: Improving On-the-Fly from Language Corrections**](https://arxiv.org/abs/2403.12910) (Shi et al., 2024) — Real-time language corrections to a hierarchical policy; corrections also stored for offline updates.
- [**Steering Robots with Inference-Time Interactions**](https://arxiv.org/abs/2506.14287) (Wang) — Systematizes ITPS + PoCo-style composition + TAMI hard-mode classifiers.

### Subgoal / trajectory conditioning on non-VLA goal-conditioned policies

- [**SuSIE: Subgoal Synthesis via Image Editing**](https://arxiv.org/abs/2310.10639) (Black, Nakamoto et al.) — InstructPix2Pix edits the current obs into a subgoal image; a goal-conditioned low-level policy follows it. Frozen at deployment.
- [**RT-Trajectory: Robotic Task Generalization via Hindsight Trajectory Sketches**](https://arxiv.org/abs/2311.01977) (DeepMind, ICLR 2024) — Conditions RT-1 on 2D / 2.5D trajectory sketches from humans, generators, or planners. "Sketch as prompt."

---

## License

Released under the [MIT License](./LICENSE).
