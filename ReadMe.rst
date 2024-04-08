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

* ``constrain_value``: limit the range of a value, truncating as needed.
  This may be used to constrain the value returned by ``compute_pid`` to reasonable values.

Recommendations for using ``compute_pid``:

* In order to allow tuning flexibility, use a ``p_coeff`` significantly larger than 1 (e.g. 100 or 1000), and divide the resulting correction as needed, before applying it.
  (This helps compensate for the lack of floating=point values in microBlocks).
* Limit the returned correction to sane values, perhaps using ``constrain_value``.
* For some motors you may find that tiny corrections have no effect (but may make the motors whine), and that somewhat larger corrections are unreliable (e.g. due to friction).
  Two ways to help this situation:

  * If the correction is so tiny as to be useless, change it to 0.
  * If the correction is in an intermediate range, where it might be useful but is not reliable, increase it as needed.

* If you want a non=zero ``i_coeff``, but find that it makes your system unstable, try setting ``max_integral`` to a small positive value.


DC Motors.ubl
=============

Control DC motors that have incremental encoders.
Each motor is specified by a different index: 1, 2, 3...
This library requires the ``PID.ubl`` library.

In order to use this library to control a specific board (such as the WPI XRP robot kit) you must override the ``init`` method to set various global variables (all of whose names start with _motors__); see the provided example (which is for the WPI XRP robot kit) for more information.
We suggest that you write a separate library for each board you wish to support, so users do not have to figure out the init override.
The provided ``init`` is for the WPI XRP robot kit, and assumes the user has only 2 motors (even though the board supports 4).

The main blocks are:

* ``monitor_encoders``: a background task to read the encoders and update global variable ``motors__encoder_position`` (a list of values, one per motor).
  You must start this block before doing anything that uses the encoders.
  It is an infinite loop, so please do not put any other blocks after it.
* ``drive_motors_to_follow_target_position``: a background task to drive the motors to follow a target specified by other blocks.
  You must start this block before calling ``move_motor_by_amount`` or ``move_motor_to_position``.
  It is an infinite loop, so please do not put any other blocks after it.
* ``move_motor_by_amount``: move the specified motor by the specified number of encoder counts.
  This requires that ``monitor_encoders`` and ``drive_motors_to_follow_target_position`` are both running.
* ``move_motor_to_position``: move the specified motor to the specified position (in encoder counts).
  This requires that ``monitor_encoders`` and ``drive_motors_to_follow_target_position`` are both running.
* ``stop_all_motors``: stop all motors.
* ``stop_motor``: stop the specified motor.
