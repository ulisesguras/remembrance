# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run all tests
python tests/test_memory.py

# Run tests with pytest (verbose)
python -m pytest tests/ -v

# Run a specific use case
python cases/health_assistant.py
python cases/customer_support.py
python cases/energy_optimizer.py
python cases/robotics_coordinator.py
python cases/spatial_intelligence.py
python cases/cognitive_agent.py
```

Install (editable, includes dev extras):

```bash
pip install -e ".[dev]"
```

No external dependencies in core — pure Python.

## Architecture

**remembrance** is a pip-installable memory-layer framework for AI agents, not an LLM wrapper. The nine memory modules in `remembrance/memory/` are independent, composable primitives. `BaseAgent` in `remembrance/agent/__init__.py` wires all nine together into one ready-to-subclass class. The top-level `remembrance/__init__.py` re-exports `BaseAgent` and `AgentConfig` for convenience.

```
remembrance/          ← installable package root
├── __init__.py       ← re-exports BaseAgent, AgentConfig
├── agent/
│   └── __init__.py   ← BaseAgent, AgentConfig
└── memory/
    └── *.py          ← nine independent memory modules
```

### Memory taxonomy

| Module | Dimension | What it holds |
|--------|-----------|---------------|
| `sensory.py` | By Duration | Raw inputs with TTL-based decay; `perceive()` + `flush()` |
| `working.py` | By Duration | Active scratchpad, capacity-limited, priority-evicted |
| `episodic.py` | By Duration | Append-only event log; searchable by keyword or tag |
| `semantic.py` | By Duration | Key-value fact store; pluggable vector backend via `use_vector_backend()` |
| `procedural.py` | By Function | Named skills registry with optional handler callables |
| `strategic.py` | By Function | Self-reflection log + strategy registry; tracks `systematic_biases()` |
| `emotional.py` | By Scope | Per-subject valence/arousal affect traces; `high_friction_subjects()` |
| `prospective.py` | By Scope | Future intentions; `time` and `event` trigger types |
| `collective.py` | By Scope | Shared swarm pool; confidence-weighted `consensus()` API |

### BaseAgent

`BaseAgent` (`remembrance/agent/__init__.py`) exposes high-level convenience methods over the nine layers:

- `perceive(modality, content)` → sensory
- `learn(key, value, tags)` → semantic **and** collective (simultaneously)
- `remember_episode(...)` → episodic
- `reflect(decision, strategy, outcome, lesson, ...)` → strategic
- `intend(description, trigger_type, trigger_value, action_description)` → prospective
- `memory_summary()` → dict with counts across all layers

Override `_setup()`, `think()`, and `act()` to specialize an agent.

### Extending the framework

- **New use case**: add a file under `cases/`; must exercise at least 4 of the 9 layers.
- **New memory module**: add under `remembrance/memory/`, add tests in `tests/`, keep the module under ~150 lines.
- **Vector backend for SemanticMemory**: pass `embed_fn` and a DB client to `semantic.use_vector_backend()`.
- **Distributed CollectiveMemory**: pass a Redis/DB client as the `backend` arg to `CollectiveMemory()`.
- **No external dependencies** may live in `remembrance/memory/` unless hidden behind a backend interface.

### Test structure

All 29 tests are in a single file `tests/test_memory.py` using `unittest`. Each memory class has its own `TestXxxMemory` suite; `TestBaseAgent` covers the integration path. `TestStrategicMemory` appears after the `if __name__ == "__main__"` block but is still collected by pytest.

## Project identity
- Founded: March 2, 2026
- Author: Mario Ulises Guras (ulguras@gmail.com)
- Stage: Open source, seeking accelerator partners
- MCP registered: mcp.so

## Design philosophy
This is infrastructure, not an application. Modules must remain backend-agnostic 
and dependency-free in core. Vector/Redis backends are always opt-in via interface injection.

## Conventions
- PRs must include tests; coverage per new module required
- Cases must exercise ≥4 of 9 layers (enforced by convention, not CI)
- Public API surface lives in `remembrance/__init__.py` (re-exports) and `remembrance/agent/__init__.py`; memory modules are internal primitives