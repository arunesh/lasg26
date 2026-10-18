 I've built the barebones notebook at instruct_gpt/smollm_quickstart.ipynb, using SmolLM2-135M. It has 18 cells and no configs, and I ran it start to finish on this Mac in about 6 minutes, 5 of them fine-tuning.

  Can the Mac load it? Easily. The 135M model is about 0.5 GB, and even the 1.7B version (~7 GB) fits in your 32 GB.

  What can 5 minutes of fine-tuning do? On your Mac that was 342 steps, or about 2,700 short Dolly examples at 8 per batch. It's enough to change the model's behaviour clearly, though not its knowledge:

  ┌──────────────────────┬──────────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────┬───────────────────────────────────────────────────────────────────────────┬──────────────────────────────┐
  │        Prompt        │                             Base                             │         Base, prompted as "Question: … Answer:"          │                            Our 5-min fine-tune                            │    SmolLM2-135M-Instruct     │
  ├──────────────────────┼──────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┼──────────────────────────────┤
  │ Capital of France?   │ Rambles about Paris's population and the Celts               │ "Paris." then starts writing more quiz questions         │ "Paris", then stops                                                       │ Paris, plus a long paragraph │
  ├──────────────────────┼──────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┼──────────────────────────────┤
  │ Three red fruits     │ Starts writing a quiz ("10) What is the name of the fruit?") │ Strawberries, raspberries, blackcurrants… then more quiz │ "Red apples, strawberries and raspberries"                                │ A wrong list of 30+ colours  │
  ├──────────────────────┼──────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┼──────────────────────────────┤
  │ Why is the sky blue? │ "The answer can be found by looking at a diagram…"           │ A rambling answer                                        │ One mostly correct sentence about scattering by air molecules, then stops │ Somewhat wrong explanation   │
  └──────────────────────┴──────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────────────────────────┴──────────────────────────────┘

  What the notebook shows, cell by cell:
  1. Base model, bare prompt: it treats your question as the start of a document and keeps writing, often more questions.
  2. Base model, made to look like a Q&A page: it answers, but has no idea when to stop.
  3. Official Instruct model: it prints the chat template so you can see the special tokens the model actually receives, then answers the same questions.
  4. 5-minute fine-tune: the training loop is about 15 lines. Loss is computed only on the answers, and each answer ends with the end-of-text token, which is how the model learns to stop.
  5. Before vs after: all four side by side, plus a cell for your own question.

  What 5 minutes doesn't buy:
  - The answers copy Dolly's style, which is terse.
  - The official Instruct model writes longer, more structured answers. SmolLM2's own instruction tuning (fine-tuning, then preference training) used far more data.
  - Neither model is reliably right at 135M (see the red-fruits row).
  - Raising MINUTES to 15–20 should improve fluency but probably won't fix facts.

  To run it, use the venv from the README and open smollm_quickstart.ipynb. The first run downloads about 1 GB of models and data.
