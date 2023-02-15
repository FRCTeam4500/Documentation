We need to balance for the 2023 season. We will be focused on balancing in autonomous.

We currently have 2 methods.

## Method 1: Experimentation with Rotation Around Axes

### Explanation of this Method
**gyro.getPitch():** check rotation around the x-axis.
**gyro.getRoll():** check rotation around the y-axis.
**gyro.getYaw():** check rotation around the z-axis.
We play around with values of getPitch(), getRoll(), and getYaw() until we find ideal value ranges for them and then we program those ranges in if-statements.

### Sample Code of this Method 
The range of -2 to 2 for roll, pitch, and yaw are the ranges of angles that the robot should be in. Also, the function is moveRobotCentric() because depending on where the robot is facing, it should move all around the charging station accordingly to adjust for the imbalance. To clarify, moveRobotCentric() is to have the robot move relative to the direction it is facing and moveFieldCentric() is to have the robot move relative to the direction of the field (which means that no matter which way the robot is facing, the robot will move the same way if you hold down move forward).

```java
private void balance() {
        AHRSAngleGetterComponent gyro = new AHRSAngleGetterComponent(I2C.Port.kMXP);
        double anglePitch;
        double angleRoll;
        int timesEqualRoll = 0;
        int timesEqualPitch = 0;

        while (timesEqualRoll < 10 || timesEqualPitch < 10) {
        anglePitch = Math.toDegrees(gyro.getPitch());
        angleRoll = Math.toDegrees(gyro.getRoll());
        if (false) { // Nate code
        if (-2 < anglePitch && anglePitch < 2) {
        timesEqualPitch++;
        }
        if (-2 < angleRoll && angleRoll < 2) {
        timesEqualRoll++;
        }
        moveRobotCentric(-anglePitch/30, -angleRoll/30, 0);
        } else { // Vincent Bryan code

        if (anglePitch > 2) {
        timesEqualPitch = 0;
        moveRobotCentric((anglePitch - 2) / 10, 0, 0);
        } else if (anglePitch < -2) {
        timesEqualPitch = 0;
        moveRobotCentric(-((anglePitch + 2) / 10), 0, 0);
        } else {
        timesEqualPitch++;
        }
        if (angleRoll > 2) {
        timesEqualRoll = 0;
        moveRobotCentric(0, (angleRoll - 2) / 10, 0);
        } else if (angleRoll < -2) {
        timesEqualRoll = 0;
        moveRobotCentric(0, (angleRoll - 2) / 10, 0);
        } else {
        timesEqualRoll++;
        }
        }
        new WaitCommand(.05); // Stops continuous running
        }
        }
}
```

## Method 2: Manual Tuning of [[PID]]

This method uses [[PID]] math in order to create smooth, fast movement to target angle.

### [[Arcade Drive PID Balancing]] Example
It's not super ideal to use but if you want to refer to it, click it on the link above.

### Explanation of our Actual [[PID]]
We first [[Tuning PID|tune]] the [[PID]] to get desired kP, kI, and kD values.

It's important to note that the optimal values for the gains will change based on different conditions, such as the battery level of the robot, the surface the robot is operating on, and other variables. It may be necessary to adjust the gains during a competition to ensure the best performance which would require continuous testing :(

We can bypass the battery power limitation by assuming a completely charged battery at the start of each match. Values will either need to be changed according to sample acrylic, or we will need to find proper fields.

### Sample Code of this Method (Post-Experimentation so we know kP, kI, and kD values)
Still need to adjust this code because we're thinking of making it so there's a [[Joystick|joystick]] button changing the kP, kI, and kD values instead of having it automatically happen. However, having it manually done would mean that we would need to continually un-program those joystick buttons after we finish testing for a specific competition :(

```java
// Define the desired angle
double desiredAngle = 0.0;

// PID controller variables
double kP = 0.1; // Lowest val, only up from here
double kI = 0.01;
double kD = 0.01;

// Integral error
double integralError = 0.0;

// Previous error
double previousError = 0.0;

// Main control loop
while (true) {
    // Get the current pitch angle from the AHRS
    double currentPitch = getAHRS().getPitch();

    // Calculate the error between the desired angle and current pitch
    double error = desiredAngle - currentPitch;

    // Update the integral error
    integralError += error;

    // Calculate the derivative of the error
    double derivativeError = error - previousError;
    
    // Calculate the control signal
    double controlSignal = kP * error + kI * integralError + kD * derivativeError;

    // Update the previous error
    previousError = error;

    // Calculate the desired motor speeds
    double[] motorSpeeds = swerveDriveKinematics(controlSignal);

    // Set the motor speeds
    setMotorSpeeds(motorSpeeds);

    // Wait for the next control loop iteration
    waitForNextIteration();
}
```

## Method 3: robot-centric movement
The idea here is that when the robot is pitching,
that means we need to move forward (or backward).
And when the robot is rolling,
that means we need to move left (or right).
Then, we use `getPitch` and `getRoll` from the gyro,
with `robotCentricMove`, and some carefully-selected constants,
to move the robot accordingly.
Sample code:
```java
	double k1,k2;
	double pitchAngle = Math.toDegrees(gyro.getPitch());
	double rollAngle = Math.toDegrees(gyro.getRoll());
	robotCentricMove(pitchAngle*k1, rollAngle*k2, 0);
```
