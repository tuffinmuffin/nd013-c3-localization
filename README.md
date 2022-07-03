Intro
=====


This is the 3rd major course for The Udacity self driving car. This section focused on using a lidar to localize the location of car based on lidar scan given a known map.

The final project gave us two options. We can use **Iterative closest Point** (https://en.wikipedia.org/wiki/Iterative_closest_point) or NOrmal Distributions Transforms (NDT) (https://en.wikipedia.org/wiki/Normal_distributions_transform)

Both options can give good solutions to estimate new poses based on a change in the scan and previous estiamted poses.


In the final project the objective was to come up with parameters to optimize one of these algorithms to meet the time performance and accuray performance requirments.


The NDT and ICP algorithms were both explored during the course. For the final project the Point Cloud Library (PCL) versions will be used.

Given both ICP and NDT were solved during the course using the library removed a source of error out from the final testing.

Another technique to mention here as it is one of the adjustable parameters is how to reduce the number of points each algorthm will see. This is done by group points into cubes and only matching cubes. The values in the cube relates to the number of points that fall into the cube giving stronger returns. The primary parameter I will use below is the "leaf" parameter that adjusts the size of the cubes.

How to Run
----------
The application mostly follows the usage given by course.

Support Libs and example refernces should be extracted from support-libs.zip

extract support libs (or copy the c3-main.cpp file as this is only file with updates)
```
    gunzip support-libs.zip
```

Configure
```
    cmake .
```

Compile
```
    make
```

Run Simulator
.. code-block:: shell

    #optional depending on env but can not run as root without modifing code
    su - student
    ./run_carla.sh

After the simulator is running and the code is built an executable name `cloud_loc` is created.
Running it without paremters pick the "defaults"

Running with the the following sytnax allows paramters to change. The CLI parser is very crude and will not tolerate deviation.

```
    ./cloud_loc <algo> <iterations> <leaf size>
```

Where
* **<algo>** is either 'ndt' or 'icp"
* **<iterations>** is an integer for number of iterations to run algo
* **<leaf size>** is a floating point value for leaf size

Adding this simnple CLI allowed much faster attempts on values. In addition to the above added CLI interface the data presentation
was changed to also present the critical tuned values. This helped with keeping data collection simpler


----------

One of my first attempts to solve the problem was using ICP. This resulted in an error of 1.24 which was very close.

![Early attempt](c3-project/images/icp_leaf1.0_itr5_speed3.png)

This used ICP with 5 iterations and a leaf size of 1.0.


Several further attempts followed. It was hard to be patient enough at times due to the simulation appearing to be very slow. Increasing the speed about 5-6 ticks caused error to quickly exceed 1.2 meters. Even if it only ever reached 1.3 meters that still would be failing.

Some high speeds or extereme paramers would also lose all tracking and no further pose tracking would occur.

I found a good set of parameters uing NCP with a leaf size of 0.7 and 4 iterations. The resulted in a very low error and seemed to meet the requirments for the project.

```
    ./cloud_loc ndt 4 0.7
```

![Good NDT](c3-project/images/ndt_leaf0.7_itr4_speed3.png)

In the image I also inlcuded the shell output showing my debug statments for tracking key presses. As shown this was taken at speed 3.

One issue appeared to be the speed of the simulator. It appeared the simulator required more resources than the VMs really could provide.

One snapshop (which was fairly consistent) is the simulator took up 2 total CPUs. But most of the time my algorthims were using <15% of a CPU.
![Performance issues](c3-project/images/cpu_usage.png)
Snaptop of CPU usage. When running ICP the CPU usage was higher in the matching app than using NDT.


ICP was finicky. There was one region of the map it generally did not perform well on. Even with a very good final result the section marked on the map bellow seemed to provide challenges.

![Bad ICP](c3-project/images/icp_leaf0.1_itr30_speed3.png)


A good version of IPC was found with ICP iterations 30, leaf size 0.2

```
./cloud_loc icp 30 0.2
```
![Good ICP](c3-project/images/icp_leaf0.2_itr30_speed3.png)


As you can see from the 2 ICPs above the only change between is leaf size to pass vs fail.




# Conclision
Using either algorithm presented in this course can work both worked well and had diffrent areas they seemed to perform well in.

A passing NDT and ICP was shown above that met the reqs of <1.2 meter error over 170 meters travel.

The simulation speed vs how the algorithm would perform was hard to seperate. The simulation did not appear to be real time and the algorithm is being run on provided hardware instead of target hardware. Moving this to a real time system would need a more disciplened way of comparing real run time performance.
