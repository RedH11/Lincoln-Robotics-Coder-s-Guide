# Lincoln-Robotics-Coders-Guide

## Setting Up Android Studios by Hunter Webb

This is pretty simple, but just install Android Studios from [here](https://developer.android.com/studio). Also, the platform that you will likely be using it Github to share code which you are going to familiarize yourself with, and while some will opt to learn the command line code to use it (pro gamer flex right there) I, as a normal person trying to maintain a social life alongside robotics, opt to use Github Desktop which is a UI interface that allows you to use Android Studios easily with Github by cloning the repository and then opening it with Android studios and pushing all commits and pulling all changes through the program. Otherwise, a third alternative is to learn how to use the VCS tab in an Android Studios Project to push and pull code which is also simple enough.

With that done, open your project and simply start coding after you have pulled the right repository to use for Android Studios.

## Using EasyOpenCV by Hunter Webb

EasyOpenCV is a great way to take images from your phone's camera and do useful stuff with it! In the year that I did robotics, I even was able to wrangle it to do a basic neural network to detect different objects. It worked! Buttt... the phone couldn't handle the processing and 1 frame every few seconds wouldn't cut it in competition, so we opted to just use the basics of seeing the colors on the screen to determine what was there with our code. 

To start the process, you can always reference the code in CLoud-Robotics-FTC in my branch, HunterW, and in teamcode there is the Vision folder with my neural network code that contains the camera.

The first step is to initialize the camera so that you can start its stream when you want to. The method I used was:
```
public void initCamera() {
        int cameraMonitorViewId = hardwareMap.appContext.getResources().getIdentifier("cameraMonitorViewId", "id", hardwareMap.appContext.getPackageName());
        phoneCam = OpenCvCameraFactory.getInstance().createInternalCamera(OpenCvInternalCamera.CameraDirection.BACK, cameraMonitorViewId);
        phoneCam.openCameraDevice();
        phoneCam.setPipeline(new StageSwitchingPipeline());
        phoneCam.startStreaming(rows, cols, OpenCvCameraRotation.UPRIGHT); //display on RC
}
```


This uses the class StageSwitchingPipeline which communicates the frame or MAT (for Matrix of pixels) to the code that you want to use. In my code I embedded this in another class and used it to pull the MAT's for the neural network to analyze. 

```
class StageSwitchingPipeline extends OpenCvPipeline {
	...
	code
	...
}
```

Alternatively, you can save the frames to the phone or display them on the screen as shown in my code, and you can also draw shapes on the MAT. Therefore, if you want the robot to look at a certain part of the screen to see if something is there when it is in a reproducible position in competition (likely the starting position) you can draw a shape to find the coordinates of where the object(s) will/won't be to use for pathing/decision making.



## Using RoadRunner by Jonathan Zhao

So you want to learn how to build trajectories, huh kid?

Well, here’s a guide to do just that.

**Please note that everything in Roadrunner is subject to change, we utilized v0.4.7 and are tailoring this tutorial towards v0.5.0, the newest version at this tutorial’s writing. Moreover, this is essentially a more explicit version of the tutorials that can already be found on the Roadrunner official website

**Some useful complementary packages include:
- Proguard
	- This circumvents that pesky method limit Android has. Should come installed with the quickstart.
- FTCDashboard
	- This is used often in the quickstart to display various diagnostics and cool-looking graphs. Should come installed with the quickstart.
- RevHubExtensions2
	- This contains many useful RevHub functions. The most useful (for us) is probably the bulk reading option, which allows us to gather data in one go instead of performing sequential reads (which adds a ~10ms delay per read). If you’re a smart kiddo, go ahead and try to optimize the various repetitive operations such as motor encoder position reading. May come installed with the quickstart.

**If you ever get lost, sample code from our year can be found at [this link](https://github.com/FTC7604/SkyStone/tree/roadrunnerDev/TeamCode/src/main/java/org/firstinspires/ftc/teamcode)**

There will be TWO main parts to building a trajectory:
- The Drive
- The TrajectoryBuilder

Shall we start?
### Step ONE: The Drive

The backbone of all your movements will be the Drive. For those of you who aren’t lazy bastards, there’s an option to custom-create your Drive from scratch, but beware, you’ll need to know exactly what you’re doing or you’ll just end up crashing your robot into the wall at ludicrous speed.

Anyhow, for the lazy majority of you, there is a customized project called **Roadrunner-quickstart** which, as the name implies, contains everything needed for a quickstart to Roadrunner. This includes various files in **TeamCode** under the **drive** folder which contain all sample drive bases. Under the **opmode** folder are a bunch of tests for tuning and whatnot: pay no attention to those fow now. At this point, you’ll need to take a good, long gander at your robot. Is it mecanum? Is it holonomic? Or are you lazy and just went for a glorified push-bot? Choose the right drive base accordingly.

Afterwards, go to the constructor. If you don’t know what a constructor is, kindly ask someone else to be the coder. There you will (hopefully) find a lot of TODO comments near the bottom of the constructor. These mainly include **changing names until it fits your hardware configuration, remapping the axes of the IMU, reversing a few motors, and then finally having the option to use localization.** This means odometry or any external ways of calculating your robot’s position on the field. If you have odometry wheels and encoders, go ahead and use them! They can improve your accuracy on the field significantly, although you could go ahead and try to use Roadrunner without localization. Roadrunner will just estimate using encoder readings from your powered wheels, which slowly drifts off. If you want to use localization go ahead and read the section below. Otherwise, skip ahead.
	
#### Using Localization
You poor bastard. You mad lad. You absolute shell of a human being. You desire more tuning than you have to do? I respect the hell out of that. Here’s more tuning.
Customize the StandardTrackingWheelLocalizer.java to your odometry setup. If you’re using three wheels to track position, you can leave it mostly as is besides a few minor adjustments to a few values. These include:
- TICKS_PER_REV
	- This is the ticks per revolution of your odometry encoders, it should be found online on the store page or whatever.
- WHEEL_RADIUS
	- This is the radius of your odometry wheels.
- GEAR_RATIO
	- This is the gear ratio, output to input, although in most cases it should be one.
- LATERAL_DISTANCE
	- This is the distance between your front/back odometry wheels
- FORWARD_OFFSET
	- This is the offset of your strafe odometry wheels

Oftentimes odometry wheels are not perfectly set up to align with this, so you can fiddle around with the Pose2d values you see in the constructor inside an array. These denote the positions of your odometry wheels relative to the center of your drive base: x is front/back where going towards the front of your robot is positive, y is left/right where going towards the left of your robot is positive, and the last value is the orientation of your wheel.

If you’re using two wheel tracking (one front/back wheel, one strafing wheel, and an external gyro), you’ll need to change the class to fit the **TwoTrackingWheelLocalizer** class. This can be done by getting rid of one of the wheels (probably either left or right), adding in your gyro or whatever you’re using to measure heading to the class, changing the word after extends at the top to “TwoTrackingWheelLocalizer” and then just writing in any required methods (such as getHeading()).
Back to the Drive
	So, you think you’re all set up? No; like all things in life, it won’t work straight out the box. You’ll need a ton of tuning and whatnot. Guess where these are? That’s right: spread out all over the place in the quickstart, and halfway through the season you’ll find this one constant you didn’t tune correctly so you’ll have to take a week’s break for just tuning, although state will have been cancelled so, guess what, all your efforts over the past few months integrating a brand new library were wasted, but I digress. I will take some time to list out all the important tuning constants.
Tuning
	The vast majority of tuning will be found in DriveConstants.java. Very little can be done automatically here, because every robot built is unique in its own special little way. Please remember that you need to be careful with division, as things like “1 / 5” will return 0, while “1.0 / 5.0” will return 0.2 due to Java int and double behaviors. Moreover, there is a great graphic on the Roadrunner documentation website that describes how to properly tune a few constants. At the time of writing, there will be the following constants to tune:
- TICKS_PER_REV
This constant denotes the amount of ticks per revolution of the motor. Check online for this one for your motors.
- MAX_RPM
This constant denotes the maximum speed of the motor. Check online for this one for your motors.
- RUN_USING_ENCODER
	- This constant denotes whether you will be running using encoders. Quick rundown of the difference between true/false:
		- TRUE
			- Motor velocity will be maintained by a PID and affected by voltage. It may be slightly slower than simply running by power.
		- FALSE
			- Motors are directly sent power indiscriminately by a dictatorial battery. It may be faster than using the encoder PIDs.
- MOTOR_VELO_PID
	- If you choose to run using encoders, you’ll have to deal with tuning it. Familiarize yourself a bit with PIDs before delving into this! Poke around a bit online, read the Roadrunner excerpt on PIDs, or just read the terrible summary at the bottom of this tutorial. Tuning can be done using the DriveVelocityPIDTuner.java opmode by connecting using the FTCDashboard that complements Roadrunner.
- WHEEL_RADIUS
	- Should be self-explanatory. Sometimes, movements may not be accurate even if this is dead accurate, so you might be able to fiddle around with this. You can test this with StraightTest.java under opmodes
- GEAR_RATIO
	- Please note this excludes the internal gearing of any motors. It is measured as the output divided by the input, or the number of revolutions of the wheel per revolution of the corresponding motor.
- TRACK_WIDTH
	- This is the distance between wheels on opposite sides of the robot. On a car, for example, it would be the distance between both of your front wheels or between both of your back wheels. If your robot doesn’t turn properly, it might have something to do with this. Calibration can be done with the TrackWidthTuner.java opmode.
- Feedforward control
	- These only need to be tuned if you choose not to run using encoders. It enables good feedforward control for wheels. We haven’t had much experience tuning these, so you may just need to look up in the documentation or online how other people have done it.
	- kV
	- kA
	- kStatic
	- All three of these represent their own corresponding constants. I never really took physics, so I don’t know what the hell’s going on here, but I do know that you can tune these using an opmode called the DriveFeedforwardTuner.java. It involves some time and effort, as linear regressions on data taken from powering the wheels are used to calculate these constants.
- BASE_CONSTRAINTS
	- These are perhaps the most important constants in terms of speed. If you want to be the next Lightning McQueen, tune these perfectly for your robot. Too much will lead the robot to quickly break itself (never exceed >80% of your robot’s actual capability!), and too little will slow you down to a crawl. I believe the constants are ordered as followed, although check documentation to make sure you don’t mix them up:
		- Max velocity
		- Max acceleration
		- Max jerk
		- 		- Max angular velocity
		- Max angular acceleration
		- Max angular jerk

Oh, you thought tuning was over? No, sir/ma’am, there’s even more. These may be found in your corresponding drive classes. Mecanum/holonomic/tank have their own corresponding parameters. We used mecanum, which includes heading and translational (strafing) PID constants, but we’re not sure for the others. Those will involve you personally investigating them, and definitely not me. At the very end, after running all the various tuners, see all your hard work pay off with the lame SplineTest.java. Have fun with your first month of using Roadrunner.

### Step TWO: The TrajectoryBuilder

The hard part’s finally done, unless you mess up tuning somewhere along the way in which case you’ll have to perform every test again. Anyhow, after all that, you can finally build some cool ass paths, right? Yes. No more catches.

As a quick setup, it’s important to set the pose estimate of the drive to wherever you start. This can be done using the setPoseEstimate(Pose2d) method. This way, you can create trajectories using absolute positions relative to the field rather than relative to your robot.

The core part of trajectory building is, unsurprisingly the TrajectoryBuilder which can be fetched from the drive class using their public trajectoryBuilder(...) methods. To actually force your robot to follow the trajectory, you should call the drive’s followTrajectory(Trajectory) method. Afterwards, it’s essentially up to you to explore all the different pathing possibilities. The main ones we used were splineTo(Pose2d) and strafeTo(Vector2d). We also used reverse(), but that method has been removed in v0.5.0 and has been replaced with setting whether the TrajectoryBuilder is reversed or not within its constructor. Aside from that, we also sometimes used forward(double), back(double), strafeLeft(double), and strafeRight(double). Some work better than others for some reason. A full list of the 0.5.0 methods can be found in this documentation. To finish a trajectory, simply call .build() at the very end of the long list of methods, which will return a Trajectory instance. As an example, here’s a quick writeup to reverse to the center of the field (0, 0) from wherever it is, and then move 5 inches:

```
drive.followTrajectory(
	drive.trajectoryBuilder(drive.getPoseEstimate(), true)
		.splineTo(new Pose2d(0, 0, 0)) // Drives the robot in a spline curve to the point 0, 0 with a heading (IN RADIANS) of 0
		.forward(5) // Moves the robot forward 5 inches
		.build()
);
```

Some methods take Vector2d as an argument, while some take Pose2d as an argument. The difference is that a Vector2d is purely position, while a Pose2d ends with the heading. Please note we are by no means experts at Roadrunner, so there are various things we never really looked into. These include HeadingInterpolators, which allow you to customize how the robot turns in its movements, and adding markers, which allow commands to execute at certain points along the trajectory. Lastly, while building trajectories using the TrajectoryBuilder, it is very important to keep the following ideas in mind:
Trajectories default to going forwards. If you tell your robot to move backwards by an inch without properly reversing it, it will do a whole ass circle.

Trajectories have not (in our experience) done well when combining methods meant for different movement styles. For example, using basically any other methods with splineTo will create the most scuffed trajectory you’ll ever see.
There is an option to run trajectories without blocking the main thread (you can do other things while the trajectory is chugging along). This usually comes from adding “Async” to the end of the method name, such as followTrajectoryAsync. To move the robot along the trajectory, you have to call in a loop the update() method, which updates the robot’s estimated position and sets its power. This should be called in a while loop that runs until the isBusy() method returns false.

### (Optional) Step THREE: The TrajectoryGenerator

TrajectoryBuilders too simple for you? Want a real challenge? Well, you’re absolutely stupid. However, I will present you the option of doing so, although I never delved into it, so this part you will have to figure out on your own.

One way to further customize your trajectories to your absolute liking is using a combination of the TrajectoryGenerator and either the PathBuilder or PathSegments. The TrajectoryGenerator is used solely to generate a Trajectory, which the robot can follow, from a Path instance by calling TrajectoryGenerator.generateTrajectory(Path, DriveConstraints). A PathSegment is what it sounds like: one segment of a larger Path. These go deep, ranging from things like LineSegments to QuinticSplines to LinearInterpolators and a whole bunch of other stuff. All you have to do is convert the special segments into the more generic PathSegment, put them in a list of some sort, and then generate a Path using the constructor Path(List<PathSegment>). Alternatively, you can use a PathBuilder, which is basically the same as a TrajectoryBuilder. The main difference between a Path and a Trajectory is that a Path contains no motion profiles, only the raw movement of the robot, while a Trajectory takes into account the physical constraints of the robot and adjusts motion profiles accordingly. Either way, you’ll have to convert whatever you have into a Trajectory for the robot to be able to follow it.

### PID Tuning

PIDs are traditionally used as control systems. It is an acronym of Proportional Integral Derivative, which the calculus kiddos will understand right quick. So how does PID actually work? Well, it takes the current state of the system, looks at the previous errors, and at the endpoint of the system (where you want it to be), and then makes adjustments as needed. The way the change is calculated is by multiplying the P term by the proportional value, the I term by the integral value, and the D value by the derivative value.
Proportional is the linear aspect of the system: the system will linearly go towards its endpoint from its current state. The following table illustrates this with a P value of 1:

| Current State |     Goal      |    Change     |
| ------------- | ------------- | ------------- |
|     1.0       |     0.0       |    -1.0       |
|     100.0     |     0.0       |   -100.0      |
|    -5.0       |     0.0       |    +5.0       |

So we should only use P, right? Sadly, life is not always that easy. Excessive P values lead to oscillations since the system continuously overcorrects the error, overshooting the endpoint and constantly going back and forth between too much and too little. In the case of motor velocity, a pure P controller (only P, other constants are 0) is probably fine, although one would have to be careful as to not overtune P.
That’s only one third of it all. There’s still integral and derivative! Now, an integral is essentially the sum of a set of values over time. In our case, the integral represents the sum of the error over time multiplied by the time difference between when we read those errors, so it’ll keep it smaller than it would be. All of this is done automatically, so unless you really want to delve into the nitty gritty, don’t worry about it (hey, look, my team’s name). All you really need to know is that adjusting the integral term pushes the current state closer to the endpoint. For example, if for some reason a pure P controller is unable to reach the end goal, you could sprinkle in a little I to push it closer to the end. However, this adds instability to the system, where certain circumstances might lead the controller to spaz out. Hopefully you don’t do that, but only time will tell. Note that, as the Roadrunner page discusses, it is usually a bad idea to try to add I in, and you would more likely use something called a feedforward controller which adds a constant to the change to counteract any underlying factors, such as gravity or friction. I’m not sure if this is automatically utilized or not in Roadrunner, so it may require you to customize the code to your liking.
Lastly, there’s the derivative term. A derivative is simply the rate of change of something at that exact moment. We can’t do that, so what we can do is get a rough estimate using the magic of first principles. In other words, we take the previous error reading, subtract that from our current error reading, and then divide by the time difference. This is also automatic. What this will do is dampen changes, causing the system to be slightly slower at responding to changes. It is best used to stop oscillations from P. In other words, it may be best to tune P-D-I, which is less catchy that PID but may work better. The biggest downside to using this constant for drive velocity is that, when our team used it, it may have caused our robot to be slower to respond to errors while following trajectories, causing it to be off. Thus, be careful.
All in all, **be careful tuning your PIDs**. They form the foundation of how well your robot follows trajectories, and messing it up may cause large errors much later down the line.

# Tips and Tricks for Making an Auto with Roadrunner by Hunter Webb

After the knowledge storm you have been pounded with in the previous section you are probably ecstatic to start testing your robot. 

But wait?! I decided to skip this section and test my robot by myself and now it is just turning in circles, and now i'm confused as to why nothing is working. Now, my friend, you have completed the first step of creating the most perfect roadrunner code, failing miserably. 

I had my robot kamakazi into three different walls, the shins of a builder who was mocking the robots movement, and tornado spin in circles across the field so many times before I had a really solid auto. This is all part of the process, and **I am now here** to show you how to quickly and precisely code to get your robot going correctly. 

Firstly, tell yourself that you are valuable because you are, and what makes you valuable is your **TIME** which **CANNOT** be wasted when you are working. So a few ground rules for our pilgrimage for the auto we want.

1. NO building in our auto time
2. NO having other people want to drive around and mess with the robot
3. YES having a sidekick builder to help you set up the field / plug in the phone is great
4. NO testing once and assuming that it works
5. YES take breaks
6. NO NO NO NO NO STARTING YOUR AUTO THE NIGHT, DAY, OR WEEK BEFORE THE COMPETITION
	- It is death, trust me here, I am on your side. At least have two weeks before the tournament to start it, and I was able to do that with two free periods every other day to just do the auto. 
	
Now, lets get to work.

## Pre-Testing Tips

### Think about the Path of your Full Auto

Have a clear picture in your mind or physically on paper of how you want the robot to move and where you want it to move. We can't be wasting our time making that idea clear while we have the robot zooming around. 

### Zoom but Not Too Much

Start with a lower max velocity / acceleration in roadrunner for your robot (in the DriveConstants call) initially and work up from there, but keep in mind that wheel slippage from moving faster can mess with your pathing.

### Strafing is Wack

I used strafing in a critical way in our auto, but it completely messed up the internal localization (it thought that it had strafed 10 inches farther than it had). Also, depending on the battery strafing was wack. Although, this was without odometry so you would likely have more luck there, but I would still use strafing with caution.

### Heading is in Radians

THE HEADING IS IN RADIANS, REPEAT, THE HEADING IS IN RADIANS AND WATCH KHAN ACADEMY IF YOU'RE LIKE WTF ARE RADIANS. 
The heading is the third parameter in the Pose2D class they use, so the following code has the robot ending up facing 180 degrees from its initial heading

```
.splineTo(new Pose2d(15, 20, Math.PI))

// Math.PI == 180 degrees
// Math.PI / 2 == 90 degrees
// -Math.PI / 2 == -90 degrees

```

## Testing Tips

### Localization

This is where Jonathan and I split on how we made our autos. He preferred to have (0, 0) be the center of the field like roadrunner uses for their GUI. However, I found it much more intuitive to have (0, 0) be where the robot began for my autos which ends up being just about as good as Jonathan's method with some advantages (ex. you can code without needing the GUI) and disadvantages (ex. you have to transform coordinates in the GUI to be what you want). 

However, the following tips still apply regardless of which way you want to do it, and I trust you to do what is best for you. Mostly because it doesn't matter whether or not I trust you. But be assured, I do.

If you choose my method, a helpful comment I would put into documents to help me work out which way the robot would go with coordinate changes is here:
```
/*
        Notes:

        (In terms of the original heading)
        -Y: Robot goes to a position towards the right
        Y: Robot goes to a position towards the left
        X: Robot moves to a position Fowards from the current position
        -X: Robot moves to a position Backwards from the current position

	Positive angle turns left
        Negative angle turns right
 */
```

### Driving in Parts

One of the biggest advantages that I believe my team had in our auto was our philosophy of *perfection in pieces*.

The **BEST** way for you to make an auto quickly and accurately is to take that image of what you want your path to be, and break it down into piece to make. In our competition format, this included picking up a lego block thing and putting it on a platform and going back for more lego blocks. 

Thus, the path was broken down into:
1. Pick up the block 
2. Drive to the platform
3. Drop the block off
4. Drive back to where the other blocks are
5. Pick up another block
...

Then we code each step in roadrunner with a different trajectory builder and wait()/Thread.sleep() methods when needed for motors and servos to do motor/servo things. 

This step by step process ensures that you don't rush ahead to Step 3: Drop the block off when you can't even accurately pick up the block in the first place. Then, when you fix picking up the block, your step three doesn't really work because the robot is in a different place with a different heading. 


### Small Angle Corrections

IMU's (aka Gyroscopes) aren't perfectly accurate. But, they seem to be consistantly inaccurate which is amazing. That means that with our heading we can add or subtract small angle values to make the robot have a heading that is straighter. 

Ex:

```
facingBackAngle = Math.PI/2;
correctionAngle = Math.PI/16; # Some small angle which can shift the angle to correct it

drive.followTrajectory(
	drive.trajectoryBuilder(drive.getPoseEstimate(), true)
		.reverse() # NOTE: THIS IS DEPRACATED IN THE RECENT VERSION SO READ THE ROADRUNNER DOCUMENTATION 
		.splineTo(new Pose2d(30, 20, facingBackAngle + correctionAngle)) // Aligned to face backwards when going to the platform
		.build()
);
```


