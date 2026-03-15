# HANDS: Dual 3D-Printed Hand Towel Folding Robot

Real-time control software for a dual-hand towel-folding robot with AI-powered perception and autonomous motion execution.

## 🎯 Project Goal

Build **dual 3D-printed robotic hands** that can:
- **Detect garments** using cameras (Raspberry Pi AI camera + RGB-D depth)
- **Estimate cloth pose** with 3D vision (OpenCV + YOLO + depth estimation)
- **Execute folding motion** with human-like smoothness and precision
- **Achieve >95% reliable clean folds** every time
- **Operate autonomously** with trained AI agents (imitation learning + reinforcement learning)

### Key Specifications
- **Hands**: 2x identical 7-DOF 3D-printed hands (14 DOF total)
- **Actuators**: MG996R servo motors (7 per hand, 14 total)
- **Cameras**: 1x Raspberry Pi AI camera + 1x RGB-D depth camera
- **Target Speed**: Fold time <30 seconds per towel
- **Reliability**: >95% successful clean folds
- **Control Latency**: <50ms end-to-end (perception + decision + motion)

### Success Metrics
| Metric | Target | Status |
|--------|--------|--------|
| Single hand kinematics | <1ms | ⏳ In progress |
| Servo response time | <100ms | ⏳ In progress |
| Towel detection accuracy | >98% | ⏳ In progress |
| Fold success rate | >95% | ⏳ In progress |
| Fold time | <30s | ⏳ In progress |
| Dual-arm synchronization | <20ms skew | ⏳ In progress |

---

## 📋 Requirements

### Hardware (What You Have)

**Hands:**
- 2x identical 3D-printed hands
- 7 DOF per hand (14 total DOF)
- Design files: `models/hand_*.stl` or CAD files

**Actuators:**
- **14x MG996R servo motors** (7 per hand)
  - Voltage: 4.8-6V
  - Speed: 0.16 sec/60° (4.8V)
  - Torque: 9 kg·cm (4.8V)
  - Control: PWM (1000-2000 µs pulse width)
  - Feedback: Built-in potentiometer

**Vision System:**
- **1x Raspberry Pi AI Camera Module** (or equivalent)
  - 12MP, fixed focus
  - Purpose: RGB garment detection
- **1x RGB-D Depth Camera** (supported: Intel RealSense D435, Azure Kinect, or Luxonis OAK)
  - Purpose: Depth estimation for pose detection

**Compute & Control:**
- **Raspberry Pi 4** (8GB RAM) or **Jetson Nano** (for vision + inference)
- **Arduino** (for PWM servo control) or direct GPIO control
- **USB hub** (for camera and depth sensor)
- **Power bank or 5V/2A+ supply** (for Raspberry Pi)
- **Servo power supply**: 6V/10A+ (for 14 servos)

**Optional:**
- Laundry basket with towel feeder
- Workspace mat (for stable cloth positioning)
- LED ring light (for consistent vision)
- Force sensors on fingertips (future: force feedback)

### Software Requirements

**Core Dependencies:**
- **Python 3.8+** (3.10+ recommended)
- **numpy, scipy** - Kinematics and math
- **OpenCV** - Vision and image processing
- **PyYAML** - Configuration
- **PySerial / RPi.GPIO** - Servo control

**Vision & AI:**
- **YOLO** - Object detection (towel detection)
- **MediaPipe** - Hand pose estimation (optional alternative)
- **TensorFlow/PyTorch** - AI training (Phase 5)
- Depth camera SDK (e.g., `pyrealsense2` for RealSense)

**Development:**
- **pytest** - Testing
- **jupyter** - Prototyping
- **cProfile** - Performance profiling

**Installation:**
```bash
pip install -r requirements.txt
```

### System Requirements

- **OS**: Raspberry Pi OS (Bullseye+), Ubuntu 20.04+, or macOS
- **Compute**: RPi 4 (8GB) or equivalent (Jetson, desktop PC)
- **RAM**: 4GB minimum, 8GB+ for vision inference
- **Disk**: 1GB for code + models
- **Power**: Dedicated servo power supply (6V/10A+)
- **Network**: USB connection or WiFi for RPi to desktop (for development)

---

## 📈 Project Status & Progress

**Current Phase**: Phase 1 - Single Hand Kinematics  
**Status**: Hardware assembled, software framework starting  
**Last Updated**: 15 Mar 2026

### Development Roadmap

These phases follow the intermediate goals in order:

