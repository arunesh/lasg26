# InstructGPT, small and from scratch

`instruct_gpt.ipynb` walks through [Ouyang et al. 2022 (InstructGPT)](https://arxiv.org/abs/2203.02155) on a **Pythia** base model, which only predicts the next token:

1. **SFT** on Dolly-15k demonstrations
2. **Reward model** on Anthropic HH-RLHF preference pairs (pairwise loss, initialised from SFT)
3. **RL**: a from-scratch PPO-style clipped policy gradient with the paper's KL-to-SFT penalty and optional ptx loss (RLOO baseline instead of a value network)
4. **Evaluation**: base vs few-shot base vs SFT vs RL. Covers RM win rate, stopping and template-leak rates, repetition, ROUGE-L, likelihood of human demonstrations, and WikiText perplexity (alignment tax)
5. **Interpretability**: hook-based tracing of attention, residual stream and MLP neurons, then segment-level attention, head-change maps, logit lens, exact direct logit attribution, and weight/representation drift, all compared across base → SFT → RL on identical token sequences

## Profiles

Selected with the `IG_PROFILE` env var. The default is `gpu0` when CUDA is available, otherwise `local`.

| profile | model | approx. time |
|---|---|---|
| `smoke` | pythia-70m, a few steps of each stage | ~3 min (checks the pipeline end to end) |
| `local` | pythia-160m | ~45–60 min on an M-series Mac |
| `local_410m` | pythia-410m | ~1.5–2 h on an M-series Mac |
| `gpu0` | pythia-1.4b | ~1–1.5 h on the A100-40GB |

To swap the base model while keeping a profile's training budget, set `IG_MODEL`, e.g. `IG_MODEL=HuggingFaceTB/SmolLM2-135M`. That model has been tested end to end; GPT-2 and other Llama-style models should also work.

Checkpoints and logs go to `checkpoints/<profile>-<model>/`. Re-running the notebook reuses them (`REUSE_CHECKPOINTS = True`), so you can jump straight to the analysis.

## Run locally

```bash
cd instruct_gpt
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m ipykernel install --user --name instruct_gpt --display-name "Python (instruct_gpt .venv)"
IG_PROFILE=smoke jupyter lab          # first a quick pipeline check, then drop IG_PROFILE
```

The notebooks default to the **"Python (instruct_gpt .venv)"** kernel. If you see `No module named 'datasets'`, the notebook is running on a different Python: pick that kernel from the kernel menu (in VS Code, "Select Kernel" at the top right).

## Run on gpu0

```bash
# 1. copy the folder (from the repo root on your laptop)
rsync -av --exclude checkpoints --exclude .venv instruct_gpt/ gpu0:~/instruct_gpt/

# 2. one-time environment setup on gpu0 (may need: sudo apt install python3.10-venv)
ssh gpu0
cd ~/instruct_gpt && python3 -m venv .venv && source .venv/bin/activate
pip install -U pip && pip install -r requirements.txt
python -m ipykernel install --user --name instruct_gpt --display-name "Python (instruct_gpt .venv)"
python -c "import torch; print(torch.cuda.is_available(), torch.cuda.get_device_name())"

# 3a. interactive: start Jupyter inside tmux so it survives disconnects
tmux new -s ig
source .venv/bin/activate && jupyter lab --no-browser --port 8890
#     then on your laptop:
ssh -N -L 8890:localhost:8890 gpu0     # open the http://localhost:8890/?token=... URL Jupyter printed

# 3b. or headless: run the whole notebook in tmux and keep the executed copy
mkdir -p runs && jupyter nbconvert --to notebook --execute instruct_gpt.ipynb \
    --output runs/gpu0.ipynb --ExecutePreprocessor.timeout=-1

# 4. optional: bring the trained checkpoints back to analyse locally (IG_PROFILE=gpu0 reuses them)
rsync -av gpu0:~/instruct_gpt/checkpoints/ instruct_gpt/checkpoints/
```

VS Code's Remote-SSH extension also works: open `~/instruct_gpt` on gpu0 and choose the `.venv` kernel.
