# Test plan for scaling vCPUs and Gorouters

This is the test plan that outlines out we will do our testing of scaling the vCPUs and GoRouters.

# Test assumptions

It is assumed that we already know the 1 vCPU/1GoRouter configuration. If we had to use X apps per cell and Y cells,
and Z instance of JMeter to saturate 1 instance of GoRouter on 1 vCPU, we should assume that this will scale linearly.
We are constrained by only being able to deploy 15 apps/cell, at 16GB. That leaves us ~20 cells per node. We would require at
least a 4 node cluster for the larger nodes.

vCPUs | # GoRouter | Apps/Cell | # Cells | JMeter Instances
------|------------|-----------|---------|-----------------
   1  |     1      |    X      |    Y    |      Z
   2  |     1      |    X      |   2Y    |     2Z
   4  |     1      |    X      |   4Y    |     4Y
   8  |     1      |    X      |   8Y    |     8Y
   1  |     2      |    X      |   2Y    |     2Z
   2  |     2      |    X      |   4Y    |     4Z
   4  |     2      |    X      |   8Y    |     8Y
   8  |     2      |    X      |  16Y    |    16Y
   1  |     4      |    X      |   4Y    |     4Z
   2  |     4      |    X      |   8Y    |     8Z
   4  |     4      |    X      |  16Y    |    16Y
   8  |     4      |    X      |  32Y    |    32Y
   1  |     8      |    X      |   8Y    |     8Z
   2  |     8      |    X      |  16Y    |    16Z
   4  |     8      |    X      |  32Y    |    32Y
   8  |     8      |    X      |  64Y    |    64Y
   
   
