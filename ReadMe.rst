microBlocks libraries to run the wheels on the XRP robot kit.

Contents
========

Contains the following microBlocks libraries:

* ``PID.ubl``: PID (proportional, integral, derivitive) controllers.
* ``DC Motors.ubl``: control DC motors that have incremental encoders.
* ``XRP.ubl``: control the WPI XRP robot kit.

PID.ubl
=======

Multiple PID controllers.
Each controller is specified by a different index: 1, 2, 3...

The main blocks are:

* ``compute_pid``: compute the next PID correction for the specified PID loop.
  The inputs are as follows:

  * ``index``: index of the PID loop: 1, 2, 3...
  * ``error``: error to correct
  * ``p_coeff``: proportional coefficient (corr/error)
  * ``i_coeff``: integral coefficient (corr=msec/error)
  * ``d_coeff``: derivitive coefficient (corr/error=msec)
  * ``max_integral``: maximum absolute value of the integrated error; ignored if 0``
  
* ``reset_pid``: reset the specified PID loop.
  It may be useful to reset a PID before commanding a new move.

* ``constrain_value``: limit the range of a value, intended to post-process a scaled correction from a PID loop.
  This may be used to constrain the correction returned by ``compute_pid`` to useful values.
  The algorithm is as follows (where ``|value|`` means the absolute value of ``value``):

  * If ``|value| < deadband``: return 0.
    This can keep the system from jittering or whining after a move.
  * If ``|value| < minimum``: return ``minimum`` with the sign of ``value``.
    This prevents applying a correction that is too small to do anything useful.
    For example the WPI XRP motor drives cannot reliably move when the effort is less than about 200.
  * If ``|value| > maximum``: return ``maximum`` with the sign of ``value``.'
    This prevents applying more correction than the system can handle.
    For example the WPI XRP motor drives have a maximum "effort" of 1023.

Recommendations for using ``compute_pid``:

* In order to allow tuning flexibility, use a ``p_coeff`` significantly larger than 1 (e.g. 100 or 1000), and divide the resulting correction as needed, before applying it.
  (This helps compensate for the lack of floating=point values in microBlocks).
* Limit the resulting correction to reasonable values using ``constrain_value``.
* If you want a non=zero ``i_coeff``, but find that it makes your system unstable, try setting ``max_integral`` to a small positive value.


DCMotors.ubl
============

Control DC motors that have incremental encoders.
Each motor is specified by a different index: 1, 2, 3...
This library requires the ``PID.ubl`` library.

This library must be configured by a board-specific library in order to control that board.
The board-specific library must define the following blocks (which take no arguments):

* ``get_num_motors``: return the number of DC motors supported by your system.
* ``_init_dcmotors_system_variables``: initialize system-specific DCMotors variables.

See ``XRP.ubl`` for an example.

The main DCMotors blocks:

* ``move_motor_distance_speed``: move the specified motor by the specified number of encoder counts.
* ``stop_all_motors``: stop all motors.
* ``stop_motor``: stop the specified motor.

Background tasks which the user should not have to touch.
Note that each of this is an infinite loops, so if you do decide to run them manually,
any code you put after either of them will never run:

* ``monitor_encoders``: read the encoders and update global variable ``motors__encoder_position`` (a list of values, one per motor).
* ``drive_motors_to_follow_target``: drive the motors to follow a target specified by other blocks.

XRP
===

Control the WPI XRP robot.
The non-wheel-related code was originally written by Steve Spaeth.
