# 📘 Day 1 - IBM MQ Practice

## 🛠️ IBM MQ 9.4.0.10 Installation on Linux (Trial Version)

This guide walks through the steps required to install the IBM MQ Trial version (9.4.0.10) on a Linux system.

### 📦 Prerequisites

- A 64-bit Linux system (RHEL/CentOS or similar)
- `sudo` privileges
- `yum` package manager (for RPM-based systems)

---

### 🚀 Installation Steps


# 1. Move the downloaded MQ tarball to /opt/softwares
```bash
mv 9.4.0.10-IBM-MQTRIAL-LinuxX64.tar.gz /opt/softwares/
```
# 2. Navigate to /opt/softwares and extract the archive
```bash
cd /opt/softwares/
tar -xzvf 9.4.0.10-IBM-MQTRIAL-LinuxX64.tar.gz
```
# 3. List extracted files and verify current directory
```bash
ls -ltr
pwd
```
# 4. Navigate into the MQ Server directory
```bash
cd MQServer/
ls -ltr
```
# 5. Accept IBM MQ License
```bash
sudo ./mqlicense.sh
```
# 6. Create an MQ RPM package build directory
```bash
sudo ./crtmqpkg mq94 /opt/mqm
```
# 7. Install required packages to build RPMs
```bash
sudo yum install -y rpmbuild
sudo yum install -y rpm-build
```
# 8. Re-run the package creation script (after installing build tools)
```bash
sudo ./crtmqpkg mq94 /opt/mqm
```
# 9. Navigate to the generated RPM packages directory
```bash
cd /var/tmp/mq_rpms/mq94/x86_64
```
# 10. Install all MQ RPM packages
```bash
sudo rpm -ivh *
```
# 11. Verify IBM MQ version
✅ Verification Output Example
```bash
dspmqver

Name:        IBM MQ
Version:     9.4.0.10
Level:       p940-001-24113
BuildType:   IKAP - (Production)
```
📌 Notes
mqlicense.sh: Required to accept IBM's license before installation can proceed.

crtmqpkg: Used to build RPM packages from the extracted files.

The rpm -ivh * command installs all necessary MQ components.

You can change the install directory from /opt/mqm to a custom location if required.

📚 References
[IBM MQ Documentation](https://www.ibm.com/docs/en/ibm-mq/9.2.x?topic=mq-introduction).

[MQ Trial Download](https://www.ibm.com/docs/en/ibm-mq/9.4.x?topic=information-mq-downloads)

