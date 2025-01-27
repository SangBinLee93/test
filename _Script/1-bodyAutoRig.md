---
title : "[2023,2024] Matrix Auto Rig"
header:
  teaser: /assets/images/portfolio/0_autorig_thumbnail.png
  overlay_image: /assets/images/portfolio/0_autorig_thumbnail.png
  overlay_filter: 0.5
tages:
  - maya
  - rig
  - rnd
  - python
  - math
---

[RIG][Maya Math][Python][Git Cooperation]

# Matrix AutoRig Development

20231001~20231107

1. Common
    - [Matrix Node Connection] based rig.
    - Point/Orient/Parent Constraint is not used. Only IK and polvectorConstraint is used.
    - The final calculation formulas for the wrist, shoulder, hip, knee, ankle etc. have Quaternion calculations applied to prevent them from candywrap.
    - The rotateOrder is open for the animator to modify.
      (to Animator : try it out and if you provide feedback on the default values, we will make modifications.)

2. Remained Development Progress
    - FK & IK switching ( gui will be provided )
    - Asset publish, pickwalk will be added.
    - The scale attribute & Squash are meaningless except for the overall scale. 
    (Development is suspended because of realistic pipeline)

# **1. Spine**

## **1-1. Spine Structure**  

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/1_spine.png)  

The calculations are done in the order of FK → IK → Micro.

FK is the same as the existing advanced skeleton.

All controllers are always being calculated, and the visibility can be adjusted in **[M_IkSpine1_CTL]**.

## **1-2. M_IkSpine1_CTL**  

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/2_M_IkSpine1_CTL.png)  

**[visMod]**

**[fk+hip]**: The default mode that will be used the most.  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/3_spineDefault.mp4" type="video/mp4">
</video>  

Using IK, you can adjust the translation/rotation of Pelvis on top of the FK structure.  

**[all]**: Shows the middle and chest controllers of IK.  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/4_spine_all.mp4" type="video/mp4">
</video>  

Middle controller **(M_IKSpine2_CTL)**
- not affect the first and last controllers.
- **[topSmooth]**
    For the joint (M_Spine2Part1_JNT) closer to the chest,
  
    10:  
    Does not apply rotation based on IK translation.  
    Can express the stiffness of the sternum.  
    
    0:  
    Applies rotation based on translation.  

    (default: 5)
- **[micro_CTL]**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/5_microCTL.mp4" type="video/mp4">
</video>  

A controller that can move each joint independently from the hierarchy. It is left open for similar use.

- **[endSmooth]**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/6_endSmooth.mp4" type="video/mp4">
</video>  


Adjusts the rotation angle of Spine1 rotated by M_IKSpine1_CTL.

10: Aligns the rotation angle of Spine1 with Pelvis.

0: Spine1 looks towards the next joint direction (Spine1Part1).

(default 0)

- **[centerFollow]**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/7_centerFollow.mp4" type="video/mp4">
</video>  

10: Adjusts the coordinate value of the middle controller (M_IKSpine2_CTL) to be in the middle.

(default 0)

## **1-3. M_Spine3_micro_CTL**

The green Micro_CTL mentioned earlier is the same, but it has been modified to easily express breathing.  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/8_Breath.mp4" type="video/mp4">
</video>  

First, set the target scale values of ChestX, Y, Z to the desired shape.

Using only the chestBreath attribute, you can express the breath of the rib cage.

Using Belly X, Z, bellySpread, and bellyH,

Set the target scale values, and

Using only the bellyBreath attribute, you can express the breath of the abdomen.

BellyH adjusts the height of the affected joint,

BellySpread determines how far the influence spreads from that height.

BellyH allows you to give key values to express the movement of a wobbling belly.

## **1-4. RootX_M**

[ **centerBetweenFitVis** ]

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/9_centerBetweenFit.mp4" type="video/mp4">
</video>  

Shows the dummy controller between the feet.

You can use matchTransform as shown in the video.

# **2. Head**

## **2-1. Head Aim**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/10_headAim.mp4" type="video/mp4">
</video>  

The Head_aim_CTL (aim mode) has been added, similar to the existing eye-aim controller.

If you don't use it, you can turn off the aimVis on the Head controller.

- **[rotateZ] (tilt)** is also supported.
    
    rotateY is simply opened to visually see the axis of rotateZ.
    
- **[neckAim]:**
    
    Turn off the automatic axis adjustment of M_FkNeck_neg_CTL.
    
- **[headAim]:**
    
    Turn off the aiming effect of the headAim controller.
    
- **[headAimUp]:**
    
    Turn off the up and down aiming. If there are any abnormal symptoms such as sudden rotation, turning this off may fix the issue. (There are occasional errors when calculating together up and down due to the difficulty of axis calculation.)


