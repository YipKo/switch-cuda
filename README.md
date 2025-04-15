switch-cuda
===========

Sometimes, it becomes necessary to switch to an earlier version of CUDA in order to run older code on a machine that is
actually set up to use the current version of the CUDA toolkit.

This is as simple as adjusting the values of a few environment variables, yet it is cumbersome to do manually.
Therefore, this repository provides the bash script [`switch-cuda.sh`](switch-cuda.sh), which adjusts the environment to
use a specific version of CUDA **for the current bash session**.

**Notice:** [`switch-cuda.sh`](switch-cuda.sh) has been written for and tested on Ubuntu 16.04, but is easily adapted
for other platforms.


Usage
-----

```
source switch-cuda.sh [VERSION]
```

Notice that the script has to be sourced rather than executed, as it performs changes of environment variables that are
supposed to persist after the script has finished. 

If a version number is provided, than all relevant environment variables are adjusted to the required CUDA version
(including `PATH`, `LD_LIBRARY_PATH`, `CUDA_HOME`, and `CUDA_ROOT`). 

If the special argument `default` is provided, the script clears these environment variables and restores the system’s
default CUDA setup (as determined by the `/usr/local/cuda` symbolic link). 

If no version is provided, however, then the script simply prints all versions of CUDA that have been found on the used
machine (in `/usr/local`). 


Examples
--------

```
$ source switch-cuda.sh 
The following CUDA installations have been found (in '/usr/local'):
* cuda-8.0
* cuda-9.0
* cuda-9.1
```

```
$ source switch-cuda.sh 9.0
Switched to CUDA 9.0.
```

```
$ source switch-cuda.sh default
CUDA environment has been reset to default.
```

Conda Integration
--------
You can automatically switch to a specific CUDA version when activating a conda environment by adding a hook inside the environment’s `etc/conda/activate.d` and `etc/conda/deactivate.d` directories.

**Example:**

Assuming your environment is located at `~/miniconda3/envs/YOUR_ENV_NAME` and your "switch-cuda.sh" is located at `~/switch-cuda.sh`. The CUDA version you want to use in your environment is `12.1`.

Create the following two scripts:
```
~/miniconda3/envs/YOUR_ENV_NAME/etc/conda/activate.d/set-cuda.sh
~/miniconda3/envs/YOUR_ENV_NAME/etc/conda/deactivate.d/reset-cuda.sh
```

**activate.d/set-cuda.sh**
```
#!/bin/bash
source ~/Projects/switch-cuda.sh 12.1
```
**activate.d/reset-cuda.sh**
```
#!/bin/bash
source ~/Projects/switch-cuda.sh default
```

**Usage**

Now, whenever you activate `YOUR_ENV_NAME`:
```
conda activate YOUR_ENV_NAME
```
It will automatically switch to `12.1`.

And when you deactivate:
```
conda deactivate
```
It will reset back to the system default CUDA setup.


