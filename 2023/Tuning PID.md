It can be done through a process of trial and error, where the gains are adjusted incrementally and the resulting behavior of the robot is observed. Here are some general guidelines for tuning the PID controller:

1.  Start with _kP_ at 0.1, _kI_ at 0.01, and _kD_ at 0.01.
    
2.  Increase the value of _kP_ until the robot starts to **oscillate**. Then set the value to half.
    
3.  Adjust the value of _kI_ until the robot hits the target angle at as fast a time as possible (which happens with higher _kI_ value), but a value less than a value where there's creeping overshooting.
    
4.  Adjust the value of _kD_ to **dampen the oscillations** of the robot. A high value of _kD_ will reduce the **amplitude** of the oscillations, but may also slow down the **response time** of the robot.
    
5.  Repeat the process of adjusting the gains until the robot maintains balance on the charging station with **minimal oscillation**.