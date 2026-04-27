# FinBen2

FinBen2 is a financial-domain large language model evaluation framework built on top of [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness). It provides a clean directory layout for organizing custom financial tasks, evaluation runners, and result aggregation, while keeping the underlying harness as a Git submodule so that upstream improvements can be pulled in at any time.

This repository is the successor of [Yan2266336/FinBen](https://github.com/Yan2266336/FinBen), redesigned around the official `lm-evaluation-harness` rather than a forked copy of `finlm_eval`.

## Repository Structure

```
FinBen2/
├── lm-evaluation-harness/   # Git submodule -> Yan2266336/lm-evaluation-harness (fork of EleutherAI/lm-evaluation-harness)
├── tasks/                   # Custom financial evaluation tasks (YAML configs, prompt templates, etc.)
├── results/                 # Evaluation outputs, logs, and aggregated metrics
├── scripts/                 # Shell entry points for running evaluations (e.g. run_xxx.sh)
├── aggregate.py             # Utility for aggregating per-task results into summary tables
├── .gitmodules              # Submodule configuration
└── README.md
```

> The `lm-evaluation-harness` directory is **not** a regular folder. It is a Git submodule pointed at a specific commit of the fork `Yan2266336/lm-evaluation-harness`, which itself tracks the upstream `EleutherAI/lm-evaluation-harness`.

## Getting Started

### 1. Clone the repository (with submodules)

```bash
# Recommended: clone everything at once
git clone --recurse-submodules git@github.com:Yan2266336/FinBen2.git
cd FinBen2
```

If you have already cloned the repository without `--recurse-submodules`, initialize the submodule manually:

```bash
git submodule update --init --recursive
```

### 2. Set up the Python environment

```bash
conda create -n finben2 python=3.10 -y
conda activate finben2

# Install lm-evaluation-harness from the submodule (editable mode)
cd lm-evaluation-harness
pip install -e .
cd ..
```

Add any extra dependencies your custom tasks need (e.g. `pip install -r requirements.txt`).

## Running Evaluations

A typical run uses `lm_eval` from the submodule together with task configs in `tasks/` and writes outputs into `results/`.

```bash
lm_eval \
    --model hf \
    --model_args pretrained=meta-llama/Llama-3.1-8B-Instruct \
    --tasks my_finben_task \
    --include_path ./tasks \
    --batch_size 8 \
    --output_path ./results/llama3_8b
```

Convenience scripts (e.g. `scripts/run_xxx.sh`) wrap the most common configurations.

After collecting results, aggregate them into a single summary:

```bash
python aggregate.py --results_dir ./results --output ./results/summary.csv
```

## Adding a New Task

1. Create a new directory under `tasks/<your_task_name>/`.
2. Add a task YAML following the `lm-evaluation-harness` task format.
3. (Optional) Add prompt templates, few-shot examples, or post-processing utilities.
4. Reference the task with `--tasks <your_task_name> --include_path ./tasks`.

For format details, see the [task guide](https://github.com/EleutherAI/lm-evaluation-harness/blob/main/docs/new_task_guide.md) in the harness documentation.

## Keeping the Submodule in Sync with Upstream

The submodule points at the fork `Yan2266336/lm-evaluation-harness`. To pull in the latest changes from `EleutherAI/lm-evaluation-harness`:

```bash
# 1. Enter the submodule
cd lm-evaluation-harness

# 2. Add the upstream remote (only the first time)
git remote add upstream https://github.com/EleutherAI/lm-evaluation-harness.git

# 3. Fetch and merge upstream changes
git fetch upstream
git checkout main
git merge upstream/main

# 4. Push the updated branch back to your fork
git push origin main

# 5. Record the new submodule pointer in FinBen2
cd ..
git add lm-evaluation-harness
git commit -m "chore: bump lm-evaluation-harness to latest upstream"
git push
```

To quickly update the submodule to the latest commit on its tracked branch without going into the directory:

```bash
git submodule update --remote --merge lm-evaluation-harness
git add lm-evaluation-harness
git commit -m "chore: update lm-evaluation-harness submodule"
```

## Acknowledgements

- [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) — the underlying evaluation framework.
- [The-FinAI/FinBen](https://github.com/The-FinAI/FinBen) — original FinBen benchmark suite that inspired this repository.

## License

This repository follows the license of `lm-evaluation-harness` (MIT) for the submodule code. Custom code in `tasks/`, `scripts/`, and `aggregate.py` is released under the MIT License unless stated otherwise.
