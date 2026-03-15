# HANDS Project - Copilot Coding Guide

## Project Vision
Build a **real-time control system for a 7 DOF robotic arm** with kinematics, dynamics simulation, trajectory planning, and hardware integration.

### End Goal
- **Real-time control** of 7-axis robotic arm with sub-millisecond response times
- **Full software stack**: simulation, planning, control, and hardware interface
- **Production-ready**: safe, tested, and documented

---

## Architecture Overview

```
┌─────────────────────────────────────────┐
│      Application Layer (Examples)       │
├─────────────────────────────────────────┤
│    Planning & Control (High Level)      │
│  - Trajectory generation & planning     │
│  - Path planning & collision detection  │
├─────────────────────────────────────────┤
│  Arm Core (Mid Level - TIME CRITICAL)   │
│  - Forward/Inverse Kinematics           │
│  - Dynamics model                       │
│  - Real-time state management           │
├─────────────────────────────────────────┤
│  Control & Hardware (Low Level)         │
│  - Motor control & PID loops            │
│  - Safety checks                        │
│  - Hardware interface (serial/GPIO)     │
├─────────────────────────────────────────┤
│  Simulation & Visualization             │
│  - PyBullet physics                     │
│  - 3D rendering                         │
└─────────────────────────────────────────┘
```

---

## Development Guidelines

### 1. Performance & Real-Time Constraints
- **Core arm module** (kinematics, dynamics): target <5ms compute time
- **Control loop**: 100-500 Hz (10-2ms cycles)
- **Path planning**: can be offline or slow (background thread)
- Always profile with `timeit` and `cProfile` before optimization
- Use `numpy` vectorization over Python loops

### 2. Code Organization
- **`src/arm/`**: Core algorithms, keep lightweight & vectorized
- **`src/control/`**: Hardware abstraction, keep time-critical
- **`src/planning/`**: Path planning (can be slower)
- **`src/simulation/`**: Visualization, for testing/development only
- **`tests/`**: Unit tests for kinematics, dynamics, control logic

