# Deepdrive [![Build Status](https://travis-ci.org/deepdrive/deepdrive.svg?branch=master)](https://travis-ci.org/deepdrive/deepdrive)

The easiest way to experiment with self-driving AI

## Simulator requirements

- Linux or Windows
- Python 3.5 or 3.6 (Recommend Miniconda for Windows)
- 3GB disk space
- 8GB RAM

## Optional - baseline agent requirements

- CUDA capable GPU (tested and developed on 970, 1070, and 1060's)
- Tensorflow 1.1+ [See Tensorflow install tips](#tensorflow-install-tips)

## Install

```
git clone https://github.com/deepdrive/deepdrive
cd deepdrive
```

> Optional - Activate the Python / virtualenv where your Tensorflow is installed, then

#### Linux
```
python install.py
```

#### Windows
Make sure the Python you want to use is in your PATH, then

> Tip: We highly recommend using [Conemu](https://conemu.github.io/) for your Windows terminal

```
python install.py
```

#### Cloud

We've tested on Paperspace's ML-in-a-Box Linux public template which already has Tensorflow installed and just requires

```
python install.py
```

If you run into issues, try starting the sim directly as Unreal may need to install some prerequisetes (i.e. DirectX needs to be installed on the Paperspace Parsec Windows box). The default location of the Unreal sim binary is in your user directory under <kbd>Deepdrive/sim</kbd>.

## Usage

> If you're in PyCharm - make sure you disable the SciView in Settings > Tools > Python Scientific > Show Plots in Toolwindow

Run the **baseline** agent
```
python main.py --baseline --experiment-name my-baseline-test
```

Run in-game path follower
```
python main.py --path-follower --experiment-name my-path-follower-test
```

**Record** training data for imitation learning / behavioral cloning
```
python main.py --record --record-recovery-from-random-actions
```

**Train** on recorded data
```
python main.py --train
```

**Train** on the same dataset we used 

Grab the [dataset](#dataset)
```
python main.py --train --recording-dir <the-directory-with-the-dataset>
```

**Change cameras** to one of the rigs in `camera_config.py`

```
python main.py --camera-rigs="three_cam_rig"
```

### Key binds 

* <kbd>Esc</kbd> - Pause
* <kbd>Alt+Tab</kbd> - Control other windows* <kbd>;</kbd> - Toggle FPS
* <kbd>m</kbd> - Toggle manual driving (control the car yourself)
* <kbd>1</kbd> - Chase cam
* <kbd>2</kbd> - Orbit (side) cam
* <kbd>3</kbd> - Hood cam
* <kbd>4</kbd> - Free cam (use WASD to fly)
* WASD or Up, Down, Left Right - steer / throttle
* <kbd>Space</kbd> - Handbrake
* <kbd>Shift</kbd> - Nitro
* <kbd>H</kbd> - Horn
* <kbd>L</kbd> - Light
* <kbd>R</kbd> - Reset
* <kbd>E</kbd> - Gear Up
* <kbd>Q</kbd> - Gear down
* <kbd>Z</kbd> - Show mouse
* <kbd>`</kbd><kbd>`</kbd> - Unreal console (first press releases game input capture)


## Benchmark

| Agent  |  10 lap avg score  | Weights |  Deepdrive version |
| :---    | ---:   | :---    |   ---: |
|Baseline agent (trained with imitation learning)|[1691](https://docs.google.com/spreadsheets/d/1ryFaMFJhcTMBuhXZv0eMFHO35NMcXE2_MFLYqeUosfM/edit#gid=0)|[baseline_agent_weights.zip](https://d1y4edi1yk5yok.cloudfront.net/weights/baseline_agent_weights.zip)|2.0|
|Path follower |[*1069](https://docs.google.com/spreadsheets/d/1T5EuEobdVFn5ewdYTO20i9CqcZ-jIEsAihlV5lpvLQQ/edit#gid=0)| N/A (see [3D spline follower](https://github.com/crizCraig/deepdrive-beta/blob/bde6b8c48314c34a96ce0942fc398fae840720ee/Source/DeepDrive/Private/Car.cpp#L409))|2.0|

*The baseline agent currently outperforms the path follower it was trained on, likely due to the slower
speed the at which the baseline agent drives, resulting in lower lane deviation and g-force penalties. 
Interestingly, reducing the path follower speed causes it to crash at points where it otherwise loses traction and drifts, 
so the baseline agent has actually learned a more robust turning function than the original hardcoded path follower
it was trained on.

## Dataset

100GB (8.2 hours of driving) of camera, depth, steering, throttle, and brake of an 'oracle' path following agent. We rotate between three different cameras: normal, wide, and semi-truck - with random camera intrisic/extrinsic perturbations at the beginning of each episode (lap). This boosted performance on the benchmark by 3x. We also use DAgger to collect course correction data as in previous versions of Deepdrive.

1. Get the [AWS CLI](https://github.com/aws/aws-cli)
2. Ensure you have 104GB of free space
3. Download our dataset of mixed Windows (Unreal PIE + Unreal packaged) and Linux + variable camera and corrective action recordings 
(generated with `--record`)
```
cd <the-directory-you-want>
aws --no-sign-request --region=us-west-1 s3 sync s3://deepdrive/data/baseline .
```

If you'd like to check out our Tensorboard training session, you can download the 13GB
[tfevents files here](https://d1y4edi1yk5yok.cloudfront.net/tensorflow/baseline_tensorflow_train_and_eval.zip),
unzip, and run

```
tensorboard --logdir <your-unzipped-baseline_tensorflow_train_and_eval>
```

## Frame rate issues on Linux

If you experience low frame rates on Linux, you may need to install NVIDIA’s display drivers including their OpenGL drivers. We recommend installing these with CUDA which bundles the version you will need to run the baseline agent. Also, make sure to [plugin your laptop](https://help.ubuntu.com/community/PowerManagement/ReducedPower). If CUDA is installed, skip to testing [OpenGL](#opengl).

## Tensorflow install tips

- Use the Tensorflow instructions on GitHub for the release you wish to install (not the docs website), i.e. here's the [latest](https://github.com/tensorflow/tensorflow/releases/latest) - so for v1.6.0, you would use [https://github.com/tensorflow/tensorflow/blob/v1.6.0/tensorflow/docs_src/install](https://github.com/tensorflow/tensorflow/blob/v1.6.0/tensorflow/docs_src/install).
- Make sure to install the CUDA / cuDNN major and minor version the Tensorflow instructions specify.  i.e. CUDA 9.0 / cuDNN 7.0. These will likely be older than the latest version NVIDIA offers. You can see all [CUDA  releases here](https://developer.nvidia.com/cuda-toolkit-archive).
- Use the packaged install, i.e. deb[local] on Ubuntu, referred to in [this guide](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
- If you are feeling dangerous and use the runfile method, be sure to follow [NVIDIA’s instructions](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) on how to disable the Nouveau drivers if you're on Ubuntu.

## OpenGL

`glxinfo | grep OpenGL` should return something like:
```
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: GeForce GTX 980/PCIe/SSE2
OpenGL core profile version string: 4.5.0 NVIDIA 384.90
OpenGL core profile shading language version string: 4.50 NVIDIA
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile
OpenGL core profile extensions:
OpenGL version string: 4.5.0 NVIDIA 384.90
OpenGL shading language version string: 4.50 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)
OpenGL extensions:
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 384.90
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL ES profile extensions:
```
You may need to disable secure boot in your BIOS in order for NVIDIA’s OpenGL and tools like nvidia-smi to work. This is not Deepdrive specific, but rather a general requirement of Ubuntu’s NVIDIA drivers.

## Citing

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1248998.svg)](https://doi.org/10.5281/zenodo.1248998)

Bibtex
```
@misc{craig_quiter_2018_1248998,
  author       = {Craig Quiter and
                  Maik Ernst},
  title        = {deepdrive/deepdrive: 2.0},
  month        = mar,
  year         = 2018,
  doi          = {10.5281/zenodo.1248998},
  url          = {https://doi.org/10.5281/zenodo.1248998}
}
```


## Development

To run tests in PyCharm, go to File | Settings | Tools | Python Integrated Tools and change the default test runner 
to py.test.

## Thanks

Special thanks to [Rafał Józefowicz](https://scholar.google.com/citations?user=C7zfAI4AAAAJ) for contributing the original [training](#tensorflow_agent/train) code used for the baseline agent

[License](LICENSE.md)


