# Advanced Lab: Real-time Stream Processing with AWS Flink

## Overview
In this advanced lab exercise, you will build and deploy a sophisticated real-time stream processing pipeline using AWS Flink on Amazon Kinesis Data Analytics. You will learn how to:
- Create an input data source with Amazon Kinesis Data Streams.
- Develop an advanced Apache Flink application with stateful processing, windowing, and exactly-once semantics.
- Write aggregated results to Amazon S3.
- Configure checkpointing and fault tolerance.
- Monitor, scale, and troubleshoot your application using CloudWatch.
- Clean up the deployed resources.

## Prerequisites
- **AWS Account:** Ensure you have an active AWS account with permissions for Amazon Kinesis Data Streams, Kinesis Data Analytics, S3, IAM, and CloudWatch.
- **Flink Knowledge:** Familiarity with Apache Flink’s APIs (Java or Scala) and streaming concepts (windowing, state management, checkpointing).
- **Development Tools:** An IDE for Java/Scala development and Maven/Gradle for dependency management.
- **AWS CLI (Optional):** For interacting with AWS services via the command line.

---

## Step 1: Set Up the Input Data Stream

1. **Create a Kinesis Data Stream:**
   - Go to the **Amazon Kinesis** console.
   - Select **Data Streams** and click **Create data stream**.
   - **Name:** `AdvancedFlinkInputStream`
   - **Shard Count:** 1 (adjust as needed for throughput)
   - Click **Create data stream**.

2. **Purpose:**  
   This stream will serve as the dynamic real-time data source for your Flink application.

---

## Step 2: Develop an Advanced Apache Flink Application

1. **Application Overview:**
   - The application reads data from `AdvancedFlinkInputStream`, performs stateful processing using windowing functions, aggregates data, and writes results to an Amazon S3 bucket.
   - It demonstrates:
     - **Stateful Processing:** Using keyed windows.
     - **Windowing:** Tumbling or sliding windows.
     - **Fault Tolerance:** With checkpointing for exactly-once processing.
     - **External Sink:** Writing aggregated results to S3.

2. **Sample Code (Java):**
   - Create a new Maven/Gradle project and add dependencies for Apache Flink and the Kinesis connector.
   - Example snippet:
     ```java
     import org.apache.flink.api.common.eventtime.WatermarkStrategy;
     import org.apache.flink.api.common.functions.AggregateFunction;
     import org.apache.flink.api.common.serialization.SimpleStringSchema;
     import org.apache.flink.connector.file.sink.FileSink;
     import org.apache.flink.connector.file.sink.writer.SimpleStringEncoder;
     import org.apache.flink.streaming.api.datastream.DataStream;
     import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
     import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
     import org.apache.flink.streaming.api.windowing.time.Time;
     import org.apache.flink.streaming.connectors.kinesis.FlinkKinesisConsumer;
     import org.apache.flink.streaming.connectors.kinesis.config.ConsumerConfigConstants;

     import java.time.Duration;
     import java.util.Properties;

     public class AdvancedFlinkProcessor {
         public static void main(String[] args) throws Exception {
             // Set up the execution environment
             final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
             env.enableCheckpointing(60000); // Enable checkpointing every 60 seconds

             // Configure Kinesis consumer properties
             Properties kinesisConsumerConfig = new Properties();
             kinesisConsumerConfig.setProperty(ConsumerConfigConstants.AWS_REGION, "us-east-1");
             kinesisConsumerConfig.setProperty(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

             // Create a Kinesis consumer for the input stream
             DataStream<String> inputStream = env.addSource(
                 new FlinkKinesisConsumer<>("AdvancedFlinkInputStream", new SimpleStringSchema(), kinesisConsumerConfig)
             ).assignTimestampsAndWatermarks(
                 WatermarkStrategy
                     .<String>forBoundedOutOfOrderness(Duration.ofSeconds(10))
                     .withTimestampAssigner((element, timestamp) -> System.currentTimeMillis())
             );

             // Example processing: Count words in tumbling windows
             DataStream<String> aggregatedStream = inputStream
                 .flatMap((String value, org.apache.flink.util.Collector<String> out) -> {
                     for (String word : value.split(" ")) {
                         out.collect(word);
                     }
                 })
                 .returns(String.class)
                 .keyBy(word -> word)
                 .window(TumblingEventTimeWindows.of(Time.seconds(30)))
                 .aggregate(new AggregateFunction<String, Integer, String>() {
                     @Override
                     public Integer createAccumulator() {
                         return 0;
                     }
                     @Override
                     public Integer add(String value, Integer accumulator) {
                         return accumulator + 1;
                     }
                     @Override
                     public String getResult(Integer accumulator) {
                         return "Count: " + accumulator;
                     }
                     @Override
                     public Integer merge(Integer a, Integer b) {
                         return a + b;
                     }
                 });

             // Define a sink to write results to Amazon S3
             FileSink<String> s3Sink = FileSink
                 .forRowFormat(new org.apache.flink.core.fs.Path("s3://<YOUR_S3_BUCKET>/flink-output/"),
                               new SimpleStringEncoder<String>("UTF-8"))
                 .build();
             aggregatedStream.sinkTo(s3Sink);

             // Execute the Flink job
             env.execute("Advanced AWS Flink Real-time Stream Processor");
         }
     }
     ```
   - **Note:** Replace `<YOUR_S3_BUCKET>` with your S3 bucket name. Adjust window size, watermark strategy, and aggregation logic as needed.
