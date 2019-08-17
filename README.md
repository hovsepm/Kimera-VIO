# SparkVIO: Visual Inertial Odometry

[![Build Status](http://ci-sparklab.mit.edu:8080/buildStatus/icon?job=VIO/master)](http://ci-sparklab.mit.edu:8080/job/VIO/job/master/)

## What is SparkVIO?

SparkVIO is a Visual Inertial Odometry pipeline for accurate State Estimation from **Stereo** + **IMU** data. (Mono-only capabilities soon to be released).

We kindly ask to cite the papers below if you find this library useful:

 - **TODO:** add our latest paper.

Backend optimization is based on:

 - C. Forster, L. Carlone, F. Dellaert, and D. Scaramuzza. **On-Manifold Preintegration Theory for Fast and Accurate Visual-Inertial Navigation**. IEEE Trans. Robotics, 33(1):1-21, 2016.

 - L. Carlone, Z. Kira, C. Beall, V. Indelman, and F. Dellaert. **Eliminating Conditionally Independent Sets in Factor Graphs: A Unifying Perspective based on Smart Factors.** IEEE Intl. Conf. on Robotics and Automation (ICRA), 2014.

Alternatively, the `Regular VIO` backend, using structural regularities, is described in this paper:

- A. Rosinol, T. Sattler, M. Pollefeys, and L. Carlone. **Incremental Visual-Inertial 3D Mesh Generation with Structural Regularities**. IEEE Int. Conf. on Robotics and Automation (ICRA), 2019.

## Demo

<div style="text-align:center">
<img src="media/spark_vio_release.gif"/>
</div>

## Installation

Tested on Mac, Ubuntu 14.04 & 16.04.

> Note: if you want to avoid building all these dependencies yourself, we provide a docker image [below](#From-Dockerfile) that will install them for you.

### Prerequisites:

- [GTSAM](https://github.com/borglab/gtsam) >= 4.0
- [OpenCV](https://github.com/opencv/opencv) >= 3.0
- [OpenGV](https://github.com/laurentkneip/opengv) 
- [Glog](http://rpg.ifi.uzh.ch/docs/glog.html), [Gflags](https://gflags.github.io/gflags/), [Gtest](https://github.com/google/googletest/blob/master/googletest/docs/primer.md) (installed automagically).

> Installation instructions below.

### Install GTSAM

#### GTSAM's dependencies:

Install Boost: `sudo apt-get update && sudo apt-get install -y libboost-all-dev`

#### GTSAM's Optional dependencies (highly recommended for speed)

Install [Intel Threaded Building Blocks (TBB)](http://www.threadingbuildingblocks.org/): `sudo apt-get install libtbb-dev`

#### GTSAM Source Install

Clone GTSAM: `git clone git@github.com:borglab/gtsam.git`

> (last tested with commit `0c3e05f375c03c5ff5218e708db416b38f4113c8`)

Make build dir, and run `cmake`:

```bash
cd gtsam
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_BUILD_TYPE=Release -DGTSAM_USE_SYSTEM_EIGEN=OFF .. 
```

Ensure that:
- TBB is enabled: check for `--   Use Intel TBB                  : Yes` after running `cmake`.
- Compilation is in Release mode: check for `--   Build type                     : Release` after running `cmake`.
- Use GTSAM's Eigen, **not** the system-wide one (OpenGV and GTSAM must use same Eigen, see OpenGV install instructions below).

Compile and install GTSAM:
```bash
make $(nproc) check # (optional, runs unit tests)
sudo make $(nproc) install
```

> Alternatively, replace `$(nproc)` by the number of available cores in your computer.

> Note 1a: if you use MKL in gtsam, you may need to add to `~/.bashrc` a line similar to:
>  ```source /opt/intel/parallel_studio_xe_2018/compilers_and_libraries_2018/linux/mkl/bin/mklvars.sh intel64```
> (Alternatively, type `locate compilervars.sh` and then `source $output file.sh that you got from locate$`, add to your .bashrc to automate).

> Note 1b: sometimes you may need to add `/usr/local/lib` to `LD_LIBRARY_PATH` in `~/.bashrc` (if you get lib not found errors at run or test time).

> Note: we are considering to enable EPI in GTSAM, which will require to set the GTSAM_THROW_CHEIRALITY_EXCEPTION to false (cmake flag).

> Note: for better performance when using the IMU factors, set GTSAM_TANGENT_PREINTEGRATION to 'false' (cmake flag)

> Note: also add `-march=native` to `GTSAM_CMAKE_CXX_FLAGS` for max performance (at the expense of the portability of your executable). Check [install gtsam](https://github.com/borglab/gtsam/blob/develop/INSTALL.md) for more details. Note that for some systems, `-march=native` might cause problems that culminates in the form of segfaults when you run the unittests.

### Install OpenCV

#### OpenCV's dependencies:
- on Mac:
```bash
homebrew install vtk # (to check)
```

- on Linux:
```bash
sudo apt-get install -y libvtk5-dev   # (libvtk6-dev in ubuntu 17.10)
sudo apt-get install -y pkg-config libgtk2.0-dev
```

#### OpenCV Source Install

Download OpenCV and run cmake:
```bash
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout tags/3.3.1
mkdir build
cd build
cmake -DWITH_VTK=On .. # Use -DWITH_TBB=On if you have TBB
```

Finally, build and install OpenCV:
```bash
sudo make -j $(nproc) install
```

> Alternatively, replace `$(nproc)` by the number of available cores in your computer.

### Install OpenGV
Clone the repo:
```bash
git clone https://github.com/laurentkneip/opengv 
```

- using cmake-gui, set: the eigen version to the GTSAM one (for me: /Users/Luca/borg/gtsam/gtsam/3rdparty/Eigen). if you don't do so, very weird error (TODO document) appear (may be due to GTSAM and OpenGV using different versions of eigen!)
- in the opengv folder do:

```bash
cd opengv
mkdir build
cd build
cmake ..
sudo make -j $(nproc) install
```

> Alternatively, replace `$(nproc)` by the number of available cores in your computer.

### Glog, Gflags & Gtest
Glog, Gflags, and Gtest will be automatically downloaded using cmake unless there is a system-wide installation found (gtest will always be downloaded).

## Install SparkVIO

### From Source:

Clone this repository: 
```bash
git clone git@github.mit.edu:SPARK/VIO.git SparkVIO
```

Build SparkVIO:
```bash
cd SparkVIO
mkdir build
cd build
cmake ..
make -j $(nproc)
```

> Alternatively, replace '$(nproc)' by the number of available cores in your computer.

### From Dockerfile:

If you want to avoid building all these dependencies yourself, we provide a docker image that will install all dependencies for you.
For that, you will need to install [Docker](https://docs.docker.com/install/).

Once installed, clone this repo, build the image and run it:
> Note: while this repo remains private, you'll need to **specify your ssh keys**: replace `<username>` below, with an ssh key that has read access to the repo in github.mit.edu (check your profile settings for such ssh key). Also, since docker build doesn't handle user input, ensure your ssh key does **not** have a passphrase.

```bash
# Clone the repo
git clone git@github.mit.edu:SPARK/VIO.git SparkVIO
cd SparkVIO

# Build the image
docker build --rm -t spark_vio -f ./scripts/docker/Dockerfile . \
--build-arg SSH_PRIVATE_KEY="$(cat /home/<username>/.ssh/id_rsa)"

# Run an example dataset
./scripts/docker/spark_vio_docker.bash
```

## Running examples

### Euroc Dataset

stereoVIOEuroc:
- Download [EuRoC](http://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) dataset. You can just download this [file](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip) to do a first test, which corresponds to the ```MH_01_easy``` EuRoC dataset.
- Add the comment ```%YAML:1.0``` at the top of each .yaml file in the dataset (each folder has one sensor.yaml).
You can do this manually or alternatively paste and run the following bash script from within the dataset folder:

```bash
sed -i '1 i\%YAML:1.0' body.yaml
sed -i '1 i\%YAML:1.0' */sensor.yaml
```

You have two ways to start the example:
- Using the script ```stereoVIOEuroc.bash``` inside the ```scripts``` folder:
  - Run: ```./stereoVIOEuroc.bash -p "PATH_TO_DATASET/MH_01_easy"```
  - Optionally, you can try the VIO using structural regularities, as in [Toni's thesis](https://www.research-collection.ethz.ch/handle/20.500.11850/297645), by specifying the option ```-r```: ```./stereoVIOEuroc.bash -p "PATH_TO_DATASET/MH_01_easy" -r```
- Alternatively, have a look inside the script to see how to change extra parameters that control the pipeline itself.

### Kitti dataset

- Download raw data from [Kitti ](http://www.cvlibs.net/datasets/kitti/raw_data.php?type=residential) (in order to have IMU messages). For example:
  - Download unsynced + unrectified raw dataset (e.g. [2011\_09\_26\_drive\_0005\_extract](https://s3.eu-central-1.amazonaws.com/avg-kitti/raw_data/2011_09_26_drive_0005/2011_09_26_drive_0005_extract.zip)).
  - Download as well the calibration data (three files) from raw dataset (e.g. [2011\_09\_26\_calib](https://s3.eu-central-1.amazonaws.com/avg-kitti/raw_data/2011_09_26_calib.zip)), and save them inside the folder `2011_09_26`.

- Run: `./scripts/stereoVIOEuroc.bash -p PATH_TO_DATASET -d 1 -r -parallel` where you specify the path to the dataset (e.g. path to '2011_09_26_drive_0005_extract' folder). (Again, the `-r` is enabling using the structural regularities and the `-parallel` is for using VIO in parallel mode)


Tips for usage
----------------------
- The 3D Visualization window implements the following keyboard shortcuts (you need to have the window in focus, click on it):
    - Press 't': toggle freezing visualization (as of know, this blocks the whole pipeline, it might change once the visualization is threaded).
    - Press 'v': prints to the terminal the pose of the current viewpoint of the 3D visualization window.
    - Press 'w': prints to the terminal the size of the 3D visualization window.
    - Do not press 'q': unless you want to terminate the pipeline in an abrupt way, see #74.

     These last two shortcuts are useful if you want to programmatically set the initial viewpoint and size of the screen when launching the 3D visualization window (this is done at the constructor of the 3DVisualizer class).
    - Press 's': to get a screenshot of the 3D visualization window.
    - Press '0', '1', or '2': to toggle the 3D mesh representation (only visible if the gflag 'visualize_mesh' is set to true).
    - Press 'a': to toggle ambient light for the 3D mesh ('visualize_mesh' has to be set to true).
    - Press 'l': to toggle lighting for the 3D mesh ('visualize_mesh' has to be set to true).

> For these shortcuts to take effect you need to have the window in focus. Note also that there might be a big delay between your keypress and the actual processing of the information as this is purely sequential with the VIO pipeline itself; this might change after parallelizing the pipeline).

- The 3D Visualization allows to load an initial `.ply` file.
This is useful for example for the Euroc dataset `V1_01_*` where we are given a point-cloud of the scene. Note that the `.ply` file must have the following properties in the header, and its corresponding entries:
  - A vertex element: `element vertex 3199068` (actual number depends on how many vertices has the ply file).
  - A face element: `element face 0`(actual number depends on how many faces has the ply file)
  - Properties may vary, as for now, any given `.ply` file will be displayed as a point-cloud, not a mesh (due to some weirdness in opencv).
  - For example, the ground-truth point-cloud provided in `V1_01_easy` dataset must be modified to have the following header:
      ```
      ply
      format ascii 1.0
      element vertex 3199068
      property float x
      property float y
      property float z
      property float intensity
      property uchar diffuse_red
      property uchar diffuse_green
      property uchar diffuse_blue
      element face 0
      property list uint8 int32 vertex_indices
      end_header
      ```

Tips for development
----------------------
- To make the pipeline deterministic:
    - Disable TBB in GTSAM (go to build folder in gtsam, use cmake-gui to unset ```GTSAM_WITH_TBB```).
    - Change ```ransac_randomize``` flag in ```params/trackerParameters.yaml``` to 0, to disable ransac randomization.

> Note: these changes are not sufficient to make the output repeatable between different machines.

> Note to self: remember that we are using ```-march=native``` compiler flag, which will be a problem if we ever want to distribute binaries of this code.
>

For Developers: Pull Requests
======================

## Use code linter

To contribute to this repo, ensure your commits pass the linter pre-commit checks.

### Dependencies

Install the following dependencies to run the linter:

 * **requests** `pip install requests`
 * **pylint** `pip install pylint`
 * **yapf** `pip install yapf`
 * **clang-format**
   * Compatible with `clang-format-3.8 - 6.0`
   * Ubuntu: `sudo apt install clang-format-${VERSION}`
   * macOS:
     ```
     brew install clang-format
     ln -s /usr/local/share/clang/clang-format-diff.py /usr/local/bin/clang-format-diff
     ```

### Installation

```bash
cd $THIS_REPO
git submodule update --init
echo "source $(realpath ./dev_tools/linter/setup_linter.sh)" >> ~/.bashrc  # Or the matching file for
                                                   # your shell.
bash
```

Then you can install the linter in your repository:

```
cd $THIS_REPO
init_linter_git_hooks
```
For more information about the linter, check [here](https://github.com/ToniRV/linter).
