#### **Benchmarking Command & Explanation**

The test was conducted using **Apache Benchmark (`ab`)** with the following command:

```sh
ab -n 10000 -c 100 http://server-ip/
```

- `-n 10000` → Total number of requests (10,000)
- `-c 100` → Number of concurrent users (100)
- `http://server-ip/` → URL of the static test page

This command simulates 100 users making repeated requests to a simple HTML file to evaluate the server’s request-handling speed and efficiency.

The test was conducted using **Apache Benchmark (`ab`)** with the following command:

`ab -n 500 -c 10 http://server-ip/testfile10M.bin`

- `-n 500` → Total number of requests (500)
- `-c 10` → Number of concurrent users (10)
- `http://server-ip/testfile10M.bin` → URL of the 10MB test file

This command simulates **10 concurrent users** downloading a large file repeatedly, providing insights into the throughput and efficiency of each web server in handling large payloads.

The test was conducted using **Apache Benchmark (`ab`)** with the following command:

`ab -n 20000 -c 1000 http://server-ip/`

- `-n 20000` → Total number of requests (20,000)
- `-c 1000` → Number of concurrent users (1,000)
- `http://server-ip/` → URL of the test page

This command simulates a heavy traffic load by sending **1,000 concurrent requests** to the test page. The goal is to measure the number of requests the server can handle per second while monitoring latency and potential failures.
