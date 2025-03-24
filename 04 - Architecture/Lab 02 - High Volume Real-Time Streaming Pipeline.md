# Advanced AWS Lab Exercise: High Volume Real-Time Streaming Pipeline

This lab guides you through building a robust real-time streaming pipeline that ingests data from simulated IoT devices and processes it at scale using AWS services. You will set up the following components:

- **Device:** Simulated IoT devices using AWS IoT Core.
- **Cloud (IoT Core):** Secure ingestion of device data.
- **Stream Storage (Kinesis):** Buffer and store high volume streaming data.
- **Real-Time Processing (Flink):** Process the data using Apache Flink via Kinesis Data Analytics.
- **Timeseries Storage:** Store processed data in Amazon Timestream.

---

## Prerequisites

- An AWS account with appropriate permissions to create IoT Core, Kinesis, Kinesis Data Analytics, and Timestream resources.
- Basic familiarity with AWS IoT Core, Kinesis, Apache Flink, and Timestream.
- AWS CLI installed (optional but recommended).
- A development environment for running a sample device simulator (Python is recommended).

---

## Lab Components Overview

1. **AWS IoT Core:** Create a thing, certificate, and IoT policy; simulate device data.
2. **Kinesis Data Streams:** Create a stream to ingest data from IoT Core.
3. **AWS IoT Rule:** Route device messages from IoT Core to Kinesis.
4. **Kinesis Data Analytics for Apache Flink:** Process the incoming data stream.
5. **Amazon Timestream:** Store time series data output from your Flink job.

---

## Step 1: Set Up AWS IoT Core

### 1.1 Create a Thing, Certificate, and Policy

1. **Create an IoT Thing:**
   - Navigate to the AWS IoT Core console.
   - Create a new thing (e.g., `HighVolumeDevice`).

2. **Generate and Activate a Certificate:**
   - In the IoT Core console, generate a new certificate.
   - Download the certificate, public key, and private key.
   - Activate the certificate.

3. **Create an IoT Policy:**
   - Create a policy that permits IoT actions (publish/subscribe) as needed. For example:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": "iot:*",
           "Resource": "*"
         }
       ]
     }
     ```
   - Attach this policy to your certificate.

### 1.2 Simulate High Volume Device Data

Create a Python script to simulate an IoT device that sends data at high frequency:

```python
import time
import json
import random
import paho.mqtt.client as mqtt

# Replace with your AWS IoT endpoint (e.g., a3xxxxxx-ats.iot.region.amazonaws.com)
mqtt_endpoint = "YOUR_IOT_ENDPOINT"
topic = "devices/highvolume/data"

# Configure MQTT client with TLS settings (certificates may be required)
client = mqtt.Client()
client.tls_set()  # Default TLS configuration; customize if needed
client.connect(mqtt_endpoint, 8883)

print("Starting high volume data simulation...")
while True:
    # Create a sample payload
    data = {
        "deviceId": "HighVolumeDevice",
        "timestamp": int(time.time()),
        "temperature": round(random.uniform(20.0, 30.0), 2),
        "humidity": round(random.uniform(30.0, 60.0), 2)
    }
    # Publish data to the IoT topic
    client.publish(topic, json.dumps(data))
    time.sleep(0.1)  # Adjust the sleep interval to simulate high volume traffic
