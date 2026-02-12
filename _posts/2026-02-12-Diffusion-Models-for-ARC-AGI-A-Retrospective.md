---
title: "Diffusion Models for ARC-AGI: A Retrospective"
---

In early July 2024, I spent about two weeks applying diffusion models to the ARC-AGI abstract reasoning benchmark. It was my first ML project after taking the fastai courses on diffusion. I solved one task, nearly solved a second, and moved on before running a systematic evaluation.

I never wrote any of it up at the time. Recently, seeing diffusion approaches gain traction in the ARC community, I wanted to go back and document what I'd tried while the details were still recoverable from my notebooks and ChatGPT logs.

For timestamps: here's me on [June 27, 2024](https://x.com/C_H_Wood/status/1806436104138297747) suggesting the approach could work, and on [July 2, 2024](https://x.com/C_H_Wood/status/1807884670438359072) celebrating solving the first task.

## Why Diffusion for ARC?

The idea didn't start with diffusion. It started with a question: if I have an input and an output and the output is a complex combination of transformations to the input, but I have no idea what those transformations are, how do I figure out what they are?

My first instinct was signal processing. Could a Fourier transform decompose an unknown transformation into simpler constituent parts? The answer was no, not for arbitrary transformations. Fourier analysis decomposes signals into frequency components, but it doesn't discover unknown mappings between inputs and outputs. That's a learning problem. But the question pushed my thinking in a specific direction: what if I represented the data as something signal-like? ARC grids are discrete matrices with integer values 0 through 9. If I treated the output as a "clean signal" and the input as a "corrupted signal," then the transformation rule becomes the corruption, and learning to reverse it means learning the rule.

That's what led me to diffusion. And the key insight, which I still stand by: the forward process in diffusion is just a function. It's conventionally stochastic Gaussian noise, but nothing requires that. If you define the forward process as a deterministic transformation and train a model to reverse it, a model that succeeds has learned the transformation. The "noise" doesn't have to be random. If the model learns to remove noise that was added in a structured, non-random way, it has learned the function that added it.

In mid-2024, nobody in the ARC community was talking about diffusion. The dominant approaches were program synthesis, DSLs, and LLMs.

## What I Built

ARC grids are single-channel matrices of integers 0-9, with height and width varying between 3 and 30. I had a harness that compressed these and embedded them into a standardized 16-channel, 16x16 representation. All the diffusion work operated on these embeddings.

The core approach was latent diffusion: an autoencoder to learn a latent space from the embeddings, then a UNet denoiser in that space, treating the input as noisy and predicting the residual to reach the output.

Getting to a version that worked took six iterations. I tried deeper autoencoders, bigger latent dimensions, more UNet complexity, a second-stage denoising model, random training data to make the autoencoder general-purpose, and various combinations. Each failure was informative: bigger latents made the denoising problem exponentially harder regardless of autoencoder quality, and more UNet depth and dropout couldn't compensate for a poorly structured latent space.

The breakthrough was representational. My earlier versions collapsed the grid into a flat 1x1 latent vector and artificially reshaped it into spatial dimensions for the UNet to convolve over. The version that worked let the encoder's stride-3 convolution naturally produce a 256x3x3 spatial latent, where each of the 9 latent positions corresponded to a 3x3 region of the 9x9 grid. The latent mirrored the spatial structure of the problem. With that single change, the UNet simplified from 5 levels to 2, loss dropped by an order of magnitude, and the autoencoder loss improved 100x over any flat-latent version. This is the version that solved a task.

About a week later, I tried removing the autoencoder and having the UNet predict the input-to-output residual directly in pixel space. Both approaches were single-step residual prediction. The question was whether the learned latent representation was doing useful work. For the task I'd already solved (where the output is the input tiled according to a pattern), the pixel-space version was getting the raw tiled input as a starting point, already structurally close to the answer. The latent version worked through an additional layer of abstraction. The pixel-space result still wasn't as promising. The fact that the latent approach performed better despite the pixel version having a head start is a signal that the learned latent space was contributing something meaningful. When I returned to the latent approach for the next task, I made small refinements: centering the normalization around zero, simplifying the decoder, and experimenting with different augmentation strategies (shifted color mappings in addition to random permutations).

All versions trained per-task with heavy augmentation: color permutation and geometric transforms to generate thousands of training pairs from each task's few examples.

