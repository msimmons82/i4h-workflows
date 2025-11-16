# 🤖 Robotic Ultrasound Workflow

![Robotic Ultrasound Workflow](../../docs/source/robotic_us_workflow.jpg)

---

## 🔬 Technical Overview

The Robotic Ultrasound Workflow is a comprehensive solution designed for healthcare professionals, medical imaging researchers, and ultrasound device manufacturers working in the field of autonomous ultrasound imaging. This workflow provides a robust framework for simulating, training, and deploying robotic ultrasound systems using NVIDIA's advanced ray tracing technology. By offering a physics-accurate ultrasound simulation environment, it enables researchers to develop and validate autonomous scanning protocols, train AI models for image interpretation, and accelerate the development of next-generation ultrasound systems without requiring physical hardware.

The workflow features a state-of-the-art ultrasound sensor simulation that leverages GPU-accelerated ray tracing to model the complex physics of ultrasound wave propagation. The simulator accurately represents:
- Acoustic wave propagation through different tissue types
- Tissue-specific acoustic properties (impedance, attenuation, scattering)
- Real-time B-mode image generation based on echo signals
- Dynamic tissue deformation and movement
- Multi-frequency transducer capabilities

This physics-based approach enables the generation of highly realistic synthetic ultrasound images that closely match real-world data, making it ideal for training AI models and validating autonomous scanning algorithms. The workflow supports multiple AI policies (PI0, GR00T N1) and can be deployed using NVIDIA Holoscan for clinical applications, providing a complete pipeline from simulation to real-world deployment.

### 🎯 Isaac Sim/Lab Integration

This workflow is built on **NVIDIA Isaac Sim** and **NVIDIA Isaac Lab**. When you run the workflow scripts, Isaac Sim/Lab provides:

- **🤖 Robot Physics**: Accurate Franka arm dynamics with precise end-effector control for ultrasound probe manipulation
- **🔧 Real-Time Control**: Live robot control through AI policies, teleoperation, or automated scanning protocols
- **📊 Sensor Integration**: Multi-modal data streams including RGB cameras, depth sensing, and ultrasound B-mode imaging

---

## 📋 Table of Contents

