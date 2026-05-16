# Background & Related Work

Papers excluded from the main [Rudder](./README.md) list because they fall outside *inference-time steering of a frozen VLA*, but provide essential context. Three categories:

1. **Trained-in VLA reasoning / memory** — the "steering" is a capability baked into the VLA via training, not an external intervention applied at deployment.
2. **Online VLA adaptation** — methods that modify policy weights (or add trainable modules updated online) during deployment, crossing from steering into adaptation.
3. **Generic CV diffusion / flow-matching guidance** — the conceptual machinery (CFG, classifier guidance, MPGD, particle methods, etc.) that most VLA action-expert steering methods import from the CV literature.

## Contents

- [Trained-in VLA reasoning / memory](#trained-in-vla-reasoning--memory)
- [Online VLA adaptation](#online-vla-adaptation)
- [Generic CV diffusion / flow-matching guidance](#generic-cv-diffusion--flow-matching-guidance)

---

## Trained-in VLA reasoning / memory

The model is trained to produce intermediate reasoning or to consume memory tokens; the runtime behavior is a property of the trained model, not an external steering signal.

- [**CoT-VLA: Visual Chain-of-Thought Reasoning for VLA Models**](https://arxiv.org/abs/2503.22020) (Zhao et al., 2025) — Autoregressively generates a future image as visual chain-of-thought, then attends to it for action prediction.
- [**ECoT: Embodied Chain-of-Thought**](https://arxiv.org/abs/2407.08693) (Zawalski et al., 2024) — Language analogue of CoT-VLA: plan / subtask / bounding box / end-effector position reasoning text before action, trained into OpenVLA.
- [**MemoryVLA: Perceptual-Cognitive Memory for VLA Models**](https://arxiv.org/abs/2508.19236) — VLA architecture augmented with perceptual + cognitive memory banks.
- [**ReMem-VLA: Recurrent Dual-Level Memory for VLA**](https://arxiv.org/abs/2603.12942) (2026) — Recurrent dual-level memory architecture for long-horizon VLA tasks.

---

## Online VLA adaptation

Methods that modify the policy's weights or attach trainable modules updated during deployment. Steering shades into adaptation here.

- [**Policy Decorator: Model-Agnostic Online Refinement for Large Policy Models**](https://arxiv.org/abs/2412.13630) — Trains a model-agnostic residual policy via online RL on top of frozen large BC policies; bounded exploration. 8 tasks across ManiSkill / Adroit.
- [**FlowCorrect: Online Correction of Pretrained Flow Policies**](https://arxiv.org/abs/2605.11809) (2026) — Frozen flow policy plus a lightweight LoRA / gating correction module trained from small amounts of human relative corrections.

---

## Generic CV diffusion / flow-matching guidance

Background machinery. Many VLA action-expert steering methods are robotics imports of these techniques.

### Classifier and classifier-free guidance

- [**Diffusion Models Beat GANs on Image Synthesis**](https://arxiv.org/abs/2105.05233) (Dhariwal & Nichol, 2021) — Classifier guidance.
- [**Classifier-Free Diffusion Guidance**](https://arxiv.org/abs/2207.12598) (Ho & Salimans, 2022) — The CFG that everyone uses.
- [**Autoguidance: Guiding a Diffusion Model with a Bad Version of Itself**](https://arxiv.org/abs/2406.02507) (Karras et al., NeurIPS 2024) — Replaces the unconditional model with a degraded conditional one. Record FIDs.
- [**In-Situ Autoguidance**](https://arxiv.org/abs/2510.17136) — Achieves autoguidance without an auxiliary model via stochastic perturbation.
- [**PAG: Perturbed-Attention Guidance**](https://arxiv.org/abs/2403.17377) — Training-free, conditioning-free quality booster by perturbing self-attention.

### Reward-guided sampling

- [**DPS: Diffusion Posterior Sampling**](https://arxiv.org/abs/2209.14687) (Chung et al., 2022) — Foundation of most training-free guidance.
- [**Universal Guidance for Diffusion Models**](https://arxiv.org/abs/2302.07121) (Bansal, Goldstein et al.) — Any off-the-shelf classifier + self-recurrence.
- [**MPGD: Manifold Preserving Guided Diffusion**](https://arxiv.org/abs/2311.16424) (He, Ermon et al., ICLR 2024) — Projects guidance onto the data-manifold tangent space; 3.8× faster.
- [**DOODL: Direct Optimization of Diffusion Latents**](https://arxiv.org/abs/2303.13703) (Wallace, Ermon, Naik, ICCV 2023) — Uses EDICT for invertible diffusion ODE; exact gradients w.r.t. initial noise.
- [**ReNO: Reward-Based Noise Optimization**](https://arxiv.org/abs/2406.04312) (Eyring et al., NeurIPS 2024) — 100× faster than DOODL on one-step models; combines 4 rewards to avoid hacking.
- [**SVDD: Soft Value-Based Decoding in Diffusion**](https://arxiv.org/abs/2408.08252) (Li et al.) — SMC framework with intermediate-step potentials.
- [**FK Steering: Feynman-Kac Steering of Diffusion Models**](https://arxiv.org/abs/2501.06848) (Singhal et al.) — Subsumes TDS and SVDD.
- [**Inference-Time Scaling for Diffusion Models**](https://arxiv.org/abs/2501.09732) (Ma et al., 2025) — Design-space paper: random search over noise, zero-order search, search over paths.

### Compositional / energy-based

- [**Composable Diffusion Models**](https://arxiv.org/abs/2206.01714) (Liu, Du et al., ECCV 2022) — Treats multiple conditions as score functions and adds them.
- [**Reduce, Reuse, Recycle: Compositional Generation with Energy-Based Diffusion Models and MCMC**](https://arxiv.org/abs/2302.11552) (Du, NeurIPS 2023) — Corrects ill-conditioned product via Langevin / HMC MCMC at intermediate noise.

### Editing and inversion

- [**SDEdit: Image Synthesis and Editing with Stochastic Differential Equations**](https://arxiv.org/abs/2108.01073) (Meng et al., 2022) — Add noise to a real input, re-denoise with a new condition.
- [**Prompt-to-Prompt Image Editing with Cross-Attention Control**](https://arxiv.org/abs/2208.01626) (Hertz et al., 2022) — Edit images via cross-attention map control.

### Particle and restart methods

- [**Particle Guidance**](https://arxiv.org/abs/2310.13102) (Corso et al., ICLR 2024) — Non-IID sampling with a joint particle potential for diversity.
- [**Restart Sampling for Improving Generative Processes**](https://arxiv.org/abs/2306.14878) (Xu, Jaakkola, NeurIPS 2023) — Alternates ODE-deterministic denoising with forward noise injections.

### Control and adapter methods

- [**ControlNet: Adding Conditional Control to Text-to-Image Diffusion Models**](https://arxiv.org/abs/2302.05543) (Zhang et al.) — Locked backbone + zero-conv side branch.
- [**T2I-Adapter: Lightweight Adapters for Diffusion Models**](https://arxiv.org/abs/2302.08453) (Mou et al.) — Small adapters provide structural control.

### Flow matching

- [**Flow Matching for Generative Modeling**](https://arxiv.org/abs/2210.02747) (Lipman et al.) — Foundational flow matching paper.
- [**Guided Flows for Generative Modeling and Decision Making**](https://arxiv.org/abs/2311.13443) (Zheng et al., 2023) — CFG for flow matching; decision-making bridge.
- [**CFG-Zero***](https://arxiv.org/abs/2503.18886) (Fan et al.) — Early zeroing + optimized guidance scale for flow-matching CFG.
- [**Improving CFG of Flow Matching via Manifold Projection**](https://arxiv.org/abs/2510.05529) (Cai et al.) — Manifold projection + Anderson acceleration for stable flow CFG.
