# Streams Handson

#### Part 1

The objectives of this section is to 

1. Create an EMR Cluster with Flink runtime using AWS console
2. Make Flink Dashboard accessible externally
3. Deploy, test and monitor a Flink application

#### Instructions

1. Log into AWS console and select EMR service. In the EMR service  page,  click on the "Create cluster" button and selet the "Go to advanced options" link.

2. Select the EMR release 5.30.0. Under the "Software Configuration" section, select Hadoop and Flink applications. EMR will provision these selected application frameworks during the cluster launch process.

3. In the "Edit software settings" subsection, select the "Enter configuration" radio button and key in the following text

   ```
   classification=flink-conf,properties=[taskmanager.numberOfTaskSlots=4,jobmanager.memory.flink.size=2g]
   ```

4. In the Steps section, add a step with the following details.  This will start a long running YARN session in the "detached" mode  where the job manager gets 1 GB of heap space and the task managers 4 GB of heap space assigned. 

   ![Steps configuration](https://github.com/rspamzn/streams-handson/blob/master/resources/steps.png)

5. Click on the "Next" button and leave the Hardware Configurations to the defaults in the wizard. Click on the "Next" button

6. In the "General Options", remove the "Termination Protection" check. Provide your own name for the cluster and click the "Next" button

7. In the Security options page, choose your existing key pair. Leave the rest to the defaults as shown in the wizard and click on "Create cluster" button.

8. Wait for the cluster status to change to "Running". Check the "Steps" tab to make sure that the Status of the steps show Running also as below

   ![](https://github.com/rspamzn/streams-handson/blob/master/resources/running.png)