- [🚀 Quick Start](#-quick-start)
- [⚡ Running Workflows](#-running-workflows)
- [🔧 Detailed Setup Instructions](#-detailed-setup-instructions)
- [🛠️ Troubleshooting](#️-troubleshooting)

---

## 🚀 Quick Start

### ⏱️ Installation Timeline
**Estimated Setup Duration:** 30-40 minutes (network-dependent asset downloads)

### 🔍 System Prerequisites Validation

#### GPU Architecture Requirements
- **NVIDIA GPU**: RT Core-enabled architecture (Ampere or later)
- **Compute Capability**: ≥8.6
- **VRAM**: ≥24GB GDDR6/HBM
- **Unsupported**: A100, H100 (lack RT Cores for ray tracing acceleration)

   <details>
   <summary>🔍 GPU Compatibility Verification</summary>

   ```bash
   nvidia-smi --query-gpu=name,compute_cap --format=csv,noheader
   ```

   Verify output shows compute capability ≥8.6 (Ampere/Ada Lovelace/Hopper with RT Cores)
   </details>

#### Driver & System Requirements
- **Operating System**: Ubuntu 22.04 LTS / 24.04 LTS (x86_64)
- **NVIDIA Driver**: ≥555.x (RTX ray tracing API support)
- **CUDA Toolkit**: ≥12.6 (OptiX 8.x compatibility) but < 13.0
- **Memory Requirements**: ≥24GB GPU memory, ≥64GB system RAM
- **Storage**: ≥100GB NVMe SSD (asset caching and model downloading)

   <details>
   <summary>🔍 Driver Version Validation</summary>

   ```bash
   nvidia-smi --query-gpu=driver_version --format=csv,noheader,nounits
   ```
   </details>

   <details>
   <summary>🔍 CUDA Toolkit Verification</summary>

   ```bash
   nvcc --version | grep "release" | awk '{print $6}' | cut -d',' -f1
   ```
   </details>

#### Software Dependencies
- **Python**: 3.11 (exact version required)
- **Conda**: Miniconda or Anaconda ([installation guide](https://www.anaconda.com/docs/getting-started/miniconda/install))

#### Communication Middleware
- **RTI Connext Data Distribution Service (DDS)**: Please download [RTI Connext](https://content.rti.com/l/983311/2025-07-08/q5x1n8) to provide access to the DDS. To obtain a license/activation key, please [click here](https://content.rti.com/l/983311/2025-07-25/q6729c). Additional information on Connext can be found here: https://www.rti.com/products/third-party-integrations/nvidia.  For any questions, please email evaluations@rti.com.

---

### 🐍 Conda Environment Setup

The robotic ultrasound workflow requires conda-based environment management due to mixed pip and system package dependencies.

Installation reference: [Miniconda Installation Guide](https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions)

#### 1️⃣ Repository Clone
```bash
git clone https://github.com/isaac-for-healthcare/i4h-workflows.git
cd i4h-workflows
```

#### 2️⃣ Environment Creation & Dependency Installation
```bash
conda create -n robotic_ultrasound python=3.11 -y
conda activate robotic_ultrasound
bash tools/env_setup_robot_us.sh
```

**⚠️ Expected Build Time**: The environment setup process takes 40-60 minutes. You may encounter intermediary warnings about macaroon bakery library dependencies - these are non-critical and can be ignored.

#### 3️⃣ Environment Variable Configuration
```bash
export PYTHONPATH=`pwd`/workflows/robotic_ultrasound/scripts:$PYTHONPATH
export RTI_LICENSE_FILE=<path-to-your-rti-license-file>
```

   <details>
   <summary>💾 Persistent Environment Configuration</summary>

   ```bash
   echo "export PYTHONPATH=`pwd`/workflows/robotic_ultrasound/scripts:\$PYTHONPATH" >> ~/.bashrc
   echo "export RTI_LICENSE_FILE=<path-to-your-rti-license-file>" >> ~/.bashrc
   source ~/.bashrc
   ```

   This ensures the environment variables are automatically set when you open new terminals.

   **Note:** If you have `robotic_surgery` workflow scripts or previous versions of `robotic_ultrasound` workflow scripts in your `PYTHONPATH`, you can reset it to include only the robotic_ultrasound scripts by running `export PYTHONPATH=$(pwd)/workflows/robotic_ultrasound/scripts`
   </details>


✅ **Installation Complete** - Your robotic ultrasound simulation environment is operational.

---

## ⚡ Running Workflows

### 🔬 Experimental Configurations

#### 🤖 Policy-Based Control with PI0
```bash
conda activate robotic_ultrasound
(python -m policy_runner.run_policy --policy pi0 & python -m simulation.environments.sim_with_dds --enable_cameras & wait)
```

**What Happens in Isaac Sim:**
- **🏥 Medical Scene**: Isaac Sim creates a space with Franka robotic arm positioned next to a hospital bed with patient model
- **🧠 AI Policy Control**: Gr00t or PI0 neural network processes real-time visual input to autonomously control ultrasound scanning motions
- **📡 DDS Communication**: Real-time data exchange between AI policy and robot simulation via distributed messaging
- **📸 Multi-Camera Setup**: Multiple RGB cameras provide different viewpoints of the scanning procedure

**How to Interact with Isaac Sim:**
- **🖱️ Scene Navigation**: Use mouse controls to orbit around the phantom and robot for different viewing angles
- **⏸️ Simulation Control**: Spacebar to pause/resume the autonomous scanning sequence
- **📊 Real-Time Monitoring**: Observe robot joint states, camera feeds, and policy decisions in the GUI
- **🔍 Component Inspection**: Click on robot links (under `Stage` -> `World` -> `envs` -> `env_0` -> `Robot`) to view detailed properties

> **Note:**
> You may see "IsaacSim is not responding". It can take approximately several minutes to download the assets and models from the internet and load them to the scene. If this is the first time you run the workflow, it can take up to 10 minutes.
> It may take an additional 1 or 2 minutes for the policy to start inferencing, so the robot arm may not move immediately.

> ⏳ **Initial Load Time**: First execution may require 10+ minutes for asset download and scene initialization

#### 🔊 Integrated Ultrasound Raytracing Pipeline
```bash
conda activate robotic_ultrasound
(python -m policy_runner.run_policy --policy pi0 & \
python -m simulation.environments.sim_with_dds --enable_cameras &  \
python -m simulation.examples.ultrasound_raytracing & \
python -m utils.visualization & \
wait)
```

**What Happens in Isaac Sim:**
- **🤖 Autonomous Scanning**: AI-controlled robot performs systematic ultrasound examination of patient anatomy
- **📊 Real-Time B-Mode**: Live ultrasound images generated based on simulated acoustic reflections and tissue properties
- **🎭 Multi-Modal Fusion**: Simultaneous RGB camera feeds and ultrasound imaging for comprehensive scene understanding

**Key Isaac Sim Technologies:**
- **⚡ RTX Ray Tracing**: Hardware-accelerated acoustic wave simulation through complex tissue geometries
- **🔄 Synchronized Simulation**: Coordinated robot motion, phantom scanning, and ultrasound image generation
- **📈 Performance Monitoring**: Real-time FPS, physics step timing, and ray tracing performance metrics

#### 🎮 Manual Teleoperation Interface
```bash
conda activate robotic_ultrasound
(python -m simulation.examples.ultrasound_raytracing & \
python -m simulation.environments.teleoperation.teleop_se3_agent --enable_cameras & \
python workflows/robotic_ultrasound/scripts/utils/visualization.py & \
wait)
```

**What Happens in Isaac Sim:**
- **🎮 Manual Control**: Direct 6-DOF (position + orientation) control of the ultrasound probe via keyboard, SpaceMouse, or gamepad
- **🩻 Live Ultrasound Feedback**: Real-time B-mode images update as you manually scan different anatomical regions
- **📸 Multi-Camera Views**: Observe your scanning technique from multiple camera perspectives simultaneously
- **🔧 Interactive Physics**: Feel realistic probe-to-phantom contact detection and constraints during manual scanning

**Isaac Sim Control Features:**
- **⌨️ Keyboard Mapping**: Reference [Teleoperation Documentation](./scripts/simulation/environments/teleoperation/README.md#keyboard-controls)
- **🖱️ 3D Mouse Support**: Direct SE(3) control for intuitive ultrasound probe manipulation
- **📊 Real-Time Feedback**: Live visualization of probe position, camera feed, and ultrasound image

> 🔄 **Process Termination**: Use `Ctrl+C` followed by `bash workflows/robotic_ultrasound/reset.sh` to cleanly terminate all distributed processes.

---

### 🎯 Workflow Component Matrix

| Category | Script | Usage Scenario | Purpose | Documentation | Key Requirements | Expected Runtime |
|----------|--------|----------------|---------|---------------|------------------|------------------|
| **🚀 Quick Start** | [simulation/imitation_learning/pi0_policy/eval.py](scripts/simulation/imitation_learning/pi0_policy/eval.py) | First-time users, policy testing | PI0 policy evaluation | [Simulation README](./scripts/simulation/imitation_learning/README.md) | PI0 policy, Isaac Sim | 2-5 minutes |
| **🔄 Multi-Component** | [simulation/environments/sim_with_dds.py](scripts/simulation/environments/sim_with_dds.py) | Full pipeline testing | Main simulation with DDS communication | [Simulation README](./scripts/simulation/environments/README.md#simulation-with-dds) | Isaac Sim, DDS | Continuous |
| **🎮 Interactive Control** | [simulation/environments/teleoperation/teleop_se3_agent.py](scripts/simulation/environments/teleoperation/teleop_se3_agent.py) | Manual control, data collection | Manual robot control via keyboard/gamepad | [Simulation README](./scripts/simulation/environments/teleoperation/README.md) | Isaac Sim, input device | Continuous |
| **🩺 Ultrasound Simulation** | [simulation/examples/ultrasound_raytracing.py](scripts/simulation/examples/ultrasound_raytracing.py) | Realistic ultrasound imaging | Physics-based ultrasound image generation | [Simulation README](scripts/simulation/examples/README.md) | RayTracing Simulator | Continuous |
| **🤖 Policy Inference** | [policy_runner/run_policy.py](scripts/policy_runner/run_policy.py) | Policy deployment | Generic policy runner for PI0 and GR00T N1 models | [Policy Runner README](scripts/policy_runner/README.md) | Model inference, DDS | Continuous |
| **🧠 Policy Training** | [training/pi_zero/train.py](scripts/training/pi_zero/train.py) | Model development | Train PI0 imitation learning models | [PI0 Training README](scripts/training/pi_zero/README.md) | Training data, GPU | Depends on the dataset size |
| **🧠 Policy Training** | [training/gr00t_n1/train.py](scripts/training/gr00t_n1/train.py) | Advanced model development | Train GR00T N1 foundation models | [GR00T N1 Training README](scripts/training/gr00t_n1/README.md) | Training data, GPU | Depends on the dataset size |
| **🔄 Data Processing** | [training/convert_hdf5_to_lerobot.py](scripts/training/convert_hdf5_to_lerobot.py) | Data preprocessing | Convert HDF5 data to LeRobot format | [GR00T N1 Training README](scripts/training/gr00t_n1/README.md#data-conversion) | HDF5 files | Depends on the dataset size |
| **📈 Evaluation** | [simulation/evaluation/evaluate_trajectories.py](scripts/simulation/evaluation/evaluate_trajectories.py) | Performance analysis | Compare predicted vs ground truth trajectories | [Evaluation README](scripts/simulation/evaluation/README.md) | Trajectory data | Depends on the dataset size |
| **🏗️ State Machine** | [simulation/environments/state_machine/liver_scan_sm.py](scripts/simulation/environments/state_machine/liver_scan_sm.py) | Automated data collection | Automated liver scanning protocol | [Simulation README](scripts/simulation/environments/state_machine/README.md) | Isaac Sim | 5-15 minutes |
| **🗂️ Data Collection** | [simulation/environments/state_machine/liver_scan_sm.py](scripts/simulation/environments/state_machine/liver_scan_sm.py) | Automated data collection | Automated liver scanning protocol | [Simulation README](scripts/simulation/environments/state_machine/README.md) | Isaac Sim | 5-15 minutes |
| **🔄 Replay** | [simulation/environments/state_machine/replay_recording.py](scripts/simulation/environments/state_machine/replay_recording.py) | Data validation | Replay recorded robot trajectories | [Simulation README](scripts/simulation/environments/state_machine/README.md#replay-recordings) | Recording files | 2-5 minutes |
| **🎯 Customization Tutorial** | [tutorials/assets/bring_your_own_patient/README.md](../../tutorials/assets/bring_your_own_patient/README.md) | Patient data integration | Convert CT/MRI scans to USD for simulation | [Patient Tutorial](../../tutorials/assets/bring_your_own_patient/README.md) | MONAI, medical imaging data | Variable |
| **🎯 Customization Tutorial** | [tutorials/assets/bring_your_own_robot/replace_franka_hand_with_ultrasound_probe.md](../../tutorials/assets/bring_your_own_robot/replace_franka_hand_with_ultrasound_probe.md) | Robot customization | Replace Franka hand with ultrasound probe | [Robot Tutorial](../../tutorials/assets/bring_your_own_robot/replace_franka_hand_with_ultrasound_probe.md) | Isaac Sim, CAD/URDF files | 30-60 minutes |
| **🥽 XR Teleoperation Tutorial** | [tutorials/assets/bring_your_own_xr/README.md](../../tutorials/assets/bring_your_own_xr/README.md) | Mixed reality control | OpenXR hand tracking with Apple Vision Pro | [Bring Your Own XR README](../../tutorials/assets/bring_your_own_xr/README.md) | Isaac Lab, CloudXR Runtime, Apple Vision Pro | 10-15 minutes |
| **📊 Visualization** | [utils/visualization.py](scripts/utils/visualization.py) | Monitoring simulations, debugging | Real-time camera feeds and ultrasound display | [Utils README](./scripts/utils/README.md) | DDS, GUI | Continuous |
| **🏥 Hardware-in-the-loop** | [holoscan_apps/clarius_cast/clarius_cast.py](scripts/holoscan_apps/clarius_cast/clarius_cast.py) | Hardware-in-the-loop | Clarius Cast ultrasound probe integration | [Holoscan Apps README](scripts/holoscan_apps/README.md) | Clarius probe, Holoscan | Continuous |
| **🏥 Hardware-in-the-loop** | [holoscan_apps/clarius_solum/clarius_solum.py](scripts/holoscan_apps/clarius_solum/clarius_solum.py) | Hardware-in-the-loop | Clarius Solum ultrasound probe integration | [Holoscan Apps README](scripts/holoscan_apps/README.md) | Clarius probe, Holoscan | Continuous |
| **🏥 Hardware-in-the-loop** | [holoscan_apps/realsense/camera.py](scripts/holoscan_apps/realsense/camera.py) | Hardware-in-the-loop | RealSense depth camera integration | [Holoscan Apps README](scripts/holoscan_apps/README.md) | RealSense camera, Holoscan | Continuous |
| **📡 Communication** | [dds/publisher.py](scripts/dds/publisher.py) | Data streaming | DDS data publishing utilities | [DDS README](scripts/dds/README.md) | DDS license | Continuous |
| **📡 Communication** | [dds/subscriber.py](scripts/dds/subscriber.py) | Data reception | DDS data subscription utilities | [DDS README](scripts/dds/README.md) | DDS license | Continuous |

### 🎓 Understanding the Isaac Sim Workflow Architecture

When you run robotic ultrasound workflow scripts, here's how they integrate with Isaac Sim:

```
📦 Workflow Script Launch
    ↓
🚀 Isaac Sim Initialization
    ├── 🌍 Medical Scene Creation (Patient Room)
    ├── 🤖 Franka Robot Loading (7-DOF Arm + Ultrasound Probe)
    ├── 🏥 Environment Setup (Hospital Bed, Patient Model)
    └── 📸 Sensor Configuration (RGB Cameras, Ultrasound Transducer)
    ↓
⚙️ Simulation Loop
    ├── 🧠 Control Logic (AI Policy/Teleoperation/State Machine)
    ├── 🔄 Physics Step (Robot Dynamics + Phantom Scanning)
    ├── 🩻 Ultrasound Ray Tracing (Acoustic Wave Simulation)
    ├── 📊 Sensor Updates (Camera Feeds + B-Mode Images)
    └── 📡 DDS Communication (Real-Time Data Streaming)
```

**Core Isaac Sim Components for Ultrasound:**

- **🌍 World**: Medical environment with realistic patient room and equipment
- **🤖 Franka Articulation**: 7-DOF robotic arm with precise end-effector control
- **🩻 Ultrasound Simulator**: GPU-accelerated acoustic ray tracing for B-mode image generation
- **📸 Multi-Camera System**: RGB and depth cameras for visual feedback and policy input
- **📡 DDS Integration**: Real-time communication between simulation and AI policies
- **🔧 Interactive Controls**: Teleoperation interfaces for manual probe control

**Script-to-Simulation Flow:**
1. **Isaac Sim Launch**: Python script initializes simulation app with medical environment
2. **Robot & Patient Setup**: Franka arm, ultrasound probe, and patient anatomy are loaded
3. **Sensor Configuration**: Cameras and ultrasound transducer are positioned and calibrated
4. **Control Mode Selection**: AI policy, teleoperation, or automated scanning is activated
5. **Real-Time Loop**: Robot moves probe, ultrasound images generate, data streams via DDS
6. **Visualization**: Multi-modal rendering shows robot motion, phantom scanning, and ultrasound images

---

## 🔧 Detailed Setup Instructions

<details>
<summary>📋 Advanced Configuration & Troubleshooting</summary>

### 🔩 Hardware Architecture Requirements

#### Compute Infrastructure
- **Operating System**: Ubuntu 22.04 LTS / 24.04 LTS (x86_64)
- **GPU Architecture**: NVIDIA RTX/GeForce RTX/Quadro RTX with RT Cores
- **Memory Requirements**: ≥24GB GPU memory, ≥64GB system RAM
- **Storage**: ≥500GB NVMe SSD (asset caching, simulation data and model weights)

### 🏗️ Framework Architecture Dependencies

The robotic ultrasound workflow is built on the following dependencies:
- [IsaacSim 4.5.0](https://docs.isaacsim.omniverse.nvidia.com/4.5.0/index.html)
- [IsaacLab 2.1.0](https://isaac-sim.github.io/IsaacLab/v2.1.0/index.html)
- [Gr00T N1](https://github.com/NVIDIA/Isaac-GR00T)
- [Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1/tree/main)
- [openpi](https://github.com/Physical-Intelligence/openpi) and [lerobot](https://github.com/huggingface/lerobot)
- [Raytracing Ultrasound Simulator](https://github.com/isaac-for-healthcare/i4h-sensor-simulation/tree/v0.3.0/ultrasound-raytracing)
- [RTI Connext DDS](https://www.rti.com/products)

### 🐳 Docker Installation Procedures

Please refer to the [Robotic Ultrasound Docker Container Guide](./docker/README.md) for detailed instructions on how to run the workflow in a Docker container.

### 🔨 Conda Installation Procedures

#### 1️⃣ NVIDIA Graphics Driver Installation
Install or upgrade to the latest NVIDIA driver from [NVIDIA website](https://www.nvidia.com/en-us/drivers/)

#### 2️⃣ CUDA Toolkit Installation
Install CUDA from [NVIDIA CUDA Quick Start Guide](https://docs.nvidia.com/cuda/cuda-quick-start-guide/index.html)

```bash
# Download CUDA installer
 wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
 sudo dpkg -i cuda-keyring_1.1-1_all.deb
 sudo apt-get update
 sudo apt-get -y install cuda-toolkit-12-8
 echo 'export PATH=/usr/local/cuda-12.8/bin:$PATH' >> ~/.bashrc
 echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.8/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
 source ~/.bashrc
```

#### 3️⃣ RTI DDS License Configuration
1. Obtain a license/activation key, please [click here](https://content.rti.com/l/983311/2025-07-25/q6729c)
2. Download license file
3. Configure environment variable:
```bash
export RTI_LICENSE_FILE=/path/to/rti_license.dat
```

#### 4️⃣ Conda Environment Initialization
```bash
# Install Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# Create isolated environment
conda create -n robotic_ultrasound python=3.11 -y
conda activate robotic_ultrasound
```

#### 5️⃣ Dependency Resolution & Installation
```bash
git clone https://github.com/isaac-for-healthcare/i4h-workflows.git
cd i4h-workflows

# Install base dependencies + policy-specific packages
bash tools/env_setup_robot_us.sh --policy pi0     # PI0 policies
# OR
bash tools/env_setup_robot_us.sh --policy gr00tn1 # GR00T N1 foundation models
# OR
bash tools/env_setup_robot_us.sh --policy none    # Base dependencies only
```
The environment for pi0 and gr00tn1 has conflicts with each other. You can only install one of them at a time.

**Dependency Conflict Resolution:**
Expected PyTorch version conflicts between IsaacLab (2.5.1) and OpenPI (2.6.0) are non-critical and can be ignored.

#### 6️⃣ Raytracing Ultrasound Simulator Installation

Choose one of the following options:
- **(Use pre-built binary)** Current [installation script](../../tools/env_setup_robot_us.sh) will download the pre-built binary and install it to `workflows/robotic_ultrasound/scripts/raysim`.

- **(Compiling from source)** Install and build following instructions in [Raytracing Ultrasound Simulator](https://github.com/isaac-for-healthcare/i4h-sensor-simulation/tree/v0.3.0/ultrasound-raytracing#bare-metal-installation) and copy the `raysim` folder to `workflows/robotic_ultrasound/scripts/`.

### 📦 Asset Management

#### Automated Asset Retrieval

Asset retrieval is done automatically when running the workflow.

#### Manual Asset Retrieval
```bash
i4h-asset-retrieve
```

**Asset Storage**: `~/.cache/i4h-assets/<sha256>/`
**Total Size**: 2.7GB (asset, incremental download)
**Reference**: [Asset Catalog Documentation](https://github.com/isaac-for-healthcare/i4h-asset-catalog/blob/v0.3.0/docs/catalog_helper.md)

#### Model Management

There are two models in the workflow available on Hugging Face:
- [GR00T N1 with Cosmos](https://huggingface.co/nvidia/Liver_Scan_Gr00t_Cosmos_Rel)
- [PI0 with Cosmos](https://huggingface.co/nvidia/Liver_Scan_Pi0_Cosmos_Rel)

Model retrieval is done automatically when running the workflow. You can also download the models manually from Hugging Face.

### 🔧 Environment Configuration

#### Required Environment Variables
```bash
# Python module resolution
export PYTHONPATH=<i4h-workflows-root>/workflows/robotic_ultrasound/scripts:<i4h-workflows-root>

# DDS communication middleware
export RTI_LICENSE_FILE=<path-to-rti-license>

# Optional: CUDA runtime optimization
export CUDA_VISIBLE_DEVICES=0  # Single GPU selection
```

</details>

---

## 🛠️ Troubleshooting

### ⚠️ Common Integration Issues

#### 🔧 Dependency Resolution Conflicts
```bash
ERROR: pip's dependency resolver does not currently take into account all the packages...
isaaclab 0.34.9 requires torch==2.5.1, but you have torch 2.6.0 which is incompatible.
```
**Resolution**: These PyTorch version conflicts are expected and non-blocking. The workflow maintains compatibility across version differences.

#### 🔗 Module Import Resolution
**Symptoms**: `ModuleNotFoundError` or `Error while finding module specification for 'xxx'` during script execution
**Resolution**: Verify `PYTHONPATH` includes both `scripts/` directory and repository root.

#### 🔧 PI0 model fails to load
**Symptoms**: When using the PI0 policy for the scanning task, the model fails to load, or Isaac Sim appears stuck on the loading screen.
**Resolution**: Verify that your machine can reach Google APIs (`googleapis.com`). Although the model is hosted on Hugging Face, additional files are fetched from Google during initialization.

### 🆘 Support Resources

- **Technical Documentation**: Component-specific README files
- **Issue Tracking**: [GitHub Issues](https://github.com/isaac-for-healthcare/i4h-workflows/issues)
