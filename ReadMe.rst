microBlocks libraries to run the wheels on the XRP robot kit.

Contents
========

Contains the following microBlocks libraries:

* ``PID.ubl``: PID (proportional, integral, derivitive) controllers.
* ``DC Motors.ubl``: control DC motors that have incremental encoders.

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


DC Motors.ubl
=============

Control DC motors that have incremental encoders.
Each motor is specified by a different index: 1, 2, 3...
This library requires the ``PID.ubl`` library.

In order to use this library to control a specific board (such as the WPI XRP robot kit) you must override the ``_init_dcm_library`` method to set various global variables (all of whose names start with _motors__); see the provided example (which is for the WPI XRP robot kit) for more information.
We suggest that you write a separate library for each board you wish to support, so users do not have to figure out the init override.
The provided ``_init_dcm_library`` is for the WPI XRP robot kit, and assumes the user has only 2 motors (because the standard kit only has two, even though the board supports 4).

The main blocks are:

* ``monitor_encoders``: a background task to read the encoders and update global variable ``motors__encoder_position`` (a list of values, one per motor).
  You must start this block before doing anything that uses the encoders.
  It is an infinite loop, so please do not put any other blocks after it.
* ``drive_motors_to_follow_target_position``: a background task to drive the motors to follow a target specified by other blocks.
  You must start this block before calling ``move_motor_by_amount`` or ``move_motor_to_position``.
  It is an infinite loop, so please do not put any other blocks after it.
* ``stop_all_motors``: stop all motors.
* ``stop_motor``: stop the specified motor.

The following main blocks all require that ``monitor_encoders`` and ``drive_motors_to_follow_target_position`` are both running:

* ``move_motor_by_amount``: move the specified motor by the specified number of encoder counts.
* ``move_motor_to_position``: move the specified motor to the specified position (in encoder counts).
* ``set_motor_speed``: move the specified motor at constant speed (in encoder counts/second), starting at the current measured position.
* ``set_motor_target``: a lower-level block that allows you to set the target position (encoder counts), speed (encoder counts/second), and starting time (milliseconds, -1 for "now").
  The blocks above it all call this method.
