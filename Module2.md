# Streams Handson - Module 2

The objectives of Module2 are:

1. To explore the deployment of the flink application from [Module1](https://github.com/rspamzn/streams-handson) onto Kinesis Data Analytics (KDA)
3. Test and monitor the application

###  Part 1 - Setting up Pre-Requisites

1. We will reuse the Kinessis Data Streams and the application Jar file that we created in Module1. If you skipped Module1, you may just do the first 2 steps of "Part3 - Preparing to run the job" section.
2. KDA expects the application jar to be in an S3 bucket. Open the Amazon S3 console at https://console.aws.amazon.com/s3/ 
3. Choose **Create bucket**.

4. Enter `ka-app-code-<username>` in the **Bucket name** field. Add a suffix to the bucket name, such as your user name, to make it globally unique. Choose **Next**.
5. In the **Configure options** step, keep the settings as they are, and choose **Next**.
6. In the **Set permissions** step, keep the settings as they are, and choose **Next**.
7. Choose **Create bucket**.
8. In the Amazon S3 console, choose the **ka-app-code-`<username>`** bucket, and choose **Upload**.
9. In the **Select files** step, choose **Add files**. Navigate to the `aws-kinesis-analytics-java-apps-1.0.jar` file that you created in the previous step. Choose **Next**.
10. You don't need to change any of the settings for the object, so choose **Upload**.

Your application code is now stored in an Amazon S3 bucket where your application can access it.



------



### Part 2 - Create and Run the Application

1. To access the Flink web console, we will use the SSH tunnel with port forwarding method. This method needs a browser extension  similar to FoxyProxy. Use the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html) to install and configure FoxyProxy on your browser.

2.  Add SSH to the inbound rules of the security group associated with the master node. The security group can be found in the Summary tab -> "Security and acces" section as below ![](https://github.com/rspamzn/streams-handson/blob/master/resources/summary.png)

3. Set up the SSH tunnel to the master node using the [instructions here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-ssh-tunnel-local.html). You may also use the following command by replacing the keypair name and the public DNS name for the master node. 

   ```
   ssh -i SteamKeypair.pem  -N -L 8157:ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:8088 -L 8158:ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:20888 hadoop@ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com
   ```

4. With SSH tunnel on, use the browser where the FoxyProxy is configured and access the url - http://localhost:8157. This will open the Resource Manager web interface. ![](https://github.com/rspamzn/streams-handson/blob/master/resources/applications.png). Click on the "Application Master" link as circled below.

5. This link will not be accessible as the port 20888 is not open in the security configuration. Instead, we have enabled the SSH port forwarding earlier from the local port 8158. Replace the ip address and port to localhost and 8158 (http://localhost:8158/proxy/application_1600269053659_0001/#/overview). This will open Flink's web console as below.![](https://github.com/rspamzn/streams-handson/blob/master/resources/flinkweb.png)

   

------



### Part 3 - Preparing to Run the Job

1. Clone the repository https://github.com/rspamzn/streams-handson This contains the following artifacts

   - The app folder contains a simple Java application that we will use for testing. This Application reads data from a Kinesis Data Stream (ExampleInputStream) and publish the output( by adding the PROCESSED_TIME attribute) to another Kinesis Stream (ExampleOutputStream). Before we submit this application, we will create these Kinesis Data Streams and enable appropriate permissions.
   - The eventgen folder contains a python program that will generate and push the input data into ExampleInputStream.

2. Create the ExampleInputStream and ExampleOutputStream using the [instructions here](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-streams.html)

3. Copy the target jar file into the master node so that it can be submitted later. Replace the keypair and the node IP.

   ```
   scp -i ~/cloudLabs/SteamKeyPair.pem ./aws-kinesis-analytics-java-apps-1.0.jar  hadoop@ec2-13-250-29-109.ap-southeast-1.compute.amazonaws.com:~/
   ```

4. At this point, we will enbale the permission for the streams application to read and write to the Kinesis Data Streams. Create an IAM policy using the json document as below.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "ReadInputStream",
               "Effect": "Allow",
               "Action": "kinesis:*",
               "Resource": "*"
           },
           {
               "Sid": "WriteOutputStream",
               "Effect": "Allow",
               "Action": "kinesis:*",
               "Resource": "*"
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



