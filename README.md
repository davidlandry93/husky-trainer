# husky-trainer

A set of nodes that can do teach and repeat on a Clearpath Husky A200,
under ROS.

Tested under ROS Hydro.

## How it works

During the teach phase, the robot records point clouds of its surrondings along
with their positions. The commands sent to the robot are recorded as well.

During the repeat phase, the commands are played back again. The robot compares
the point clouds he reads to the point clouds he saved during the teach phase,
and computes the transformation to go from the reading to the recorded point
cloud. This transformation is then sent to a controller that will try to
compensate the error.

## Usage

### Teach

A launchfile is provided to launch the teach phase. 

```Shell
$ roslaunch husky_trainer husky-teach.launch
```

Then place the robot in the starting position and press Y on the gamepad to
start the teaching.  Move around, the point clouds are recorded as you go.  When
you are done stop the recording by hitting C-c in the console.  The files are
recorded in the CWD. Run the repeat node in the same directory you ran the teach
in and everything should be fine.

### Repeat

There is a launchfile for the repeat too, but you have to specify what config
file is to be used for the point cloud matching. The config file should be
compatible with libpointmatcher's options. 

```Shell
$ roslaunch husky_trainer husky-repeat.launch icp_config:=/abs/path/to/conf.yaml
```

For the repeat phase, the RB button acts as a deadman switch. Hold it to start
the playback.

## Nodes

This section documents the individual nodes, in case you want to play
with the launchfiles.

### teach

No parameters are interfaced. Note that there must be a node that records point
clouds to disk that is running for the teach to be useful. The clouds that
should be saved are sent on the `/anchor_points` topic. The `cloud_recorder`
node from the `pointcloud-tools` was used for testing.

### repeat

This nodes replays the commands on the robot and corrects the error according to
the scanned point clouds.

#### Parameters

- `lx`. This is the gain value applied to the error in the `x` axis (the front
  of the robot). Default: 0.0.
- `ly`. Same thing, but for the `y` axis, which is pointing towards the right
  side of the robot. Default: 0.0.
- `lt`. Same thing, but the gain is applied to the angular error, in the
  trigonometrical convention. Default: 0.0.
- `lookahead`. Many controllers will compute the error according to the position
  the robot should be in x miliseconds from now instead of the actual position. This
  parameter is used to specify the lookahead, in seconds. Default: 0.2.
- `readings_topic`. Used to specify what topic the node should use to get clouds
  and compare them to the anchor points. Default: `/cloud`.
- `working_directory`. Is used to specify a working directory different that the
  pwd, if you want the clouds to be saved elsewhere. This is mainly used by the
  launchfile. Optional.

### command_repeater

This node is used to repeat a desired Twist command at a constant rate. The node
listens to a topic for desired commands, and repeats them on another topic at a
constant rate. As a security, if no desired commands are sent in a certain time
, the node will stop sending commands. 

#### Parameters

- `input`. The name of the topic the desired commands are sent on.
- `output`. The name of the topic the commands are repeated on.
- `timeout`. The number of seconds the node waits before stopping the robot in
  case he stops receiving commands.
