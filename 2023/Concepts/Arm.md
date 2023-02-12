The arm on our 2023 robot is a telescoping arm, which allows it to extend and retract in order to reach different heights. The arm has a motor that controls its movement and its position can be set using an encoder. The arm also has a tilt mechanism which is comprised of a motor and another encoder that allows it to tilt to different angles. The arm and the intake are synchronized to ensure efficient and fast movement during the game.

```java
import com.revrobotics.CANSparkMax; 
import com.revrobotics.ControlType; 

public class Arm { 
	private CANSparkMax armMotor; 
	
	public Arm(int motorID) { 
		armMotor = new CANSparkMax(motorID, CANSparkMaxLowLevel.MotorType.kBrushless); 
	} 
	
	public void moveArm(double position) { 
		armMotor.getPIDController().setReference(position, ControlType.kPosition); 
	} 
	
	public void configArm() { 
		armMotor.restoreFactoryDefaults(); 
		armMotor.setIdleMode(CANSparkMax.IdleMode.kBrake); 
		armMotor.setOpenLoopRampRate(0.25); 
		armMotor.setClosedLoopRampRate(0.25); 
		armMotor.setSensorPhase(true); 
	} 
}
```

