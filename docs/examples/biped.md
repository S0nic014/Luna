# Python biped build



```python
import pymel.core as pm
from luna import Logger
from luna_rig.core import pybuild
from luna_rig import components
from luna_rig import importexport
from luna.utils import devFn
# devFn.reload_rig_functions()
# devFn.reload_rig_components()


class RigBuild(pybuild.PyBuild):
    def run(self):
        self.spine = components.FKIKSpineComponent.create(side="c",
                                                          name="spine",
                                                          start_joint="c_spine_00_jnt",
                                                          end_joint="c_spine_02_jnt")
        self.left_leg = components.FKIKComponent.create(side="l",
                                                        name="leg",
                                                        start_joint="l_leg_00_jnt",
                                                        end_joint="l_leg_02_jnt",
                                                        meta_parent=self.spine,
                                                        attach_point=self.spine.AttachPoints.HIPS)
        self.right_leg = components.FKIKComponent.create(side="r",
                                                         name="leg",
                                                         start_joint="r_leg_00_jnt",
                                                         end_joint="r_leg_02_jnt",
                                                         meta_parent=self.spine,
                                                         attach_point=self.spine.hips_control)
        self.left_clavicle = components.FKComponent.create(side="l",
                                                           name="clavicle",
                                                           start_joint="l_clavicle_00_jnt",
                                                           end_joint="l_clavicle_00_jnt",
                                                           add_end_ctl=True,
                                                           meta_parent=self.spine,
                                                           attach_point=self.spine.AttachPoints.CHEST)
        self.right_clavicle = components.FKComponent.create(side="r",
                                                            name="clavicle",
                                                            start_joint="r_clavicle_00_jnt",
                                                            end_joint="r_clavicle_00_jnt",
                                                            add_end_ctl=True,
                                                            meta_parent=self.spine,
                                                            attach_point=self.spine.AttachPoints.CHEST)
        self.left_arm = components.FKIKComponent.create(side="l",
                                                        name="arm",
                                                        start_joint="l_shoulder_00_jnt",
                                                        end_joint="l_wrist_00_jnt",
                                                        meta_parent=self.left_clavicle,
                                                        attach_point=0)
        self.right_arm = components.FKIKComponent.create(side="r",
                                                         name="arm",
                                                         start_joint="r_shoulder_00_jnt",
                                                         end_joint="r_wrist_00_jnt",
                                                         meta_parent=self.right_clavicle,
                                                         attach_point=0)
        self.head = components.FKComponent.create(side="c",
                                                  name="head",
                                                  start_joint="c_neck_00_jnt",
                                                  end_joint="c_head_01_jnt",
                                                  add_end_ctl=False,
                                                  meta_parent=self.spine,
                                                  attach_point=self.spine.AttachPoints.CHEST)
        self.left_arm.param_control.shape = "hand"
        self.right_arm.param_control.shape = "hand"

        # Extra features
        self.left_clavicle.add_auto_aim(self.left_arm.ik_control)
        self.right_clavicle.add_auto_aim(self.right_arm.ik_control, mirrored_chain=True)

        # Parent skeleton
        self.spine.bind_joints[0].setParent(self.character.deformation_rig)
        for comp in self.character.get_meta_children():
            comp.attach_to_skeleton()

    def post(self):
        # Add spaces
        self.left_leg.pv_control.add_world_space()
        self.left_leg.pv_control.add_space(self.left_leg.ik_control, "IK")
        self.right_leg.pv_control.add_world_space()
        self.right_leg.pv_control.add_space(self.right_leg.ik_control, "IK")
        self.left_arm.pv_control.add_world_space()
        self.left_arm.pv_control.add_space(self.left_arm.ik_control, "IK")
        self.right_arm.pv_control.add_world_space()
        self.right_arm.pv_control.add_space(self.right_arm.ik_control, "IK")
        self.left_arm.ik_control.add_world_space()
        self.left_arm.ik_control.add_space(self.head.controls[-1], "Head")
        self.left_arm.pv_control.add_space(self.head.controls[-1], "Head")
        self.right_arm.ik_control.add_world_space()
        self.right_arm.ik_control.add_space(self.head.controls[-1], "Head")
        self.right_arm.pv_control.add_space(self.head.controls[-1], "Head")

        # Create sets
        self.character.create_controls_set()
        self.character.create_joints_set()
        # Import data
        importexport.CtlShapeManager().import_asset_shapes()


if __name__ == "__main__":
    result = RigBuild("character", "Dummy")

```
## Build results
---
### Rig:

![biped_rig](/docs/biped_rig.png)

### Node graph:
![biped_graph](/docs/biped_graph.png)