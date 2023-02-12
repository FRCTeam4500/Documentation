We need to balance for the 2023 season. We will be focused on balancing in autonomous.

We currently have 2 methods.

<<<<<<< Updated upstream
## Method 1: Experimentation with Rotation Around Axes
=======
## Method 1: Simple, Easy
>>>>>>> Stashed changes

### Explanation of this Method
**gyro.getPitch():** check rotation around the x-axis.
**gyro.getRoll():** check rotation around the y-axis.
**gyro.getYaw():** check rotation around the z-axis.
We play around with values of getPitch(), getRoll(), and getYaw() until we find ideal value ranges for them and then we program those ranges in if-statements.
It may be necessary to adjust the values during a competition to ensure the best performance which would require continuous testing :(

### Sample Code of this Method (post-experimentation so we've determined ranges)
The range of -2 to 2 for roll, pitch, and yaw are fake experimental values for the ranges of angles that the robot should be in

```java
private void balance() {
	AHRSAngleGetterComponent gyro = new AHRSAngleGetterComponent(I2C.Port.kMXP); // Port on the RIO
	boolean go = true;
	double angle; // Initial angle
	int timesEqual = 0; // Check target match in a row
	
	while (go) {
		anglePitch = Math.toDegrees(gyro.getPitch());
		angleRoll = Math.toDegrees(gyro.getRoll());
		angleYaw = Math.toDegrees(gyro.getYaw());
		
		if(anglePitch > 2 || anglePitch < -2 || angleRoll > 2 || angleRoll < -2 || angleYaw > 2 || angleYaw < -2) {
			if (anglePitch > 2) {
				moveFieldCentric((anglePitch - 2) / 10, 0, 0);
			if (anglePitch < -2) {
				moveFieldCentric(-((anglePitch - 2) / 10), 0, 0);
			}
			if (angleRoll > 2) {
				moveFieldCentric(0, (angleRoll - 2) / 10, 0);
			if (angleRoll < -2) {
				moveFieldCentric(0, -((angleRoll - 2) / 10), 0);
			}
			if (angleYaw > 2) {
				moveFieldCentric(0, 0, (angleYaw - 2) / 10);
			if (angleYaw < -2) {
				moveFieldCentric(0, 0, -((angleYaw - 2) / 10));
			}
		}
		
		else { // If target match
			timesEqual++;
		}
		
		if (timesEqual > 10) { // 10 is a placeholder, if engage is secured
			go = false;
		}
			
		new WaitCommand(.05); // Stops continuous running
	}
}
```

## Method 2: Manual Tuning of [[PID]]

This method uses [[PID]] math in order to create smooth, fast movement to target angle.

### [[Arcade Drive PID]] Example
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
double kP = 0.1;
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