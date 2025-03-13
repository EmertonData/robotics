
<img src="media/logo ED_.jpg">

<h1 align="center">
  <p>AI Robotics by Emerton Data in collaboration with LeRobot Hugging Face</p>
</h1>

<p align="center">
  <img src="media/lerobot-logo-thumbnail.png" style="width:60%">
</p>


<h2 align="center">
  <p>Build Your Own Robot</p>
</h2>

<div align="center">
  <img src="media/robot_img.jpg" width="50%"></div>

  <p>Unleash your robot’s potential in no time – just a few clicks on your laptop.</p>
  <p>Then kick back and enjoy watching it come to life and perform on its own! 🚀</p>

# Introduction

## Table of Contents

  - [A. Install repository](#a-install-repository)
  - [B. Configure the Motors](#b-configure-the-motors)
  - [C. Step-by-Step Assembly Instructions](#c-step-by-step-assembly-instructions)
  - [D. Calibrate](#d-calibrate)
  - [E. Teleoperate](#e-teleoperate)
  - [F. Set up cameras](#f-set-up-cameras)
  - [G. Record a dataset](#g-record-a-dataset)
  - [H. Visualize a dataset](#h-visualize-a-dataset)
  - [I. Replay an episode](#i-replay-an-episode)
  - [J. Train a policy](#j-train-a-policy)
  - [K. Evaluate your policy](#k-evaluate-your-policy)

# Tutorial
## A. Install repository

On your computer, create a folder that will contain all the code. 
Then, start a new shell and navigate to the folder you just created. 
Follow the steps below by running the commands in your shell.

### 1. Download the source code:
```bash
git clone https://github.com/EmertonData/robotics.git
cd robotics
```

### 2. Create a virtual environment with Python >=3.10 and activate it, e.g. with 
```bash
python -m venv .venv
```
or 
```bash
python3 -m venv .venv
```

### 3. To activate it:

On macOS:
```bash
source .venv/bin/activate
```

On Windows:
```bash
.venv\Scripts\activate
```

### 4. Install dependencies for the feetech motors:
```bash
pip install -e ".[feetech]"
```

*EXTRA: For Mac*: install extra dependencies for recording datasets:
```bash
brew install ffmpeg
```

Great :wink:! You are now done installing the repository and we can begin configuring the SO100 arms :robot:.

## B. Configure the Motors

> [!NOTE]
> Throughout this tutorial you will find videos on how to do the steps, the full video tutorial can be found here: [assembly video](https://www.youtube.com/watch?v=FioA2oeFZ5I).

### 1. Find the USB ports associated to each arm

Designate one bus servo adapter and 6 motors for your leader arm, and similarly the other bus servo adapter and 6 motors for the follower arm. It's convenient to label them and write on each motor if it's for the follower `F` or for the leader `L` and it's ID from 1 to 6 (F1...F6 and L1...L6).

#### a. Run the script to find port

<details>
<summary><strong>Video finding port</strong></summary>
  <video src="https://github.com/user-attachments/assets/4a21a14d-2046-4805-93c4-ee97a30ba33f"></video>
  <video src="https://github.com/user-attachments/assets/1cc3aecf-c16d-4ff9-aec7-8c175afbbce2"></video>
</details>

To find the port for each bus servo adapter, run the utility script:
```bash
python lerobot/scripts/find_motors_bus_port.py
```

#### b. Example outputs

Example output when identifying the leader arm's port (e.g., `/dev/tty.usbmodem575E0031751` on Mac, or possibly `/dev/ttyACM0` on Linux):
```
Finding all available ports for the MotorBus.
['/dev/tty.usbmodem575E0032081', '/dev/tty.usbmodem575E0031751']
Remove the usb cable from your MotorsBus and press Enter when done.

[...Disconnect leader arm and press Enter...]

The port of this MotorsBus is /dev/tty.usbmodem575E0031751
Reconnect the usb cable.
```
Example output when identifying the follower arm's port (e.g., `/dev/tty.usbmodem575E0032081`, or possibly `/dev/ttyACM1` on Linux):
```
Finding all available ports for the MotorBus.
['/dev/tty.usbmodem575E0032081', '/dev/tty.usbmodem575E0031751']
Remove the usb cable from your MotorsBus and press Enter when done.

[...Disconnect follower arm and press Enter...]

The port of this MotorsBus is /dev/tty.usbmodem575E0032081
Reconnect the usb cable.
```

#### c. Update config file

IMPORTANTLY: Now that you have your ports, update the **port** default values of [`SO100RobotConfig`](../lerobot/common/robot_devices/robots/configs.py) (../lerobot/common/robot_devices/robots/configs.py). You will find something like:
```python
@RobotConfig.register_subclass("so100")
@dataclass
class So100RobotConfig(ManipulatorRobotConfig):
    calibration_dir: str = "personal/.cache/calibration/so100"
    # `max_relative_target` limits the magnitude of the relative positional target vector for safety purposes.
    # Set this to a positive scalar to have the same value for all motors, or a list that is the same length as
    # the number of motors in your follower arms.
    max_relative_target: int | None = None

    leader_arms: dict[str, MotorsBusConfig] = field(
        default_factory=lambda: {
            "main": FeetechMotorsBusConfig(
                port="/dev/tty.usbmodem58760431091",  <-- UPDATE HERE
                motors={
                    # name: (index, model)
                    "shoulder_pan": [1, "sts3215"],
                    "shoulder_lift": [2, "sts3215"],
                    "elbow_flex": [3, "sts3215"],
                    "wrist_flex": [4, "sts3215"],
                    "wrist_roll": [5, "sts3215"],
                    "gripper": [6, "sts3215"],
                },
            ),
        }
    )

    follower_arms: dict[str, MotorsBusConfig] = field(
        default_factory=lambda: {
            "main": FeetechMotorsBusConfig(
                port="/dev/tty.usbmodem585A0076891",  <-- UPDATE HERE
                motors={
                    # name: (index, model)
                    "shoulder_pan": [1, "sts3215"],
                    "shoulder_lift": [2, "sts3215"],
                    "elbow_flex": [3, "sts3215"],
                    "wrist_flex": [4, "sts3215"],
                    "wrist_roll": [5, "sts3215"],
                    "gripper": [6, "sts3215"],
                },
            ),
        }
    )
```

### 2. Set IDs for all 12 motors


<details>
<summary><strong>Video configuring motor</strong></summary>
  <video src="https://github.com/user-attachments/assets/ef9b3317-2e11-4858-b9d3-f0a02fb48ecf"></video>
  <video src="https://github.com/user-attachments/assets/f36b5ed5-c803-4ebe-8947-b39278776a0d"></video>
</details>

> **❗IMPORTANT!** Remember the ID of each motor!

Plug your first motor F1 (Follower 1) and run this script to set its ID to 1. It will also set its present position to 2048, so expect your motor to rotate. 
> [!NOTE]
> Replace the text after --port to the corresponding follower control board port and run this command in cmd:
```bash
python lerobot/scripts/configure_motor.py \
  --port /dev/tty.usbmodem58760432961 \
  --brand feetech \
  --model sts3215 \
  --baudrate 1000000 \
  --ID 1
```

> [!NOTE]
> These motors are currently limited. They can take values between 0 and 4096 only, which corresponds to a full turn. They can't turn more than that. 2048 is at the middle of this range, so we can take -2048 steps (180 degrees anticlockwise) and reach the maximum range, or take +2048 steps (180 degrees clockwise) and reach the maximum range. The configuration step also sets the homing offset to 0, so that if you misassembled the arm, you can always update the homing offset to account for a shift up to ± 2048 steps (± 180 degrees).

Then unplug your motor and plug the second motor and set its ID to 2.
```bash
python lerobot/scripts/configure_motor.py \
  --port /dev/tty.usbmodem58760432961 \
  --brand feetech \
  --model sts3215 \
  --baudrate 1000000 \
  --ID 2
```

Redo the process for all your motors until ID 6. **Do the same for the 6 motors of the leader arm.**

> **❗IMPORTANT!** Remember the ID of each motor!


## C. Step-by-Step Assembly Instructions
> [!NOTE]
> From step 1 to 22, it is the same for both leader and follower arms.
> **❗IMPORTANT!** Make sure to not turn the motors in any way! They should stay in the same position after the calibration

### Before: Remove the gears of the 6 leader motors

<details>
<summary><strong>Video removing gears</strong></summary>

<video src="https://github.com/user-attachments/assets/0c95b88c-5b85-413d-ba19-aee2f864f2a7"></video>

</details>


Follow the video for removing gears. You need to remove the gear for the motors of the leader arm. As a result, you will only use the position encoding of the motor and reduce friction to more easily operate the leader arm.


---

### Additional Guidance

<details>
<summary><strong>Video assembling arms</strong></summary>

<video src="https://github.com/user-attachments/assets/488a39de-0189-4461-9de3-05b015f90cca"></video>

</details>

**Note:**
This video provides visual guidance for assembling the arms, but it doesn't specify when or how to do the wiring. Inserting the cables beforehand is much easier than doing it afterward. The first arm may take a bit more than 1 hour to assemble, but once you get used to it, you can assemble the second arm in under 1 hour.

---

### First Motor

**Step 1: Insert Wires**
- Insert two wires into the first motor.

  <img src="media/tutorial/img1.jpg" style="height:300px;">

**Step 2: Install in Base**
- Place the first motor into the base.

  <img src="media/tutorial/img2.jpg" style="height:300px;">

**Step 3: Secure Motor**
- Fasten the motor with 4 screws. Two from the bottom and two from top.

**Step 4: Attach Motor Holder**
- Slide over the first motor holder and fasten it using two screws (one on each side).

  <img src="media/tutorial/img4.jpg" style="height:300px;">

**Step 5: Attach Motor Horns**
- Install both motor horns, securing the top horn with a screw. Try not to move the motor position when attaching the motor horn, especially for the leader arms, where we removed the gears.

  <img src="media/tutorial/img5.jpg" style="height:300px;">
<details>
  <summary><strong>Video adding motor horn</strong></summary>
  <video src="https://github.com/user-attachments/assets/ef3391a4-ad05-4100-b2bd-1699bf86c969"></video>
</details>

**Step 6: Attach Shoulder Part**
- Route one wire to the back of the robot and the other to the left or in photo towards you (see photo).
- Attach the shoulder part.

  <img src="media/tutorial/img6.jpg" style="height:300px;">

**Step 7: Secure Shoulder**
- Tighten the shoulder part with 4 screws on top and 4 on the bottom
*(access bottom holes by turning the shoulder).*


---

### Second Motor Assembly

**Step 8: Install Motor 2**
- Slide the second motor in from the top and link the wire from motor 1 to motor 2.

  <img src="media/tutorial/img8.jpg" style="height:300px;">

**Step 9: Attach Shoulder Holder**
- Add the shoulder motor holder.
- Ensure the wire from motor 1 to motor 2 goes behind the holder while the other wire is routed upward (see photo).
- This part can be tight to assemble, you can use a workbench like the image or a similar setup to push the part around the motor.

  <div style="display: flex;">
    <img src="media/tutorial/img9.jpg" style="height:250px;">
    <img src="media/tutorial/img10.jpg" style="height:250px;">
    <img src="media/tutorial/img12.jpg" style="height:250px;">
  </div>

**Step 10: Secure Motor 2**
- Fasten the second motor with 4 screws.

**Step 11: Attach Motor Horn**
- Attach both motor horns to motor 2, again use the horn screw.

**Step 12: Attach Base**
- Install the base attachment using 2 screws.

  <img src="media/tutorial/img11.jpg" style="height:300px;">

**Step 13: Attach Upper Arm**
- Attach the upper arm with 4 screws on each side.

  <img src="media/tutorial/img13.jpg" style="height:300px;">

---

### Third Motor Assembly

**Step 14: Install Motor 3**
- Route the motor cable from motor 2 through the cable holder to motor 3, then secure motor 3 with 4 screws.

**Step 15: Attach Motor Horn**
- Attach both motor horns to motor 3 and secure one again with a horn screw.

  <img src="media/tutorial/img14.jpg" style="height:300px;">

**Step 16: Attach Forearm**
- Connect the forearm to motor 3 using 4 screws on each side.

  <img src="media/tutorial/img15.jpg" style="height:300px;">

---

### Fourth Motor Assembly

**Step 17: Install Motor 4**
- Slide in motor 4, attach the cable from motor 3, and secure the cable in its holder with a screw.

  <div style="display: flex;">
    <img src="media/tutorial/img16.jpg" style="height:300px;">
    <img src="media/tutorial/img19.jpg" style="height:300px;">
  </div>

**Step 18: Attach Motor Holder 4**
- Install the fourth motor holder (a tight fit). Ensure one wire is routed upward and the wire from motor 3 is routed downward (see photo).

  <img src="media/tutorial/img17.jpg" style="height:300px;">

**Step 19: Secure Motor 4 & Attach Horn**
- Fasten motor 4 with 4 screws and attach its motor horns, use for one a horn screw.

  <img src="media/tutorial/img18.jpg" style="height:300px;">

---

### Wrist Assembly

**Step 20: Install Motor 5**
- Insert motor 5 into the wrist holder and secure it with 2 front screws.

  <img src="media/tutorial/img20.jpg" style="height:300px;">

**Step 21: Attach Wrist**
- Connect the wire from motor 4 to motor 5. And already insert the other wire for the gripper.
- Secure the wrist to motor 4 using 4 screws on both sides.

  <img src="media/tutorial/img22.jpg" style="height:300px;">

**Step 22: Attach Wrist Horn**
- Install only one motor horn on the wrist motor and secure it with a horn screw.

  <img src="media/tutorial/img23.jpg" style="height:300px;">

---

### Follower Configuration

**Step 23: Attach Gripper**
- Attach the gripper to motor 5.

  <img src="media/tutorial/img24.jpg" style="height:300px;">

**Step 24: Install Gripper Motor**
- Insert the gripper motor, connect the motor wire from motor 5 to motor 6, and secure it with 3 screws on each side.

  <img src="media/tutorial/img25.jpg" style="height:300px;">

**Step 25: Attach Gripper Horn & Claw**
- Attach the motor horns and again use a horn screw.
- Install the gripper claw and secure it with 4 screws on both sides.

  <img src="media/tutorial/img26.jpg" style="height:300px;">

**Step 26: Mount Controller**
- Attach the motor controller on the back.

  <div style="display: flex;">
    <img src="media/tutorial/img27.jpg" style="height:300px;">
    <img src="media/tutorial/img28.jpg" style="height:300px;">
  </div>

*Assembly complete – proceed to Leader arm assembly.*

---

### Leader Configuration

For the leader configuration, perform **Steps 1–22**. Make sure that you removed the motor gears from the motors.

**Step 23: Attach Leader Holder**
- Mount the leader holder onto the wrist and secure it with a screw.

  <img src="media/tutorial/img29.jpg" style="height:300px;">

**Step 24: Attach Handle**
- Attach the handle to motor 5 using 4 screws.

  <img src="media/tutorial/img30.jpg" style="height:300px;">

**Step 25: Install Gripper Motor**
- Insert the gripper motor, secure it with 3 screws on each side, attach a motor horn using a horn screw, and connect the motor wire.

  <img src="media/tutorial/img31.jpg" style="height:300px;">

**Step 26: Attach Trigger**
- Attach the follower trigger with 4 screws.

  <img src="media/tutorial/img32.jpg" style="height:300px;">

**Step 27: Mount Controller**
- Attach the motor controller on the back.

  <div style="display: flex;">
    <img src="media/tutorial/img27.jpg" style="height:300px;">
    <img src="media/tutorial/img28.jpg" style="height:300px;">
  </div>

*Assembly complete – proceed to calibration.*

## D. Calibrate

Next, you'll need to calibrate your SO-100 robot to ensure that the leader and follower arms have the same position values when they are in the same physical position. This calibration is essential because it allows a neural network trained on one SO-100 robot to work on another.

#### a. Manual calibration of follower arm

> [!IMPORTANT]
> Contrarily to step 6 of the [assembly video](https://youtu.be/FioA2oeFZ5I?t=724) which illustrates the auto calibration, we will actually do manual calibration of follower for now.

You will need to move the follower arm to these positions sequentially:

| 1. Zero position                                                                                                                                             | 2. Rotated position                                                                                                                                                   | 3. Rest position                                                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="media/so100/follower_zero.webp?raw=true" alt="SO-100 follower arm zero position" title="SO-100 follower arm zero position" style="width:100%;"> | <img src="media/so100/follower_rotated.webp?raw=true" alt="SO-100 follower arm rotated position" title="SO-100 follower arm rotated position" style="width:100%;"> | <img src="media/so100/follower_rest.webp?raw=true" alt="SO-100 follower arm rest position" title="SO-100 follower arm rest position" style="width:100%;"> |

Make sure both arms are connected and run this script to launch manual calibration:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_follower"]'
```

#### b. Manual calibration of leader arm
Follow step 6 of the [assembly video](https://youtu.be/FioA2oeFZ5I?t=724) which illustrates the manual calibration. You will need to move the leader arm to these positions sequentially:

| 1. Zero position                                                                                                                                       | 2. Rotated position                                                                                                                                             | 3. Rest position                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="media/so100/leader_zero.webp?raw=true" alt="SO-100 leader arm zero position" title="SO-100 leader arm zero position" style="width:100%;"> | <img src="media/so100/leader_rotated.webp?raw=true" alt="SO-100 leader arm rotated position" title="SO-100 leader arm rotated position" style="width:100%;"> | <img src="media/so100/leader_rest.webp?raw=true" alt="SO-100 leader arm rest position" title="SO-100 leader arm rest position" style="width:100%;"> |

Run this script to launch manual calibration:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_leader"]'
```

## E. Teleoperate

**Simple teleop**
Then you are ready to teleoperate your robot! Run this simple script (it won't connect and display the cameras):
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=teleoperate
```

## F. Set up cameras

> [!NOTE]
> Follow [this guide to setup your cameras](https://github.com/huggingface/lerobot/blob/main/examples/7_get_started_with_real_robot.md#c-add-your-cameras-with-opencvcamera) if needed.


To identify the cameras connected, run the following utility script, which will save a few frames from each detected camera:
```bash
python lerobot/common/robot_devices/cameras/opencv.py \
    --images-dir outputs/images_from_opencv_cameras
```

The output will look something like this if you have two cameras connected:
```
Mac or Windows detected. Finding available camera indices through scanning all indices from 0 to 60
[...]
Camera found at index 0
Camera found at index 1
[...]
Connecting cameras
OpenCVCamera(0, fps=30.0, width=1920.0, height=1080.0, color_mode=rgb)
OpenCVCamera(1, fps=24.0, width=1920.0, height=1080.0, color_mode=rgb)
Saving images to outputs/images_from_opencv_cameras
Frame: 0000	Latency (ms): 39.52
[...]
Frame: 0046	Latency (ms): 40.07
Images have been saved to outputs/images_from_opencv_cameras
```


#### Teleop with displaying cameras

```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=teleoperate
```

## G. Record a dataset

Once you're familiar with teleoperation, you can record your first dataset with SO-100.

<details>
<summary><strong>Hugging Face Hub Setup (Optional)</strong></summary>
If you want to use the Hugging Face hub features for uploading your dataset and you haven't previously done it, make sure you've logged in using a write-access token, which can be generated from the [Hugging Face settings](https://huggingface.co/settings/tokens):
```bash
huggingface-cli login --token ${HUGGINGFACE_TOKEN} --add-to-git-credential
```

Store your Hugging Face repository name in a variable to run these commands:
```bash
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
```
</details>

Set_up your username, under which the dataset will be saved:
```bash
HF_USER=your_username
echo $HF_USER
```

Record 2 episodes:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a block and put it in the bin." \
  --control.repo_id=${HF_USER}/so100_test \
  --control.tags='["so100","tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=15 \
  --control.reset_time_s=10 \
  --control.num_episodes=2 \
  --control.push_to_hub=false
```

Note: You can resume recording by adding `--control.resume=true`. 


## H. Visualize a dataset

<details>
<summary><strong>If uploaded to Hugging Face Hub (Optional)</strong></summary>
If you uploaded your dataset to the hub with `--control.push_to_hub=true`, you can [visualize your dataset online](https://huggingface.co/spaces/lerobot/visualize_dataset) by copy pasting your repo id given by:
```bash
echo ${HF_USER}/so100_test
```
</details>

If you didn't upload with `--control.push_to_hub=false`, you can visualize the dataset locally with (a window can be opened in the browser `http://127.0.0.1:9090` with the visualization tool):
```bash
python lerobot/scripts/visualize_dataset_html.py \
  --repo-id ${HF_USER}/so100_test
```

## I. Replay an episode

Now try to replay the first episode on your robot:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=replay \
  --control.fps=30 \
  --control.repo_id=${HF_USER}/so100_test \
  --control.episode=0
```

## J. Train a policy

To train a policy to control your robot, use the [`python lerobot/scripts/train.py`](../lerobot/scripts/train.py) script. A few arguments are required. Here is an example command:
```bash
python lerobot/scripts/train.py \
  --dataset.repo_id=${HF_USER}/so100_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so100_test \
  --job_name=act_so100_test \
  --policy.device=cuda \
  --wandb.enable=true
```

Let's explain it:
1. We provided the dataset as argument with `--dataset.repo_id=${HF_USER}/so100_test`.
2. We provided the policy with `policy.type=act`. This loads configurations from [`configuration_act.py`](../lerobot/common/policies/act/configuration_act.py). Importantly, this policy will automatically adapt to the number of motor sates, motor actions and cameras of your robot (e.g. `laptop` and `phone`) which have been saved in your dataset.
4. We provided `policy.device=cuda` since we are training on a Nvidia GPU, but you could use `policy.device=mps` to train on Apple silicon.
5. We provided `wandb.enable=true` to use [Weights and Biases](https://docs.wandb.ai/quickstart) for visualizing training plots. This is optional but if you use it, make sure you are logged in by running `wandb login`.

Training should take several hours. You will find checkpoints in `outputs/train/act_so100_test/checkpoints`.


## K. Evaluate your policy

> [!REMEMBER]
> The cameras have to be in the same position as when recording the dataset.

You can use the `record` function from [`lerobot/scripts/control_robot.py`](../lerobot/scripts/control_robot.py) but with a policy checkpoint as input. For instance, run this command to record 10 evaluation episodes:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=${HF_USER}/eval_act_so100_test \
  --control.tags='["tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=10 \
  --control.push_to_hub=true \
  --control.policy.path=outputs/train/act_so100_test/checkpoints/last/pretrained_model
```

As you can see, it's almost the same command as previously used to record your training dataset. Two things changed:
1. There is an additional `--control.policy.path` argument which indicates the path to your policy checkpoint with  (e.g. `outputs/train/eval_act_so100_test/checkpoints/last/pretrained_model`). You can also use the model repository if you uploaded a model checkpoint to the hub (e.g. `${HF_USER}/act_so100_test`).
2. The name of dataset begins by `eval` to reflect that you are running inference (e.g. `${HF_USER}/eval_act_so100_test`).
