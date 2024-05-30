# ACT: Action Chunking with Transformers

Original Project Websites:

* [ALOHA](https://tonyzhaozh.github.io/aloha/)
* [Mobile ALOHA](https://mobile-aloha.github.io/)

This repo contains the implementation of ACT, together with 2 simulated environments:

* Transfer Cube
* Bimanual Insertion

You can train and evaluate ACT in simulation or on real hardware.
For real hardware, you would also need to install [ALOHA](https://github.com/Interbotix/aloha).

# Repo Structure

* act
  * ``detr`` Model definitions of ACT, modified from [DETR](https://github.com/facebookresearch/detr)
  * ``policy.py`` An adaptor for ACT policy
  * ``sim_env.py`` Mujoco + DM_Control environments with joint space control
  * ``ee_sim_env.py`` Mujoco + DM_Control environments with EE space control
  * ``scripted_policy.py`` Scripted policies for sim environments
  * ``constants.py`` Constants shared across files
  * ``utils.py`` Utils such as data loading and helper functions
  * scripts
    * ``imitate_episodes.py`` Train and Evaluate ACT
    * ``record_sim_episodes.py`` Record episodes using the simulator
    * ``visualize_episodes.py`` Save videos from a .hdf5 dataset

# Installation

There are two recommended ways to install ACT: using ``conda`` or ``venv``.
Using ``venv`` is preferred due its ease of use against frameworks like ROS.

## Installation Using venv

```bash
sudo apt-get install python3-venv
python3 -m venv ~/act # creates a venv "act" in the home directory, can be created anywhere
source ~/act/bin/activate
pip install dm_control==1.0.14
pip install einops
pip install h5py
pip install ipython
pip install matplotlib
pip install modern_robotics
pip install mujoco==2.3.7
pip install opencv-python
pip install packaging
pip install pexpect
pip install pyquaternion
pip install pyyaml
pip install rospkg
pip install torch
pip install torchvision
pip install transforms3d
cd /path/to/act/detr && pip install -e .
```

## Installation Using conda

### Manual Installation

```bash
conda create -n aloha python=3.8.10
conda activate aloha
pip install dm_control==1.0.14
pip install einops
pip install h5py
pip install ipython
pip install matplotlib
pip install mujoco==2.3.7
pip install opencv-python
pip install packaging
pip install pexpect
pip install pyquaternion
pip install pyyaml
pip install rospkg
pip install torch
pip install torchvision
cd /path/to/act/detr && pip install -e .
```

### Installation from File

```bash
conda env create --file=conda_env.yaml
```

# Example Usage

To set up a new terminal, run:

```bash
source ~/aloha/bin/activate # or conda activate aloha
cd /path/to/act
```

## Simulated experiments

We use ``sim_transfer_cube_scripted`` task in the examples below.
Another option is ``sim_insertion_scripted``.
To generate 50 episodes of scripted data, run:

```bash
python3 record_sim_episodes.py \
  --task_name sim_transfer_cube_scripted \
  --dataset_dir <data save dir> \
  --num_episodes 50
```

To can add the flag ``--onscreen_render`` to see real-time rendering.
To visualize the episode after it is collected, run

```bash
python3 visualize_episodes.py \
  --dataset_dir <data save dir> \
  --episode_idx 0
```

To train ACT:

```bash
# Transfer Cube task
python3 imitate_episodes.py \
  --task_name sim_transfer_cube_scripted \
  --ckpt_dir <ckpt dir> \
  --policy_class ACT \
  --kl_weight 10 \
  --chunk_size 100 \
  --hidden_dim 512 \
  --batch_size 8 \
  --dim_feedforward 3200 \
  --num_epochs 2000 \
  --lr 1e-5 \
  --seed 0
```

To evaluate the policy, run the same command but add ``--eval``.
This loads the best validation checkpoint.
The success rate should be around 90% for transfer cube, and around 50% for insertion.
To enable temporal ensembling, add flag ``--temporal_agg``.
Videos will be saved to ``<ckpt_dir>`` for each rollout.
You can also add ``--onscreen_render`` to see real-time rendering during evaluation.

For real-world data where things can be harder to model, train for at least 5000 epochs or 3-4 times the length after the loss has plateaued.
Please refer to [tuning tips](https://docs.google.com/document/d/1FVIZfoALXg_ZkYKaYVh-qOlaXveq5CtvJHXkY25eYhs/edit?usp=sharing) for more info.

### Updates:
You can find all scripted/human demo for simulated environments [here](https://drive.google.com/drive/folders/1gPR03v05S1xiInoVJn7G7VJ9pDCnxq9O?usp=share_link).

### *New*: [ACT tuning tips](https://docs.google.com/document/d/1FVIZfoALXg_ZkYKaYVh-qOlaXveq5CtvJHXkY25eYhs/edit?usp=sharing)
TL;DR: if your ACT policy is jerky or pauses in the middle of an episode, just train for longer! Success rate and smoothness can improve way after loss plateaus.
