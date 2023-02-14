## How Limelight Works
Limelight is a camera system that can detect AprilTags and reflective tape.
[Limelight Docs](https://docs.limelightvision.io/en/latest/)

## Vision.java
Nothing here yet.

## VisionComponent.java
The idea behind VisionComponent is that

```java
package frc.robot.component;

/**
 * Add your docs here.
 */
public interface VisionComponent {
    enum CameraMode {
        VisionProcessor, DriverCamera
    }

    /**
     * Determines whether any valid targets are detected by the VisionComponent
     * component. Targets are the final output of a vision component, after
     * thresholding and filtering contours. Usually, the vision component is
     * configurable, and the thresholds and contours can be changed.
     *
     * @return whether any valid targets are detected
     */
    boolean hasValidTargets();

    /**
     * Gets the horizontal offset (in radians) from the crosshair to the target. The
     * crosshair is essentially the origin from which offset to targets are
     * calculated. With a Limelight, this crosshair can be calibrated so that
     * different positions may be considered "on target" relative to the camera.
     * <br/>
     * <br/>
     * A positive angle means that the target to the right of the crosshair, and a
     * negative angle means that the target is to the left. Zero means that the
     * target is perfectly lined up horizontally with the crosshair.
     *
     * @return the horizontal offset
     */
    double getHorizontalOffsetFromCrosshair();

    /**
     * Gets the vertial offset (in radians) from the crosshair to the target. See
     * {@link #getHorizontalOffsetFromCrosshair()} for more details on crosshairs.
     * <br/>
     * <br/>
     * A positive angle means that the target is above the crosshair, and a negative
     * angle means that the target is below the crosshair. Zero means that the
     * target is perfectly level with the crosshair.
     *
     * @return the vertical offset
     */
    double getVerticalOffsetFromCrosshair();

    /**
     * Gets the percentage of the camera's FOV that is taken up by the target. 0%
     * means that the target does not exist, and 100% means that the camera is
     * completely covered by the target. This can be used as a rough measurement of
     * distance from the target.
     *
     * @return the percentage of area of camera view covered by the target
     */
    double getTargetArea();

    /**
     * Gets the rotation (in radians) of the target relative to the vision
     * processor. A positive value means the target is rotated counter-clockwise,
     * and a negative value means the target is rotated clockwise. The rotation is
     * within a range of 0 to pi/2 radians.
     *
     * @return the rotation of the target
     */
    double getSkew();

    /**
     * Sets the camera mode of the component. See {@link CameraMode} for more
     * details.
     *
     * @param mode the desired camera mode
     */
    void setCameraMode(CameraMode mode);

    /**
     * Sets the vision processing pipeline used by the component. A "pipeline"
     * refers to the process the vision component takes to calculate the location,
     * rotation, and size of the target. A vision processor can have multiple
     * pipelines so that each configuration can be switched out as needed.
     *
     * @param index - the index of the desired pipeline
     */
    void setPipeline(int index);
}
```

## LimelightVisionComponent.java
```java
package frc.robot.component.hardware;

import edu.wpi.first.networktables.NetworkTable;
import edu.wpi.first.networktables.NetworkTableInstance;
import frc.robot.component.TransformSupplier;
import frc.robot.component.VisionComponent;
import frc.robot.utility.Transform3D;

/**
 * An {@link VisionComponent} wrapper for a Limelight vision processor.
 */
public class LimelightVisionComponent implements VisionComponent, TransformSupplier {

    private NetworkTable table;

    /**
     * Creates a new VisionComponent component, which is essentially a wrapper
     * around the networktable entries modified by a Limelight.
     */
    public LimelightVisionComponent() {
        table = NetworkTableInstance.getDefault().getTable("limelight");
    }

    @Override
    public boolean hasValidTargets() {
        double value = getEntry("tv");
        if (value == 1) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public double getHorizontalOffsetFromCrosshair() {
        return Math.toRadians(getEntry("tx"));
    }

    @Override
    public double getVerticalOffsetFromCrosshair() {
        return Math.toRadians(getEntry("ty"));
    }

    @Override
    public double getTargetArea() {
        return getEntry("ta");
    }

    @Override
    public double getSkew() {
        double rawDegrees = getEntry("ts");
        double adjustedDegrees;
        if (Math.abs(rawDegrees) < 45) {
            adjustedDegrees = -rawDegrees;
        } else {
            adjustedDegrees = -(90 + rawDegrees);
        }
        return Math.toRadians(adjustedDegrees);
    }

    @Override
    public void setCameraMode(CameraMode mode) {
        if (mode == CameraMode.DriverCamera) {
            setEntry("camMode", 1);
        } else if (mode == CameraMode.VisionProcessor) {
            setEntry("camMode", 0);
        } // Maybe add exception for bad entry
    }

    @Override
    public void setPipeline(int index) {
        setEntry("pipeline", index);
    }

    private double getEntry(String key) {
        return table.getEntry(key).getDouble(0);
    }

    private void setEntry(String key, Number value) {
        table.getEntry(key).setNumber(value);
    }

    private double[] getCameraTranslationEntry() {
        return table.getEntry("camtran").getDoubleArray(new double[6]);
    }

    @Override
    public Transform3D getTransform() {
        double[] translation = getCameraTranslationEntry();
        return new Transform3D(translation[0], translation[1],
                translation[2], translation[3], translation[4],
                translation[5]);
    }
}
```