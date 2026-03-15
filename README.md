# HANDS: 7 DOF Robotic Arm Control System

Real-time control software for a 7-degree-of-freedom robotic arm with kinematics, dynamics simulation, trajectory planning, and hardware integration.

## 🎯 Project Goal

Build a **production-ready, real-time control system** for a 7 DOF robotic arm with:
- ✅ Real-time motor control (100-500 Hz control loops)
- ✅ Forward/Inverse Kinematics solvers
- ✅ Dynamics simulation with PyBullet
- ✅ Trajectory planning and path planning
- ✅ Collision detection and avoidance
- ✅ 3D visualization and monitoring
- ✅ Hardware abstraction layer
- ✅ Safety systems and limits enforcement

**Target**: <10ms end-to-end latency for motion commands

---

## 📊 Implementation Progress

### Phase 1: Kinematics Foundation
- [ ] Define 7 DOF arm parameters (DH parameters)
- [ ] **Forward Kinematics** - Compute end-effector from joint angles
- [ ] **Inverse Kinematics** - Solve joint angles from end-effector pose
- [ ] Visualization utilities
- [ ] Unit tests for singularity handling

### Phase 2: Dynamics & Simulation
- [ ] URDF model definition
- [ ] PyBullet simulation environment setup
- [ ] Dynamics calculations (inertia, gravity, torques)
- [ ] Collision detection framework
- [ ] Sim-to-reality validation

### Phase 3: Trajectory Planning
- [ ] Joint-space trajectory generation (smooth interpolation)
- [ ] Cartesian path planning
- [ ] Obstacle avoidance (RRT*, potential fields)
- [ ] Time-optimal trajectory optimization
- [ ] Trajectory validation and smoothing

### Phase 4: Real-Time Control
- [ ] Motor hardware interface abstraction
- [ ] PID controllers for each joint
- [ ] Feedforward compensation
- [ ] Safety limit enforcement (velocity, acceleration, torque)
- [ ] Emergency stop system
- [ ] Command logging and telemetry

### Phase 5: Integration & Testing
- [ ] End-to-end integration tests
- [ ] Performance profiling and benchmarking
- [ ] Hardware validation
- [ ] Demo applications
- [ ] Documentation completion

---

## 📁 Project Structure

```
hands/
├── .github/
│   └── copilot-instructions.md      # Development guide (THIS FILE)
│
├── docs/                            # Documentation
│   ├── DESIGN.md                    # Architecture decisions
│   ├── KINEMATICS.md                # Math and algorithms
│   └── API.md                       # API reference
│
├── src/
│   ├── arm/
│   │   ├── __init__.py
│   │   ├── arm.py                   # Main Arm class (orchestrator)
│   │   ├── kinematics.py            # FK/IK solvers
│   │   ├── dynamics.py              # Physics model
│   │   └── trajectory.py            # Trajectory generation
│   │
│   ├── control/
│   │   ├── __init__.py
│   │   ├── motor_interface.py       # Hardware abstraction
│   │   ├── controller.py            # PID and motion controllers
│   │   └── safety.py                # Safety checks and limits
│   │
│   ├── planning/
│   │   ├── __init__.py
│   │   ├── path_planner.py          # RRT*, potential fields
│   │   ├── collision.py             # Collision detection
│   │   └── utils.py                 # Planning utilities
│   │
│   ├── simulation/
│   │   ├── __init__.py
│   │   ├── simulator.py             # PyBullet wrapper
│   │   ├── visualization.py         # 3D visualization
│   │   └── urdf_loader.py           # URDF utilities
│   │
│   └── utils/
│       ├── __init__.py
│       ├── config.py                # Configuration management
│       ├── math_helpers.py          # Matrix/rotation utilities
│       └── logging.py               # Logging setup
│
├── tests/                           # Unit tests
│   ├── test_kinematics.py
│   ├── test_dynamics.py
│   ├── test_control.py
│   └── test_planning.py
│
├── examples/                        # Demo scripts
│   ├── demo_kinematics.py           # FK/IK example
│   ├── sim_trajectory.py            # Simulation demo
│   └── hardware_test.py             # Hardware integration
│
├── models/                          # URDF and configs
│   ├── arm_config.yaml              # DH parameters, limits
│   ├── arm.urdf                     # URDF model
│   └── assets/                      # 3D meshes
│
├── requirements.txt                 # Python dependencies
├── setup.py                         # Package setup
├── pytest.ini                       # Test config
├── mypy.ini                         # Type checking config
├── README.md                        # THIS FILE - UPDATE with each phase
└── LICENSE

```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- Windows 10/11 or Linux

### Installation

```bash
# Clone repository
cd c:\Users\gowth\Documents\GitHub\hands

# Create virtual environment
python -m venv venv
.\venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Install package in development mode
pip install -e .
```

### Quick Start

```python
from hands.arm import SevenDOFArm
import numpy as np

# Initialize arm (loaded from YAML config)
arm = SevenDOFArm(config_file='models/arm_config.yaml')

# Forward kinematics
q = np.array([0, 0, 0, 0, 0, 0, 0])  # Zero configuration
T = arm.forward_kinematics(q)
print(f"End-effector position: {T[:3, 3]}")

# Inverse kinematics
target_pose = arm.get_current_tcp()  # Get current TCP
q_solution = arm.inverse_kinematics(target_pose)
print(f"Joint angles for target: {q_solution}")
```

---

