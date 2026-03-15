# HANDS Project - Copilot Coding Guide

## 🎯 Project Identity & Direction

### What This Is
**HANDS** is a **professional-grade, production-ready real-time control system** for 7 DOF robotic arms.

**Status**: Phase 1 - Kinematics Foundation (see README.md for detailed progress)  
**Scope**: Commercial deployment with sub-millisecond control loop latency  
**Domain**: Robotics - kinematics, dynamics, trajectory planning, hardware control  
**Language**: Python 3.8+ with advanced developer audience (assume numpy/scipy proficiency)

### Project Intent
- Build **plug-and-play arm control** that abstracts hardware details
- Enable **real-time motion execution** at 100-500 Hz with <10ms latency
- Provide **safe, validated, tested** code suitable for production deployment
- Match or exceed performance of commercial robotic middleware (ROS, Drake)

### Where to Find Context
- **README.md**: Live project status, implementation checklist, performance metrics
- **docs/DESIGN.md**: Architecture, module responsibilities, data flow
- **models/arm_config.yaml**: Hardware parameters (DH, joint limits, motor specs)
- **Current implementation phase**: Check README.md "Current Phase" at bottom

---

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

## Production & Real-Time Systems Checklist

### Code Level
- [ ] **Zero allocations in control loop** - Pre-allocate arrays, reuse buffers
- [ ] **No GC pauses** - Avoid any object creation in <10ms hot path
- [ ] **Type-stable functions** - Same input types always return same output types (Python/Cython friendly)
- [ ] **JAX/numba compatible** - Use numpy operations (enables JIT compilation later)
- [ ] **Thread-safe state** - Use locks if multiple threads access arm state
- [ ] **No blocking I/O** - Hardware reads are async or in background thread