<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/11_headAimGlobal.mp4" type="video/mp4">
</video>  

- **visMatchCtl** (M_FkHead_aim_matchTranslation_CTL)
    
    Due to calculation issues with neckautoTwist, it is not possible to constrain the head aim controller. ㅠㅠ
    
    Therefore, I have prepared an alternative method for you to use as follows.
    
    (Similar to RootX's centerBetweenFit.)
    
    - Give a key to the aim controller and turn on autokey.
    - Use visMatchCtl (dummy) (can also be a locator or other object) to first align the gaze.
    - Use modify - match Transform - match Transform to attach frame by frame.
 
    *20231112 upated*
    It can cause cycle error sometimes with worldMatrix, You Can turn on/off with the Attribute **[visMatchCtl]** on Condition. This will block the node state.


## **2-2 Head**

- **neck auto twist**  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/12_neckTwist.mp4" type="video/mp4">
</video>  

To allow the key to be set using only the head as much as possible, the left and right twists are automatically inputted.

Adjustment is possible with the **[neckTwist]** of the Head controller.

*20231112 upated*
It can cause cycle error sometimes with worldMatrix, You Can turn on/off with the Attribute **[neckTwistOnOff]**. This will block the node state.


- **globalRotY**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/13_globalRotY.mp4" type="video/mp4">
</video>  

By default, global follows the Sky. Due to other functionalities, it could only be implemented outside of the sky.

To provide an offset to global (to avoid gimbal lock), **[globalRotY]** has been included.

Since the rotationOrder has also been changed, there should be less gimbal lock by default.

## **2-3 M_FkNeck_neg_CTL**

- It doesn't matter if you just write it as is, but
In terms of calculation, it is recommended to use the gimbal mode and rotate from the y-axis.
The rotateY value is designed to be negated in order to not be affected.
The headAim controller is intended to be used to make slight modifications to the neck after using it.

## 2-4 Neck

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/14_Neck.mp4" type="video/mp4">
</video>  

- It is the same as the advanced skeleton.
- You can modify the degree to which the neck controller is affected by [w0~w2] for the 0th, 1st, and 2nd levels.
- [Function Addition] If you find it inconvenient to use, you can turn on [showInbetween] to allow direct modifications with the controller.

# **3. Arm & Finger**

## **3-1. FK Shoulder**

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/15_FKArm.png)  

[makeT]

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/16_Shoulder_makeT.mp4" type="video/mp4">
</video>  

Create the T-pose as the default axis for the shoulder.

When lifting the arm up,

accurate representation can be achieved using only the z-axis.

[Global]

10: Turn on the global option, but follow the followTarget.

(If you need it elsewhere, please request it from the rigging team.)

## **3-2. FKArm + IKFinger**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/17_FKHand.mp4" type="video/mp4">
</video>  

When using IKFinger described later,

if the arm is FK,

the base of all fingers follows FKWrist_CTL by default,

and when you want IkFingers not to be affected,

use the rotation of [**L_Finger_FKIK**].

## **3-3. IKArm + IKFinger**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/18_IKHand.mp4" type="video/mp4">
</video>  

When using IKFinger described later,

if the arm is IKfkaus,

the base of all fingers follows IkArm_CTL by default,

and when you want IkFingers not to be affected,

use the rotation of **[L_IkArm_endLocal_CTL]** or

**[L_Finger_FKIK]**.

## **3-3. IK Finger**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/19_IkFinger.mp4" type="video/mp4">
</video>  

If it doesn't move as you want, we recommend turning on visPV to see how it moves.

You can also use **[tipRot]** to create bending movements.

You can also use FK additionally.

## **3-5. IKArm**

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/20_IkArm_CTL_joint.png)  
<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/21_IKArm.mp4" type="video/mp4">
</video>  

By using the jointOrient option of the joint,

trans - Parent mode = world axis space

rotation - Object/Gimbal mode = Local axis space

can be used.

<details>
<summary>Gimbal Lock</summary>

## Gimbal Lock
![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/22_Gimbal_Lock_Plane.gif)  

When representing rotation in a three-dimensional axis using x, y, and z axes (Euler method),

there is a moment when two or more axes overlap.

In other words, at this time, one or more axes become mathematically equal and useless.

This phenomenon is called Gimbal Lock, and it is one of the causes of bouncing or not moving as desired.

To minimize this phenomenon, when representing rotations in general,

deciding which axis will take the top position in the hierarchy (similar to the hierarchy of fk controllers)

is called rotateOrder,

Gimbal Mode is a mode where only one axis can move, and it is a mode where this phenomenon can be visually observed.
</details>

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/23_IkArmVis.png)  

