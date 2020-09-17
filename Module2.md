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



### Part 2 - Create and Configure the Application

1. Open the Kinesis Data Analytics console at https://console.aws.amazon.com/kinesisanalytics.

2. On the Kinesis Data Analytics dashboard, choose **Create analytics application**.

3. On the **Kinesis Analytics - Create application** page, provide the application details as follows:

   - For **Application name**, enter `MyApplication`.
   - For **Description**, enter `My java test app`.
   - For **Runtime**, choose **Apache Flink**.
   - Leave the version pulldown as **Apache Flink 1.8 (Recommended Version)**.

4. For **Access permissions**, choose **Create / update IAM role `kinesis-analytics-MyApplication-us-east-1`**.

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/kdacreate.png)

   This action has created these IAM resources, named using your application name and Region as below. Edit the IAM policy to add permissions to access the Kinesis data streams

   - Policy: `kinesis-analytics-service-MyApplication-us-east-1`
   - Role: `kinesis-analytics-MyApplication-us-east-1`

5. Open the IAM console at https://console.aws.amazon.com/iam/.

6. Choose **Policies**. Choose the **`kinesis-analytics-service-MyApplication-us-east-1`** policy that the console created for you in the previous section.

7. On the **Summary** page, choose **Edit policy**. Choose the **JSON** tab.

8. Add the highlighted section of the following policy example to the policy. Replace the sample account IDs (`012345678901`) with your account ID and **`username`** with your username

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "ReadCode",
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:GetObjectVersion"
         ],
         "Resource": [
           "arn:aws:s3:::ka-app-code-username/aws-kinesis-analytics-java-apps-1.0.jar"
         ]
       },
       {
         "Sid": "DescribeLogGroups",
         "Effect": "Allow",
         "Action": [
           "logs:DescribeLogGroups"
         ],
         "Resource": [
           "arn:aws:logs:us-east-1:012345678901:log-group:*"
         ]
       },
       {
         "Sid": "DescribeLogStreams",
         "Effect": "Allow",
         "Action": [
           "logs:DescribeLogStreams"
         ],
         "Resource": [
           "arn:aws:logs:us-east-1:012345678901:log-group:/aws/kinesis-analytics/MyApplication:log-stream:*"
         ]
       },
       {
         "Sid": "PutLogEvents",
         "Effect": "Allow",
         "Action": [
           "logs:PutLogEvents"
         ],
         "Resource": [
           "arn:aws:logs:us-east-1:012345678901:log-group:/aws/kinesis-analytics/MyApplication:log-stream:kinesis-analytics-log-stream"
         ]
       },
       {
         "Sid": "ReadInputStream",
         "Effect": "Allow",
         "Action": "kinesis:*",
         "Resource": "arn:aws:kinesis:us-east-1:012345678901:stream/ExampleInputStream"
       },
       {
         "Sid": "WriteOutputStream",
         "Effect": "Allow",
         "Action": "kinesis:*",
         "Resource": "arn:aws:kinesis:us-east-1:012345678901:stream/ExampleOutputStream"
       }
     ]
   }
   ```

9. On the **MyApplication** page, choose **Configure**.

10. On the **Configure application** page, provide the **Code location**:

    - For **Amazon S3 bucket**, enter `ka-app-code-<username>`.
    - For **Path to Amazon S3 object**, enter `aws-kinesis-analytics-java-apps-1.0.jar`.

11. Under **Access to application resources**, for **Access permissions**, choose **Create / update IAM role `kinesis-analytics-MyApplication-us-east-1`**.

12. Under **Monitoring**, ensure that the **Monitoring metrics level** is set to **Application**.

13. For **CloudWatch logging**, select the **Enable** check box.

14. Choose **Update**.

    ![](https://github.com/rspamzn/streams-handson/blob/master/resources/kdaupdate.png)



------



### Part 4 - Run the Application

1. On the **MyApplication** page, choose **Run**. Confirm the action. Choose the default for the next step on the snapshots. Wait until the application status changes to **Running** 

2. When the application is running, refresh the page. The console shows the **Application graph**.

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/appdag.png)

3. The Kinesis Data Streams monitoring page will also show the number of records written as a verification that the application is processing the data as and when its available ![](https://github.com/rspamzn/streams-handson/blob/master/resources/processed.png)

------