3. **Build the Application:**
   - Package your application as an executable JAR file.

---

## Step 3: Deploy the Flink Application on Amazon Kinesis Data Analytics

1. **Create a Kinesis Data Analytics Application:**
   - Open the **Amazon Kinesis Data Analytics** console.
   - Click **Create application**.
   - **Application Type:** Select **Apache Flink**.
   - **Application Name:** `AdvancedFlinkStreamProcessor`
   - Choose the appropriate Flink runtime version (e.g., Flink 1.13 or later).

2. **Upload Your Application Code:**
   - In the application wizard, upload the JAR file you built.
   - Specify the main class (e.g., `com.example.AdvancedFlinkProcessor`).

3. **Configure Input and Output:**
   - **Input:**  
     Map your application to the `AdvancedFlinkInputStream` Kinesis Data Stream.
   - **Output:**  
     Your code writes results to an S3 bucket. Ensure your S3 bucket permissions and IAM roles are correctly configured.

4. **Advanced Settings:**
   - Configure checkpointing, parallelism, and scaling options as needed.
   - Enable CloudWatch logging for real-time monitoring.

5. **Review and Create:**
   - Confirm your settings and create the application.
   - Start the application and monitor its state until it reaches `RUNNING`.

---

## Step 4: Test and Monitor the Application

1. **Inject Test Data:**
   - Use the AWS CLI or a Kinesis data generator to send sample records to `AdvancedFlinkInputStream`.
   - Example CLI command:
     ```bash
     aws kinesis put-record --stream-name AdvancedFlinkInputStream --partition-key key1 --data "flink advanced stream processing example"
     ```
2. **Monitor Application Metrics:**
   - In the Kinesis Data Analytics console, navigate to the **Monitoring** tab.
   - Observe CloudWatch metrics (e.g., checkpoint success, record processing rate).
   - Check CloudWatch Logs for application logs and error messages.

3. **Verify Output:**
   - Go to your S3 bucket and check the output directory (`flink-output/`) for result files.
   - Confirm that the aggregated data (e.g., word counts) is correctly written.

---

## Step 5: Auto Scaling and Fault Tolerance

1. **Configure Parallelism:**
   - Adjust the parallelism of your Flink application in the Kinesis Data Analytics settings to scale with your workload.
2. **Fault Tolerance:**
   - Verify that checkpointing is working by reviewing the application’s recovery logs.
   - Test fault tolerance by simulating failures (e.g., stopping the application and restarting it).

---

## Step 6: Clean Up Resources

1. **Stop the Flink Application:**
   - In the Kinesis Data Analytics console, stop the `AdvancedFlinkStreamProcessor` application.
2. **Delete the Application:**
   - Delete the Kinesis Data Analytics application once stopped.
3. **Delete the Kinesis Data Stream:**
   - In the Kinesis console, delete `AdvancedFlinkInputStream`.
4. **Remove S3 Data (Optional):**
   - Delete the output files in your S3 bucket if they are no longer needed.
5. **Clean Up Other Resources:**
   - Remove any CloudWatch log groups or IAM roles that were created specifically for this lab.

---

## Lab Conclusion

In this advanced lab exercise, you:
- Set up a Kinesis Data Stream as a real-time data source.
- Developed an advanced Apache Flink application featuring stateful windowing, exactly-once processing, and an external S3 sink.
- Deployed the application using Amazon Kinesis Data Analytics with advanced configuration options.
- Tested, monitored, and scaled your application while ensuring fault tolerance.
- Cleaned up all resources to prevent unnecessary charges.

This lab demonstrates how AWS Flink can be used for robust, production-grade real-time stream processing, enabling dynamic data handling and advanced analytics.

Happy Streaming and Advanced Processing!
