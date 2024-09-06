# 

Minimal repro demo of a bug where `uv pip install -e .` on the workspace root does not produce
editable dependencies.

This repro repo has a virtual workspace root named `ws_editable_dep_repro` and one package:

```
packages/
└── demo_cli
    ├── pyproject.toml
    ├── README.md
    └── src
        └── demo_cli
            ├── __init__.py
            └── main.py
```

- This package named `demo_cli` exposes an entrypoint in its `pyproject.toml` named `repro-demo-entrypoint`
- The `repro-demo-entrypoint` entrypoint points to the `demo_cli.main:run_cli` function.
- The `run_cli` function source code is just an `assert False`.
- If you run the CLI, it throws the `AssertionError`
- If you go and edit the CLI to remove the bug, and re-run the entrypoint, it still throws, the dep
  was not edited to reflect your changes:

```
editable-ws-dep-bug-simplified $ cat packages/demo_cli/src/demo_cli/main.py 
def run_cli() -> None:
    assert True
editable-ws-dep-bug-simplified $ repro-demo-entrypoint 
Traceback (most recent call last):
  File "/home/louis/miniconda3/envs/uv-ws-edit-dep-repro/bin/repro-demo-entrypoint", line 8, in
<module>
    sys.exit(run_cli())
             ^^^^^^^^^
  File
"/home/louis/miniconda3/envs/uv-ws-edit-dep-repro/lib/python3.12/site-packages/demo_cli/main.py",
line 2, in run_cli
    assert False
           ^^^^^
AssertionError
```

## Environment

This was done in a conda env:

```sh
conda create -n uv-ws-edit-dep-repro python=3.12 -y
conda activate uv-ws-edit-dep-repro
```

## Workaround

Going into the indidual package and running `uv pip install -e .` works

```
editable-ws-dep-bug-simplified/packages/demo_cli $ uv pip install -e .
Resolved 1 package in 2ms
   Built demo-cli @ file:///home/louis/lab/uv/editable-ws-dep-bug-simplified/packages/demo_cli
Prepared 1 package in 659ms
Uninstalled 1 package in 0.43ms
Installed 1 package in 1ms
 ~ demo-cli==0.1.0 (from file:///home/louis/lab/uv/editable-ws-dep-bug-simplified/packages/demo_cli)
editable-ws-dep-bug-simplified/packages/demo_cli $ gcd # gcd is aliased to `cd $(git rev-parse --show-toplevel)'
editable-ws-dep-bug-simplified $ repro-demo-entrypoint 
editable-ws-dep-bug-simplified $ 
```

(The reinstall corrects the package source)
