# NVIDIA Quadro P620 — Installation Manual

**Windows 10 · Miniconda · PyTorch · Visual Studio Code**

Recorded: 11 September 2026  
Status: The user confirmed that the setup steps in this conversation are working.

This manual records the working configuration and the steps to reproduce it. It covers Python scripts and Jupyter notebooks. If your environment already works, use the verification and daily workflow sections; reinstalling is unnecessary.

## Contents

1. [Documented configuration](#1-documented-configuration)
2. [Check the NVIDIA driver](#2-check-the-nvidia-driver)
3. [Install Miniconda](#3-install-miniconda)
4. [Create the environment and install PyTorch](#4-create-the-environment-and-install-pytorch)
5. [Configure Visual Studio Code](#5-configure-visual-studio-code)
6. [Verify a GPU calculation](#6-verify-a-gpu-calculation)
7. [Configure Jupyter notebooks](#7-configure-jupyter-notebooks)
8. [Daily workflow](#8-daily-workflow)
9. [Troubleshooting](#9-troubleshooting)

## 1. Documented configuration

| Component | Configuration |
|---|---|
| Operating system | Windows 10, 64-bit |
| GPU | NVIDIA Quadro P620 |
| GPU architecture | Pascal; CUDA compute capability 6.1 |
| GPU memory | 2 GB |
| Environment manager | Miniconda |
| Conda environment name | `p620` |
| Python | 3.11.x |
| PyTorch | 2.7.1, CUDA 11.8 build |
| torchvision | 0.22.1 |
| torchaudio | 2.7.1 |
| VS Code extensions | Python and Jupyter, published by Microsoft |
| Notebook support | `ipykernel` installed inside `p620` |

The installer linked below is Miniconda 26.1.1-1 with Python 3.11. The VS Code, driver, and ipykernel versions were not recorded. Keep the PyTorch version and CUDA build pinned when reproducing this setup.

Hardware references: [NVIDIA previous Quadro GPUs](https://www.nvidia.com/en-us/products/workstations/previous-quadro-desktop-gpus/) and [CUDA legacy GPU capabilities](https://developer.nvidia.com/cuda/gpus/legacy). Package versions: [PyTorch 2.7.1 installation commands](https://pytorch.org/get-started/previous-versions/#v271).

> **Windows 10 compatibility:** Anaconda stopped validating new package releases on Windows 10 on 30 June 2026. Existing environments can continue working; an archived installer does not guarantee compatibility with packages released later. This manual records a user-confirmed working setup. See [Anaconda's Windows support policy](https://www.anaconda.com/blog/windows-operating-system-support-update).

### Where to run commands

- Run installation commands in **Anaconda Prompt**, opened from the Windows Start menu.
- After configuring VS Code, use its integrated terminal with `p620` active.
- Run Python code in a `.py` file or notebook cell.
- Enter commands without copying prompt prefixes such as `(p620)`.

## 2. Check the NVIDIA driver

Open Command Prompt or Anaconda Prompt and run:

```bat
nvidia-smi
```

Confirm that the output lists the Quadro P620 and an NVIDIA driver version.

If the GPU is already listed and GPU calculations work, retain the working driver. For a new installation with no working driver, download a driver that lists **Quadro P620** and **Windows 10 64-bit** as supported from [NVIDIA Driver Downloads](https://www.nvidia.com/Download/index.aspx), install it, restart Windows if requested, and repeat the check.

The prebuilt PyTorch packages below provide the CUDA runtime libraries needed for this setup. A separate CUDA Toolkit installation is needed only for additional development tasks such as compiling custom CUDA code.

## 3. Install Miniconda

Skip this section if Miniconda is already installed.

1. Download the official [Miniconda Windows 64-bit installer with Python 3.11](https://repo.anaconda.com/miniconda/Miniconda3-py311_26.1.1-1-Windows-x86_64.exe).
2. Run the downloaded `.exe`.
3. Review the license and select **I Agree** if you accept it.
4. Choose **Just Me**.
5. Select an installation folder without spaces or special characters. Use the default location if it meets that condition.
6. Set the installer options as follows.

| Installer option | Choice |
|---|---|
| Create shortcuts | Checked |
| Add Miniconda to PATH | Unchecked |
| Register as default Python | Unchecked if another Python installation is already in use |

7. Click **Install**, then **Finish**.
8. Open **Anaconda Prompt** from the Start menu.

Verify the installation:

```bat
conda --version
```

Expected result: a Conda version number.

See the [official Miniconda Windows installation guide](https://www.anaconda.com/docs/getting-started/miniconda/install/windows-gui-install).

## 4. Create the environment and install PyTorch

Run these commands in **Anaconda Prompt**.

### Create the environment once

First, check existing environments:

```bat
conda info --envs
```

If `p620` does not exist, create it:

```bat
conda create -n p620 python=3.11 pip
```

Review any repository terms presented. If you agree, accept them; type `y` when asked to proceed with package installation.

Activate the environment:

```bat
conda activate p620
python --version
```

Expected results:

- The prompt begins with `(p620)`.
- Python reports version `3.11.x`.

### Install the GPU packages

For a new environment, run:

```bat
python -m pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/cu118
python -m pip install ipykernel
```

The `cu118` index selects CUDA 11.8 packages. This is the package combination used in the working instructions. See [PyTorch's official version-specific commands](https://pytorch.org/get-started/previous-versions/#v271).

Using `python -m pip` installs packages through the Python interpreter that is currently active.

Check the interpreter and installed build:

```bat
python -c "import sys; print(sys.executable)"
python -c "import torch; print('PyTorch:', torch.__version__); print('CUDA runtime:', torch.version.cuda)"
```

The interpreter path should point to `p620`. Expected build information:

```text
PyTorch: 2.7.1+cu118
CUDA runtime: 11.8
```

## 5. Configure Visual Studio Code

Install [Visual Studio Code](https://code.visualstudio.com/download) if it is not already installed.

### Install the extensions

Press **Ctrl + Shift + X** and install these Microsoft extensions:

| Extension | Extension ID |
|---|---|
| Python | `ms-python.python` |
| Jupyter | `ms-toolsai.jupyter` |

### Open the project and select Python

1. Use **File → Open Folder** to open your project directory.
2. Press **Ctrl + Shift + P**.
3. Run **Python: Select Interpreter**.
4. Choose the entry containing **p620** and **Python 3.11**.

The selected interpreter controls Python execution and debugging. See the [VS Code environment guide](https://code.visualstudio.com/docs/python/environments).

If `p620` is absent, run this in Anaconda Prompt:

```bat
conda activate p620
python -c "import sys; print(sys.executable)"
```

Copy the path. Choose **Enter interpreter path** in VS Code's interpreter selector and paste it.

### Verify the integrated terminal

Close old terminals, then use **Terminal → New Terminal**. VS Code normally activates the selected environment automatically.

Run:

```bat
python -c "import sys; print(sys.executable)"
```

Confirm that the path matches the one printed from Anaconda Prompt.

## 6. Verify a GPU calculation

Create `check_gpu.py` inside the project folder and paste the following code.

```python
import sys
import torch

print("Python:", sys.executable)
print("PyTorch:", torch.__version__)
print("CUDA available:", torch.cuda.is_available())

if torch.cuda.is_available():
    print("GPU:", torch.cuda.get_device_name(0))

    # Allocate a tensor on the GPU and perform matrix multiplication.
    x = torch.randn(256, 256, device="cuda")
    result = x @ x
    print("Result device:", result.device)

    # Reading the result waits for the calculation to finish.
    print("Result sum:", result.sum().item())
```

Save the file. Select **Run Python File in Terminal** at the top right of the editor. Alternatively, use the activated project terminal:

```bat
python check_gpu.py
```

A successful run should report:

```text
Python: <path to the p620 environment's python.exe>
PyTorch: 2.7.1+cu118
CUDA available: True
GPU: Quadro P620
Result device: cuda:0
Result sum: <a numeric value>
```

The precise GPU name may include an NVIDIA prefix. The sum varies because the input is random.

**Success requires the calculation to complete without an error.** A `True` result from `torch.cuda.is_available()` alone does not prove that the installed build can execute kernels on this GPU.

See [running Python files in VS Code](https://code.visualstudio.com/docs/python/run).

## 7. Configure Jupyter notebooks

A notebook has its own kernel selection. Set it explicitly even when the workspace interpreter is already `p620`.

1. Open an existing `.ipynb` file, or create `gpu_test.ipynb`.
2. Click the kernel selector at the top right.
3. Choose **Select Another Kernel → Python Environments → p620**, if those intermediate choices appear.
4. Paste the verification code from Section 6 into a cell.
5. Press **Shift + Enter**.

Confirm that the Python path, PyTorch build, GPU name, and completed calculation match the script results.

If you install or change packages while a notebook is open, **restart its kernel** before testing again. See the [VS Code notebook guide](https://code.visualstudio.com/docs/datascience/jupyter-notebooks).

### Optional: register a named kernel

If environment discovery is still unsuccessful, run this in Anaconda Prompt:

```bat
conda activate p620
python -m ipykernel install --user --name p620 --display-name "Python (Quadro P620)"
```

Reload VS Code. In the notebook's kernel picker, look under the available Jupyter kernels for **Python (Quadro P620)**.

## 8. Daily workflow

### Python scripts

Open your project, select `p620`, and verify the terminal interpreter when needed. Run scripts using **Run Python File in Terminal**.

From Anaconda Prompt, activate the environment before running project commands:

```bat
conda activate p620
```

Change to the folder containing your script, then run:

```bat
python check_gpu.py
```

### Notebooks

Open the notebook, choose the `p620` kernel, and run cells with **Shift + Enter**.

### Package installation

Activate `p620` before installing project dependencies:

```bat
conda activate p620
python -m pip install <package-name>
```

Replace `<package-name>` with the required package. Review dependency requirements before upgrading the pinned PyTorch packages.

Selecting this environment makes the GPU-enabled libraries available. Your training code still needs to move both the model and its input tensors to the CUDA device, as the verification script does for its tensor.

## 9. Troubleshooting

| Symptom | What to check or do |
|---|---|
| `conda` is not recognized | Open Anaconda Prompt from the Start menu and run the command there. |
| `EnvironmentNameNotFound: p620` | Run `conda info --envs`; create the environment using Section 4 if it is absent. |
| VS Code shows Python 3.13 or another environment | Select the Python 3.11 interpreter belonging to `p620`; confirm `sys.executable`. |
| `ModuleNotFoundError: No module named 'torch'` | Check the interpreter first, then install PyTorch in `p620` using Section 4. |
| Script works but notebook fails | Select the `p620` notebook kernel, restart it, and compare `sys.executable` in both places. |
| `torch.cuda.is_available()` is `False` | Check `nvidia-smi`, the selected interpreter, `torch.__version__`, and `torch.version.cuda`. |
| CUDA runtime prints `None` | The selected interpreter has a CPU-only PyTorch build. Use the targeted repair below. |
| GPU architecture warning or “no kernel image is available” | Confirm the CUDA 11.8 PyTorch build. A different build may lack support for the P620. |
| `CUDA out of memory` | Reduce batch size, image resolution, video frames, or model size. The P620 has only 2 GB of GPU memory. |
| Packages were installed but a notebook cannot import them | Confirm the environment and restart the notebook kernel. |

### Terminal activation problems

If the VS Code terminal uses the wrong Python, close all VS Code windows. Open Anaconda Prompt, activate `p620`, change to your project directory, then launch VS Code:

```bat
conda activate p620
code .
```

If `code` is not recognized, open VS Code normally and use the explicit interpreter path from Section 5. You can continue running installation commands in Anaconda Prompt.

### Targeted repair for the wrong PyTorch build

Use this only when diagnostics show a CPU-only or incompatible build. A working installation does not need repair.

Close running notebook kernels and Python processes using the environment. In Anaconda Prompt, run:

```bat
conda activate p620
python -m pip install --upgrade torch==2.7.1+cu118 torchvision==0.22.1+cu118 torchaudio==2.7.1+cu118 --index-url https://download.pytorch.org/whl/cu118
```

The explicit `+cu118` suffix requires the intended CUDA build. Restart the notebook kernel or Python process, then repeat the calculation in Section 6.

### Diagnostic commands

Run these in Anaconda Prompt when investigating an issue:

```bat
conda activate p620
python --version
python -c "import sys; print(sys.executable)"
python -c "import torch; print('PyTorch:', torch.__version__); print('CUDA runtime:', torch.version.cuda); print('CUDA available:', torch.cuda.is_available()); print('Compiled architectures:', torch.cuda.get_arch_list())"
nvidia-smi
```

Record the full error message together with these outputs. They distinguish an interpreter selection problem from a package or driver problem.

