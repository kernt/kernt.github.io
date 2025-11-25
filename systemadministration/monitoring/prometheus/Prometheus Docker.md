# Configure Docker to Expose Metrics

To allow Prometheus to gather Docker metrics, modify the Docker `/etc/docker/daemon.json` configuration file. Add the following code:

```JSON
{  
  "metrics-addr" : "127.0.0.1:9323"  
}
```

- **Port 9323**: The port from which Prometheus will scrape Docker metrics.
- Restart Docker with the following command:

`sudo systemctl restart docker`

- If the above setting is not working then we can also add the below stanza to the `daemon.json` file.

```JSON 
{  
  "metrics-addr": "0.0.0.0:9323"  
}
```