# OS-Jackfruit: Mini Container Runtime with Kernel Memory Monitor



OS-Jackfruit is a lightweight container runtime written in C with a Linux kernel module for monitoring and enforcing container memory limits.

The project simulates a simplified Docker-like environment using:

* Linux namespaces
* `clone()`
* `chroot()`
* UNIX domain sockets
* Kernel `ioctl()` communication
* Logging threads and bounded buffers
* Kernel timers and workqueues

---

## Features

### User Space Runtime (`engine.c`)

* Start, stop, run, list, and log containers
* Process isolation using namespaces
* Root filesystem isolation using `chroot()`
* Per-container logs
* Supervisor-client architecture using UNIX sockets
* Threaded logging system with bounded buffer
* Graceful shutdown and cleanup

### Kernel Module (`monitor.c`)

* Tracks container memory usage
* Soft memory limit warnings
* Hard memory limit enforcement using `SIGKILL`
* Periodic monitoring using timers + workqueues
* Character device interface through `/dev/container_monitor`
* Communication with user space using `ioctl()`

---

## Technologies Used

* C
* Linux Kernel Module Programming
* POSIX Threads
* UNIX Domain Sockets
* Linux Namespaces
* `clone()`
* `chroot()`
* `ioctl()`
* Workqueues and Timers
* Git and GitHub

---

## Project Structure

```text
.
├── engine.c
├── monitor.c
├── monitor_ioctl.h
├── Makefile
├── monitor.ko
├── cpu_hog
├── memory_hog
├── rootfs-base/
├── rootfs-alpha/
├── rootfs-beta/
├── logs/
└── README.md
```

---

## How It Works

1. The supervisor starts and creates a control socket.
2. Containers are launched using `clone()` with isolated namespaces.
3. Each container uses its own root filesystem through `chroot()`.
4. The runtime registers containers with the kernel module.
5. The kernel module periodically checks memory usage.
6. If memory exceeds:

   * Soft limit → warning logged
   * Hard limit → process killed
7. Logs are collected and stored in separate files.

---

## Build Instructions

### 1. Install Required Packages

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r) git
```

### 2. Compile the User-Space Runtime

```bash
gcc engine.c -o engine -lpthread
```

### 3. Compile the Test Programs

```bash
gcc cpu_hog.c -o cpu_hog
```

```bash
gcc memory_hog.c -o memory_hog
```

### 4. Compile the Kernel Module

```bash
make
```

This generates:

* `monitor.ko`
* `monitor.o`
* `monitor.mod.o`

### 5. Create Root Filesystem Structure

```bash
mkdir -p rootfs-alpha/bin
mkdir -p rootfs-beta/bin
mkdir -p rootfs-base/bin
```

Copy required binaries:

```bash
cp /bin/sh rootfs-alpha/bin/
cp /bin/sh rootfs-beta/bin/
cp /bin/sh rootfs-base/bin/
```

Copy test programs:

```bash
cp cpu_hog rootfs-beta/
cp memory_hog rootfs-alpha/
```

For ARM64 systems, also copy required libraries:

```bash
mkdir -p rootfs-alpha/lib/aarch64-linux-gnu
mkdir -p rootfs-alpha/lib
mkdir -p rootfs-beta/lib/aarch64-linux-gnu
mkdir -p rootfs-beta/lib
```

```bash
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-alpha/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-alpha/lib/
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-beta/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-beta/lib/
```

---

## Commands Used

### Load Kernel Module

```bash
sudo insmod monitor.ko
ls /dev/container_monitor
```

### Start Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

### Start Container

```bash
sudo ./engine start alpha ./rootfs-alpha /memory_hog --soft-mib 40 --hard-mib 80
```

```bash
sudo ./engine start beta ./rootfs-beta /cpu_hog
```

### Run Container in Foreground

```bash
sudo ./engine run beta ./rootfs-beta "/cpu_hog 10"
```

### List Containers

```bash
sudo ./engine ps
```

### View Logs

```bash
sudo ./engine logs beta
```

### Stop Container

```bash
sudo ./engine stop beta
```

### View Kernel Logs

```bash
sudo dmesg | tail -20
```

---

## Screenshots

### 1. Project Folder Structure

<img width="1919" height="279" alt="image" src="https://github.com/user-attachments/assets/21c73553-7a9e-4669-8eab-7c465082e6e3" />


### 2. Kernel Module Compilation

<img width="1795" height="570" alt="image" src="https://github.com/user-attachments/assets/f5886cbb-3d80-4cc4-8f32-2bd87e3ea0d4" />


### 3. Loading Kernel Module

<img width="1761" height="327" alt="image" src="https://github.com/user-attachments/assets/aa4350b4-50c4-44ff-8dba-cfb33fae4f12" />


### 4. Supervisor Startup

<img width="1725" height="216" alt="image" src="https://github.com/user-attachments/assets/0ef21e76-301c-41c3-8136-bf84f62b2121" />


### 5. Starting Alpha Container

<img width="1868" height="186" alt="image" src="https://github.com/user-attachments/assets/8841c058-d841-4581-a832-0309ed17ba0e" />


### 6. Starting Beta Container

<img width="1919" height="124" alt="image" src="https://github.com/user-attachments/assets/e109d198-2d73-4450-a073-888761751335" />


### 7. Container List (`ps`)
<img width="1792" height="485" alt="image" src="https://github.com/user-attachments/assets/aada6b33-2c61-47b5-80ff-f54a008b3b0f" />



### 8. Viewing Logs

<img width="1595" height="687" alt="image" src="https://github.com/user-attachments/assets/68d1a8cd-aeff-4972-8d60-324236ee0d8c" />


### 9. Memory Limit Trigger from Kernel Module

<img width="1902" height="520" alt="image" src="https://github.com/user-attachments/assets/29e9d0b2-fd0c-4b58-bd33-2a5e9c18ffcf" />


### 10. Stopping Container

<img width="1823" height="258" alt="image" src="https://github.com/user-attachments/assets/4a751a25-31fd-45b2-81da-568d43451855" />
<img width="495" height="646" alt="image" src="https://github.com/user-attachments/assets/863c522a-0165-4b3a-a6f4-925f11e5668d" />



### 11. Final Cleanup
<img width="1812" height="283" alt="image" src="https://github.com/user-attachments/assets/42286848-2190-4dab-bf75-428217f3194b" />



---

## Important Concepts

### Namespaces Used

* `CLONE_NEWPID`
* `CLONE_NEWUTS`
* `CLONE_NEWNS`

### Memory Monitoring

* Soft limit generates a warning
* Hard limit kills the process

### Logging System

* Producer thread reads container logs
* Consumer thread writes logs to file
* Uses a bounded buffer to avoid overflow

### Kernel Techniques Used

* Linked lists
* Mutexes
* Workqueues
* Timers
* Character devices
* IOCTL communication

---

## Challenges Faced

* Permission issues with `clone()`
* Root vs non-root socket access
* Missing runtime libraries inside rootfs
* Dynamic `/proc` and `/sys` files causing Git issues
* GitHub authentication using personal access token

---

## Step-by-Step Reproduction Guide

If someone else wants to run this project from scratch, follow these steps in order.

### Step 1: Clone the Repository

```bash
git clone https://github.com/CJC1406/OS-Jackfruit.git
cd OS-Jackfruit
```

### Step 2: Install Dependencies

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r) git
```