------------------------------------------------------------------------------------------------------------------------
⏺ I've built the barebones notebook at instruct_gpt/smollm_quickstart.ipynb, using SmolLM2-135M. It has 18 cells and no configs, and I ran it start to finish on this Mac in about 6 minutes, 5 of them fine-tuning.

  Can the Mac load it? Easily. The 135M model is about 0.5 GB, and even the 1.7B version (~7 GB) fits in your 32 GB.

  What can 5 minutes of fine-tuning do? On your Mac that was 342 steps, or about 2,700 short Dolly examples at 8 per batch. It's enough to change the model's behaviour clearly, though not its knowledge:

  ┌─────────────────────┬──────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────┬─────────────────────────────┐
  │       Prompt        │                           Base                           │       Base, prompted as "Question: … Answer:"        │                         Our 5-min fine-tune                          │    SmolLM2-135M-Instruct    │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────┼─────────────────────────────┤
  │ Capital of France?  │ Rambles about Paris's population and the Celts           │ "Paris." then starts writing more quiz questions     │ "Paris", then stops                                                  │ Paris, plus a long          │
  │                     │                                                          │                                                      │                                                                      │ paragraph                   │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────┼─────────────────────────────┤
  │ Three red fruits    │ Starts writing a quiz ("10) What is the name of the      │ Strawberries, raspberries, blackcurrants… then more  │ "Red apples, strawberries and raspberries"                           │ A wrong list of 30+ colours │
  │                     │ fruit?")                                                 │ quiz                                                 │                                                                      │                             │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────┼─────────────────────────────┤
  │ Why is the sky      │ "The answer can be found by looking at a diagram…"       │ A rambling answer                                    │ One mostly correct sentence about scattering by air molecules, then  │ Somewhat wrong explanation  │
  │ blue?               │                                                          │                                                      │ stops                                                                │                             │
  └─────────────────────┴──────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────┴─────────────────────────────┘

  What the notebook shows, cell by cell:
  1. Base model, bare prompt: it treats your question as the start of a document and keeps writing, often more questions.
  2. Base model, made to look like a Q&A page: it answers, but has no idea when to stop.
  3. Official Instruct model: it prints the chat template so you can see the special tokens the model actually receives, then answers the same questions.
  4. 5-minute fine-tune: the training loop is about 15 lines. Loss is computed only on the answers, and each answer ends with the end-of-text token, which is how the model learns to stop.
  5. Before vs after: all four side by side, plus a cell for your own question.

  What 5 minutes doesn't buy:
  - The answers copy Dolly's style, which is terse.
  - The official Instruct model writes longer, more structured answers. SmolLM2's own instruction tuning (fine-tuning, then preference training) used far more data.
  - Neither model is reliably right at 135M (see the red-fruits row).
  - Raising MINUTES to 15–20 should improve fluency but probably won't fix facts.

  To run it, use the venv from the README and open smollm_quickstart.ipynb. The first run downloads about 1 GB of models and data.


------------------------------------------------------------------------------------------------------------------------
⏺ Yes, it works with SmolLM2-135M. I ran the whole notebook with it in the quick test mode: training, the reward model, RL, evaluation and all the tracing sections. No code changes were needed. I added one thing, an
  IG_MODEL setting that swaps the model and keeps everything else:

  IG_MODEL=HuggingFaceTB/SmolLM2-135M jupyter lab

  Checkpoints go to their own folder, so the Pythia and SmolLM runs don't overwrite each other. I also added a note on this to Section 2 of the notebook and to the README.

  What's different about SmolLM2:

  - Modern design. It's built like Llama and current large models: a deep, narrow network (30 layers, 9 attention heads) where Pythia-160m has 12 layers and 12 heads. Pythia-160m is closer to GPT-2.
  - More to look at. The attention and change heatmaps become 30×9 instead of 12×12.
  - The "which layers push it to stop" breakdown is still exact. The sum of the parts matched the actual value to 3 decimals.
  - The MLP neuron view means something slightly different. SmolLM2's MLP has an extra gating step, so the "neuron activations" are taken one step before the final multiplication. I noted this in the notebook.

  The catch is that the base model already half follows instructions. Before any fine-tuning, SmolLM2-135M answered "Translate to French: Good morning…" with "Bonjour, mademoiselle." It also explained why the sky is blue.
  Pythia-70m produced unrelated text for the same prompts. SmolLM2's training data includes synthetic, textbook-style and some instruction-like text.

  It still shows the classic signs of an untuned model: it never stopped on its own (0 of 8 answers), and it kept writing new ### Instruction: blocks. Fine-tuning still teaches it to stop and stay on the answer, but the
  before/after contrast is smaller than with Pythia.

  The test-mode numbers come from a few dozen training steps on 8 prompts, so they only show that everything runs, not how good the results are.

  My suggestion: keep Pythia-160m as the main run, because it shows InstructGPT's core story cleanly: a model that only predicts text learns to follow instructions. Then run SmolLM2-135M as a comparison, which is now one
  setting. Comparing the two is itself a good discussion point: how much does pretraining on instruction-like data shrink what fine-tuning and RL add?
------------------------------------------------------------------------------------------------------------------------
⏺ I've created the notebook at instruct_gpt/instruct_gpt.ipynb, along with a README.md (local and gpu0 instructions), requirements.txt and a .gitignore for checkpoints. It runs end to end: I executed the whole thing with a
  tiny model (IG_PROFILE=smoke, about 3 minutes on your Mac). The results that matter, the real 160m and 1.4B runs, haven't been run yet. Nothing is committed.

  Model choice

  I picked Pythia (EleutherAI). It was trained only on the Pile, so it has essentially no chat or instruction data, which keeps the before/after contrast honest. It comes in a range of sizes, interpretability tools support
  it well, and every block simply adds to the residual stream, which makes some of the analyses below exact. The notebook also compares it with GPT-2, SmolLM2 and Qwen base models; Qwen's base models already follow chat
  formats, which defeats the purpose.

  I timed training on your Mac: a batch of 8 takes about 2.7 s for 410m and 1.2 s for 160m. That set the profiles:

  ┌─────────────────────────────────────┬─────────────┬──────────────────────────────────┐
  │               Profile               │    Model    │           Approx. time           │
  ├─────────────────────────────────────┼─────────────┼──────────────────────────────────┤
  │ local (default on the Mac)          │ pythia-160m │ ~45–60 min                       │
  ├─────────────────────────────────────┼─────────────┼──────────────────────────────────┤
  │ local_410m                          │ pythia-410m │ ~1.5–2 h                         │
  ├─────────────────────────────────────┼─────────────┼──────────────────────────────────┤
  │ gpu0 (default when CUDA is present) │ pythia-1.4b │ ~1–1.5 h (estimated) on the A100 │
  └─────────────────────────────────────┴─────────────┴──────────────────────────────────┘

  The InstructGPT pipeline

  - Step 1, supervised fine-tuning: on Dolly-15k, whose task categories are close to the paper's. Loss is computed only on the response, and the end-of-text token is what teaches the model to stop.
  - Step 2, reward model: initialised from the fine-tuned model, using the paper's pairwise loss on single-turn Anthropic HH-RLHF pairs. It is normalised so a typical human answer scores 0, as in the paper.
  - Step 3, RL: written from scratch, not with a library. It uses the paper's objective (reward minus a KL penalty to the fine-tuned model, plus an optional pretraining-loss term) and PPO-style clipping. In place of a value
    network, each sample is compared against the average of the other samples for the same prompt.

  Metrics

  - Reward-model win rate against the fine-tuned model, the stand-in for the paper's human preference win rate.
  - How often the model stops on its own, and how often it slips back into writing the template instead of answering.
  - Repetition, overlap with reference answers (ROUGE-L), and how likely the model finds held-out human answers.
  - WikiText perplexity, the stand-in for the paper's "alignment tax" on standard benchmarks.
  - A check of the reward model against the trivial rule "the longer answer wins". In the smoke run that rule beat the reward model (66% vs 62%), and reward correlated with length at +0.65, a good lead-in to discussing
    reward hacking.

  Tracing attention and activations

  - Capture: PyTorch hooks record attention weights, each block's output, MLP neuron activations and the residual stream. Traces are saved to disk for each model and probe prompt.
  - During training: every N steps it logs cheap summary numbers: the probability of stopping at the right place, attention from the answer back to the instruction, and attention parked on the first token.
  - Same tokens for every model: comparisons run base, fine-tuned and RL models on the same fixed prompt-plus-answer text, so any difference comes from the weights, not from different generated text.
  - Analyses:
    - attention heatmaps per head, plus an optional interactive view (bertviz)
    - attention by segment and layer
    - which heads changed most
    - layer-by-layer prediction of the next token ("logit lens")
    - which layers push the model to stop (exact attribution; the smoke run confirms the parts sum to the actual value)
    - how much each layer's representation and weights changed
    - MLP neurons whose behaviour shifted
    - a "trace your own prompt" cell

  Section 12 lists further experiments, including DPO, KL and pretraining-loss sweeps, an LLM judge, lm-eval-harness, and TransformerLens or nnsight for activation patching.

  Running it

  - Library versions on your Mac's system Python are too old (transformers 4.47), so use the README's venv steps.
  - Two transformers 5.x behaviours the code handles:                                                                                               
    - Pythia now loads in fp16 unless told otherwise, so the code asks for fp32.
    - Attention weights are only available with the slower "eager" kernel, so tracing switches to it temporarily.
  - gpu0 has no torch installed yet; the README has the setup and an SSH-tunnelled Jupyter or headless nbconvert run.

  Should I kick off the local profile in the background now so the checkpoints are ready, or set up the environment on gpu0 and start the 1.4B run there?
