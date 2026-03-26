Create a new policy implementation for the AIC competition.

Arguments: $ARGUMENTS (policy name in PascalCase)

Steps:
1. Create a new Python file at `aic_example_policies/aic_example_policies/ros/<PolicyName>.py`
2. Use this template structure:

```python
from aic_model.policy import Policy
from aic_task_interfaces.msg import Task
from aic_control_interfaces.msg import MotionUpdate, TrajectoryGenerationMode
from aic_model_interfaces.msg import Observation
from geometry_msgs.msg import Pose, Twist
from std_msgs.msg import Header

class <PolicyName>(Policy):
    def __init__(self, parent_node):
        super().__init__(parent_node)
        # Initialize your model/state here

    def insert_cable(self, task: Task, get_observation, move_robot, send_feedback) -> bool:
        # Main control loop
        send_feedback(f"Starting {task.plug_name} -> {task.port_name}")
        start_time = self.time_now()

        while (self.time_now() - start_time).nanoseconds / 1e9 < task.time_limit:
            obs = get_observation()
            if obs is None:
                self.sleep_for(0.05)
                continue

            # TODO: Process observation and compute action
            # TODO: Send command via move_robot()

            self.sleep_for(0.05)  # ~20Hz loop

        return False  # Change to True when insertion succeeds
```

3. Register the policy in `aic_example_policies/setup.py` if using entry points
4. Test with: `pixi run ros2 run aic_model aic_model --ros-args -p use_sim_time:=true -p policy:=aic_example_policies.ros.<PolicyName>`
