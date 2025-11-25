Siege

`siege -c2100 -t3M -f urls.txt`

The test was conducted using `siege` with the following command:

`siege -c50 -t2M -d1 http://server-ip/`

- `-c50` → Simulates 50 concurrent users
- `-t2M` → Runs the test for 2 minutes
- `-d1` → Each user waits **randomly up to 1 second** between requests
- `http://server-ip/` → URL of the static test page

The test was conducted using `siege` with the following command:

`siege -c200 -b -t5M http://server-ip/`

- `-c200` → Simulates 200 concurrent users
- `-b` → Runs in **benchmark mode**, meaning no random delays between requests
- `-t5M` → Runs for **5 minutes**
- `http://server-ip/` → URL of the static test page

This test stresses the server by forcing it to handle a **continuous high load with no pauses**, exposing any weaknesses in request handling, resource allocation, and connection management.

The test was conducted using `siege` with the following command:

`siege -c100 -t3M -f urls.txt`

- `-c100` → Simulates 100 concurrent users
- `-t3M` → Runs for **3 minutes**
- `-f urls.txt` → Uses a file containing multiple URLs for testing

The `urls.txt` file contained a mix of different content types:

http://server-ip/
http://server-ip/contact.html
http://server-ip/testfile10M.bin

