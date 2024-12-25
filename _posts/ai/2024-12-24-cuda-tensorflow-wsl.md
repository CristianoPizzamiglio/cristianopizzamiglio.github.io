---
layout: post
title: "Installing TensorFlow, CUDA and cuDNN in Windows Subsystem for Linux"
date: 2024-12-24 00:00:00 -0400
---

As reported in the [TensorFlow documentation](https://www.tensorflow.org/install/pip#windows-native), TensorFlow 2.10 is the last release supporting GPU on Windows native. To leverage GPUs while working on a Windows machine, newer TensorFlow releases must be installed in the Windows Subsystem for Linux (WSL). In this post I would like to provide a clear and concise guide on how to install TensorFlow, CUDA and cuDNN on WSL. Moreover, we will see how to setup a Python interpreter, available through WSL, in the PyCharm IDE.

Before starting with the step-by-step instructions, let's briefly review what WSL, CUDA and cuDNN are. If you are already familiar with these tools, feel free to skip this section.

* [WSL](https://learn.microsoft.com/en-us/windows/wsl/about): a tool that allows you to run a Linux environment directly from a Windows machine, without the need for partitioning the hard drive and running a dual boot system. I have already talked about WSL in a [previous post](https://cristianopizzamiglio.com/2022/08/14/carbon-wsl.html).

* [CUDA](https://blogs.nvidia.com/blog/what-is-cuda-2/): the NVIDIA *Compute Unified Architecture* is a parallel computing platform and programming model that helps developers speed up their applications by harnessing the power of GPU accelerators.

* [cuDNN](https://docs.nvidia.com/cudnn/index.html#:~:text=The%20NVIDIA%20CUDA%C2%AE%20Deep,primitives%20for%20deep%20neural%20networks.): NVIDIA *CUDA Deep Neural Network* is a GPU-accelerated library of primitives for deep neural networks. cuDNN provides highly tuned implementations for standard routines such as forward and backward convolution, attention, matmul, pooling, and normalization.

In this tutorial I will install Ubuntu 22.04 and TensorFlow 2.13. According to the [TensorFlow GPU compatibility matrix](https://www.tensorflow.org/install/source#gpu), CUDA 11.8, cuDNN 8.6 and Python 3.9-3.11 are compatible with TensorFlow 2.13. The steps required to install TensorFlow, CUDA and cuDNN are reported below.

#### 1. Install GPU Driver

Download and install the NVIDIA GPU driver on your Windows machine. You can download the driver from the [NVIDIA website](https://www.nvidia.com/en-us/drivers/). To check the GPU status, open PowerShell and enter the following command:

```powershell
nvidia-smi
```
If the GPU is properly installed, it should appear in the output.

A few notes on the command above. As stated [here](https://developer.nvidia.com/system-management-interface):
> The NVIDIA System Management Interface (nvidia-smi) is a command line utility, based on top of the NVIDIA Management Library (NVML), intended to aid in the management and monitoring of NVIDIA GPU devices. This utility allows administrators to query GPU device state and with the appropriate privileges, permits administrators to modify GPU device state.  It is targeted at the TeslaTM, GRIDTM, QuadroTM and Titan X product, though limited support is also available on other NVIDIA GPUs.


#### 2. Install WSL

Install WSL by running the PowerShell as an administrator. By default, the latest Ubuntu distribution will be installed, i.e. Ubuntu 24.04. If that works for you, simply type

```powershell
wsl --install
```
otherwise, you need to specify the desired version of the Linux distribution

```powershell
wsl --install -d Ubuntu-22.04
```
This step could require several minutes but it’s quite straightforward. Afterwards, you will be asked to create a username and password.


#### 3. A note on Python and WSL

Python is installed by default, as you can check by typing

```bash
python3 --version
```

On Ubuntu 22.04 the default Python version is 3.10. To install a different version, type
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.9
```
However, if you check again the Python version, the system will return the default Python version, i.e. 3.10. To make 3.9 the default version type
```bash
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
```
The command
```bash
sudo update-alternatives --config python3
```
will output a table listing the Python versions available and their priority level. If the following error message appears later on
```bash
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 https://ppa.launchpadcontent.net/deadsnakes/ppa/ubuntu noble InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Traceback (most recent call last):
File "/usr/lib/cnf-update-db", line 3, in <module>
    import apt_pkg
ModuleNotFoundError: No module named 'apt_pkg'
Reading package lists... Done
E: Problem executing scripts APT::Update::Post-Invoke-Success 'if /usr/bin/test -w /var/lib/command-not-found/ -a -e /usr/lib/cnf-update-db; then /usr/lib/cnf-update-db > /dev/null; fi'
E: Sub-process returned an error code
```
then run the commands below to fix the issue

```bash
sudo apt remove python3-apt
sudo apt autoremove
sudo apt autoclean
sudo apt install python3-apt
```
<p style="margin-bottom: 10px;"></p>

#### 4. Install `pip`

`pip` is not installed by default. To install it, enter the commands

```bash
sudo apt update
sudo apt install python3-pip
```
To check whether `pip` is properly installed, just type
```bash
pip --version
```
<p style="margin-bottom: 10px;"></p>


#### 5. Create a Python Virtual Environment

Let's create a Python virtual environment. First, you need to install the `venv` module by entering

```bash
sudo apt install python3.10-venv
```
To keep things tidy, I suggest to create a `venvs` folder to store your virtual environments. To create the folder, simply type
```bash
mkdir venvs
cd venvs
```
The virtual environment `my-venv` is created by typing the command
```bash
python3 -m venv my-venv
```
and to activate it
```bash
cd my-venv/bin/
source activate
```
<p style="margin-bottom: 10px;"></p>


#### 6. Install TensorFlow

To install TensorFlow 2.13 enter

```bash
pip install tensorflow==2.13
```
If you have a `requirements.txt` file, run the following command by specifying its path
```bash
pip install -r /mnt/c/Users/Username/Documents/requirements.txt
```
In the code above, the file is located into the `C:` folder in the Windows machine, where `/mnt/c/` is the mount point for the `C:` drive on WSL.


#### 7. Install CUDA

To install CUDA on WSL, type the commands

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda-repo-ubuntu2204-11-8-local_11.8.0-520.61.05-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-11-8-local_11.8.0-520.61.05-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-11-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-11-8
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
source ~/.bashrc
```
Run the following to check whether CUDA is properly installed
```bash
nvcc --version
```
<p style="margin-bottom: 10px;"></p>


#### 8. Install cuDNN

The commands below allows to install cuDNN

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt-get install libcudnn8=8.6.0*
sudo apt-get install libcudnn8-dev=8.6.0*
```
If the installation is successful, the following command will show the cuDNN version.
```bash
dpkg -l | grep cudnn
```
<p style="margin-bottom: 10px;"></p>


#### 9. Check TensorFlow GPU Detection

Start a Python session and write

```python
import tensorflow as tf
print(tf.config.list_physical_devices('GPU'))
```

If TensorFlow has access to the GPU, then the output of the `print` statement should look similar to the following

```python
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

On the contrary, an empty list will be printed.


#### 10. Setup a WSL Python Interpreter in PyCharm

To leverage a Python interpreter available through WSL, PyCharm Professional Edition is required (a 30-day trail version can be downloaded). Go to `Settings | Project | Python Interpreter`, click the `Add Interpreter` link, select `On WSL`, and wait until introspection is completed. Read [here](https://www.jetbrains.com/help/pycharm/using-wsl-as-a-remote-interpreter.html#configure-wsl) for further details.


Now you are ready to develop and train your neural networks in PyCharm using TensorFlow, and harness the power of your NVIDIA GPU directly from your Windows machine thanks to WSL.
