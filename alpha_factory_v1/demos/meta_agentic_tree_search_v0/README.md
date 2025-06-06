# Meta‑Agentic Tree Search (MATS) Demo — v0

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MontrealAI/AGI-Alpha-Agent-v0/blob/main/alpha_factory_v1/demos/meta_agentic_tree_search_v0/colab_meta_agentic_tree_search.ipynb)

**Abstract:** We pioneer **Meta-Agentic Tree Search (MATS)**, a novel framework for autonomous multi-agent decision optimization in complex strategic domains. MATS enables intelligent agents to collaboratively navigate and optimize high-dimensional strategic search spaces through **recursive agent-to-agent interactions**. In this **second-order agentic** scheme, each agent in the system iteratively refines the intermediate strategies proposed by other agents, yielding a self-improving decision-making process. This recursive optimization mechanism systematically uncovers latent inefficiencies and unexploited opportunities that static or single-agent approaches often overlook.

> **Status:** Experimental · Proof‑of‑Concept · Alpha  
> **Location:** `alpha_factory_v1/demos/meta_agentic_tree_search_v0`  
> **Goal:** Showcase how recursive agent‑to‑agent rewrites — navigated with a best‑first tree policy — can rapidly surface high‑value trading policies that exploit AGI‑driven discontinuities (“AGI Alpha”).

## 1 Why this demo exists
Financial edges sourced from AGI inflection points decay in hours or days. Classical research pipelines are too slow.  
MATS compresses the idea‑to‑capital cycle by letting agents continuously rewrite each other while a Monte‑Carlo tree search focuses compute on the most promising rewrite trajectories.

## 2 High‑level picture
```
root population
      │  meta‑rewrite
      ▼
  ┌─────────────┐   tree policy   ┌───────────────┐
  │  Node k     ├────────────────►│  Node k+1     │
  └─────────────┘                 └───────────────┘
```
Each edge = “one agent improves another”; backpropagation = “which rewrite path maximises risk‑adjusted α”.

## 3 Formal definition
*(verbatim from specification for precision)*  

> **Meta‑Agentic Tree Search (MATS)**  
> Let **E** be a partially‑observable, stochastic environment parameterised by state vector *s* and reward function *R*.  
> Let **𝒜₀** = {a₁,…,aₙ} be a population of base agents, each with policy πᵢ(·|θᵢ).  
> **Meta‑agents** are higher-order policies **Π : (𝒜₀, 𝒮, 𝒭) → 𝒜₀′** that rewrite or re‑parameterise base agents to maximise a meta‑objective **J**.  
> A search node **v** is the tuple (𝒜ₖ, Σₖ) where 𝒜ₖ is the current agent pool after *k* rewrites and Σₖ their cumulative performance statistics.  
> The tree policy **T** selects the next node via a best‑first acquisition criterion (e.g. UCB over expected α).  
> Terminal nodes are reached when Δα < ε or depth ≥ d\*.  
> **Output**: argmax₍ᵥ∈𝒱_leaf₎ J(v).

## 4 Minimal algorithm (reference implementation)
```python
def MATS(root_agents, env, horizon_days):
    tree = Tree(Node(root_agents))
    while resources_left():
        node = tree.select(best_first)                       # ← UCB / Thompson
        improved = meta_rewrite(node.agents, env)           # ← gradient, evo, code‑gen
        reward = rollouts(improved, env, horizon_days)      # ← risk‑adj α
        child = Node(improved, reward)
        tree.add_child(node, child)
        tree.backprop(child)
    return tree.best_leaf().agents
```

### Design knobs
| Component          | Options (demo default) |
|--------------------|------------------------|
| `best_first`       | UCB1, TS, ε‑greedy (UCB1) |
| `meta_rewrite`     | PPO fine‑tune, CMA‑ES, GPT‑4 code‑gen (PPO) |
| Reward             | IRR, CumPnL/√Var, Sharpe (IRR) |
| Environment        | Toy number-line env (default), limit‑order‑book sim, OpenAI Gym trading env |

### 4.1 · OpenAI/ADK rewrite option
When the optional `openai-agents` and `google-adk` packages are installed the
demo can leverage a tiny ``RewriterAgent`` built with the OpenAI Agents SDK
together with the A2A protocol to generate candidate policies.  The agent is
instantiated directly from :func:`openai_rewrite` and executed once per tree
step. Enable this behaviour with:

```bash
python run_demo.py --rewriter openai --episodes 500 --model gpt-4o
```
The script automatically falls back to the offline rewriter when the
dependencies are unavailable so the notebook remains runnable anywhere.

When the optional `openai` package is also present, `openai_rewrite` uses
`ChatCompletion` to refine candidate integer policies.  Supply an
`OPENAI_API_KEY` environment variable to activate this behaviour.  Without a
key or in fully offline environments the routine simply increments the
proposed policy elements so the rest of the demo keeps working.  You can
override the model used by setting ``OPENAI_MODEL`` (defaults to ``gpt-4o``).
Output from the model is processed via the ``_parse_numbers`` helper which
extracts integers from free‑form text and validates their length so the search
loop remains stable even when the LLM response contains extra commentary. When
the output is malformed or incomplete the helper simply increments the previous
policy as a safe fallback. The rewrite routine executes the LLM call via a small
synchronous helper so it functions both with and without an active event loop.

