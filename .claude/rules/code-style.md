# Code Style Rules

## Python
- Format with `black` (default settings)
- Sort imports with `isort` (profile=black)
- Type check with `pyright` (basic mode)
- Use type hints for function signatures
- Follow PEP 8 naming: snake_case for functions/variables, PascalCase for classes
- ROS 2 Python packages use `ament_python` build type

## C++
- Format with `clang-format` v19 (Google style via `.clang-format`)
- ROS 2 C++ packages use `ament_cmake` build type
- Header files in `include/<package_name>/`
- Source files in `src/`

## ROS 2 Conventions
- Package names: lowercase with underscores (e.g., `aic_model`)
- Node names: lowercase with underscores
- Topic names: lowercase with slashes (e.g., `/aic_controller/pose_commands`)
- Message fields: snake_case
- Use `package.xml` format 3

## Policy Code
- Subclass `aic_model.policy.Policy`
- Keep `insert_cable()` as the main entry point
- Use `get_observation()` callback - do NOT subscribe to topics directly
- Use `move_robot()` callback - do NOT publish to topics directly
- Use `send_feedback()` for debug messages
- Always check for cancellation in long-running loops
- Handle timeouts gracefully (return False if time_limit exceeded)

## File Organization
- Policy implementations go in `aic_example_policies/aic_example_policies/ros/`
- Register new policies in `setup.py` entry_points or as importable modules
- Test scripts go in `aic_model/test/`
- Configuration YAML files in respective `config/` directories
