# Traffic Camera Vehicle Detection Dashboard

This project is a traffic monitoring system that continuously scrapes traffic camera images from the [OneMotoring Traffic Cameras](https://onemotoring.lta.gov.sg/content/onemotoring/home/driving/traffic_information/traffic-cameras.html), processes those images to detect vehicles using YOLO object detection, and displays the results in a real-time dashboard built using Dash.

## Features

- **Web Scraping**: Periodically scrapes traffic camera images from the website.
- **Vehicle Detection**: Detects cars, trucks, buses, and motorbikes in the images using a YOLO model.
- **Real-time Dashboard**: Displays vehicle counts and images with bounding boxes drawn around detected vehicles.
- **Image Processing**: Processes camera images in parallel and updates the data periodically.

## How It Works

1. **Scraping**: The scraper fetches traffic camera images from the OneMotoring traffic information page and stores them locally. It parses the main traffic camera page to get a list of camera URLs and then extracts image snapshots from each camera.

2. **YOLO Object Detection**: After downloading the images, YOLOv10x is used to detect vehicles (cars, trucks, buses, and motorbikes) in the images. Each detected vehicle is classified and the results are drawn on the images as bounding boxes.

3. **Real-time Dashboard**: The processed images, along with vehicle counts, are displayed in a real-time dashboard built with Dash. The dashboard refreshes periodically (every 10 seconds) to show updated counts and processed images.

4. **Parallel Execution**: The scraping and image processing tasks are executed in parallel using asyncio, which helps efficiently manage multiple network requests and I/O operations.

5. You can view the dashboard at http://localhost:8050/

## Project Enhancement and Production Considerations

### 1. Handling Data at Scale

When dealing with a large number of traffic camera streams and simultaneous image processing, several strategies can be implemented to manage data effectively:

- **Distributed Scraping**: Utilize distributed systems like **Apache Kafka** or **Celery** for task management to divide scraping tasks across multiple workers. This ensures that you can handle multiple streams simultaneously.
  
- **Batch Processing**: Rather than processing images one by one, implement **batch processing** where images are grouped together and processed in chunks to optimize resource usage.

- **Database Storage**: Instead of saving all the processed data as flat files (i.e., JSON and images in the file system), consider using:
  - **Cloud-based storage** like AWS S3 or Google Cloud Storage for storing large volumes of images.
  - **NoSQL databases** (e.g., MongoDB) to store metadata and vehicle counts as they provide faster querying capabilities and horizontal scaling.

- **Asynchronous Processing**: Using **asyncio** and **aiohttp** ensures that I/O-bound tasks (like downloading images) are done concurrently without blocking the CPU. This is key for managing multiple streams in parallel. (We are already doing this)

- **Stream Processing**: Use stream processing frameworks such as **Apache Flink** or **Apache Spark Streaming** to process real-time data streams at scale efficiently. These frameworks allow continuous data ingestion and processing.

### 2. Making Inference Faster

To speed up vehicle detection inference, consider the following optimizations:

- **Model Optimization**:
  - Use **smaller models** (such as YOLOv10s or YOLOv10n) or **model quantization** to reduce the computation load while retaining acceptable accuracy.
  - Apply **pruning** to remove unnecessary weights from the model, which can significantly reduce inference time.
  
- **Hardware Acceleration**:
  - Deploy the YOLO model on **GPUs** or **TPUs** to accelerate inference.
  - Use **NVIDIA TensorRT** for faster inferences on NVIDIA GPUs by optimizing the model specifically for the target hardware.

- **Parallel Processing**:
  - Use frameworks like **Dask** or **Ray** to parallelize the image processing task across multiple CPU/GPU cores.
  - Use **multi-threading** or **multi-processing** to process multiple images concurrently.

- **Edge Computing**:
  - If real-time inference is critical, deploy the model closer to where the data is generated (edge devices like cameras) to reduce latency and bandwidth usage.
  - **ONNX Runtime** can be used to convert the model into a format optimized for edge devices, ensuring low-latency inferences.

### 3. Taking to Production: Challenges and Considerations

Bringing the project to production involves addressing several key challenges:

- **Deployment and Infrastructure**:
  - **Containerization**: Use **Docker** to containerize the application, making it easy to deploy across different environments. **Kubernetes** can help with orchestrating containers for load balancing and scaling.
  - **Serverless Architecture**: For on-demand scaling, use a **serverless** approach where scraping and image processing are triggered by events (such as new camera images) using AWS Lambda or Google Cloud Functions.

- **Scalability**:
  - The system needs to handle hundreds or even thousands of camera streams concurrently. Leveraging **auto-scaling** infrastructure like AWS EC2 Auto Scaling or Kubernetes’ Horizontal Pod Autoscaler can handle sudden increases in load.
  
- **Latency and Real-Time Constraints**:
  - Ensure that processing delays are minimal to maintain real-time reporting. Using GPUs or specialized hardware like TPUs can help achieve low-latency inference times.
  
- **Monitoring and Alerts**:
  - Set up **monitoring** with tools like **Prometheus** and **Grafana** to track the health of the scraping and processing systems.
  - Use logging frameworks like **ELK Stack** to track errors and performance bottlenecks in production.

- **Data Privacy**:
  - When processing real-time traffic images, be cautious of privacy issues, especially if the cameras capture sensitive areas. Implement **data anonymization** techniques like **blurring license plates or faces**.

- **Handling Failures**:
  - Use **retry mechanisms** and **circuit breakers** for handling failed requests or slow network connections.
  - Implement backup strategies for long-term data retention and disaster recovery.

### 4. Evaluation and Improvement Ideas

#### Evaluation of Results

- **Accuracy of Detection**: Evaluating detection accuracy is crucial. This can be done by comparing model predictions against a manually annotated validation set of images.
  - Metrics like **precision, recall, F1-score**, and **mean Average Precision (mAP)** should be used to assess performance.
  
- **Latency**: Measure the time it takes for the entire process (from scraping to vehicle detection and displaying the results) and optimize the slowest parts.

- **Resource Utilization**: Monitor CPU, memory, and disk usage during scraping and inference to ensure resources are being used efficiently.

#### Gaps and Areas for Improvement

- **False Positives/Negatives**: There may be instances where the YOLO model detects vehicles incorrectly. Fine-tuning the model or using more specialized vehicle detection models could improve accuracy.
  
- **Handling Occlusions**: If vehicles are occluded (partially blocked), the current model might miss them. Implementing **advanced object tracking** (e.g., SORT) could help maintain detection accuracy in crowded scenes.

- **Lighting Conditions**: Vehicle detection performance may degrade in low-light conditions. Using techniques like **image enhancement** or training the model with low-light data can help mitigate this issue.

- **Vehicle Counting**: Add a post-processing step to track and count vehicles more accurately over time instead of simply counting the number of vehicles in each frame.

### 5. Related Use Cases

This traffic monitoring system has several other potential applications beyond vehicle detection:

- **Pedestrian Detection and Safety**:
  - Modify the model to detect pedestrians in urban areas. This could help improve road safety by alerting authorities to crowded or dangerous areas.
  
- **Smart Parking Solutions**:
  - Use the system for smart parking management by detecting empty parking spaces and integrating it into a parking assistance app.

- **Traffic Flow Monitoring**:
  - Extend the system to estimate traffic flow by calculating vehicle density and speed. This data can be fed into smart city systems to optimize traffic lights or reroute vehicles.

- **Accident Detection**:
  - By analyzing traffic camera streams in real-time, the system could detect accidents or unusual behavior (like sudden stops or crashes) and notify emergency services.

- **Weather-Related Traffic Alerts**:
  - Integrate weather data to monitor its impact on traffic, such as detecting road congestion during storms or fog, and provide real-time alerts to drivers.

## Conclusion

Scaling this system for production involves solving several technical challenges, including optimizing model inference, managing large amounts of traffic data, and ensuring real-time performance. With improvements to the underlying infrastructure, model, and evaluation mechanisms, this system could be deployed as a robust traffic monitoring and management solution for smart cities.


