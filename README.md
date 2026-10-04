# revo2_isaac_description

This package provides the BrainCo Revo2 hand models for [mc_isaac](https://github.com/isri-aist/mc_isaac). (Isaac Sim simulation of
[mc_rtc](https://jrl-umi3218.github.io/mc_rtc/) controllers), like `revo2_mj_description` does for
[mc_mujoco](https://github.com/rohanpsingh/mc_mujoco).

It is a data-only package (no compilation, no mc_rtc dependency): USD models and one mc_isaac description file per
mc_rtc robot module.

It installs:

- `share/mc_isaac/<module>.yaml`: one description per mc_rtc robot module of `mc_revo2`
  - `revo2_left_hand.yaml`
  - `revo2_right_hand.yaml`
- `share/revo2_isaac_description/usd/`: the USD models referenced by these descriptions

The G1 robot with two Revo2 hands is provided by `g1_isaac_description`.

## Models

| mc_rtc module (`MainRobot` alias) | Description | USD | DOF |
|---|---|---|---|
| `revo2_left_hand` (`Revo2_LeftHand`, `Revo2`) | left hand alone | `revo2_left.usd` | 11 (6 actuated + 5 mimic) |
| `revo2_right_hand` (`Revo2_RightHand`) | right hand alone | `revo2_right.usd` | 11 (6 actuated + 5 mimic) |

The hands are simulated with a fixed base (mc_isaac adds a fixed joint to the world if the USD does not have one).

Joints (`<side>` is `left` or `right`):

- actuated: `<side>_thumb_metacarpal_joint`, `<side>_thumb_proximal_joint` and
  `<side>_(index|middle|ring|pinky)_proximal_joint`
- mimic: `<side>_(thumb|index|middle|ring|pinky)_distal_joint`

The distal joints are PhysX mimic joints (`PhysxMimicJointAPI` in the USD): PhysX makes them follow their proximal joint,
they have no drive and the commands mc_rtc sends for them are ignored. Their measured position and velocity are sent back
to mc_rtc like for any other joint.

mc_isaac looks descriptions up by **robot module name** (`robot.module().name` in mc_rtc), then robot name, then the
`MainRobot` value for the main robot, so both `MainRobot: Revo2_RightHand` and any additional robot using these modules
work.

## Description files

Each `<module>.yaml` contains:

```yaml
usd: <absolute path of the installed USD>
fixed: true
drives:            # PhysX joint drives, regex groups (fullmatch) over the Isaac joint names
  - {joints: "left_thumb_metacarpal_joint", stiffness: 20.0, damping: 1.0}
  ...
```

Units and meaning are the same as IsaacLab `ImplicitActuatorCfg` (applied by the mc_isaac server through the PhysX tensor
API, in radians). Keys not given (`max_effort`, `max_velocity`, `armature`) keep the values stored in the USD. Joints that
no group matches keep the gains stored in the USD (mc_isaac prints a warning, except for mimic joints).

Gains (from `revo2_mj_description` `pdgains/revo2_<side>_hand/PDgains_sim.dat`):

| Joints | Stiffness (N.m/rad) | Damping (N.m.s/rad) |
|---|---|---|
| `<side>_thumb_metacarpal_joint` | 20 | 1.0 |
| `<side>_thumb_proximal_joint` | 40 | 1.2 |
| `<side>_(index\|middle\|ring)_proximal_joint` | 35 | 1.0 |
| `<side>_pinky_proximal_joint` | 30 | 0.8 |

With `simulation: torque_control: true` in the IsaacSim plugin configuration, the gains are set to 0 and the mc_rtc
torques are applied as joint efforts instead.

## Build and install

```bash
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=<prefix>
make install
```

Installed in the mc_isaac (or mc_rtc) prefix, the descriptions are found without configuration; otherwise add
`<prefix>/share/mc_isaac` to `description_paths` in the IsaacSim plugin configuration
(`~/.config/mc_rtc/plugins/IsaacSim.yaml`).

Option `-DSRC_MODE=ON` makes the descriptions point to the USD files of the source tree (no USD copy at install).

## Model generation

The USD files come from the IsaacLab assets of the Revo2 hands, converted from the Revo2 URDFs with the Isaac Sim URDF
importer (mimic joints converted to `PhysxMimicJointAPI`). They are single-file USDs (only external reference: Isaac
Sim built-in materials), so mc_isaac can upload them as is to the Isaac server.

## Adding a variant

1. Add the USD in `usd/` (single file, joint names identical to the mc_rtc URDF; if it references other files, list
   them in `extra_files` of the description, see `g1_isaac_description`).
2. Add a line `"<module name> <side> <usd file>"` to `REVO2_MODELS` in `CMakeLists.txt` (or a new template in
   `mc_isaac/`).
3. Rebuild and install.
