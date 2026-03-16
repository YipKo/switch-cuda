switch-cuda
===========

<details>
<summary><strong>🤖For LLM Agents: </strong></summary>
Fetch the installation guide and follow it:

**Step 1: Ask user parameters**

- `SCRIPT_PATH` (default: `~/switch-cuda.sh`)
- Conda environment name (`ENV_NAME`)
- CUDA version (`CUDA_VERSION`), selected from detected installed versions
- Whether to overwrite existing hook files

**Step 2: Validate and detect**

```bash
test -f "$SCRIPT_PATH" || wget -O "$SCRIPT_PATH" https://raw.githubusercontent.com/YipKo/switch-cuda/refs/heads/master/switch-cuda.sh
source "$SCRIPT_PATH"          # list installed CUDA versions
conda info --envs
test -n "$CONDA_PREFIX" || echo "please activate target env first"
```

**Step 3: Create hook directories**

```bash
mkdir -p "$CONDA_PREFIX/etc/conda/activate.d"
mkdir -p "$CONDA_PREFIX/etc/conda/deactivate.d"
```

**Step 4: Write activate hook**

```bash
cat > "$CONDA_PREFIX/etc/conda/activate.d/set-cuda.sh" <<EOF
#!/bin/bash
source "$SCRIPT_PATH" "$CUDA_VERSION"
EOF
chmod +x "$CONDA_PREFIX/etc/conda/activate.d/set-cuda.sh"
```

**Step 5: Write deactivate hook**

```bash
cat > "$CONDA_PREFIX/etc/conda/deactivate.d/reset-cuda.sh" <<EOF
#!/bin/bash
source "$SCRIPT_PATH" default
EOF
chmod +x "$CONDA_PREFIX/etc/conda/deactivate.d/reset-cuda.sh"
```

**Step 6: Verify**

```bash
conda deactivate && conda activate "$ENV_NAME"
echo "$CUDA_HOME"
which nvcc
```

</details>

Docs
----

- Chinese tutorial: https://yipko.com/posts/work/cuda-switcher-with-conda/
- English tutorial: https://yipko.com/en/posts/work/cuda-switcher-with-conda/

Sometimes you need to switch to an older CUDA version to run legacy code on a machine configured with a newer toolkit.

This can be done by changing a few environment variables, but doing it manually is tedious. This repository provides [`switch-cuda.sh`](switch-cuda.sh), which switches CUDA **for the current shell session**.

**Notice:** [`switch-cuda.sh`](switch-cuda.sh) was originally tested on Ubuntu and is typically easy to adapt for other Linux distributions.


Install
-------

Check whether the script already exists locally. Download only if missing:

```bash
test -f ~/switch-cuda.sh && echo "switch-cuda.sh already exists, skip download" || wget -O ~/switch-cuda.sh https://raw.githubusercontent.com/YipKo/switch-cuda/refs/heads/master/switch-cuda.sh
```


Usage
-----

```bash
source switch-cuda.sh [VERSION]
```

You must use `source` (not execute directly), because the script modifies environment variables that should persist in the current shell.

- If a version is provided (e.g. `11.8`), it updates `PATH`, `LD_LIBRARY_PATH`, `CUDA_HOME`, and `CUDA_ROOT`.
- If `default` is provided, it clears these variables and restores system default CUDA (from `/usr/local/cuda` symlink).
- If no version is provided, it lists installed CUDA versions found under `/usr/local`.


Examples
--------

```bash
$ source switch-cuda.sh
The following CUDA installations have been found (in '/usr/local'):
* cuda-11.5
* cuda-11.8
* cuda-12.6
```

```bash
$ source switch-cuda.sh 11.8
Switched to CUDA 11.8.
```

```bash
$ source switch-cuda.sh default
CUDA environment has been reset to default.
```


Conda Integration
-----------------

You can auto-switch CUDA when activating/deactivating a conda environment by adding hooks in `etc/conda/activate.d` and `etc/conda/deactivate.d`.

**Example assumptions:**

- Conda env: `YOUR_ENV_NAME`
- Script path: `~/switch-cuda.sh`
- Desired CUDA for this env: `12.1`

Create hook directories:

```bash
conda activate YOUR_ENV_NAME
mkdir -p "$CONDA_PREFIX/etc/conda/activate.d"
mkdir -p "$CONDA_PREFIX/etc/conda/deactivate.d"
```

`$CONDA_PREFIX/etc/conda/activate.d/set-cuda.sh`

```bash
#!/bin/bash
source ~/switch-cuda.sh 12.1
```

`$CONDA_PREFIX/etc/conda/deactivate.d/reset-cuda.sh`

```bash
#!/bin/bash
source ~/switch-cuda.sh default
```

Now:

```bash
conda activate YOUR_ENV_NAME
```

switches to CUDA `12.1`, and:

```bash
conda deactivate
```

restores system default CUDA.
