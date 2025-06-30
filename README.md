# Loadable-Kernel-Module-Linux

# 🛡️ Linux Kernel Module: execve Hook for Program Restriction

This is a **Linux Kernel Module (LKM)** designed to enhance system security by **intercepting the `execve` syscall** and preventing execution of blacklisted programs by unauthorized (non-root) users. It leverages **ftrace hooking** and dynamic symbol resolution via `kprobes` to implement real-time enforcement of execution restrictions at the kernel level.



## 📌 What This Project Does

This kernel module:
- Intercepts the **`execve` system call** via **ftrace**.
- Checks the program being executed against a **hardcoded blocklist**.
- Allows **only root (UID 0)** to run blocked programs.
- Blocks execution and logs a kernel message if a restricted program is executed by any other user.

### 🔐 Why?
- Adds an extra layer of security on Linux systems.
- Prevents usage of potentially dangerous or admin-level programs (like `nmap`, `netcat`, `tcpdump`) by regular users.


## ⚙️ How It Works

### 🧱 Core Components
- **`fh_sys_execve`**: Custom function to replace `execve`, handles blocking logic.
- **`is_blocked()`**: Checks if the given binary path is in the blocklist.
- **`check_path()`**: Safely copies and resolves the real path of the binary.
- **`ftrace_hook` struct**: Wraps function hooking details.
- **`kprobe`**: Used to dynamically resolve `kallsyms_lookup_name`, since it's not exported in modern kernels.
- **`within_module()`**: Ensures the hook doesn't intercept itself (avoids recursion).

### 📝 Blocklist Example

static const char *blocklist[] = {
    "/usr/bin/nmap",
    "/usr/bin/netcat",
    "/usr/bin/nc",
    "/usr/bin/wireshark",
    "/usr/bin/tcpdump",
    "/usr/bin/vim",
    NULL
};

You can expand this list based on your security requirements.

---

## 🧪 How to Compile and Run

### 📄 Prerequisites
- Linux system with matching kernel headers:

sudo apt install linux-headers-$(uname -r)

make utility

### 🛠️ Step-by-Step

#### 1. 📦 Compile the Kernel Module
Make sure you have a `Makefile` like this and also ensure you have block_execve.c and Makefile in same directory and that directory must be your PWD:

obj-m += execve_hook.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

Then compile:

make


#### 2. 📥 Insert the Module

sudo insmod block_execve.ko

Check logs to confirm it's loaded:

dmesg | tail

You should see lines like:

execve_hook: Blocklisted program: /usr/bin/nmap
execve_hook: Module loaded successfully


#### 3. 🚫 Try Running a Blocked Program
As a non-root user:

nmap

You should see a `Permission denied` and corresponding log:

dmesg | tail


#### 4. ✅ Root Can Bypass

sudo nmap

Should run normally and log:

execve_hook: Allowed execution by root (UID 0)


#### 5. 🧼 Remove the Module

sudo rmmod execve_hook

Check logs:

dmesg | tail

You should see:

execve_hook: Module unloaded successfully




## 🧠 Notes & Limitations
- Requires **root privileges** to load/unload.
- May be blocked by security modules like SELinux or AppArmor.
- Hooking `execve` can be risky — always test on a **virtual machine** or dev environment.
- `kallsyms_lookup_name` access depends on kernel security policies.




> "With great power comes great responsibility — use kernel hooks wisely. 🧠🛡️"