| Phase | Goal | Status | Est. Timeline |
|-------|------|--------|---------------|
| **1** | Single hand kinematics + timing | ⏳ Starting | Week 1-2 |
| **2** | Servo control + PID tuning | ⏳ Planned | Week 2-3 |
| **3** | Camera integration + pose estimation | ⏳ Planned | Week 3-4 |
| **4** | Dual-hand synchronization | ⏳ Planned | Week 4-5 |
| **5** | AI agents + autonomous folding | ⏳ Planned | Week 5-8 |

**See [IMPLEMENTATION.md](IMPLEMENTATION.md)** for detailed checklist of each phase.

### Recent Updates
- **15 Mar 2026**: Project setup and hardware inventory
  - ✅ 3D-printed hands assembled (7 DOF each)
  - ✅ 14x MG996R servos ready
  - ✅ Camera hardware acquired
  - ⏳ Starting Phase 1: Kinematics implementation

---

## � Quick Start

### Prerequisites

- Python 3.8+ installed
- Windows, Linux, or macOS
- Basic robotics knowledge (DH parameters, forward/inverse kinematics)

### Installation

```bash
# Clone repository
git clone https://github.com/yourusername/hands.git
cd hands

# Create virtual environment
python -m venv venv
.\venv\Scripts\activate        # Windows
# source venv/bin/activate     # Linux/macOS

# Install dependencies
pip install -r requirements.txt

# Install HANDS package in development mode
pip install -e .
```

### How to Use

#### 1. Control a Single Hand with Kinematics

```python
from hands.arm import Hand
import numpy as np

# Initialize left hand
left_hand = Hand(
    hand_id='left',
    dh_config='models/hand_left_dh.yaml',
    servo_ports=['/dev/ttyUSB0']  # Or COM3 on Windows
)

# Set home position
home_angles = np.array([90, 45, 0, 90, 0, 45, 90])  # degrees
left_hand.move_to_angles(home_angles)

# Compute forward kinematics (where is the hand?)
fingertip_pose = left_hand.forward_kinematics(home_angles)
print(f"Fingertip position: {fingertip_pose[:3, 3]}")

# Inverse kinematics (move to target position)
target_xyz = np.array([0.15, 0.05, 0.12])  # x, y, z in meters
joint_angles = left_hand.inverse_kinematics(target_xyz)
if joint_angles is not None:
    left_hand.move_to_angles(joint_angles)
else:
    print("Target unreachable")
```

#### 2. Detect Towel with Camera

```python
from hands.vision import TowelDetector
import cv2

# Initialize detector
detector = TowelDetector(model='yolov5')  # Uses YOLO for detection

# Capture frame from RGB camera
cap = cv2.VideoCapture(0)  # Raspberry Pi Camera on /dev/video0
ret, frame = cap.read()

# Detect towel
towel_bbox, confidence = detector.detect_towel(frame)
print(f"Towel detected with {confidence*100:.1f}% confidence")

# Estimate 3D pose from RGB-D camera
from hands.vision import PoseEstimator
pose_est = PoseEstimator(depth_camera='realsense')  # or 'kinect', 'oak'
pose_3d = pose_est.estimate_towel_pose(frame, depth_frame)
print(f"Towel pose: {pose_3d}")
```

#### 3. Execute Simple Fold Sequence

```python
from hands.arm import Hand, DualArmCoordinator
import numpy as np

# Initialize two hands
left = Hand('left', '/dev/ttyUSB0')
right = Hand('right', '/dev/ttyUSB1')

# Create coordinator
coord = DualArmCoordinator(left, right)

# Predefined folding sequence
sequence = [
    {'left': [90, 45, 0, 90, 0, 45, 90], 'right': [90, 45, 0, 90, 0, 45, 90]},  # Spread hands
    {'left': [70, 60, 10, 80, 15, 50, 100], 'right': [110, 50, 350, 100, 345, 40, 80]},  # Grab sides
    # ... more motion waypoints
]

# Execute synchronized motion
for step in sequence:
    coord.move_synchronized(
        left_angles=step['left'],
        right_angles=step['right'],
        speed=0.5  # 0-1 scale
    )
    print(f"Step {sequence.index(step)} complete")
```

#### 4. Full Autonomous Folding (Phase 5)

