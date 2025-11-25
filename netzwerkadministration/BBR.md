BBR (Bottleneck Bandwidth and Round-trip propagation time) is a TCP congestion control algorithm developed by Google. It aims to improve network throughput and reduce latency by dynamically adjusting the data flow based on real-time network conditions. Unlike traditional congestion control algorithms like CUBIC and Reno, which rely on packet loss to detect congestion, BBR uses bandwidth and round-trip time measurements to optimize data transmission, resulting in more efficient and stable network performance.

**Technical Features and Benefits of BBR:**

- **Improved Throughput:** BBR can achieve higher data transfer rates by utilizing available bandwidth more effectively.
- **Reduced Latency:** By avoiding packet loss and adjusting to real-time network conditions, BBR reduces latency, providing a smoother experience for applications like video streaming and online gaming.
- **Stability:** BBR offers more stable performance under varying network conditions, reducing the chances of network congestion collapse.
- **Efficiency:** It optimizes available network resources, ensuring data flows smoothly even in high-traffic scenarios.

Enabling BBR on Debian 12, 11, or 10 can enhance your network performance, especially for high-bandwidth applications. This guide will show you how to enable BBR on your Debian system.

## Step 1: Verify if BBR is Enabled Already

Before enabling BBR, checking if it’s already enabled on your system is essential. To do this, run the following command:

```bash
sysctl net.ipv4.tcp_congestion_control
```

If BBR is enabled, you’ll see the following output:

```bash
net.ipv4.tcp_congestion_control = bbr
```

If you see a different congestion control algorithm, such as cubic or reno, BBR isn’t enabled.

## Step 2: Update Debian System

Before making any changes to your system, it’s crucial to update it to ensure you have the latest packages and security fixes. To do this, run the following command:

```bash
sudo apt update && sudo apt-get upgrade
```

## Step 3: Check if BBR is Supported on your Debian System

Not all systems support BBR, so checking if your system is essential. To do this, run the following command:

```bash
sudo modprobe tcp_bbr
```

If your system supports BBR, you’ll see no output. If it doesn’t, you’ll see an error message.

## Step 4: Enable BBR via CLI Commands

To enable BBR, run the following command:

```bash
sudo sh -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo sh -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
```

These commands will set the default queuing discipline to fq and enable BBR as the congestion control algorithm.

## Step 5: Reload sysctl For BBR Enablement

To apply the changes, run the following command:

```bash
sudo sysctl -p
```

## Step 6: Verify that BBR is Now Enabled

To verify if BBR is enabled after you ran those commands to enable it, run the following command:

```bash
sysctl net.ipv4.tcp_congestion_control
```

If BBR is enabled, you’ll see the following output:

```bash
net.ipv4.tcp_congestion_control = bbr
```