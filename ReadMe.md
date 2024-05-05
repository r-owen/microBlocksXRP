microBlocks libraries to run the wheels on the XRP robot kit.

# Contents

Contains the following microBlocks libraries:

- `PID.ubl`: PID (proportional, integral, derivitive) controllers.
- `DC Motors.ubl`: control DC motors that have incremental encoders.
- `XRP.ubl`: control the WPI XRP robot kit.

# PID.ubl

One or more PID (proportional-integral-differential) controllers.
Each controller is specified by a different index: 1, 2, 3, ...

PID controllers are used to control a system using feedback: measure an error and use this to apply a correction.
A common use case is operating an actuator with an encoder, such as a robot wheel, to move a specified amount.
For an actuator, you can determine the error by comparing where you want to be with where the encoder says you are.
Feed the measured error into `pid_computePID` and scale the resulting correction to determine a drive signal.
See *Recommendations for using `pid_computePID`* for more information.

The main blocks are:

- `pid_computePID`: compute the next PID correction for the specified PID loop.
  The inputs are as follows:
  - `index`: index of the PID loop: 1, 2, 3, ...
  - `error`: error to correct
  - `pCoeff`: proportional coefficient (corr/error)
  - `iCoeff`: integral coefficient (corr=msec/error)
  - `dCoeff`: derivitive coefficient (corr/error=msec)
  - `maxIntegral`: maximum absolute value of the integrated error; ignored if 0
- `pid_resetPID`: reset the specified PID loop.
  It may be useful to reset a PID before commanding a new move.
- `pid_constrainValue`: limit the range of a value.
  This is provided to constrain the drive signal to useful values.
  The algorithm is as follows (where `|value|` means the absolute value of `value`):
  - If `|value| < deadband`: return 0.
    This can keep the system from jittering or whining after a move.
  - If `|value| < minimum`: return `minimum` with the sign of `value`.
    This prevents applying a correction that is too small to do anything useful.
    For example the WPI XRP motor drives cannot reliably move when the effort is less than about 200.
  - If `|value| > maximum`: return `maximum` with the sign of `value`.
    This prevents demanding more than the system can handle.
    For example the WPI XRP motor drives have a maximum "effort" of 1023.

Recommendations for using `pid_computePID`:

- In order to allow tuning flexibility, use a `pCoeff` significantly larger than 1 (e.g. 100 or 1000), then divide the resulting correction by a suitable factor to get your drive signal. This helps compensate for the lack of floating=point values in microBlocks.
- Limit the resulting drive signal to reasonable values using `pid_constrainValue`.
- A non-zero `iCoeff` can easily lead to oscillations or instability. To make the system more stable you can try either or both of the following:
  - Use a non-zero `dCoeff` (this is very common when specifying a non-zero `iCoeff`).
  - Specify a non-zero value for `maxIntegral`.
- If you can predict the value of the drive signal, use the prediction as a "feedfoward" signal, as explained here. Done well, this should greatly reduce the size of the PID corrections, which should make your system more responsive and stable.
  - Compute the predicted drive signal.
  - Compute the PID correction and scale it as usual (see the first recommendation) to produce a PID drive signal correction.
  - Compute drive signal = predicted drive signal + PID drive signal correction.
  - Limit the resulting drive signal to a reasonable value using `pid_constrainValue`, as usual.

# Encoded DC Motors.ubl

Control DC motors that have incremental encoders. Each motor is specified by a different index: 1, 2, 3, ... This library requires the `PID.ubl` library.

The units used in this library are:

- distance: encoder counts
- speed: encoder counts/second
- duration: milliseconds (it would be seconds if microBlocks supported floats)

**Warning: this library must be configured by a board-specific library in order to control that board.** The board-specific library must define the following blocks (which take no arguments):

- `edcmotors_getNumMotors`: return the number of DC motors you are using.
- `_edcmotors_initSystemVariables`: initialize system-specific `Encoded DC Motors` variables.
- `_edcmotors_checkSystem`: check that the motors are powered on, or similar. This block is optional, but highly recommended if there is anything the user must do to run the motors. For example the XRP has a power switch that must be on to run the motors, but not to program the board via USB. That makes it very easy to forget to turn on the power switch.

See `XRP.ubl` for an example.

The main `Encoded DC Motors` blocks:

- `edcmotors_moveMotorAtSpeed`: move the specified motor at the specified speed (encoder counts/second) for the specified duration (milliseconds, 0=forever).
- `edcmotors_stopAllMotors`: stop all motors.
- `edcmotors_stopMotors`: stop the specified motors.
- `edcmotors_waitForMotorsToStop`: wait for the specified motors to stop. This uses the predicted end time of the current move, rather than actually waiting for the wheels to stop, so it may exit early.

Background tasks which the user should not have to touch. Note that each of this is an infinite loops, so if you do decide to run them manually, any code you put after either of them will never run:

- `_edcmotors_monitorEncoders`: read the encoders and update global variable `edcmotors__encoderPosition` (a list of values, one per motor).
- `_edcmotors_driveMotorsToFollowTarget`: drive the motors to follow a target specified by other blocks.

# XRP

Control the WPI XRP robot. Many thanks to Steve Spaeth, who wrote a preliminary version of this library.

The units used in this library for the wheels are:

- distance: mm
- speed: mm/second
- duration: milliseconds

Some useful XRP facts from <https://docs.wpilib.org/en/stable/docs/xrp-robot/getting-to-know-xrp.html>:

- The wheels have a diameter of 60 mm (2.36\") and are separated by 155 mm (6.10\").
- There are 585 encoder counts per revolution of the wheel (12 counts per revolution of the motor, which is then geared down).
- Distance traveled for one revolution of the wheels: 188 mm (7.42\").
- Distance traveled for one encoder tick: 0.322 mm (0.0127\")

The main `XRP` blocks:

- `xrp_driveDistance`: drive straight for the specified distance (mm) at the specified speed (mm/second). Specify a negative distance to go backwards.
- `xrp_driveLeftRightSpeed`: drive at the specified speed (mm/second) for a specified duration (milliseconds). The speed can be different for the left and right wheels, so you can travel in an arc.
- `xrp_readRollRate`: read roll rate (degrees/second). Positive roll tilts the left edge up.
- `xrp_readPitchRate`: read pitch rate (degrees/second). Positive pitch tilts the nose up.
- `xrp_readYawRate`: read yaw rate (degrees/second). Positive yaw changes the heading to the right.
- `xrp_readDistanceSensor`: read the distance sensor (mm)
- `xrp_readLineSensors`: read the line sensors L,R (raw)
- `xrp_setServo`: set the specified servo to the specified angles (degrees)
- `xrp_stopWheels`: stop both wheels.
- `xrp_turnAngle`: change the heading by the specified amount (degrees)
- `xrp_waitForWheelsToStop`: wait for both wheels to stop. This uses the predicted end time of the current move, rather than actually waiting for the wheels to stop, so it may exit early.
