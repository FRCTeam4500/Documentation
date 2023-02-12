The Math: $$u(t)=K_{\text{p}}e(t)+K_{\text{i}}\int _{0}^{t}e(\tau )\,\mathrm {d} \tau +K_{\text{d}}{\frac {\mathrm {d} e(t)}{\mathrm {d} t}}$$
Note: You don't need to know the math because we are going to be using an experimental method to determine PID values instead of using a theoretical math method to determine them. 

The _kP_, _kI_, and _kD_ values can be adjusted to optimize the performance of the PID controller for the specific robot and conditions it will be operating in. The optimal values for these gains will depend on the specific characteristics of your robot, including the weight distribution, motor performance, drivetrain responsiveness, and other factors.

_kP_ is how much the PID oscillates.

_kI_ is the speed at which you reach the ideal value but that can lead to overshooting if it's too high.

_kD_ dampens oscillation but can have problem with response time if too high. Dampening of oscillation is related to response time because if the dampening is too large, the robot won't be able to respond to overshooting as quickly.