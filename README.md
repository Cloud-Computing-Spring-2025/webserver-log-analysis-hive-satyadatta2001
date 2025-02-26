# Web-Server-Log-Analysis

This repository is designed to test MapReduce jobs using a simple word count dataset.

## Objectives
Problem Statement
You must process a web server log file where each entry is represented as a row in a CSV file. Your tasks include:

1. Count the total number of web requests processed.
2. Analyze the frequency of HTTP status codes (e.g., 200, 404, 500).
3. Identify the top 3 most visited URLs.
4. Conduct traffic source analysis to determine the most common user agents.
5. Detect IP addresses with more than 3 failed requests (status 404 or 500).
6. Analyze traffic trends by calculating the number of requests per minute.
7. Implement partitioning by status code to optimize query performance.

## Setup and Execution

### 1. **Start the Hadoop Cluster**

Run the following command to start the Hadoop cluster:

```bash
docker compose up -d
```

### 2. **Count Total Web Requests**
 This query calculates the overall number of requests processed in the web server logs.
```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/count web pages'
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

### 3. **Analyze Status Codes**

This query determines how often each HTTP status code (e.g., 200, 404, 500) occurs in the logs:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Analyse status codes'
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

### 4. **Identify Most Visited Pages**

This query extracts the top three URLs that received the most visits from users:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Most visited pages'
SELECT url, COUNT(*) AS visits 
FROM web_server_logs 
GROUP BY url 
ORDER BY visits DESC 
LIMIT 3;
```

### 5. **Traffic Source Analysis**

This query identifies the most common user agents (browsers) used to access the web server:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/traffic source'
SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;
```

### 6. **Detect Suspicious Activity**

This query finds IP addresses that have made more than three failed requests (status codes 404 or 500):

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Detect Suspicious'
SELECT ip, COUNT(*) AS failed_requests 
FROM web_server_logs  
WHERE status IN (404, 500) 
GROUP BY ip 
HAVING COUNT(*) > 3;
```

### 7. **Analyze Traffic Trends**

This query calculates the number of requests made per minute to observe traffic patterns over time.:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Analyse Traffic'
SELECT substr(`timestamp`, 1, 16) AS time_minute, COUNT(*) AS requests 
FROM web_server_logs 
GROUP BY substr(`timestamp`, 1, 16) 
ORDER BY time_minute;
```

### 8. **Implement Partitioning**

This step creates a partitioned table in Hive based on HTTP status codes to optimize query performance for analysis:

```bash
CREATE TABLE web_logs_partitioned (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT OVERWRITE TABLE web_logs_partitioned PARTITION (status)
SELECT ip, `timestamp`, url, user_agent, status FROM web_server_logs;

```

### 9. **View the Output Of The Patitioning**

To view the output of my partitioning table, use:

```bash
'/user/hive/output/web_serverlogs_partitioned'
SELECT * FROM default.web_logs_partitioned LIMIT 100;
```

### 10. **Access Hive Server Container**
the command opens an interactive Bash session inside the Hive server container:

1. Use the following command to copy from HDFS:
    ```bash
    docker exec -it hive-server /bin/bash
    ```
### 11. **Copy Output from HDFS to Local Filesystem (Inside the Container)**

This retrieves the query results stored in HDFS (/user/hive/output) and saves them to /tmp/output inside the container:

```bash
hdfs dfs -get /user/hive/output /tmp/output
```
### 12. **Exit the Container**
   ```bash
   exit
   ```
### 11. **Check the Current Working Directory on the Host**
Displays the present working directory, which is where you want to copy the output files.
```bash
pwd
```
### 10. **Copy Output Files from the Docker Container to the Host Machine**
This command copies the /tmp/output directory from the Hive server container to your project directory on the host machine

```bash
docker cp hive-server:/tmp/output /........(Keep the link that is generated after compiling pwd)

```
### 10. **Commit it into GitHub**


```bash
git add.
git commit -m ""
git push origin main
```