## The Clue Giver / Solver Architecture

Alongside the latent diffusion work, I designed a system I called the "clue giver" and "solver."

A **clue giver** starts with the clean output image and progressively adds noise, one step at a time, until it reaches the noisy input image. At each step, it lays down a "clue" that a solver can follow. I thought of it like a maze: the clue giver starts at the exit and leaves a trail of breadcrumbs back to the entrance. The path doesn't have to be direct. It can wind, overshoot, or take detours, as long as the solver can follow it.

A **solver** (a timestep-conditioned UNet) learns to reverse each step: given the clue at timestep t, undo the noise to recover timestep t-1. Multiple solvers would each learn different paths through the noise, creating an ensemble. At test time, you give the single test input to every solver. Each one produces a different candidate output because it learned a different reversal strategy. You pick the most frequently occurring answer.

The clue giver went through several iterations. I first tried building it as a convolutional network that would learn to generate context-aware noise, accounting for the colors and shapes in each image (important because my augmented data meant noise patterns varied across a batch). I designed loss functions that balanced progress toward the target, smoothness between timesteps, and the solver's ability to undo each step. I experimented with a U-shaped convergence schedule where the start and end of the noise trajectory were tightly constrained but the middle was allowed to diverge and explore.

Eventually I simplified. The convolutional clue giver was overengineered for what I needed. I replaced it with a function that does linear interpolation between the clean and noisy embeddings, with element-wise sampling from a bounded normal distribution. The mean of the distribution at each element is the interpolated value. The min is the current embedding value (so we only add noise, never remove it). The max is the noisy embedding value. The standard deviation is proportional to the noise range at each element. In early timesteps, the distribution is centered near the clean value with a long tail toward the noisy value. In later timesteps, it shifts toward the noisy value. This gives you a controlled stochastic walk from clean to noisy, with different random seeds producing different paths.

I also switched from a linear interpolation schedule to a cosine schedule, so the process moves in smaller increments near the endpoints and bigger steps in the middle.

## The Ideas Behind the Architecture

**Denoising as stochastic search.** The most important idea: the clue giver's stochastic sampling produces a different path from clean to noisy each time it runs. Different solvers train on different sampled paths, so they internalize different reversal strategies. At test time, you give the single test input to every solver. Each one produces a different candidate output because it learned a different way to denoise. You pick the most frequently occurring answer.