```python
from hands.ai import AutonomousFoldingAgent
from hands.vision import TowelDetector, PoseEstimator
from hands.arm import DualArmCoordinator

# Load trained agent
agent = AutonomousFoldingAgent(model_path='models/folding_agent_v1.pt')

# Initialize system
detector = TowelDetector()
pose_est = PoseEstimator()
coord = DualArmCoordinator(left, right)

# Run autonomous folding
while True:
    # 1. Detect towel
    frame, depth = get_sensor_data()
    towel_pose = detector.detect_and_estimate_pose(frame, depth)
    
    if towel_pose is None:
        print("No towel detected")
        continue
    
    # 2. AI decides action
    action = agent.decide_action(towel_pose)  # Returns joint angles for both hands
    
    # 3. Execute motion
    coord.move_synchronized(
        left_angles=action['left'],
        right_angles=action['right']
    )
    
    # Check if fold is complete
    if agent.is_fold_complete(get_current_pose()):
        print("Fold successful!")
        break
```

---

## 📁 Project Structure

```
hands/
├── .github/
│   └── copilot-instructions.md          # Development guide + workflows
│
├── docs/                                # Documentation
│   ├── DESIGN.md                       # System architecture
│   ├── KINEMATICS.md                   # 7-DOF hand kinematics
│   ├── VISION.md                       # Camera calibration + pose estimation
│   └── AI.md                           # Imitation learning + RL training
│
├── src/
│   ├── arm/                            # Hand kinematics & control
│   │   ├── hand.py                    # Single hand class (left/right)
│   │   ├── kinematics.py              # 7-DOF FK/IK
│   │   ├── trajectory.py              # Motion planning
│   │   └── dual_coordinator.py        # Sync 2 hands
│   │
│   ├── control/                        # Motor & servo control
│   │   ├── servo_controller.py        # MG996R PWM control
│   │   ├── pid_tuner.py               # PID tuning tools
│   │   └── safety.py                  # Limits + e-stop
│   │
│   ├── vision/                         # Camera + pose detection
│   │   ├── towel_detector.py          # YOLO detection
│   │   ├── pose_estimator.py          # Depth + 3D pose
│   │   └── camera_calib.py            # Camera calibration
│   │
│   ├── ai/                             # AI agents (Phase 5)
│   │   ├── imitation_agent.py         # Behavioral cloning
│   │   ├── rl_agent.py                # Reinforcement learning
│   │   └── training_utils.py          # Data collection + training
│   │
│   └── utils/
│       ├── config.py                  # Config management
│       ├── math_helpers.py            # Transformations + rotations
│       └── logging.py                 # Logging setup
│
├── tests/
│   ├── test_kinematics.py
│   ├── test_servo_control.py
│   ├── test_vision.py
│   └── test_dual_sync.py
│
├── examples/
│   ├── single_hand_demo.py             # Basic single-hand control
│   ├── dual_hand_demo.py               # Synchronized 2-hand motion
│   ├── towel_detection_demo.py         # Camera + detection
│   ├── calibration_tool.py             # Hand calibration
│   └── manual_folding.py               # Manual control interface
│
├── models/
│   ├── hand_left_dh.yaml               # Left hand DH parameters
│   ├── hand_right_dh.yaml              # Right hand DH parameters
│   ├── servo_limits.yaml               # Servo angle limits
│   ├── folding_agent_v1.pt             # Trained AI model (Phase 5)
│   └── hand_*.stl                      # 3D print files
│
├── data/                               # Data for training
│   ├── demonstrations/                 # Human folding videos
│   └── trajectories/                   # Recorded motion sequences
│
├── requirements.txt
├── setup.py
├── README.md                           # This file
├── IMPLEMENTATION.md                   # Detailed progress
└── LICENSE
```

---

## 🧪 Testing & Development

### Run Unit Tests
```bash
pytest tests/ -v
```

### Type Checking
```bash
mypy src/ --strict
```

### Performance Profiling
```bash
python -m cProfile -s cumtime examples/demo_kinematics.py
```

### View Test Coverage
```bash
pytest tests/ --cov=src --cov-report=html
```

---

## 📖 Documentation

- **[README.md](README.md)** - This file: overview, requirements, quick start
- **[IMPLEMENTATION.md](IMPLEMENTATION.md)** - Detailed progress checklist for all phases
- **[.github/copilot-instructions.md](.github/copilot-instructions.md)** - Developer guide with workflows
- **[docs/DESIGN.md](docs/DESIGN.md)** - Architecture and design decisions  
- **[docs/KINEMATICS.md](docs/KINEMATICS.md)** - Mathematical background
- **[docs/API.md](docs/API.md)** - Complete API reference

