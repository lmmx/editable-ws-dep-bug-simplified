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
(uv-ws-edit-dep-repro) louis 🚶 ~/lab/uv/editable-ws-dep-bug-simplified $ vim
packages/demo_cli/src/demo_cli/main.py 
(uv-ws-edit-dep-repro) louis 🚶 ~/lab/uv/editable-ws-dep-bug-simplified $ cat
packages/demo_cli/src/demo_cli/main.py 
def run_cli() -> None:
    assert True
(uv-ws-edit-dep-repro) louis 🚶 ~/lab/uv/editable-ws-dep-bug-simplified $ repro-demo-entrypoint 
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
