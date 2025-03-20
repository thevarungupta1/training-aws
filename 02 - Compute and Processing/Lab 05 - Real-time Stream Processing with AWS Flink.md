# Lab: Real-time Stream Processing with AWS Flink

## Overview
In this lab exercise, you will deploy a real-time stream processing application using AWS Flink (via Amazon Kinesis Data Analytics for Apache Flink). This lab demonstrates how to:
- Create an Amazon Kinesis Data Stream as your data source.
- Develop a sample Apache Flink application to process streaming data.
- Deploy the application using Amazon Kinesis Data Analytics.
- Monitor the application's performance and logs.
- Clean up resources when the lab is complete.

## Prerequisites
- **AWS Account:** An active AWS account with permissions to use Amazon Kinesis Data Analytics, Kinesis Data Streams, IAM, and CloudWatch.
- **Basic Knowledge:** Familiarity with Apache Flink and streaming data concepts.
- **Development Tools:** Java or Scala IDE for building your Flink application (alternatively, use a precompiled JAR provided by AWS).
- **AWS CLI (Optional):** For managing and testing resources via the command line.

---

## Step 1: Set Up the Input Data Stream

1. **Create a Kinesis Data Stream:**
   - Navigate to the **Amazon Kinesis** console.
   - Choose **Data Streams** and click **Create data stream**.
   - **Name:** `FlinkInputStream`
   - **Shard Count:** 1 (for simplicity)
   - Click **Create data stream**.
2. **Purpose:**  
   This stream will serve as the real-time data source for your Apache Flink application.

---

## Step 2: Develop a Sample Apache Flink Application

1. **Application Overview:**
   - The sample application will read data from the `FlinkInputStream`, perform a simple transformation (such as splitting input text into words), and output the results (e.g., print them to the console or forward to another stream).
2. **Sample Code (Java):**
   - Create a new Java project in your IDE and add dependencies for Apache Flink and the Kinesis connector.
   - Example snippet:
     ```java
     import org.apache.flink.api.common.functions.FlatMapFunction;
     import org.apache.flink.api.common.typeinfo.Types;
     import org.apache.flink.streaming.api.datastream.DataStream;
     import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
     import org.apache.flink.streaming.connectors.kinesis.FlinkKinesisConsumer;
     import org.apache.flink.streaming.util.serialization.SimpleStringSchema;
     import org.apache.flink.util.Collector;
     import org.apache.flink.streaming.connectors.kinesis.config.ConsumerConfigConstants;
     import java.util.Properties;

     public class FlinkWordCount {
         public static void main(String[] args) throws Exception {
             // Set up the execution environment
             final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

             // Set Kinesis consumer properties
             Properties kinesisConsumerConfig = new Properties();
             kinesisConsumerConfig.setProperty(ConsumerConfigConstants.AWS_REGION, "us-east-1");
             kinesisConsumerConfig.setProperty(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

             // Create a Kinesis consumer
             DataStream<String> input = env.addSource(
                 new FlinkKinesisConsumer<>("FlinkInputStream", new SimpleStringSchema(), kinesisConsumerConfig)
             );

             // Process the stream: split lines into words
             DataStream<String> words = input
                 .flatMap(new FlatMapFunction<String, String>() {
                     @Override
                     public void flatMap(String value, Collector<String> out) {
                         for (String word : value.split(" ")) {
                             out.collect(word);
                         }
                     }
                 })
                 .returns(Types.STRING);

             // Print the processed data
             words.print();

             // Execute the Flink job
             env.execute("AWS Flink Real-time Word Count");
         }
     }
     ```
   - **Note:** Ensure you add the required dependencies (via Maven or Gradle) for Apache Flink and the Kinesis connector.
3. **Build the Application:**
   - Package your application as a JAR file, which will be deployed to Amazon Kinesis Data Analytics.

---

## Step 3: Deploy the Apache Flink Application on Amazon Kinesis Data Analytics

1. **Create a Kinesis Data Analytics Application:**
   - Go to the **Amazon Kinesis Data Analytics** console.
   - Click **Create application**.
   - **Application Type:** Select **Apache Flink**.
   - **Application Name:** `FlinkStreamProcessor`
   - Choose the desired Flink runtime version (e.g., Flink 1.12 or a later supported version).
2. **Configure Application Code:**
   - In the application creation wizard, upload the JAR file you built.
   - Specify the main class (e.g., `com.example.FlinkWordCount`).
3. **Configure the Input:**
   - Add an input configuration to connect the application to the `FlinkInputStream` Kinesis Data Stream.
   - Define the data format (e.g., plain text) based on your application.
4. **Configure Checkpoints and Monitoring:**
   - Set up checkpointing to ensure state consistency.
   - Optionally, enable CloudWatch logging for real-time monitoring.
5. **Review and Create:**
   - Review your configurations and create the application.
6. **Start the Application:**
   - Once created, start the `FlinkStreamProcessor` application from the console.
   - Monitor the status until it reaches the `RUNNING` state.

---

## Step 4: Test and Monitor the Application

1. **Send Test Data:**
   - Use the AWS CLI or a Kinesis Data Generator to send sample records into `FlinkInputStream`.
   - Example CLI command:
     ```bash
     aws kinesis put-record --stream-name FlinkInputStream --partition-key key1 --data "hello world from AWS Flink"
     ```
2. **Monitor Processing:**
   - Navigate to the **Monitoring** tab in the Kinesis Data Analytics console.
   - View CloudWatch metrics and logs to ensure data is processed as expected.
   - Confirm that your application is correctly splitting the text and outputting the words.

---

## Step 5: Clean Up Resources

1. **Stop the Flink Application:**
   - In the Kinesis Data Analytics console, stop the `FlinkStreamProcessor` application.
2. **Delete the Application:**
   - Once stopped, delete the Kinesis Data Analytics application.
3. **Delete the Kinesis Data Stream:**
   - Navigate to the Kinesis Data Streams console and delete the `FlinkInputStream`.
4. **Additional Cleanup (Optional):**
   - Remove any CloudWatch log groups or other resources created specifically for this lab to avoid additional charges.

---

## Lab Conclusion

In this lab exercise, you:
- Created an Amazon Kinesis Data Stream as the input data source.
- Developed a sample Apache Flink application to process real-time streaming data.
- Deployed and configured the application using Amazon Kinesis Data Analytics.
- Tested the stream processing by injecting test data and monitoring the results.
- Cleaned up the resources to prevent unwanted charges.

This lab demonstrates the power of AWS Flink for dynamic, real-time stream processing, enabling scalable and efficient data handling for modern applications.

Happy Streaming!