---

## 🤝 Contributing

Follow the development workflow:

1. **Read** [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for guidelines
2. **Pick** the appropriate workflow template for your task
3. **Implement** + test + profile 
4. **Update** both `README.md` and `IMPLEMENTATION.md` with progress
5. **Commit** with semantic message: `[Phase X] Feature: Name - Performance: Xms`

For detailed contribution guidelines, see [`.github/copilot-instructions.md`](.github/copilot-instructions.md).

---

## ⚠️ Safety & Calibration Checklist

### Before Folding Real Towels:

**Servo Calibration:**
- [ ] **Servo centers zeroed** - Run `python examples/calibration_tool.py --servo_centers` (set all 14 motors to 1500 µs central pulse)
- [ ] **Servo limits set** - Configure angle limits for each joint (**Must match 3D-printed joint stops**)
- [ ] **Servo jitter tested** - MG996R exhibits ±2-3° oscillation at rest (normal for budget servos; PID tuning mitigates)
- [ ] **Power supply stable** - 6V ±0.5V under full load (14 servos × 0.5A typical = 7A worst-case)

**Camera Calibration:**
- [ ] **RPi AI Camera intrinsics** - Run `python examples/calibration_tool.py --camera_intrinsic` (store in models/camera_intrinsic.npy)
- [ ] **RGB-D extrinsic calibration** - Run `python examples/calibration_tool.py --camera_extrinsic` (relative pose between RGB and depth cameras)
- [ ] **Depth camera depth-to-meters** - Verify depth scale constant for your camera model (e.g., RealSense: 0.001 m/unit)

**Dual-Hand Synchronization:**
- [ ] **Serial comm stress test** - Run `python tests/test_servo_control.py` (all 14 motors respond within 50ms)
- [ ] **Timestamp drift** - Monitor sync error over 5 minute run; must stay <20ms skew

**Safety & E-Stop:**
- [ ] **E-stop wired to GPIO** - Test e-stop halts all motors within 100ms
- [ ] **Joint limit enforcement active** - Servo position never exceeds ±π radians
- [ ] **Velocity ramping enabled** - No sudden jerky movements (start <20°/s)
- [ ] **Workspace clear** - No objects within 0.5m of hand workspace during calibration

### Known Limitations & Workarounds:

- **MG996R Budget Servos**: 
  - ±2-3° jitter at rest due to internal feedback resolution
  - **Workaround**: Use PID closed-loop + smoothing filter (running average over 3 samples)
  - **Not suitable for**: High-precision pick-and-place; fine for towel folding
  
- **No force/torque feedback** (Phase 5 upgrade with load cells):
  - **Current workaround**: Time-based motion (fixed duration per fold segment)
  - **Future**: Add load cells to detect cloth resistance
  
- **RPi AI camera limited FPS** (max ~30 FPS at full resolution):
  - **Workaround**: Run detection at 15 FPS, interleave with servo commands for <10ms control jitter

---

## 📞 Support & Issues

- **Bugs**: Report in GitHub Issues with reproduction steps
- **Questions**: Check [docs/KINEMATICS.md](docs/KINEMATICS.md) or [docs/API.md](docs/API.md)
- **Contributions**: See Contributing section above

---

## 📊 Performance Targets

See [IMPLEMENTATION.md](IMPLEMENTATION.md#-performance-metrics) for current performance metrics and actual measured times.

---

## 📄 License

See [LICENSE](LICENSE) file.

---

## 📅 Project Status

| Metric | Status |
|--------|--------|
| **Last Updated** | March 15, 2026 |
| **Current Phase** | 1 - Single Hand Kinematics |
| **Hardware** | 2x hands + 14x servos + 2x cameras ✅ Ready |
| **Phase Completion** | 0/5 (0%) |
| **Estimated Timeline** | 8 weeks to full autonomous folding |
| **Next Milestone** | Single hand IK solver (by Mar 20) |

**Next Steps:**
1. 🎯 Define DH parameters for 3D-printed hands (models/hand_left_dh.yaml, models/hand_right_dh.yaml)
2. 🎯 Implement forward & inverse kinematics (src/arm/kinematics.py)
3. 🎯 Basic servo PWM control (src/control/servo_controller.py)
4. 🎯 Semi-autonomous folding demo with manual towel targeting

For detailed implementation roadmap, see [IMPLEMENTATION.md](IMPLEMENTATION.md).
