# Streams Handson

#### Part 1

The objectives of this section is to 

1. Create an EMR Cluster with Flink runtime using AWS console
2. Make Flink Dashboard accessible externally
3. Deploy, test and monitor a Flink application

#### Instructions

1. Log into AWS console and select EMR service. In the EMR service  page,  click on the "Create cluster" button and selet the "Go to advanced options" link.

2. Select the most recent EMR release. Unde the "Software Configuration" section, select Hadoop and Flink applications. EMR will provision these selected application frameworks during the cluster launch process.

3. In the "Edit software settings" subsection, select the "Enter configuration" radio button and key in the following text

   ```
   classification=flink-conf,properties=[taskmanager.numberOfTaskSlots=4,taskmanager.memory.flink.size=4g]
   ```

4. In the Steps section, add a step with the following details.  This will start a long running YARN session in the "detached" mode  where the job manager gets 1 GB of heap space and the task managers 4 GB of heap space assigned. 

   ![Steps configuration](./resources/Screenshot 2020-09-16 at 10.17.20 PM.png)

   



