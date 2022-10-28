---
layout: post
title: "Installing Carbon on Windows using the Windows Subsystem for Linux"
date: 2022-08-14 00:00:00 -0400
---

The [Carbon programming language](https://github.com/carbon-language/carbon-lang) is an experimental successor to C++.

Google engineer [Chandler Carruth](https://www.linkedin.com/in/chandlerc/) first introduced Carbon at the [CppNorth conference](https://www.youtube.com/watch?v=omrY53kbVoA&ab_channel=CppNorth) in Toronto in July 2022.

As stated in Carbon's GitHub repo, the aim of the developers is to design a language to support:

>
* Performance-critical software
* Software and language evolution
* Code that is easy to read, understand, and write
* Practical safety and testing mechanisms
* Fast and scalable development
* Modern OS platforms, hardware architectures, and environments
* Interoperability with and migration from existing C++ code

The main reason behind the development of a replacement for C++ is that evolving and improving the latter is getting more and more challenging due to the piling up of technical debt.

Languages like [Go](https://go.dev/), [Kotlin](https://kotlinlang.org/) and especially [Rust](https://www.rust-lang.org/) already offer, for the right use case, a valuable alternative to C++, but none of the them allows a smooth migration from a C++ codebase.

I was eager to play with it, unfortunately the language is not natively supported on Windows yet, so the instructions reported on GitHub are not exhaustive. I did a little bit of research and eventually I managed to properly install it by leveraging the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) or WSL for short. All the steps required for the installation are listed below. I would like to thank the authors of this [article](https://tipseason.com/carbon-language-tutorial-syntax/) and this [video](https://www.youtube.com/watch?v=AaZUJUbXrQ8&ab_channel=RyanGaudion) for the valuable information they shared.

If you don't need to install Carbon locally, you can just use the [Compiler Explorer](https://carbon.compiler-explorer.com/) web app.


#### 1. Install WSL

The Windows Subsystem for Linux is a terrific tool that allows you to run a Linux environment directly from a Windows machine, so no need for partitioning the hard drive and running a dual boot system.

To install WSL, you need Windows 10 version 2004 or higher, or Windows 11.

Run the PowerShell as administrator and type
```powershell
wsl --install
```
to setup WSL and to install the [Ubuntu](https://en.wikipedia.org/wiki/Ubuntu) distribution of Linux. You can change the default distribution, but Ubuntu is always a good choice especially if you are just getting started with Linux.

This step could require several minutes but it's quite straightforward. You will have to reboot your machine and then Ubuntu will be set up for you and you will be asked to create a username and password.

Whenever you need to run Ubuntu, just type
```powershell
wsl
```
and an Ubuntu terminal session will be up and running.

If you are logged in as root, switch to a normal user to be able to perform the upcoming steps
```powershell
su -l your_username
```
You should see something like that from the terminal:

<img src="/img/posts/2022-08-14-carbon-wsl/ps-wsl-user.jpg" width="710"/>


#### 2. Install Homebrew

[Homebrew](https://en.wikipedia.org/wiki/Homebrew_(package_manager)) is a free and open-source software package management system that simplifies the installation of software on Apple's operating system, macOS and Linux.

Go to its [webpage](https://brew.sh/) where you will find the following command to copy
```powershell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
then paste it on your terminal and enter your password when requested.

Then, add Homebrew to your `PATH` variable by entering the two commands below (change username in the first command):
```powershell
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/cristiano/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

Install `build-essential`:
```powershell
sudo apt-get install build-essential
```

Finally, install [GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection) by entering
```powershell
brew install gcc
```


#### 3. Build the Carbon explorer

At this point we can just go back to the Carbon's GitHub repo and follow the instructions listed in the getting started section.

Install `bazelisk`
```powershell
brew install bazelisk
```

Then, install [LLVM](https://llvm.org/) which is used to write the Carbon compiler
```powershell
brew install llvm
```
and add it to your `PATH` variable
```powershell
export PATH="$(brew --prefix llvm)/bin:${PATH}"
```

Finally, we can download Carbon's code from GitHub by typing
```powershell
git clone https://github.com/carbon-language/carbon-lang
```
and change directory
```powershell
cd carbon-lang
```

To run the *Hello, World!* program, type the following command
```powershell
bazel run //explorer -- ./explorer/testdata/print/format_only.carbon
```

As stated in the article mentioned earlier, the meaning of this line is to

> invoke Bazel build tool to trigger explorer code that in turns runs the actual code present in the `./explorer/testdata/print/format_only.carbon` file.

At this point, you should see the output displayed on the terminal:

<img src="/img/posts/2022-08-14-carbon-wsl/carbon-hello-world.jpg" width="710"/>


#### 4. Programming in Carbon using VS Code

Another cool feature of the WSL is that you can easily [connect VS Code](https://code.visualstudio.com/docs/remote/wsl) to it.

Open VS Code and install the *Remote Development* pack from the marketplace.

<p style="text-align:center;"><img src="/img/posts/2022-08-14-carbon-wsl/remote-dev.jpg" width="360"/></p>

Now the *Remote Explorer* tab is available:

1. click on it,

1. select *WSL Targets* from the drop-down menu,

1. click the *Connect to WSL* button (as shown in the image below by the red circle) and a new VS Code window will open.

    <p style="text-align:center;"><img src="/img/posts/2022-08-14-carbon-wsl/remote-tab.png" width="360"/></p>

1. From the new VS Code window, open the `/home/your_username/carbon-lang/` folder.

    <img src="/img/posts/2022-08-14-carbon-wsl/open-folder.jpg" width="700"/>

1. From the *Explorer* tab open the `/explorer/testdata/print/format_only.carbon` file to edit the *Hello, World!* program. For instance, I have changed the string to `"Hello Cristiano!"`

1. To run the code, type again the command `bazel run //explorer -- ./explorer/testdata/print/format_only.carbon` in the VS Code terminal.

    <img src="/img/posts/2022-08-14-carbon-wsl/run-vs-code.jpg" width="700"/>

Now you are ready to experiment with the Carbon language using VS Code directly from your Windows machine thanks to WSL.







