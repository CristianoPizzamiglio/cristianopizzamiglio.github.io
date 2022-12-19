---
layout: post
title: "Installing the Polygon Mesh Processing Library in Ubuntu"
date: 2022-12-10 00:00:00 -0400
---

The [Polygon Mesh Processing Library](https://www.pmp-library.org/) (PMP) is a modern cross-platform C++ open-source library for processing and visualizing polygon surface meshes. 

As stated in [PMP's GitHub repo](https://github.com/pmp-library/pmp-library), its main features are:

>
* An efficient and easy-to-use mesh data structure
* Standard algorithms such as decimation, remeshing, subdivision, or smoothing
* Ready-to-use visualization tools
* Seamless cross-compilation to JavaScript

The PMP library was developed by [Daniel Sieger](https://www.linkedin.com/in/dsieger/) and [Mario Botsch](https://scholar.google.com/citations?user=VSiwBoYAAAAJ&hl=en) having in mind research, educational and industrial applications.

PMP is light-weight and easy-to-use alternative to well-known libraries such as [CGAL](https://www.cgal.org/) and [MeshLab](https://www.meshlab.net/). PMP comes under the [MIT license](https://choosealicense.com/licenses/mit/), currently has 886 stars on GitHub and its last release was published in August 2022. For further details, take a look at this [video](https://www.youtube.com/watch?v=RXc9af0Rq8s&ab_channel=ComputerGraphicsatTUDortmund).

All the steps necessary to install and run PMP on Ubuntu 22.04 are listed below. You can also try to install it leveraging the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) (WSL) however, as clarified [here](https://github.com/pmp-library/pmp-library/issues/39#issuecomment-686757220), the WSL is not tested by the developers, so if you are in trouble it may be not easy to get around the issues encountered. If you are interested in the WSL, take a look at [this previous post](https://cristianopizzamiglio.com/2022/08/14/carbon-wsl.html). PMP can be installed on MacOS and Windows as well.

To provide detailed instructions, I assume that you have never run C/C++ code in Ubuntu. If you don't have Ubuntu already installed on your machine, you can find a plethora of online tutorials on how to dual boot Ubuntu with Windows 10.

1. Open a terminal (`CTRL`+`ALT`+`T`).

2. Install [git](https://git-scm.com/) by typing in the terminal (you will be asked to enter your password):
    ```bash
    sudo apt install git-all
    ```
    and type
    ```bash
    git --version
    ```
    to check if the installation is successful.

3. Install the [GNU complier collection](https://gcc.gnu.org/) and other utilities necessary to compile programs:
    ```bash
    sudo apt install build-essential
    ```

4. Install cmake:
    ```bash
    sudo apt install cmake
    ```

5. Install the following packages:
    ```bash
    sudo apt install libx11-dev
    sudo apt install -y libxrandr-dev
    sudo apt install -y libxinerama-dev
    sudo apt install -y libxcursor-dev
    sudo apt install libxi-dev
    ```

6. Install OpenGL to use the viewer classes.
    ```bash
    sudo apt update
    sudo apt install libglu1-mesa-dev freeglut3-dev mesa-common-dev
    ```

7. Download PMP:
    ```bash
    git clone --recursive https://github.com/pmp-library/pmp-library.git 
    ```

8. Checkout the latest release:
    ```bash
    cd pmp-library && git checkout 2.0.1 
    ```

9. Configure and build
    ```bash
    mkdir build && cd build && cmake .. && make
    ```

10. Install PMP:
    ```bash
    sudo make install
    ```

11. Run the viewer application and load the [Stanford bunny](https://en.wikipedia.org/wiki/Stanford_bunny) (as an [OFF](https://en.wikipedia.org/wiki/OFF_(file_format)) file):
    ```bash
    ./mpview ../external/pmp-data/off/bunny.off
    ```

Further installation details can be found [here](https://www.pmp-library.org/installation.html).

Let's take a closer look at the mesh processing viewer. I've uploaded the vripped reconstruction of the Stanford Dragon that you can download [here](http://graphics.stanford.edu/data/3Dscanrep/), which has 437k vertices and 871k faces:
<img src="/img/posts/2022-12-10-pmp-library/viewer-1.png" width="710"/>

The spacebar allows to change the drawing mode:
<img src="/img/posts/2022-12-10-pmp-library/viewer-2.png" width="710"/>

Let's perform a uniform remeshing:
<img src="/img/posts/2022-12-10-pmp-library/viewer-3.png" width="710"/>

The decimation feature allows to reduce the number of triangles:
<img src="/img/posts/2022-12-10-pmp-library/viewer-4.png" width="710"/>

The program can also draw a few curvature maps (mean curvature in the image below):
<img src="/img/posts/2022-12-10-pmp-library/viewer-5.png" width="710"/>

The `W` key allows to export the processed mesh as an OFF file.

To perform more complex operations, you can write your own programs (API reference [here](https://www.pmp-library.org/usergroup0.html)):
```cpp
#include <pmp/SurfaceMesh.h>
using namespace pmp;

int main()
{
    SurfaceMesh mesh;
    mesh.read(mesh,"dragon.ply");
    // do stuff
    mesh.write(mesh,"dragon_smoothed.ply");
}
```

If you are interested in understanding the math behind mesh processing algorithms, I highly recommend reading the [*Polygon Mesh Processing*](https://www.amazon.it/Polygon-Mesh-Processing-Mario-Botsch/dp/1568814267) book by Mario Botsch.