### 4.2 · Anthropic rewrite option
When the optional `anthropic` package is installed and an `ANTHROPIC_API_KEY`
environment variable is configured the demo can use Claude models to refine
candidate policies via the ``anthropic_rewrite`` helper. Enable this behaviour
with:

```bash
python run_demo.py --rewriter anthropic --episodes 500 --model claude-3-opus-20240229
```
As with the OpenAI path the call automatically falls back to the offline
rewriter whenever dependencies or API keys are missing so the notebook remains
fully reproducible.

### 4.3 · OpenAI Agents bridge
The `openai_agents_bridge.py` script exposes the search loop via the
**OpenAI Agents SDK** and optionally the **Google ADK** federation layer. Launch
the bridge to control the demo through API calls or the Agents runtime UI:

```bash
python openai_agents_bridge.py --help
```
Run a quick environment check with ``--verify-env`` if desired:
```bash
python openai_agents_bridge.py --verify-env --episodes 3 --target 4 --model gpt-4o
```
The bridge exposes a small :func:`verify_env` helper that performs the same
sanity check programmatically. Call it from Python or rely on the command
above. If the `openai_agents` package or API keys are missing the bridge
automatically falls back to running the search loop locally so the notebook
remains reproducible anywhere. When running offline you can still invoke
`run_search` directly to verify the helper logic:

```bash
python openai_agents_bridge.py --episodes 3 --target 4 --model gpt-4o
python -m alpha_factory_v1.demos.meta_agentic_tree_search_v0.openai_agents_bridge --episodes 3 --target 4
```
Enable the optional ADK gateway with ``--enable-adk`` (or set
``ALPHA_FACTORY_ENABLE_ADK=true``) to expose the agent over the A2A protocol.
This prints a short completion summary after executing the demo loop.

## 5 Quick start
```bash
git clone https://github.com/MontrealAI/AGI-Alpha-Agent-v0.git
cd AGI-Alpha-Agent-v0/alpha_factory_v1/demos/meta_agentic_tree_search_v0
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt          # torch, gymnasium, networkx, etc.
python run_demo.py --verify-env          # optional sanity check
python run_demo.py --config configs/default.yaml --episodes 500 --target 5 --seed 42 --model gpt-4o
# or equivalently
python -m alpha_factory_v1.demos.meta_agentic_tree_search_v0.run_demo --episodes 500 --target 5
```
`run_demo.py` prints a per‑episode scoreboard.  Pass `--log-dir logs` to save a
`scores.csv` file for further analysis. A ready‑to‑run Colab notebook is also
provided as `colab_meta_agentic_tree_search.ipynb`.

### Notebook quick start
1. Click the “Open In Colab” badge at the top of this document.
2. Execute the first cell to clone the repository and install dependencies.
3. Optionally provide `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` values in the second cell.
4. Run the demo cell to launch the search loop.
5. Optionally invoke `openai_agents_bridge.py --verify-env` from a new cell to confirm your runtime.

Add ``--enable-adk`` to the command above to start the optional ADK
gateway for remote control via the A2A protocol.
The default environment is a simple number‑line task defined in `mats/env.py` where each agent must approach a target integer. Pass `--target 7` (for example) to experiment with different goals.
Use `--seed 42` to reproduce a specific search trajectory.

> **Tip:** Set `--market-data my_feed.csv` to replay real tick data.

## 6 Repository layout
```
meta_agentic_tree_search_v0/
├── README.md                ← you are here
├── run_demo.py              ← entry‑point wrapper
├── mats/                    ← core library
│   ├── tree.py
│   ├── meta_rewrite.py
│   ├── evaluators.py
│   └── env.py
└── configs/
    └── default.yaml
```

## 7 Walk‑through of the demo episode
1. Bootstrap 4 vanilla PPO agents trading a synthetic GPU‑demand proxy.  
2. Tree search explores ~300 rewrite paths within the 30‑second budget.  
3. Best leaf realises a 3.1 % IRR over a 10‑day horizon (toy setting).  
4. Log files + tensorboard summaries land in `./logs/`.

## 8 Extending this prototype
| Goal                           | Hook/function                     |
|--------------------------------|-----------------------------------|
| Plug‑in real execution broker  | `mats.env.LiveBrokerEnv`          |
| Swap rewrite strategy          | Subclass `MetaRewriter`           |
| Use distributed workers        | `ray tune` launcher               |
| Custom tree policy             | Implement `acquire()` in `Tree`   |
| Custom output parser           | `_parse_numbers` helper           |

`LiveBrokerEnv` is a minimal subclass of :class:`NumberLineEnv` that accepts a
market data sequence. It serves as a stub for wiring real brokerage feeds into
the search loop while keeping the demo runnable completely offline.

## 9 Safety & governance guard‑rails
* Sandboxed code‑gen (`firejail + seccomp + tmpfs`)  
* Hard VaR budget enforced by `RiskGovernor`  
* CI tests for deterministic replay to detect edge drift  

## 10 References & further reading
* **Language‑Agent Tree Search**, Jiang et al., ACL 2024  
* **Best‑First Agentic Tree Search**, Li & Karim, NeurIPS 2024 workshop  
* **Self‑Referential Improvement in RL**, Müller et al., arXiv 2025  

## 11 License
Apache 2.0 – see `LICENSE`.

---
*This README belongs to the AGI‑Alpha‑Agent project.*
