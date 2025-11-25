---
tags:
  - curl
  - gnu-tools
  - vlan
---
# 30 Examples of cURL Command in Linux

Now, let’s delve into some practical examples of using the cURL command. Each example will be prefaced with an explanation followed by a demonstration of the output.

# Example 1: Fetching Data from a URL

The most basic use of cURL is to fetch the contents of a webpage. Here’s how you can do it:

```bash
curl https://www.example.com
```

This command fetches the HTML content of the webpage at www.example.com.

# Example 2: Downloading a File

cURL can also be used to download files from the internet. Here’s an example:

```bash
curl -O https://www.example.com/file.txt
```

This command downloads the file file.txt from www.example.com and saves it in the current directory.

# Example 3: Sending a POST Request

You can use cURL to send POST requests to a server. Here’s how:

```bash
curl -d "param1=value1&param2=value2" -X POST http://www.example.com
```

This command sends an HTTP POST request to www.example.com with the data param1=value1&param2=value2.

# Example 4: Fetching HTTP Headers

If you want to fetch the HTTP headers from a server, you can use the -I option. Here’s an example:

```bash
curl -I https://www.example.com
```

This command fetches the HTTP headers from www.example.com.

# Example 5: Using a Proxy

If you need to use a proxy, specify it using the -x option. Here’s how:

```bash
curl -x http://proxy.example.com:8080 https://www.example.com
```

This command sends the request to www.example.com through the proxy at proxy.example.com:8080.

# Example 6: Sending Cookies

You can send cookies along with your request using the -b option. Here’s an example:

```bash
curl -b "name=value" https://www.example.com
```

This command sends a cookie with name=value to www.example.com.

# Example 7: Sending User Agent

Websites often use the user agent to deliver content suitable for the client’s browser. To send a user agent with your request, use the -A option:

```bash
curl -A "Mozilla/5.0" https://www.example.com
```

This command sends a request to www.example.com with the user agent set as Mozilla/5.0.

# Example 8: Following Redirects

Some URLs redirect to other URLs. To follow these redirects, use the -L option:

```bash
curl -L https://www.example.com
```

This command follows any redirects from www.example.com.

# Example 9: Saving Output to a File

To save the output of a cURL command to a file, use the -o option:

```bash
curl -o output.html https://www.example.com
```

This command saves the output of www.example.com to output.html.

# Example 10: Uploading Files with FTP

cURL can upload files to a server using FTP. Here’s how:

```bash
curl -T file.txt ftp://ftp.example.com --user username:password
```

This command uploads file.txt to ftp.example.com using the provided username and password.

# Example 11: Resuming a Download

If a download gets interrupted, you can resume it with the -C – option:

```bash
curl -C - -O https://www.example.com/file.txt
```

This command resumes the download of file.txt from www.example.com.

# Example 12: Downloading Multiple Files

To download multiple files, specify multiple URLs:

```bash
curl -O https://www.example.com/file1.txt -O https://www.example.com/file2.txt
```

This command downloads file1.txt and file2.txt from www.example.com.

# Example 13: Sending a DELETE Request

To send a DELETE request, use the -X DELETE option:

```bash
curl -X DELETE https://www.example.com/resource
```

This command sends a DELETE request to the URL www.example.com/resource.

# Example 14: Verbose Output

For detailed information about the request and response, use the -v option:

```bash
curl -v https://www.example.com
```

This command provides verbose output for the request to www.example.com.

# Example 15: Silent Mode

To suppress the progress meter and error messages, use the -s option:

```bash
curl -s https://www.example.com
```

This command fetches the content of www.example.com in silent mode.

# Example 16: Displaying the Download Progress

To display the download progress in a more readable format, use the # option:

```bash
curl -# -O https://www.example.com/file.txt
```

This command downloads file.txt from www.example.com and displays the progress as a progress bar.

# Example 17: Sending JSON Data

To send JSON data in a POST request, use the -H option to set the content type:

```bash
curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST https://www.example.com
```

This command sends a POST request with JSON data to www.example.com.

# Example 18: Using cURL with an API

cURL is often used to interact with APIs. Here’s an example:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com
```

This command sends a request to https://api.example.com with an authorization header.

# Example 19: Downloading Files in the Background

To download a file in the background, use the -O option with an ampersand at the end:

```bash
curl -O https://www.example.com/file.txt &
```

This command downloads file.txt from www.example.com in the background.

# Example 20: Sending Data from a File

To send data from a file in a POST request, use the @ symbol:

```bash
curl -d @data.txt -X POST https://www.example.com
```

This command sends a POST request with the data from data.txt to www.example.com.

### Example 21: Fetching Content from FTP Server

cURL can be used to fetch content from an FTP server. Here’s how:

```bash
curl ftp://ftp.example.com/file.txt --user username:password
```

This command fetches file.txt from ftp.example.com using the provided username and password.

# Example 22: Fetching Content from a Password-Protected Website

To fetch content from a password-protected website, use the -u option:

```bash
curl -u username:password https://www.example.com
```

This command fetches the content from www.example.com using the provided username and password.

# Example 23: Fetching Content from a Website with SSL

To fetch content from a website with SSL, use the -k option:

```bash
curl -k https://www.example.com
```

This command fetches the content from www.example.com, ignoring any SSL certificate warnings.

# Example 24: Sending a PUT Request

To send a PUT request, use the -X PUT option:

```bash
curl -X PUT -d "data" https://www.example.com/resource
```

This command sends a PUT request with the data “data” to www.example.com/resource.

# Example 25: Fetching the Response Headers

To fetch only the response headers, use the -I option:

```bash
curl -I https://www.example.com
```

This command fetches only the response headers from www.example.com.

# Example 26: Fetching Content from a Website with Cookies

To fetch content from a website with cookies, use the -b option:

```bash
curl -b cookies.txt https://www.example.com
```

This command fetches the content from www.example.com using the cookies stored in cookies.txt.

# Example 27: Fetching Content from a Website with Custom Headers

To fetch content from a website with custom headers, use the -H option:

```bash
curl -H "Custom-Header: Value" https://www.example.com
```

# Example 28: Fetching Content from a Website with a Timeout

To fetch content from a website with a timeout, use the -m option:

```bash
curl -m 10 https://www.example.com
```

This command fetches the content from www.example.com with a timeout of 10 seconds.

# Example 29: Fetching Content from a Website in Verbose Mode

To fetch content from a website in verbose mode, use the -v option:

```bash
curl -v https://www.example.com
```

This command fetches the content from www.example.com in verbose mode, displaying detailed information about the request and response.

# Example 30: Fetching Content from a Website and Displaying the Progress Meter

To fetch content from a website and display the progress meter, use the -# option:

```bash
curl -# https://www.example.com
```

This command fetches the content from www.example.com and displays the progress meter.

# vlan

```
curl --vlan-priority 4 https://example.com
```

[usingcurl](https://everything.curl.dev/usingcurl/netrc.html)