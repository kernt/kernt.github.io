---
tags:
  - server
  - vnc
  - tigervnc
---
## Initiale einrichtung

Password mit `vncpasswd` erstellen und in `~/.vnc/passwd` ablegen

Firewalld einrichten:

```bash
firewall-cmd --permanent --add-service=vnc-server
firewall-cmd --reload
```

TigerVNc Tools:

- Xvnc - the TigerVNC Server for Unix. Xvnc is both a VNC server and an X
  server with a "virtual" framebuffer. You should normally use the vncserver service to start Xvnc.

- vncpasswd - a program which allows you to change the VNC password used to  
  access your VNC server sessions (assuming that VNC authentication is being used.) This command must be run to set a password before using VNC authentication with any of the servers or services.
  
- vncconfig - a program which is used to configure and control a running instance of Xvnc.

- x0vncserver - an inefficient VNC server which continuously polls any X display, allowing it to be controlled via VNC. It is intended mainly as a demonstration of a simple VNC server.

It also contains the following systemd service:

- vncserver@.service - a service to start a user session with Xvnc and one of the desktop environments available on the system.