Hierarchy: **IKArm_CTL** ⊃ **sub controller** ⊃ **local controller**

Visibility is initially turned off. Please turn it on if needed.

**[soft] - Arms and legs the same**  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/24_soft.mp4" type="video/mp4">
</video>  

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/25_soft_L.mp4" type="video/mp4">
</video>  

1. Recommended usage 2) Usage based on controller length 3) Turning off soft

Adjusts the artificially distorted distance value for calculating the angle at which the arm extends, allowing for a smoother extension.

By default, as the controller moves further away, the arm extends. However, there may be situations where you cannot position the controller where you want it.

To fine-tune the position, after turning off soft,

Use the **[ikArm_vector_CTL]** while soft is turned on

to extend and fold the arm at the desired angle, making it more convenient to use.

By changing the minTransXLimitEnable attribute of **[ikArm_vector_CTL]** to off,

you can also fold the arm in the opposite direction.

Supports the **[stretchy]** feature.

Supports **[follow / followTarget]**. When follow is set to 10, it follows the followTarget.

follow 0: Follows the Sky.

followTarget: Main / RootX / Upchest / Head / Pelvis.

(If you need additional parts, please let us know.)

## **3-5. PoleArm**

- followTarget - ikArm: Same as the existing advanced skeleton.
  ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/26_PoleArm.png)  
  
- followTarget - Rotation

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/27_poleArm_rot.mp4" type="video/mp4">
</video>  
  
Using [**IkArm_pv_rot_CTL**], you can easily control the poleVector with rotation.

To address any possible distance issues, scaleZ is left open.

## **3-6. Bend Controller**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/28_BendStiffness.mp4" type="video/mp4">
</video>  

[**stiffness**]:

The numerical adjustment option for creating the shape of the bend curve is located within the middle controller of the bend.

## **3-7. Finger Attributes**

all default attributes are made with saved json files.

you can simply make new attributes with my python script.

# 4**. Leg**

## 4**-1. Skip explanation of the same section as arm setup**

- Supports stretchy functionality
- Supports soft functionality
- Supports sub/localVis
- Supports follow (0: Sky, 10: Main/RootX)

## 4**-2. makeT**  

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/28_5_makeT.png)  

When activating makeT in IKLocalLeg, the ankle will face forward.

Please kindly rotate the ankle naturally from the modeling team. Although it is advantageous for expressing a natural stride, there may be obstacles in accurate angle calculations and representations.

To give you the option, the makeT option has been included.

## 4**-3. PoleLeg**

- follow 10  
  
![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/29_pv_rot.png)  

Same as the existing advanced skeleton, but the following has been added:

**L_IkLeg_pv_rot_CTL**. You can easily move the poleVector with only scaleZ and rotateY.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/30_pv_rootFollow.png)  

- rootFollowWeight 0:

The default rotation angle of **L_IkLeg_pv_rot_CTL** follows the "Ankle".

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/31_rootFollowWeight0.mp4" type="video/mp4">
</video>  

- rootFollowWeight 10:

The default rotation angle of **L_IkLeg_pv_rot_CTL** follows the "RootX".

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/32_rootFollowWeight10.mp4" type="video/mp4">
</video>  

## 4**-4. Toe**

- **roll**
roll / rollStartAngle / roleEndAngle

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/34_RollStartAngle.png)  
rollStartAngle (red = X / blue = O)

When adjusting the roll, if the value of roll is less than or equal to rollStartAngle,
the bottom surface is attached.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/33_RollEndAngle.png)  
rollEndAngle

When adjusting the roll, if the value of roll is greater than or equal to rollStartAngle
and less than or equal to rollEndAngle,
the bending is maintained.

In other words, the smaller the difference between rollStartAngle and rollEndAngle,
the faster it unfolds,
and the larger the difference, the slower it unfolds.

- **toeBreak**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/35_ToeBreak.mp4" type="video/mp4">
</video>  

Moves up and down based on the ankle. Used for auxiliary purposes such as when sticking to the floor or when slipping.

- rock

Rotates between Inner and Outer sides as a pivot.

## 4**-5. SmartToes**

<video width="720" height="480" controls="controls">
  <source src="/assets/images/autorig/36_SmartToes.mp4" type="video/mp4">
</video>  

rotateY / rotateZ: Rotates the Toes as a pivot.

rotateX: Rotates the ToeTip as a pivot.

transZ: Rotates based on the ToeTip / Heel.

transX: Rotates based on rockInner/Outer.

## 4**-6. FKHip**  
![alt]({{ site.url }}{{ site.baseurl }}/assets/images/autorig/37_hipGlobal.png)  
Contains global orient settings.