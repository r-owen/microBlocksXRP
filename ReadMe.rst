microBlocks libraries to run the wheels on the XRP robot kit.

Contents
========

Contains the following microBlocks libraries:

* ``PID.ubl``: PID (proportional, integral, derivitive) controllers.
* ``DC Motors.ubl``: control DC motors that have incremental encoders.
* ``XRP.ubl``: control the WPI XRP robot kit.

PID.ubl
=======

One or more PID (proportional-integral-differential) controllers.
Each controller is specified by a different index: 1, 2, 3...

PID controllers are used to control a system using feedback: measure an error and use this to apply a correction.
Common examples are operating on oven element or furnace to hold a steady temperature, or controlling an encoded robot wheel to move a specified distance.
For a robot wheel, you can determine the error by comparing where you want to be with where the motor encoder is telling you that you are.
Feed the measured error into ``pid_computePID`` and scale the resulting correction to determine a drive signal.
See "Recommendations for using ``pid_computePID``" for more information.

The main blocks are:

* ``pid_computePID``: compute the next PID correction for the specified PID loop.
  The inputs are as follows:

  * ``index``: index of the PID loop: 1, 2, 3...
  * ``error``: error to correct
  * ``pCoeff``: proportional coefficient (corr/error)
  * ``iCoeff``: integral coefficient (corr=msec/error)
  * ``dCoeff``: derivitive coefficient (corr/error=msec)
  * ``maxIntegral``: maximum absolute value of the integrated error; ignored if 0``
  
* ``pid_resetPID``: reset the specified PID loop.
  It may be useful to reset a PID before commanding a new move.

* ``pid_constrainValue``: limit the range of a value.
  This is provided to constrain the drive signal to useful values.
  The algorithm is as follows (where ``|value|`` means the absolute value of ``value``):

  * If ``|value| < deadband``: return 0.
    This can keep the system from jittering or whining after a move.
  * If ``|value| < minimum``: return ``minimum`` with the sign of ``value``.
    This prevents applying a correction that is too small to do anything useful.
    For example the WPI XRP motor drives cannot reliably move when the effort is less than about 200.
  * If ``|value| > maximum``: return ``maximum`` with the sign of ``value``.'
    This prevents demanding more than the system can handle.
    For example the WPI XRP motor drives have a maximum "effort" of 1023.

Recommendations for using ``pid_computePID``:

* In order to allow tuning flexibility, use a ``pCoeff`` significantly larger than 1 (e.g. 100 or 1000).
  Then divide the resulting correction by a suitable factor to get your drive signal.
  This helps compensate for the lack of floating=point values in microBlocks.
* Limit the resulting drive signal to reasonable values using ``pid_constrainValue``.
* A non-zero ``iCoeff`` can easily lead to oscillations or instability.
  To make the system more stable you can try either or both of the following:
  
  * Use a non-zero ``dCoeff`` (this is very common when specifying a non-zero ``iCoeff``).
  * Specify a non-zero value for ``maxIntegral``.

* If you can predict the value of the drive signal, use the prediction as a "feedfoward" signal, as explained here.
  Done well, this should greatly reduce the size of the PID corrections, which should make your system more responsive and stable.

  * Compute the predicted drive signal.
  * Compute the PID correction and scale it as usual (see the first recommendation) to produce a PID drive signal.
  * Compute drive signal = predicted drive signal + PID drive signal.
  * Limit the resulting drive signal to a reasonable value using ``pid_constrainValue``, as usual.


Encoded DC Motors.ubl
=====================

Control DC motors that have incremental encoders.
Each motor is specified by a different index: 1, 2, 3...
This library requires the ``PID.ubl`` library.

This library must be configured by a board-specific library in order to control that board.
The board-specific library must define the following blocks (which take no arguments):

* ``edcmotors_getNumMotors``: return the number of DC motors you are using.
* ``_edcmotors_initSystemVariables``: initialize system-specific 'Encoded DC Motors' variables.
* ``_edcmotors_checkSystem``: check that the motors are powered on, or similar.
  This block is optional, but highly recommended if there is anything the user must do to run the motors.
  For example the XRP has a power switch that must be on to run the motors, but not to program the board via USB.
  That makes it very easy to forget to turn on the power switch switch.

See ``XRP.ubl`` for an example.

The main 'Encoded DC Motors' blocks:

* ``edcmotors_moveMotorDistanceSpeed``: move the specified motor the specified distance (encoder counts), at the specified speed (counts/second).
* ``edcmotors_moveMotorSpeed``: move the specified motor at the specified speed (encoder counts/second), indefinitely.
* ``edcmotors_stopAllMotors``: stop all motors.
* ``edcmotors_stopMotor``: stop the specified motor.

Background tasks which the user should not have to touch.
Note that each of this is an infinite loops, so if you do decide to run them manually,
any code you put after either of them will never run:

* ``_edcmotors_monitorEncoders``: read the encoders and update global variable ``edcmotors__encoderPosition`` (a list of values, one per motor).
* ``_edcmotors_driveMotorsToFollowTarget``: drive the motors to follow a target specified by other blocks.

XRP
===

Control the WPI XRP robot.
The non-wheel-related code was originally written by Steve Spaeth.

Some useful XRP facts from  https://docs.wpilib.org/en/stable/docs/xrp-robot/getting-to-know-xrp.html:

• The wheels have a diameter of 60 mm (2.36”) and are separated by 155mm (6.10”).
• There are 585 encoder counts per revolution of the wheel (12 counts per revolution of the motor, which is then geared down).
• Distance of traveled for one revolution of the wheels: 188 mm (7.42").
• Distance traveled for one encoder tick: 0.322 mm (0.0127")
