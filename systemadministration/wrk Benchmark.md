The test was conducted using `wrk` with the following command:

$ wrk -t4 -c100 -d30s http://server-ip/

- `-t4` → Number of threads (4)
- `-c100` → Number of concurrent connections (100)
- `-d30s` → Duration of the test (30 seconds)
- `http://server-ip/` → URL of the test page

This test simulates **100 users making repeated requests for 30 seconds**, providing insights into sustained performance and response times under consistent load.

The test was conducted using `wrk` with the following command:

$ wrk -t8 -c500 -d60s http://server-ip/

- `-t8` → Number of threads (8)
- `-c500` → Number of concurrent connections (500)
- `-d60s` → Duration of the test (60 seconds)
- `http://server-ip/` → URL of the test page

The test was conducted using `wrk` with the following command:

$ wrk -t4 -c50 -d30s --latency http://server-ip/testfile10M.bin

- `-t4` → Number of threads (4)
- `-c50` → Number of concurrent connections (50)
- `-d30s` → Duration of the test (30 seconds)
- `--latency` → Enables detailed latency reporting
- `http://server-ip/testfile10M.bin` → URL of the 10MB test file

This test simulates **50 concurrent users continuously downloading a 10MB file** for 30 seconds. It measures the server’s ability to sustain high transfer speeds while keeping latency minimal.

