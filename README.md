# SGX Key Recovery (BunnyHop UGP)

This project contains the implementation of the BunnyHop attack and SGX Key Recovery on Intel SGX using the SGX-Step framework. It enables timer-based single-stepping of SGX enclaves to recover cryptographic keys or other secrets from within the enclave.

## Documentation
For detailed knowledge, theoretical background, presentations, and the full project report, please check the `sgx ugp docs/` directory in this repository. It contains comprehensive resources about the setup and the SGX Key Recovery implementation.

## Project Structure
A brief overview of the main components in this directory:
- `sgx ugp docs/`: Contains the detailed setup guide, presentation, and project report.
- `app/`: Contains the target enclave applications, benchmark programs, and the key recovery code (`sgx-key-recovery`).
- `run_bunnyhop_attack.sh`: Automated script to orchestrate the BunnyHop attack and generate plots.
- `plot_bunnyhop_results.py`: Python script used to visualize the results (invoked by the runner script).
- `check_accuracy.sh`: Script to run the key recovery repeatedly and evaluate its success rate.
- `Makefile` & C files (`main.c`, etc.): Root-level files for compiling attack components.


## Prerequisites & Setup Guide

The following steps are required to configure your system for SGX-Step and run the attack.

### 1. Clone the SGX-Step Repository
```bash
git clone https://github.com/Satwik-Raj/sgx-step.git
cd sgx-step
```

### 2. Configure System Requirements
You must pass specific boot parameters to the Linux kernel to ensure the APIC timer interrupts work properly for single-stepping. 

Edit your grub configuration:
```bash
sudo vim /etc/default/grub
```
Add the following parameters to `GRUB_CMDLINE_LINUX_DEFAULT`:
```text
quiet splash iomem=relaxed no_timer_check clearcpuid=308,295,514 pti=off isolcpus=1 nmi_watchdog=0 rcupdate.rcu_cpu_stall_suppress=1 msr.allow_writes=on vdso=0
```
Update grub and reboot:
```bash
sudo update-grub && reboot
```

### 3. Build and Load `/dev/sgx-step`
SGX-Step comes with a loadable kernel module for APIC timer interrupt handling and single-stepping.
```bash
cd kernel/
./install_SGX_driver.sh
make clean load
```
*Note: If you reboot, you need to run `make clean load` again to reload the module.*

### 4. Patch and Install SGX SDK
SGX-Step patches the official Intel SGX SDK to allow custom Asynchronous Exit Pointer (AEP) stubs.
```bash
cd sdk/intel-sdk/
./install_SGX_SDK.sh
echo 'source /opt/intel/sgxsdk/environment' >> ~/.bashrc
source ~/.bashrc
sudo service aesmd status # restart if needed
```

### 5. Verify System Setup
To ensure the kernel is configured correctly, run:
```bash
sudo ./check_sys.sh
```

---

## Running the Attack

### 1. Testing Single Stepping
To verify the timer single-stepping functionality, you can run a benchmark enclave to step through 100 `nop` instructions:
```bash
cd app/bench
NUM=100 make parse
```

### 2. Automated BunnyHop Attack Pipeline
The project includes a unified runner script to orchestrate the BunnyHop attack, collect measurements, and generate visualisations.

**Usage:**
```bash
./run_bunnyhop_attack.sh [OPTIONS]
```

**Options:**
- `-n NUM_RUNS`: Number of attack repetitions (default: 10)
- `-k NUM_KEYS`: Number of random keys per run (default: 25)
- `-t TIMER_INT`: APIC timer interval for SGX-Step (default: 19)
- `-o OUTPUT_DIR`: Root output directory (default: `./results`)
- `-d`: Demo mode (skips actual attack, uses synthetic data for plotting)
- `-h`: Show help and exit

**Example:**
```bash
./run_bunnyhop_attack.sh -n 5 -k 10 -t 32
```
This will produce attack logs and output plots (including bit-error position plots and accuracy distributions) in the `./results` directory.

### 3. Manual Key Recovery Run
To recover the key manually using the standalone executable:
```bash
cd app/sgx-key-recovery/
make clean
make
sudo ./app
```
To test accuracy over 100 runs:
```bash
sudo ./check_accuracy.sh
```
