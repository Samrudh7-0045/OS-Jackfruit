# OS-Jackfruit Project

## Team Information

| Name            | SRN           |
| --------------- | ------------- |
| Samrudh C Patil | PES1UG24CS417 |
| Saksham         | PES1UG24CS417 |

---

## Project Overview

OS-Jackfruit is a multi-container runtime project demonstrating core operating systems concepts including:

* Process isolation using Linux namespaces
* Container supervision and process lifecycle management
* Inter-process communication (IPC)
* Memory monitoring and enforcement
* Scheduling experiments using process priorities (`nice` values)
* Kernel module integration for container monitoring

---

## Features Implemented

### 1. Multi-Container Supervision

* Supports running multiple containers under a single supervisor.
* Containers can be started, monitored, stopped, and logged.

### 2. Namespace Isolation

Implemented:

* PID Namespace
* UTS Namespace
* Mount Namespace

### 3. Resource Monitoring

* Soft memory limits with warnings
* Hard memory limits with process termination

### 4. Scheduling Experiments

* CPU-bound containers with different priorities
* CPU-bound vs I/O-bound process behavior under Linux CFS

---

## Build and Run

```bash
git clone https://github.com/Samrudh7-0045/OS-Jackfruit.git
cd OS-Jackfruit/boilerplate
sudo make
sudo insmod monitor.ko
sudo ./engine supervisor ./rootfs-base
```

---

## Sample Commands

### Start Containers

```bash
sudo ./engine start alpha ./rootfs-alpha "/cpu_hog 10"
sudo ./engine start beta ./rootfs-beta "/cpu_hog 10"
```

### View Containers

```bash
sudo ./engine ps
```

### View Logs

```bash
sudo ./engine logs alpha
```

### Stop Container

```bash
sudo ./engine stop alpha
```

---

## Cleanup

```bash
sudo umount rootfs-alpha/proc
sudo umount rootfs-beta/proc
sudo rmmod monitor
```

---

## Repository

GitHub Repository:

[https://github.com/Samrudh7-0045/OS-Jackfruit](https://github.com/Samrudh7-0045/OS-Jackfruit)

---

## Contributors

* Samrudh C Patil (PES1UG24CS417)
* Saksham (PES1UG24CS417)