```

## Step 2: Configure AWS IoT Core to Forward Data to Kinesis Data Streams

### 2.1 Create a Kinesis Data Stream
1. **Navigate to the Kinesis Console:**
    - Create a new Kinesis Data Stream (e.g., IoTDataStream).
    - Choose the number of shards based on your expected data throughput.

### 2.2 Create an IoT Rule to Route Data
1. **Create a New IoT Rule:**
    - In the AWS IoT console, navigate to Act > Rules and create a new rule.
    - Use an SQL statement to filter and select messages. For example:

```sql
SELECT * FROM 'devices/+/data'
```
2. **Add an Action:**
    - Choose the action to send data to a Kinesis Data Stream.
    - Specify the stream name (`IoTDataStream`).
    - Provide an IAM role that allows AWS IoT Core to write to your Kinesis stream.

---- 

## Step 3: Set Up Kinesis Data Analytics for Apache Flink
### 3.1 Create a Kinesis Data Analytics Application
1. **Navigate to Kinesis Data Analytics:**
  - Create a new Apache Flink application (e.g., `RealTimeProcessingApp`).
2. **Configure the Application Source:**
  - Set the source to the Kinesis Data Stream (`IoTDataStream`).
  - Configure the stream's region and initial position (typically `LATEST`).

### 3.2 Set Up Amazon Timestream for Time Series Storage
1. **Create a Timestream Database:**
  - Navigate to the Amazon Timestream console.
  - Create a database (e.g., IoTTimeSeriesDB).
2. **Create a Timestream Table:**
  - Within your database, create a table (e.g., DeviceMetrics).
  - Configure retention policies as needed.

### 3.3 Develop Your Apache Flink Job
You will now write a Flink job to process the streaming data. Below is a sample Java snippet illustrating a basic streaming job that reads from Kinesis, performs a simple transformation, and writes to Timestream using a custom sink.

***Note: Implementing the TimestreamSink requires using the AWS SDK for Java. This snippet provides a conceptual framework.***

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.connectors.kinesis.FlinkKinesisConsumer;
import org.apache.flink.streaming.connectors.kinesis.config.ConsumerConfigConstants;
import java.util.Properties;

public class IoTStreamingJob {
    public static void main(String[] args) throws Exception {
        // Set up the Flink execution environment
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // Configure Kinesis consumer properties
        Properties kinesisConsumerConfig = new Properties();
        kinesisConsumerConfig.setProperty(ConsumerConfigConstants.AWS_REGION, "us-east-1");
        kinesisConsumerConfig.setProperty(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

        // Create a Kinesis consumer for the IoTDataStream
        DataStream<String> inputStream = env.addSource(new FlinkKinesisConsumer<>(
            "IoTDataStream",
            new SimpleStringSchema(),
            kinesisConsumerConfig
        ));

        // Process the stream (e.g., parsing JSON, filtering, aggregation)
        DataStream<String> processedStream = inputStream
            .map(value -> {
                // Insert parsing and processing logic here
                return value;
            });

        // Write the processed results to Amazon Timestream using a custom sink
        processedStream.addSink(new TimestreamSink("IoTTimeSeriesDB", "DeviceMetrics"));

        env.execute("IoT Real-Time Processing Job");
    }
}
```
***Tip: You can implement the `TimestreamSink` by extending Flink's `SinkFunction` and using the AWS Timestream Write SDK to batch and write records to Timestream.***

### 3.4 Deploy and Test the Flink Application
1. **Package and Upload Your Application:**
    - Package your Flink job as a JAR.
    - Upload it to your Kinesis Data Analytics application.
2. **Start the Application:**
    - Monitor logs and metrics via CloudWatch to ensure data is being processed.

---

### Step 4: Validate the Pipeline
1. **Simulate Device Data:**
    - Ensure your IoT device simulation script is running.
2. **Monitor Data Ingestion:**
    - Verify messages are flowing into Kinesis Data Streams.
3. **Observe Flink Processing:**
    - Check that the Flink job is running and processing records.
4. **Query Amazon Timestream:**
    - Use the Timestream query editor or AWS CLI to run a query, for example:

```sql

SELECT * FROM "IoTTimeSeriesDB"."DeviceMetrics" LIMIT 10
```

  - Confirm that the processed records appear as expected.

----

## Step 5: Cleanup
**1. Stop the IoT Device Simulation Script.**
**2. Stop and delete the Kinesis Data Analytics application.**
**3. Delete the IoT Rule in AWS IoT Core.**
**4. Delete the Kinesis Data Stream.**
**5. Delete the Timestream Database (if no longer needed).**

----
## Conclusion

This lab exercise provides a comprehensive walkthrough of creating a high volume real-time streaming pipeline in AWS. Experiment with additional transformations, windowing strategies, and sink implementations to tailor the pipeline to your specific use cases.