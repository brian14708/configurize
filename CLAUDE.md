# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Configurize is a Python library for hierarchical configuration management. It provides a `Config` class that extends `DataClass` with features like parent-child relationships, cross-references between config nodes, diffing, and validation.

## Commands

```bash
# Install dependencies
uv sync

# Install package (enables cfshow CLI)
pip install .

# Lint/format code (uses black + isort)
python tools/linter.py [files...]

# Show a config or compare two configs
cfshow <config_file.py> [other_config.py] [--key=subconfig.path] [--query=attr_name]
```

## Architecture

### Core Classes (configurize/)

- **DataClass** (`data_class.py`): Base class providing dict-like behavior with inheritance support. Attributes defined as class variables become instance attributes on construction.

- **Config** (`config.py`): Extends DataClass with:
  - Tree structure: configs can contain sub-configs, accessible via `father()` and `root()` methods
  - Reference resolution: `Ref` objects are automatically dereferenced when accessed
  - Merging: `config.merge(dict_or_config)` applies updates, supporting dot-notation keys like `"sub.attr"`
  - Diffing: `config.diff(other)` returns a `ConfigDiff` showing attribute differences
  - Context modification: `with config.modify(attr=value):` for temporary changes
  - Attribute tracing: set `_tracing_set_attribute = True` to track all attribute modifications

- **Ref** (`reference.py`): Creates references to other config values using path syntax:
  - `.attr` - reference to self.attr
  - `..attr` - reference to parent.attr
  - `...sub.attr` - reference to grandparent.sub.attr

- **ConfigDiff** (`config.py`): Represents differences between two configs with colored repr output. Access original values via `_A()` and new values via `_B()`.

### Type Handling (allowed_types.py)

`ALLOWED_TYPES` defines serializable types (base types, sequences, mappings, numpy/torch types). `recur_to_allowed_types()` converts objects to allowed types for serialization.

### Mock Imports (mock_imports.py)

`mock_imports()` context manager allows loading config files that import unavailable dependencies (torch, etc.) by mocking missing modules. Used by `tools/show.py`.

### Key Config Attributes

- `critical_keys`: List of attribute names for consistency validation. Suffix `?` makes the check optional.
- `_allow_set_new_attr`: When False, prevents setting undefined attributes (typo protection).
- `_allow_search`: When True, attribute lookup searches the entire config tree.
- `_is_tree_node`: Controls whether node appears in tree search results.
