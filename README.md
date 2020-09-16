# Streams Handson

The objectives of this section is to 

1. Create an EMR Cluster with Flink runtime using AWS console
2. Make Flink Dashboard accessible externally
3. Deploy, test and monitor a Flink application

### Part 1 - Launch Cluster

1. Log into AWS console and select EMR service. In the EMR service  page,  click on the "Create cluster" button and selet the "Go to advanced options" link.

2. Select the EMR release 5.30.0. Under the "Software Configuration" section, select Hadoop and Flink applications. EMR will provision these selected application frameworks during the cluster launch process.

3. In the "Edit software settings" subsection, select the "Enter configuration" radio button and key in the following text

   ```
   classification=flink-conf,properties=[taskmanager.numberOfTaskSlots=4,taskmanager.memory.flink.size=4g]
   ```

4. In the Steps section, add a step with the following details.  This will start a long running YARN session in the "detached" mode  where the job manager gets 1 GB of heap space and the task managers 4 GB of heap space assigned. 

   ![Steps configuration](https://github.com/rspamzn/streams-handson/blob/master/resources/steps.png)

5. Click on the "Next" button and leave the Hardware Configurations to the defaults in the wizard. Click on the "Next" button

6. In the "General Options", remove the "Termination Protection" check. Provide your own name for the cluster and click the "Next" button

7. In the Security options page, choose your existing key pair. Leave the rest to the defaults as shown in the wizard and click on "Create cluster" button.

8. Wait for the cluster status to change to "Running". Check the "Steps" tab to make sure that the Status of the steps show Running also as below

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/running.png)

### Part 2 - Enable Flink Console Access

1. To access the Flink web console, we will use the SSH tunnel with port forwarding method. This method needs a browser extension  similar to FoxyProxy. Use the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html) to install and configure FoxyProxy on your browser.

2.  Add SSH to the inbound rules of the security group associated with the master node. The security group can be found in the Summary tab -> "Security and acces" section as below ![](https://github.com/rspamzn/streams-handson/blob/master/resources/summary.png)

3. Set up the SSH tunnel to the master node using the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-ssh-tunnel-local.html). You may also use the following command by replacing the keypair name and the public DNS name for the master node. 

   ```
   ssh -i SteamKeypair.pem  -N -L 8157:ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:8088 -L 8158:ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:20888 hadoop@ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com
   ```

4. With SSH tunnel on, use the browser where the FoxyProxy is configured and access the url - http://localhost:8157. This will open the Resource Manager web interface. ![](https://github.com/rspamzn/streams-handson/blob/master/resources/applications.png). Click on the "Application Master" link as circled below.

5. This link will not be accessible as the port 20888 is not open in the security configuration. Instead, we have enabled the SSH port forwarding earlier from the local port 8158. Replace the ip address and port to localhost and 8158 (http://localhost:8158/proxy/application_1600269053659_0001/#/overview). This will open Flink's web console as below.![](https://github.com/rspamzn/streams-handson/blob/master/resources/flinkweb.png)

### Part 3 - Preparing to Run Job

1. 







