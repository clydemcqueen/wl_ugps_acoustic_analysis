See the published report at https://clydemcqueen.github.io/wl_ugps_acoustic_analysis/

We tested the Waterlinked UGPS system off Pier 59 in Seattle and gathered acoustic logs from the G2 box using a Python
script that polled the acoustic/filtered API at 2Hz.

These logs were processed using a second Python script, producing Folium maps.

The scripts are in [ardusub_log_tools](https://github.com/clydemcqueen/ardusub_log_tools).

## Procedure

We did the normal startup procedure: getting GPS locks on the G2 and the U1, etc.
We placed the antenna in the water with the "Forward" marker pointing due West along the dock.

We flew the ROV in a rough pattern:
* North
* A big loop, starting SW then eventually returning to the antenna
* West along the dock
* East returning to the antenna
* North
* South
* Running a bit deeper towards Pier 62, then back

The tide was ebbing. Occasionally we noticed some significant current.

The logging script crashed once, perhaps because of a network error. We re-started the script after we noticed.

## Results

Output of the processing step:

~~~
$ wl_ugps_process.py --zoom 19 *.csv
Processing 2 files
-------------------
2023-06-05_09-26-08_filtered.csv
Log contains 1339 rows spanning 677.7 seconds recorded at 2.0 Hz
Removed 177 rows (13%) where position_valid == False
Rotating by -90 degrees
Translating by (47.6075779801547, -122.34390446166833)
Draw 94 red circles(s) for missing R1, -x, aft
Draw 0 black circles(s) for missing R2, +y, starboard
Draw 8 green circles(s) for missing R3, +x, forward
Draw 18 blue circles(s) for missing R4, 0, center
120 rows (9%) are missing a signal
Writing 2023-06-05_09-26-08_filtered.html
-------------------
2023-06-05_09-38-06_filtered.csv
Log contains 1382 rows spanning 699.5 seconds recorded at 2.0 Hz
Removed 176 rows (13%) where position_valid == False
Rotating by -90 degrees
Translating by (47.6075779801547, -122.34390446166833)
Draw 4 red circles(s) for missing R1, -x, aft
Draw 0 black circles(s) for missing R2, +y, starboard
Draw 45 green circles(s) for missing R3, +x, forward
Draw 13 blue circles(s) for missing R4, 0, center
62 rows (4%) are missing a signal
Writing 2023-06-05_09-38-06_filtered.html
~~~

A Folium map was generated for each log file:
* [2023-06-05_09-38-06_filtered.html](2023-06-05_09-38-06_filtered.html)
* [2023-06-05_09-26-08_filtered.html](2023-06-05_09-26-08_filtered.html)

The colored circles show readings where only 3 receivers reported. This snippet from the processing code provides
a color key (note that the receivers are numbered 1-4 in the UI but 0-3 in the logs):
~~~
Receiver('valid_r0', 'missing R1, -x, aft', 'red'), 
Receiver('valid_r1', 'missing R2, +y, starboard', 'black'), 
Receiver('valid_r2', 'missing R3, +x, forward', 'green'), 
Receiver('valid_r3', 'missing R4, +z, down', 'blue'),
~~~

## Discussion

13% of the readings were invalid, and thrown away. This may be because the U1 failed to generate a ping.

9% of the readings had valid signals at just 3 receivers. These generally occurred when the ROV was further away from
the antenna.

When the ROV was West of the antenna -- in the "Forward" direction -- the R1 receiver often failed to 
report a valid signal (the red circles). The R1 receiver was behind the R3 receiver, and both were near the pilings,
perhaps the reflection overwhelmed the signal?

The R2 receiver (along the y-axis) always reported a valid signal. (There are no black circles.)

If we consider distance and yaw instead of x and y, then by eye it seems that the bulk of the error is in the yaw,
which makes sense.

Overall the acoustic readings are pretty reasonable, except for the 13% dropout rate.

## Questions / future work

How often are acoustic readings generated?

If we poll more often do we get interpolated results?

Is there a limit to how often we can call the G2 API?

In a different test we polled acoustic/filtered and acoustic/raw back-to-back at 1Hz, and the results appeared identical. Why?

Can we characterize the error in terms of `var_distance(distance)` and `var_yaw(distance)`?
