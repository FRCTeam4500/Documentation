We need to balance for the 2023 season. We will be focused on balancing in autonomous.

We currently have 2 methods.

## Method 1: Simple, Easy [[thing]]

Use gyro.getPitch() in order to check rotation about the x-axis.

```java
private void balance() {
	AHRSAngleGetterComponent gyro = new AHRSAngleGetterComponent(I2C.Port.kMXP); // Port on the RIO
	boolean go = true;
	double angle; // Initial angle
	int timesEqual = 0; // Check target match in a row
	
	while (go) {
		angle = Math.toDegrees(gyro.getPitch());
	
		if (Math.toDegrees(angle) > 2) { // If starting on the front
			moveFieldCentric((angle-2)/10,0,0);
		} else if (Math.toDegrees(-2) < 2) { // If overshoot
			moveFieldCentric(-((angle-2)/10),0,0);
		} else { // If target match
			timesEqual++;
		}
		
		if (timesEqual > 10) { // 10 is a placeholder, if engage is secured
			go = false;
		}
			
		new WaitCommand(.05); // Stops continuous running
	}
}
```

The code above does not work, and is working on a simple system.
We have 2 degrees of freedom to engage on this year's charging station, so ideally if we stop at 2 degrees, then the remaining movement error can be eliminated.

## Method 2: Manual PID, Fast, Complicated

This method uses PID math in order to create smooth, fast movement to target angle.

Here is an example using Arcade Drive (forward and backward, left and right) -> drive.arcadeDrive(xSpeed, zSpeed); I have no idea why it's "z", deal with it :)
```java
// Initialize variables to store the readings from the gyro sensor 
double gyroAngle = 0.0; 
double gyroRate = 0.0; 
double gyroTime = 0.0; 

// Set the proportional, integral, and derivative gains for the PID controller 
double kP = 0.1; 
double kI = 0.01; 
double kD = 0.01; 

// Initialize the error, integral, and derivative terms for the PID controller 
double error = 0.0; 
double integral = 0.0; 
double derivative = 0.0; 

// Store the current time for use in the PID loop 
gyroTime = Timer.getFPGATimestamp(); 

// Continuously run the PID loop to balance the robot 

while (isAutonomous() && isEnabled()) { 
	// Update the gyro angle and rate readings 
	gyroAngle = gyro.getAngle(); 
	gyroRate = gyro.getRate(); 
	
	// Calculate the error between the target angle (0 degrees) and the current angle 
	error = 0.0 - gyroAngle; 
	
	// Calculate the integral of the error over time 
	integral += error * (Timer.getFPGATimestamp() - gyroTime); 
	
	// Calculate the derivative of the error with respect to time 
	derivative = (error - prevError) / (Timer.getFPGATimestamp() - gyroTime); 
	
	// Store the current error for use in the next iteration 
	prevError = error; 
	
	// Update the current time for use in the next iteration 
	gyroTime = Timer.getFPGATimestamp(); 
	
	// Calculate the correction value using the PID gains and error terms 
	double correction = kP * error + kI * integral + kD * derivative; 
	
	// Apply the correction value to the drive motors to keep the robot balanced 
	drive.arcadeDrive(0.0, correction); 
}
```

### Explanation
This code uses a PID (proportional-integral-derivative) controller to balance the robot on the charging station. The gyro sensor is used to measure the current angle of the robot, and the PID controller uses this information to calculate a correction value that is applied to the drive motors to keep the robot balanced. The _kP_, _kI_, and _kD_ variables store the gains for the proportional, integral, and derivative terms of the PID controller, respectively. The _error_ variable is the difference between the target angle (0 degrees) and the current angle, the _integral_ variable is the sum of the errors over time, and the _derivative_ variable is the rate of change of the error. The correction value is calculated by multiplying the error terms by the respective gains and summing them up. The "_drive.arcadeDrive_" function is used to apply the correction value to the drive motors. This will cause the robot to rotate on the charging station until it reaches the target angle of 0 degrees and stays balanced.

The adaptation of this arcadeDrive code into our Odometric/PathFollowingSwerve is a follows:


The Math: $$u(t)=K_{\text{p}}e(t)+K_{\text{i}}\int _{0}^{t}e(\tau )\,\mathrm {d} \tau +K_{\text{d}}{\frac {\mathrm {d} e(t)}{\mathrm {d} t}}$$
Note: You don't need to know the math because we are going to be using an experimental method to determine PID values instead of using a theoretical math method to determine them. 

The _kP_, _kI_, and _kD_ values can be adjusted to optimize the performance of the PID controller for the specific robot and conditions it will be operating in. The optimal values for these gains will depend on the specific characteristics of your robot, including the weight distribution, motor performance, drivetrain responsiveness, and other factors.

_kP_ is how much the PID oscillates.

_kI_ is the speed at which you reach the ideal value but that can lead to overshooting if it's too high.

_kD_ dampens oscillation but can have problem with response time if too high. Dampening of oscillation is related to response time because if the dampening is too large, the robot won't be able to respond to overshooting as quickly.

Tuning the PID controller can be done through a process of trial and error, where the gains are adjusted incrementally and the resulting behavior of the robot is observed. Here are some general guidelines for tuning the PID controller:

1.  Start with _kP_ at 0.1, _kI_ at 0.01, and _kD_ at 0.01.
    
2.  Increase the value of _kP_ until the robot starts to **oscillate**. Then set the value to half.
    
3.  Adjust the value of _kI_ until the robot hits the target angle at as fast a time as possible (which happens with higher _kI_ value), but a value less than a value where there's creeping overshooting.
    
4.  Adjust the value of _kD_ to **dampen the oscillations** of the robot. A high value of _kD_ will reduce the **amplitude** of the oscillations, but may also slow down the **response time** of the robot.
    
5.  Repeat the process of adjusting the gains until the robot maintains balance on the charging station with **minimal oscillation**.


It's important to note that the optimal values for the gains will change based on different conditions, such as the battery level of the robot, the surface the robot is operating on, and other variables. It may be necessary to adjust the gains during a competition to ensure the best performance.

We can bypass the battery power limitation by assuming a completely charged battery at the start of each match. Values will either need to be changed according to sample acrilic, or we will need to find proper fields.

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