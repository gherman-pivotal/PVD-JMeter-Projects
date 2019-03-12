# Objective

To determine GoRouter capacity given specific system resources. To do that, we will need to drive the GoRouter to saturation (CPU utilization > 100%) by making sure that the other system components/resources are not the bottleneck.
System resources for this test will be: Dell Servers with 2 CPUs each with 14 cores; with 384 GB memory, 4 1.92 TB SATA SSDs, 2 800 SSDs (Cache), and 4x 10G Network ports.

Note: This test will be limited to testing one GoRouter with a single NSX edge load balancer. The load balancer may be a bottleneck, but we need to limit the test to one component. For the revision of the test plan, I’ve limited it to testing with a single cell and a single copy of GoRouter. Once we learn from this test, we can move onto scaling configurations.

GoRouter performance is dependent on the latency of the response to the requests, the number of requests and the message bandwidth used by the requests. The performance variable that will have the biggest impact with GoRouter is the number of vCPUs assigned to the VM. For the purpose of this test we will limit the test to 1,2,4, and 8 vCPUs. We will be collecting the response time and throughput metrics using JMeter. And CPU utilization will be collected manually using esxtop within the VMs for the Gorouter(s), Cell(s) and appropriate cell.

# The Test Application

For this test, we will be using a single app. We will simulate latency by a simple delay. The payload information to/from the app will impact the bandwidth requirements. The test application should accept any incoming size message (within a range) from jmeter, and return a size specified in header of the incoming request. 

The app will have a fixed delay passed specified as a parameter or variable via the http header. A variable should be passed, along with the incoming message that tells the app the size of the message of the reply. A simple string of characters should suffice for the return message. (Assuming that there is no compression being used.)

# Test Methodology

We will do the following tests:

We will have 15 app instances running per large cell. We will vary the vCPUs of the VM for GoRouter from 1,2,4,8. With JMeter we will ramp the number of threads quadratically (1,2,4,8,16,etc) to 1024 threads. We will have at least 30 seconds of data in each test. We expect to see consistent performance during the last 15 seconds of each test. (If we don’t, we need to stop and understand why.) At some point during the scale up, we expect to see the IOPs go steady or decline slightly and the response get higher than 2 seconds. (During the 30 second run.) At this point we will conclude that we have reached saturation. If the CPU of GoRouter is not saturated we have a bottleneck elsewhere. We will be watching CPU utilization with esxtop of each VM (GoRouter, the VM containing the cell, and JMeter) to see which is saturated. We will increase the instances of JMeter or add more cells, depending on what we find to be the bottleneck. When all bottlenecks are removed, we expect to see GoRouter finally become saturated. The objective is to continue providing resources to other objects in order to push the GoRouter to hit it’s maximum capacity.

Now that we have a configuration that can saturate GoRouter, we need to get an understanding the impact of message size on performance. Repeat the tests with specifying a variety of send/receive message size. I would limit the testing to five or less set of runs. Increasing the message size will more network bandwidth and we would expect to see results lower (GoRouter saturation point will be hit sooner in our test runs) than in previous tests.

Finally, we need to get an understanding of the impact of the response time of the app on the CPU load of the GoRouter. A fixed response time between 100 - 500 millisecond should be a good start.

Once we get the results for VM for 1 vCPU, we need to a complete set of runs for 2, 4, and 8 vCPUs. We should monitor esxtop for each of the VMs to see where the bottleneck is. We will need to remove such bottlenecks until we have saturation of the GoRouter VM.

After we analyze all the data from the runs of these tests, we can move on to testing more than one copy of GoRouter. 

# Collection of Data

The following performance information should be captured via collection script. We will error on having too much information as much of the analysis will happen after the test is complete. Finding that we missed something will require us to make another run, so we want to keep that to a minimum.

1. All JMeter output should be collected via a CSV file. The CSV file will contain each interaction along with associated timing. We will most likely have to write a program to do some analysis. There are also plenty of open source tools for interpreting JMeter output.
2. Esxtop for the VM containing GoRouter, the cell/VM containing the apps, the VM containing the load balancer and esxtop/top of the JMeter instances. All these should be collected at 5 or 10 second intervals that give us the time so we can correlate the time of a particular event. Having network throughput information would be valuable as well. I would recommend that we use the following: esxtop -a -d 5 -b > my_file.csv