### Logging & Telemetry
- [ ] **Structured logging** - JSON format for machine parsing
- [ ] **Performance metrics** - Log actual compute times vs targets
- [ ] **Command history** - Log all motor commands with timestamps
- [ ] **Error tracking** - Log failures, singularities, safety violations
- [ ] **Sampling rates** - Configurable (don't log every 1kHz sample)

Example:
```python
from hands.utils.logging import get_logger
logger = get_logger(__name__)

# Log with performance data
logger.info("FK_CALCULATED", extra={
    "duration_ms": 0.8,
    "target_ms": 1.0,
    "status": "OK"
})
```

### Safety & Failure Modes
- [ ] **Graceful degradation** - System continues with reduced capability if subsystem fails
- [ ] **Emergency stop** - Hardware + software e-stop (can't be ignored)
- [ ] **Watchdog timers** - Detect runaway control loops
- [ ] **State validation** - Never send commands that violate hard constraints
- [ ] **Recovery protocols** - How to restart after failure (without damaging arm)

Example scenarios:
```python
# Joint limit violation - MUST reject before sending to hardware
if q_desired > joint_limits[joint_idx]:
    logger.error("JOINT_LIMIT_VIOLATION", extra={"joint": joint_idx, "desired": q_desired})
    raise JointLimitError(f"Joint {joint_idx} exceeds limit")

# Temperature warning - reduce speed but continue
if motor_temp > TEMP_WARNING:
    logger.warning("MOTOR_TEMP_HIGH", extra={"temp": motor_temp})
    self.velocity_scale = 0.5  # Gradual slowdown
```



## Effective Prompting Strategy for Advanced Developers

### Prompt Structure (General Template)
```
[CONTEXT] I'm working on [module] - [production/research] code
[PHASE] Current phase: [1-5] - [Phase name]
[STATUS] What exists: [Current state]
[GOAL] Implement: [Specific algorithm/feature]
[CONSTRAINTS] Requirements:
  - Performance: [target latency]
  - Safety: [hardware limits/validation]
  - Dependencies: [python libraries]
  - Testing: [what needs to be validated]
[OUTPUT] Expected output/signature: [return type/format]
```

---

### Workflow 1: Implementing New Algorithm
**Use when**: Adding FK/IK, trajectory planning, collision detection

```
[CONTEXT] Implementing new algorithm for [component] - production code
[PHASE] Phase: [X], [Module: src/arm/kinematics.py]
[INPUT] Input constraints:
  - Type: [np.ndarray shape (N, 7)]
  - Range: [joint angles in [-π, π]]
  - Batch: [Yes/No - vectorized?]
[ALGORITHM] Use: [mathematical approach]
  - Paper/reference: [if applicable]
  - DH parameters: [from arm_config.yaml]
[OUTPUT] Return: [4x4 matrix / trajectory object]
[PERFORMANCE] Target: [<1ms for FK, <10ms for IK]
[VALIDATION] Compare against: [Drake, IKPy, known poses]
[TESTS] Edge cases:
  - Singularities: [handle how?]
  - Boundary poses: [test workspace edges]
```

**Example:**
```
[CONTEXT] Implementing inverse kinematics for 7 DOF arm - production code
[PHASE] Phase 1: Kinematics Foundation (src/arm/kinematics.py)
[INPUT] Target TCP pose: 4x4 matrix, seed joint angles (optional)
[ALGORITHM] Use IKPy with DH params from arm_config.yaml, handle redundancy
[OUTPUT] np.ndarray shape (7,) - joint angles in radians
[PERFORMANCE] Target <10ms average, handle case where no solution exists
[VALIDATION] Test against known benchmark poses, compare with PyBullet simulation
[TESTS] handle singularities gracefully, return None if unreachable
```

---

### Workflow 2: Performance Debugging
**Use when**: Component is too slow, missing real-time deadlines

```
[ISSUE] [Component] is [current time] ms, target is [target time] ms
[PROFILING] Profiling data from cProfile:
  - Function breakdown: [top 3 bottlenecks]
  - Allocations: [memory growth during run]
[CURRENT] Current implementation: [algorithm/approach]
[CONSTRAINTS]
  - Can't change algorithm: [Yes/No]
  - Must support: [batch/streaming/realtime]
  - Memory budget: [X MB available]
[GOAL] Optimize to [target time] or better
[OPTIMIZATION] Strategies to try:
  - Vectorization: [loop → numpy operation]
  - Caching: [pre-compute what?]
  - JIT: [numba/JAX candidate?]
[VALIDATION] Before/after profiling must show [expected speedup]
```

**Example:**
```
[ISSUE] FK calculation is 2.5ms, target is <1ms
[PROFILING] cProfile shows:
  - Matrix multiplies: 60% of time
  - DH param lookup: 30%
  - Type conversions: 10%
[CURRENT] Using nested loops for 7 DOF chain
[GOAL] get to <1ms using vectorization
[OPTIMIZATION] Replace loops with numpy @ operators, cache DH matrices
[VALIDATION] Should be <0.5ms for single calc, <10ms for batch of 100
```

---

### Workflow 3: Hardware Integration & Testing
**Use when**: Adding motor interface, testing with real hardware

```
[COMPONENT] Hardware layer: [motor_interface.py]
[HARDWARE] Target hardware:
  - Type: [Dynamixel/MAXON/other]
  - Protocol: [serial/CAN/ethernet]
  - Count: [7 motors, 1 control board]
  - Specs: [voltage, max torque, resolution]
[SAFETY] Hard limits:
  - Joint bounds: [from arm_config.yaml]
  - Velocity limit: [deg/s]
  - Acceleration limit: [deg/s²]
  - Torque limit: [Nm]
  - E-stop: [implemented how?]
[TESTING] Protocol:
  1. Simulation test (PyBullet): [verify commands offline]
  2. Hardware dry-run: [send commands with e-stop ready]
  3. Monitored run: [log all commands, check for anomalies]
  4. Validation: [compare PyBullet vs real motion]
[ROLLBACK] If testing fails: [manual recovery procedure]
```

**Example:**
```
[COMPONENT] Adding Dynamixel XM430 servo interface
[HARDWARE] 7x XM430 servos @ 3.0A max, serial communication
[SAFETY] Joint limits [-180°, 180°], velocity <90°/s during testing
[TESTING]
  1. Mock hardware test with cProfile showing <5ms per cycle
  2. Send zero command with e-stop armed
  3. Ramp velocity from 0 to 10°/s, verify smooth acceleration
  4. Stop command received within 50ms
```

---

### Workflow 4: Simulation Validation
**Use when**: Comparing simulation results with expected real-world behavior

```
[SIMULATION] PyBullet model vs real hardware
[VALIDATION] Metrics to check:
  - FK accuracy: [sim TCP vs real TCP position difference]
  - Dynamics accuracy: [torque commands vs motion]
  - Latency: [command→response time]
[TEST_CASES] Run scenarios:
  - Resting pose (no inertia): [should match exactly]
  - Slow motion (gravity compensation): [verify torques]
  - Fast motion (dynamics): [compare acceleration profiles]
  - Collision scenarios: [safety behavior]
[TOLERANCE] Accept if error < [X mm position, Y° orientation]
[LOG] Save comparison results to: [results/validation_YYYYMMDD.json]
[FAILURE] If validation fails:
  - Adjust URDF: [inertia matrices, COM positions]
  - Adjust motor model: [friction, damping]
  - Adjust PID gains: [if control loop differs]
```

---

### Workflow 5: Code Refactoring for Production
**Use when**: Improving existing code for real-time / maintainability

```
[CURRENT] Current code analysis:
  - File: [src/control/controller.py]
  - Issue: [memory allocations in loop / type instability / bad performance]
  - Performance: [current profiling data]
[GOAL] Refactor to:
  - No allocations in real-time loop
  - Type-stable function signatures
  - Match performance targets
[TESTING] Regression tests:
  - Behavior unchanged: [same inputs → same outputs]
  - Performance improved: [X% faster]
  - Memory stable: [no growth over 1000 iterations]
[VALIDATION] Before/after comparison
  - Code review: [readability maintained?]
  - Tests pass: [all unit tests still green]
  - No numerical regression: [within 1e-10 tolerance]
```

---

## Recommended Prompt Templates (Quick Reference)

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

## README.md as Living Project Spec

**CRITICAL**: README.md must be updated immediately after each feature completes. It's the source of truth for project status.

### When to Update README.md

**After every completed feature:**
1. ✅ Complete implementation (code + tests)
2. ✅ Pass all tests: `pytest tests/`
3. ✅ Profile if time-critical: `python -m cProfile -s cumtime script.py`
4. ✅ Fill in checklist in README.md right away (don't batch updates)
5. ✅ Commit with message: `[Phase X] Feature: Description - Performance: Xms`

### Update Template

In README.md under "Implementation Progress", change:
```markdown
# BEFORE:
- [ ] Forward Kinematics - Compute end-effector from joint angles

# AFTER:
- ✅ Forward Kinematics [0.8ms avg] - Compute end-effector from joint angles
```

### Update Performance Table in README.md

When performance is verified, update the table:
```markdown
# BEFORE:
| FK calculation | <1ms | TBD |

# AFTER:
| FK calculation | <1ms | ✅ 0.8ms (2 Mar 2026) |
```

### Update "Recent Updates" Section

Always add to README.md "Recent Updates" section:
```markdown
- **[DATE]**: [Phase X] - [Feature name]
  - ✅ [What was done]
  - 📊 Performance: [Xms]
  - 🔗 Related tests: [test_file.py]
```

### Update Phase Completion

When complete with a 5-step workflow:
```markdown
# BEFORE:
### Phase 1: Kinematics Foundation
- ✅ Forward Kinematics [0.8ms avg]
- [ ] Inverse Kinematics
- [ ] Visualization
- [ ] Unit tests

# AFTER (MOVE TO NEXT PHASE):
### Phase 1: Kinematics Foundation ✅ COMPLETE
- ✅ Forward Kinematics [0.8ms avg] - 2 Mar
- ✅ Inverse Kinematics [8.5ms avg] - 3 Mar
- ✅ Visualization [<1ms render] - 4 Mar
- ✅ Unit tests [100% coverage] - 4 Mar

### Phase 2: Dynamics & Simulation
- [ ] URDF model definition
```

### Current Phase Indicator

EOF of README.md must always show current status:
```markdown
**Last Updated**: 2026-03-15
**Current Phase**: 1 - Kinematics Foundation
**Completion**: 2/5 features done (40%)
**Next Milestone**: Complete IK solver by March 20
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

## CI/CD Integration Notes

When code is updated:
1. Run `pytest tests/` - ensure all tests pass
2. Run `mypy src/` - type checking
3. Profile key components - ensure performance targets
4. **IMMEDIATELY update README.md** with checklist progress
5. Update documentation if APIs changed
6. Commit with semantic message: `[Phase X] Feature: Name - Performance: Xms`

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