This turns diffusion into a search strategy over possible transformations. The randomness lives in training, not inference. The "search" is over learned strategies, not over starting points. Multiple solvers exploring different regions of strategy space is doing the same work that random restarts do in combinatorial search. I [came back to this idea in October 2024](https://x.com/C_H_Wood/status/1846467721997046005): "diffusion and random noise injections could potentially lead to random search over various paths."

**Learned forward process.** In standard diffusion, the forward process is trivial: add Gaussian noise. But ARC transformations are structured, not random. I explored training a network to learn the noise addition process itself: given a clean embedding and a target noisy embedding, can we learn a noise schedule that incrementally corrupts one into the other? Making the forward process learnable was an attempt to let the model discover the structure of the transformation rather than imposing a noise assumption that doesn't fit. This is what led to the convolutional clue giver architecture before I simplified it.

**Branching model ensembles.** After 5 epochs of training, I'd deep-copy the UNet into 3 copies and freeze the first layer of each fork. After 15 more epochs, I'd fork again, ending up with 9 models that shared early features but diverged in their higher-level strategies. I'd pick the best result across all forks.

This is adjacent to existing techniques, but the specific combination is different. Snapshot Ensembles (Huang et al., 2017) save checkpoints at different local minima but don't fork into parallel copies. Population Based Training (Jaderberg et al., 2017) trains populations in parallel but from different random initializations. FreezeOut (Brock et al., 2017) uses progressive freezing for speed, not diversity. What I was doing was creating a branching tree through weight space where structural freezing ensures forked models agree on low-level features but are forced to explore different high-level strategies.

**Diffusion inference as optimization.** I also questioned why gradient descent optimizers like Adam and momentum hadn't been applied to the diffusion inference process itself. The denoising trajectory iteratively refines an image, which looks structurally similar to iterative parameter optimization. If you treat the pixel values as "parameters" being optimized toward a target, techniques like momentum (to smooth the trajectory) or adaptive learning rates (to adjust step sizes per dimension) seem like they should help. I didn't implement this, but the intuition connects to later work on guided diffusion and score-based refinement, where the denoising trajectory is steered with gradient-like signals.

## Results

I worked through tasks one at a time rather than running a benchmark sweep. The approach solved one task outright and came within two cells of solving a second.

Here's what the near-miss looked like on one of the training examples for the second task:

[screenshot: input grid (purple bars on black), expected output (blue/green bars on black), model prediction (blue/green bars on black, nearly identical to expected output)]

The model captured the structural transformation: colors mapped correctly, spatial arrangement preserved, overall pattern right. On the training examples it was producing correct outputs, but on the actual challenge input it was consistently off by two cells. My diagnosis was that the early convolutional layers in the UNet were losing too much spatial information during downsampling. The embedding format was also fragile: small errors in the embedding space could cascade into incorrect cell values after decoding. These are engineering problems, not conceptual ones. The ARChitects later solved similar spatial information challenges with 2D positional encoding and soft masking.

I didn't run across all 400+ training tasks, so the ceiling is unknown.

## What Happened After

I moved on to other projects. Over the following year, diffusion for ARC became a real research thread, and it's been interesting to see where the ideas overlap and diverge from what I was trying.

The ARChitects team placed 2nd in ARC Prize 2025 using masked diffusion with 8B parameters, 2D positional encoding, and recursive self-refinement. Their soft-masking approach turns every grid position into a partially noisy state and iterates, exploring solution space through stochastic refinement. There's a family resemblance to the "random search over paths" idea I was working toward, though their execution is obviously far beyond anything I built.

Trelis Research and Matthew Newton explored diffusion for ARC in 2025 and found that diffusion paths "converge quickly and either get the right answer or get stuck." They also found that majority voting over 72 augmented starting points improved scores by 20-30%, which is interesting because it's essentially the stochastic search idea: try different starting points, pick the consensus.

The ARC Prize 2025 technical report identified per-task "refinement loops" as the dominant paradigm across the top finishers. The Tiny Recursive Model (1st place paper, ~7M parameters) trains from scratch per task. NVARC (1st place Kaggle) and the ARChitects both use test-time training.

## What I'd Say Now

**What I got right:** The general direction. Framing ARC as denoising was non-obvious in mid-2024 and turned out to be productive. The stochastic search idea (different noise paths produce different candidate solutions, pick the consensus) showed up in the ARChitects' soft-masking and Trelis's majority voting work, though I can't say how independently they arrived at it. Per-task training with small models fit within the competition's compute and time budget, and it became the dominant approach.

**What I didn't finish:** The systematic evaluation. I worked through tasks individually instead of running across the full training set. I also didn't run the clean experiment that would have tested the stochastic search hypothesis directly: train on one task, generate N candidates from different noise seeds, show the correct answer appears more often than chance.

**What was hard:** The information loss in the UNet's early convolutional layers was a real barrier. The embedding format was fragile. I was two weeks into my first ML project, working alone on Colab and Kaggle free-tier GPUs, with GPT-4o as my most capable AI assistant (Claude 3.5 Sonnet had launched days earlier). These are engineering problems that iteration and better architectural choices would likely address.

**What I think now:** The approach is theoretically sound. The forward process is just a function. It doesn't have to be stochastic. My main worry is that convolutions are too lossy before the data reaches the attention layers when creating the latents. That was the failure mode on the second task, and it might be fundamental rather than fixable. But I'd want to try more creative workarounds before accepting that. I still have an itch to go back, run it properly across the full benchmark, and find out where it actually breaks. Maybe the per-task latent space can't generalize to harder transformations. Maybe the convolution bottleneck is a deeper problem than I think. I'd need to see it fail systematically before I'd be convinced to drop this line of thinking altogether.

---

*Notebooks: [ARC-AGI Embedding](https://www.kaggle.com/code/christopherhwood/arc-agi-embedding), [Notebook 92c6e6e661](https://www.kaggle.com/code/christopherhwood/notebook92c6e6e661), [Notebook 29802a10c4](https://www.kaggle.com/code/christopherhwood/notebook29802a10c4), [ARC-AGI Merged Embedding](https://www.kaggle.com/code/christopherhwood/arc-agi-merged-embedding). Reach me at [christopherhwood@gmail.com](mailto:christopherhwood@gmail.com).*