### Step 3: Compile the Project

```bash
gcc engine.c -o engine -lpthread
```

```bash
gcc cpu_hog.c -o cpu_hog
```

```bash
gcc memory_hog.c -o memory_hog
```

```bash
make
```

### Step 4: Create Root Filesystems

```bash
mkdir -p rootfs-alpha/bin
mkdir -p rootfs-beta/bin
mkdir -p rootfs-base/bin
```

Copy shell:

```bash
cp /bin/sh rootfs-alpha/bin/
cp /bin/sh rootfs-beta/bin/
cp /bin/sh rootfs-base/bin/
```

Copy test programs:

```bash
cp cpu_hog rootfs-beta/
cp memory_hog rootfs-alpha/
```

For ARM64 systems:

```bash
mkdir -p rootfs-alpha/lib/aarch64-linux-gnu
mkdir -p rootfs-alpha/lib
mkdir -p rootfs-beta/lib/aarch64-linux-gnu
mkdir -p rootfs-beta/lib
```

```bash
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-alpha/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-alpha/lib/
cp /lib/aarch64-linux-gnu/libc.so.6 rootfs-beta/lib/aarch64-linux-gnu/
cp /lib/ld-linux-aarch64.so.1 rootfs-beta/lib/
```

### Step 5: Load the Kernel Module

```bash
sudo insmod monitor.ko
ls /dev/container_monitor
```

### Step 6: Start the Supervisor

Open Terminal 1:

```bash
sudo ./engine supervisor ./rootfs-base
```

Keep this terminal open.

### Step 7: Start Containers

Open Terminal 2:

```bash
sudo ./engine start alpha ./rootfs-alpha /memory_hog --soft-mib 40 --hard-mib 80
```

```bash
sudo ./engine start beta ./rootfs-beta /cpu_hog
```

### Step 8: Check Running Containers

```bash
sudo ./engine ps
```

### Step 9: View Logs

```bash
sudo ./engine logs alpha
```

```bash
sudo ./engine logs beta
```

### Step 10: Check Kernel Monitor Output

```bash
sudo dmesg | tail -20
```

### Step 11: Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

### Step 12: Cleanup

```bash
sudo pkill -9 -f "./engine"
rm -f /tmp/engine_ctrl.sock
```

---

## Future Improvements

* CPU usage limits
* Network namespace support
* Cgroup integration
* Web dashboard for container monitoring
* Better log formatting
* Container restart policies

---

## Author

Chiranth J C
Deeksha G