## 📦 Dependencies

| Package | Purpose | Version |
|---------|---------|---------|
| numpy | Numerical computing | >= 1.21 |
| scipy | Scientific computing | >= 1.7 |
| pybullet | Physics simulation | >= 3.1 |
| ikpy | Kinematics solvers | >= 2.3 |
| pyyaml | Configuration files | >= 5.4 |
| matplotlib | Plotting/visualization | >= 3.4 |
| pytest | Testing framework | >= 6.2 |
| mypy | Type checking | >= 0.9 |

Install all: `pip install -r requirements.txt`

---

## 🔧 Development Workflow

### Before Starting Work
1. Read [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for coding standards
2. Check the **Implementation Progress** section above
3. See [docs/DESIGN.md](docs/DESIGN.md) for architecture

### When Adding a Feature
1. **Plan**: Define requirements and testing strategy
2. **Implement**: Follow guidelines in copilot-instructions.md
3. **Test**: Run `pytest tests/` - must pass
4. **Profile**: Check performance with `cProfile` if time-critical
5. **Document**: Update this README.md and code docstrings
6. **Update**: Move checkbox in Progress section to ✅

### Example: After Implementing Forward Kinematics
```bash
# Run tests
pytest tests/test_kinematics.py -v

# Profile
python -m cProfile -s cumtime examples/demo_kinematics.py

# Update README (mark as complete)
# - [ ] Forward Kinematics → - ✅ Forward Kinematics [1.2ms avg]
```

---

## 🧪 Testing & Quality

### Run Tests
```bash
# All tests
pytest tests/ -v

# Specific component
pytest tests/test_kinematics.py -v

# With coverage
pytest tests/ --cov=src --cov-report=html
```

### Type Checking
```bash
mypy src/ --strict
```

### Performance Profiling
```bash
# Profile a script
python -m cProfile -s cumtime examples/demo_kinematics.py

# Line-by-line profiling (install: pip install line_profiler)
kernprof -l -v examples/demo_kinematics.py
```

---

## 📚 Key Concepts

### Forward Kinematics (FK)
Maps joint angles → end-effector pose (position + orientation)
- **Input**: Joint angles q = [q₁, q₂, ..., q₇]
- **Output**: 4×4 homogeneous transformation matrix
- **Time target**: <1ms (critical for control loop)

### Inverse Kinematics (IK)
Maps end-effector pose → joint angles
- **Input**: Target pose (TCP position + orientation)
- **Output**: Joint angles q or null if unreachable
- **Challenge**: 7 DOF = redundant (infinite solutions possible)
- **Time target**: <10ms (can be offline but useful online)

### Trajectory Planning
Generate smooth, collision-free paths between poses
- **Joint-space**: Smooth polynomials through waypoints
- **Cartesian-space**: Linear paths in 3D
- **Obstacles**: RRT*, potential fields for avoidance

### Real-Time Control Loop
```
[100-500 Hz] → Read sensors → Compute control → Send commands → Repeat
```
- **Latency target**: <10ms total
- **Jitter**: Minimize for smooth motion
- **Safety**: Hard limits on velocity/acceleration/torque

---

## 🔌 Hardware Integration

### Motor Interface
The `src/control/motor_interface.py` abstracts hardware:
```python
from hands.control import MotorInterface

# Simulated or real hardware
motors = MotorInterface(hardware='simulation')  # or 'real'

# Send commands
motors.set_joint_angles(q_desired, velocity=0.5)
motors.read_sensors()  # Get current state
```

### Supported Hardware (Planned)
- [ ] Serial communication (Dynamixel, MAXON)
- [ ] ROS interface
- [ ] SimulationLinks (PyBullet)

---

## 📈 Performance Targets

| Metric | Target | Status |
|--------|--------|--------|
| Control loop rate | 100-500 Hz | TBD |
| FK calculation | <1ms | TBD |
| IK solving | <10ms | TBD |
| Collision check | <5ms | TBD |
| Path planning | <100ms | TBD |
| Total latency | <10ms | TBD |

---

## 🐛 Troubleshooting

### Import Errors
```bash
pip install -e .
```

### Testing Failures
Check Python version: `python --version` (need 3.8+)

### Performance Issues
Profile with: `python -m cProfile -s cumtime script.py`

---

## 📖 Documentation

- **[DESIGN.md](docs/DESIGN.md)**: Architecture and design decisions
- **[KINEMATICS.md](docs/KINEMATICS.md)**: Mathematical background
- **[API.md](docs/API.md)**: Full API reference
- **[copilot-instructions.md](.github/copilot-instructions.md)**: Developer guide

---

## 🤝 Contributing

Follow the workflow in [`.github/copilot-instructions.md`](.github/copilot-instructions.md):
1. Plan before coding
2. Implement with type hints and docstrings
3. Write tests (unit tests required)
4. Profile performance
5. Update documentation and this README
6. Commit with clear messages

---

## 📝 Recent Updates

**[To be maintained - update after each feature]**

- **Phase 1**: Initial project structure and documentation
  - ✅ Created project scaffolding
  - ✅ Defined architecture
  - ⏳ Started kinematics implementation

---

## 📄 License

See [LICENSE](LICENSE) file.

---

## 👤 Contact

For issues or questions: [Update with contact info]

---

**Last Updated**: [Update with date when changes made]
**Current Phase**: 1 - Kinematics Foundation
**Next Milestone**: Complete FK/IK solvers with tests
