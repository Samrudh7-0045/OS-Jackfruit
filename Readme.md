# OS-Jackfruit: Mini Container Runtime with Kernel Memory Monitor (Windows Host + Ubuntu VM Setup)

## Team Members

| Name            | SRN           |
| --------------- | ------------- |
| Samrudh C Patil | PES1UG24CS417 |
| Saksham         | PES1UG24CS404 |

---

## Overview

OS-Jackfruit is a lightweight container runtime implemented in C with a Linux kernel module for memory monitoring and enforcement.

This project was developed and tested using **Windows host + VirtualBox + Ubuntu VM** (instead of Mac).

Technologies used:

* Linux namespaces
* `clone()`
* `chroot()`
* UNIX domain sockets
* `ioctl()`
* POSIX threads
* Linux kernel module (`monitor.ko`)
* Git and GitHub

---

## Windows Host Environment Used

### Host System

* Windows 10/11
* Oracle VirtualBox

### Guest VM

* Ubuntu 22.04 / 24.04
* Secure Boot OFF

---

## Project Structure

```text
OS-Jackfruit/
└── boilerplate/
    ├── engine.c
    ├── monitor.c
    ├── monitor_ioctl.h
    ├── Makefile
    ├── cpu_hog.c
    ├── memory_hog.c
    ├── io_pulse.c
    ├── monitor.ko
    ├── rootfs-base/
    ├── rootfs-alpha/
    ├── rootfs-beta/
    └── README.md
```

---

## Setup and Build (As Per Project Execution)

### Clone Repository

```bash
git clone https://github.com/Samrudh7-0045/OS-Jackfruit.git
cd OS-Jackfruit/boilerplate
```

### Install Dependencies

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
sudo apt install gcc-12 g++-12 -y
```

---

## Create Root Filesystems

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz

tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta
```

---

## Build Project

```bash
sudo make
```

Copy workloads:

```bash
cp cpu_hog memory_hog io_pulse rootfs-alpha/
cp cpu_hog memory_hog io_pulse rootfs-beta/
```

---

## Load Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
sudo dmesg | tail -3
```

---

## Start Supervisor (Terminal 1)

```bash
sudo ./engine supervisor ./rootfs-base
```

---

## Container Commands (Terminal 2)

### Start Containers

```bash
sudo ./engine start alpha ./rootfs-alpha "/cpu_hog 20"
sudo ./engine start beta ./rootfs-beta "/cpu_hog 20"
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

## Memory Limit Test

```bash
sudo ./engine start memtest ./rootfs-alpha "/memory_hog 8 500" --soft-mib 20 --hard-mib 40
sudo dmesg | grep container_monitor
sudo ./engine ps
```

---

## Scheduler Experiment

```bash
sudo ./engine start cpu-high ./rootfs-alpha "/cpu_hog 20" --nice 0
sudo ./engine start cpu-low ./rootfs-beta "/cpu_hog 20" --nice 10
sudo ./engine ps
```

---

## Cleanup

```bash
sudo umount rootfs-alpha/proc
sudo umount rootfs-beta/proc
sudo rmmod monitor
sudo dmesg | tail -5
```

---

## Screenshots Included

1. Kernel Module Loaded

<img width="1013" height="356" alt="image" src="https://github.com/user-attachments/assets/86a7ba60-de12-40ef-8a7b-ac64314e64ee" />


2. Supervisor Running

<img width="957" height="35" alt="image" src="https://github.com/user-attachments/assets/78d043d9-8b75-482c-a9e3-aa5d0b024f4c" />


3. Alpha and Beta Containers

<img width="647" height="58" alt="image" src="https://github.com/user-attachments/assets/108a8300-ae88-44e4-acee-6f1e0fd638c5" />

<img width="678" height="52" alt="image" src="https://github.com/user-attachments/assets/ecde820e-cbbe-4093-8f9d-88f2a1586d85" />





4. Container Metadata (`engine ps`)

<img width="642" height="82" alt="image" src="https://github.com/user-attachments/assets/655a4061-dd45-40cc-9470-eb2f2ba419d5" />


5. Container Logs

<img width="643" height="232" alt="image" src="https://github.com/user-attachments/assets/a4eb6f21-2d9e-48b3-8b0d-2b0c76d6b147" />


6. Memory Soft/Hard Limit Output

<img width="625" height="120" alt="image" src="https://github.com/user-attachments/assets/1017f86b-4906-4d0f-bd79-c3fb3ef925da" />


7. Scheduler Experiment

<img width="661" height="402" alt="image" src="https://github.com/user-attachments/assets/4f280ee9-3eb8-4f15-bad5-015a6c265c54" />

<img width="666" height="425" alt="image" src="https://github.com/user-attachments/assets/d3b37490-92d4-449d-951c-c32d7c59a4aa" />


8. Final Cleanup

<img width="913" height="77" alt="image" src="https://github.com/user-attachments/assets/67407dea-9397-4a77-ae83-68e536c72743" />


---

## Key Concepts Demonstrated

### Namespaces

* CLONE_NEWPID
* CLONE_NEWUTS
* CLONE_NEWNS

### Memory Monitoring

* Soft limit warning
* Hard limit kill

### Scheduling

* nice 0 vs nice 10
* Linux CFS behavior

### IPC

* Pipes for logging
* UNIX sockets for control

---

## Challenges Faced

* VM stalls/crashes during setup
* Kernel module build issue (`gcc-12`)
* GitHub authentication with personal access token
* Rootfs workload copy sequencing

---

## Repository

[https://github.com/Samrudh7-0045/OS-Jackfruit](https://github.com/Samrudh7-0045/OS-Jackfruit)

---

## Contributors

Samrudh C Patil
Saksham