### 3. Key Principles
- **DRY (Don't Repeat Yourself)**: Configuration in one place (YAML/dataclass)
- **Separation of Concerns**: Simulation ≠ Real hardware control
- **Type Hints**: Use throughout for clarity and IDE support
- **Documentation**: Docstrings for all public methods (especially FK/IK)
- **Testing**: Unit tests for math-heavy code (kinematics, collision detection)

---

## Effective Prompting Strategy

### When Adding Features

**Structure your prompts like this:**

```
Context: [What module/layer] - For real-time [component] in 7 DOF arm
Current: [What exists now]
Goal: [Specific implementation goal]
Constraints: [Performance/safety/accuracy requirements]
Libraries: [Allowed libraries - numpy, scipy, ikpy, etc.]
```

### Example Prompts

**For Kinematics:**
```
Context: Implementing Forward Kinematics for 7 DOF arm - time-critical path
Goal: Calculate end-effector position/orientation from 7 joint angles (q)
Constraints: Must complete in <1ms, use DH parameters, output 4x4 matrix
Libraries: numpy only, no symbolic math
```

**For Control:**
```
Context: Real-time motor control interface for hardware integration
Goal: Implement PID controller with feedforward for smooth motion
Constraints: 100Hz control loop, safety bounds on max velocity/acceleration
Libraries: numpy, scipy optional for filtering
```

**For Path Planning:**
```
Context: Offline motion planning with collision avoidance
Goal: Generate smooth trajectory through waypoints avoiding obstacles
Constraints: Must check collisions with PyBullet, return joint-space trajectory
Libraries: numpy, scipy.interpolate, PyBullet physics engine
```

---

## Implementation Workflow

### Step 1: Define & Document (Before Coding)
- [ ] Write docstring with parameters, return values, math equations
- [ ] Define DH parameters or config in YAML
- [ ] List unit tests needed
- [ ] Note performance targets

### Step 2: Implement Core Algorithm
- [ ] Vectorize with numpy first (no loops if possible)
- [ ] Keep functions pure (no side effects)
- [ ] Use type hints
- [ ] Add inline comments for complex math

### Step 3: Integrate & Test
- [ ] Write unit tests (test edge cases, singularities for IK)
- [ ] Profile performance with `timeit`
- [ ] Validate against known benchmarks (compare with Drake, IKPy if relevant)
- [ ] Update README.md with completion status

### Step 4: Document in Code & README
- [ ] Update README.md with new feature
- [ ] Add example in `examples/` folder
- [ ] Document any assumptions or limitations

---

## Module-Specific Guidelines

### `src/arm/kinematics.py` - FK & IK
- **Input validation**: Check joint limits, handle singularities
- **Vectorization**: All computed matrices should be numpy arrays
- **Testing**: Test with known arm configurations
- **Performance**: Target <1ms for FK/IK of 7 DOF

### `src/arm/dynamics.py` - Physics Model
- **Inertia matrices**: Pre-compute or load from URDF
- **Gravity effects**: Model properly for real hardware
- **Validation**: Compare with PyBullet simulation
- **Performance**: <5ms for dynamics calculations

### `src/control/motor_interface.py` - Hardware Abstraction
- **Abstraction layer**: Work with simulated + real hardware seamlessly
- **Logging**: Track all commands sent to motors
- **Safety**: Never let commands violate hardware limits
- **Real-time**: Minimize memory allocation in control loop

### `src/planning/path_planner.py` - Motion Planning
- **Can be slow**: Run in background thread/process if needed
- **Validation**: Always check collision-free result
- **Testing**: Use simple obstacle scenarios first
- **Output format**: Joint-space trajectories (q over time)

### `src/simulation/simulator.py` - PyBullet Environment
- **Mirror reality**: URDF must match actual hardware
- **Logging**: Record all trajectories for analysis
- **Visualization**: Show reference trajectory vs actual
- **Not for control**: Don't mix simulation with real hardware

---

## Safety & Validation Checklist

- [ ] Joint limits enforced (software + hardware)
- [ ] Velocity limits checked before sending commands
- [ ] Acceleration limits reasonable
- [ ] Collision detection active in planning
- [ ] Emergency stop available
- [ ] Log all commands for debugging
- [ ] Singularity handling in IK
- [ ] Numerical stability in matrix operations

---

## Performance Targets

| Component | Target Time | Priority |
|-----------|------------|----------|
| FK calculation | <1ms | Critical |
| IK solving | <10ms | High |
| Control loop cycle | 2-10ms | Critical |
| Path planning | <100ms each | Medium |
| Collision checking | <5ms | High |
| Trajectory generation | <50ms | Medium |

---

## Recommended Prompt Templates

### For Bug Fixes
```
Bug: [Specific behavior]
Expected: [What should happen]
Actual: [What's happening]
Error: [Stack trace or symptoms]
Affected component: [Module in src/]
```

### For Performance Issues
```
Performance issue: [Component/function]
Current time: [Measured latency]
Target time: [Goal latency]
Profiling data: [cProfile output if available]
Constraints: [Memory/CPU limits]
```

### For New Features
```
Feature: [High-level goal]
Module: [Where to implement]
Dependencies: [Other modules needed]
Testing strategy: [How to validate]
Integration point: [Where it connects]
```

---

## CI/CD Integration Notes

When code is updated:
1. Run `pytest tests/` - ensure all tests pass
2. Run `mypy src/` - type checking
3. Profile key components - ensure performance targets
4. Update README.md with new capability
5. Update documentation if APIs changed

---

## Quick Reference: Project Commands

```bash
# Setup
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt

# Testing
pytest tests/ -v

# Type checking
mypy src/

# Run examples
python examples/demo_kinematics.py
python examples/sim_trajectory.py

# Profile
python -m cProfile -s cumtime examples/demo_kinematics.py
```

---

## Key Files to Understand First

1. **`src/arm/arm.py`** - Main 7 DOF arm class, orchestrates all sub-modules
2. **`src/arm/kinematics.py`** - FK/IK solvers (time-critical)
3. **`models/arm_config.yaml`** - DH parameters and hardware limits
4. **`examples/demo_kinematics.py`** - Simple usage example
5. **`README.md`** - Project status and current progress

---

## Documentation Standards

- Every public function has docstring (Google style)
- Math equations documented with LaTeX comments
- Performance characteristics noted
- Known limitations/TODOs listed
- Examples in docstrings for complex functions

Example:
```python
def forward_kinematics(joint_angles: np.ndarray) -> np.ndarray:
    """
    Compute forward kinematics for 7 DOF arm.
    
    Uses DH parameter convention. Computes T_{0→7} transformation matrix.
    
    Args:
        joint_angles: Joint angles q = [q1, q2, ..., q7] in radians (nx7)
        
    Returns:
        T: Homogeneous transformation matrix 4x4 (or Nx4x4 for batch)
        
    Performance: O(1), ~0.5ms for single calculation, vectorized
    
    Raises:
        ValueError: If joint_angles has wrong shape
        
    Example:
        >>> q = np.array([0, 0, 0, 0, 0, 0, 0])
        >>> T = forward_kinematics(q)
        >>> print(T.shape)  # (4, 4)
    """
```

---

## Status & End Goal

**Current**: Foundation phase
**Target**: Production-ready 7 DOF arm control system
**Next**: See README.md for implementation progress
