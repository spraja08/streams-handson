# Streams Handson - Module 1

This workshop has 2 modules. Module1 covers EMR with Flink, with the following objectives

1. Create an EMR Cluster with Flink runtime 
2. Make Flink's Web Console accessible externally
3. Deploy, test and monitor a Flink streaming application that reads and writes to Kinesis Data Streams

[Module 2](https://github.com/rspamzn/streams-handson/blob/master/Module2.md) covers the steps involved in using Kinesis Data Analytics to deploy and operate the same flink streaming application.

###  Part 1 - Launch Cluster

1. Log into AWS console and select EMR service. In the EMR service  page,  click on the **Create cluster** button and selet the **Go to advanced options** link.

2. Select the EMR release 5.30.0. Under the **Software Configuration** section, select Hadoop and Flink applications. EMR will provision these selected application frameworks during the cluster launch process.

3. In the **Edit software settings** subsection, select the **Enter configuration** radio button and key in the following text. EMR uses these configurations to override the defaults (in this case, the flin-conf.yaml) after provisioning the applications. Refer [here]( https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-configure-apps.html) for more info

   ```
   classification=flink-conf,properties=[taskmanager.numberOfTaskSlots=4,taskmanager.memory.flink.size=4g]
   ```

4. In the Steps section, add a step with the following details. (Refer [here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-overview.html#emr-work-cluster) to understand the concept of steps and clueter lifecycle) This will start a long running YARN session in the **detached** mode  where the job manager gets 1 GB of heap space and the task managers 4 GB of heap space assigned. 

   ```
   Jar Location : command-runner.jar
   Arguments    : flink-yarn-session -d -n 4 -jm 1024 -tm 4096
   ```

   ![Steps configuration](https://github.com/rspamzn/streams-handson/blob/master/resources/steps.png)

5. Click on the **Next** button and leave the Hardware Configurations to the defaults in the wizard. Click on the **Next** button

6. In the **General Options**, remove the **Termination Protection** check. Provide your own name for the cluster and click the **Next** button

7. In the Security options page, choose your existing key pair. Leave the rest to the defaults as shown in the wizard and click on **Create cluster** button.

8. Wait for the cluster status to change to **Running**. Check the **Steps** tab to make sure that the Status of the steps show Running also as below

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/running.png)

   

------



### Part 2 - Enable Flink Console Access

1. To access the Flink web console, we will use the SSH tunnel with port forwarding method. This method needs a browser extension  similar to FoxyProxy. Use the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html) to install and configure FoxyProxy on your browser.

2.  Add SSH to the inbound rules of the security group associated with the master node. The security group can be found in the Summary tab -> **Security and acces** section as below ![](https://github.com/rspamzn/streams-handson/blob/master/resources/summary.png)

3. Set up the SSH tunnel to the master node using the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-ssh-tunnel-local.html). You may also use the following command by replacing the keypair name and the public DNS name for the master node. 

   ```
   ssh -i SteamKeypair.pem  -N -L 8157:`ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com`:8088 -L 8158:`ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com`:20888 hadoop@`ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com`
   ```

4. With SSH tunnel on, use the browser where the FoxyProxy is configured and access the url - http://localhost:8157. This will open the Resource Manager web interface. ![](https://github.com/rspamzn/streams-handson/blob/master/resources/applications.png). Click on the **Application Master** link as circled below.

5. This link will not be accessible as the port 20888 is not open in the security configuration. Instead, we have enabled the SSH port forwarding earlier from the local port 8158. Replace the ip address and port to localhost and 8158 (http://localhost:8158/proxy/application_1600269053659_0001/#/overview). This will open Flink's web console as below.![](https://github.com/rspamzn/streams-handson/blob/master/resources/flinkweb.png)

   

------



### Part 3 - Preparing to Run the Job

1. Clone the repository https://github.com/rspamzn/streams-handson This contains the following artifacts

   - The app folder contains a simple Java application that we will use for testing. This Application reads data from a Kinesis Data Stream (ExampleInputStream) and publish the output( by adding the PROCESSED_TIME attribute) to another Kinesis Stream (ExampleOutputStream). Before we submit this application, we will create these Kinesis Data Streams and enable appropriate permissions.
   - The eventgen folder contains a python program that will generate and push the input data into ExampleInputStream.

2. Create the ExampleInputStream and ExampleOutputStream using the [instructions here](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-streams.html)

3. Copy the target jar file into the master node so that it can be submitted later. Replace the keypair and the node IP.

   ```
   scp -i `~/cloudLabs/SteamKeyPair.pem` ./aws-kinesis-analytics-java-apps-1.0.jar  hadoop@`ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:~/`
   ```

4. At this point, we will enbale the permission for the streams application to read and write to the Kinesis Data Streams. Create an IAM policy using the json document as below.

   ```
   {
       **Version**: **2012-10-17**,
       **Statement**: [
           {
               **Sid**: **ReadInputStream**,
               **Effect**: **Allow**,
               **Action**: **kinesis:***,
               **Resource**: *****
           },
           {
               **Sid**: **WriteOutputStream**,
               **Effect**: **Allow**,
               **Action**: **kinesis:***,
               **Resource**: *****
           }
       ]
   }
   ```

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/policycreate.png)

5. Add the newly created policy to EMR_EC2_DefaultRole. This is the role that is attached to all the EC2 instances that EMR launched for our cluster.![](https://github.com/rspamzn/streams-handson/blob/master/resources/policyadd.png)



------



### Part 4 - Run the Job

1. Ssh into the master node and submit the application using the following command 

   ```
   flink run -m yarn-cluster ./aws-kinesis-analytics-java-apps-1.0.jar
   ```

   The job will be submitted with a valid Jobid created with the status message similar to below

   ```
   Found Web Interface ip-172-31-35-41.ap-southeast-1.compute.internal:44861 of application 'application_1600269053659_0002'.
   Job has been submitted with JobID ae389833110e05c94196b06f10533108
   ```

2. The Flink's web console will now display the running job as below![](https://github.com/rspamzn/streams-handson/blob/master/resources/submitted.png)

3. Start pumping input data using the python script as below

   ```
   python stock.py
   ```

4. The input messages will be processed by the application which can be seen from the Taskmanager's log (Stdout) as below![](https://github.com/rspamzn/streams-handson/blob/master/resources/flinklogs.png)

5. Alternatively, the Kinesis Data Streams monitoring page will also show the number of records written as a proof that the application is processing the data as and when its available. ![](https://github.com/rspamzn/streams-handson/blob/master/resources/kdsmon.png)

   

------



