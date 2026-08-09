## 🚀 Phase 1: Downloading Binaries and Environment Preparation

To begin the installation, you must download the official deployment tools from the **SAP Software Download Center** using an authorized S-User with download privileges.

### 1. Required Technical Components
Under the *Support Packages & Patches* path, download the latest versions compatible with your specific Operating System architecture:
*   **SAPROUTER:** The core executable package for the service (`saprouter_*_*.sar`).
*   **SAPCAR:** SAP's official decompression utility (`SAPCAR_*_*.exe` or Linux binary) required to extract the packages.
*   **SAPCRYPTOLIB:** The cryptographic library mandatory for enabling Secure Network Communications (SNC) with SAP (`SAPCRYPTOLIB_*_*.sar`).
*   **C-Runtimes (Windows only):** The Microsoft Visual C++ Redistributable (VCredist) package must be installed on the host to execute SAP binaries properly. For deployment guidelines and version compatibility, refer to [SAP Note 1553465](https://me.sap.com/notes/1553465/E) and the SAP Community article [C-runtimes needed to run SAP executables](https://community.sap.com/t5/technology-blog-posts-by-sap/c-runtimes-needed-to-run-sap-executables/ba-p/13314763).

  
### 2. Server Directory Setup
Create a dedicated workspace directory on the host server to isolate the executables and enforce strict security permissions at the OS level.

```bash
# Create the dedicated SAP Router directory
mkdir -p /usr/sap/saprouter

# Move the downloaded .SAR packages to the target path
# (Replace /tmp with your actual staging file transfer directory)
mv /tmp/saprouter*.sar /usr/sap/saprouter/
mv /tmp/sapcrypto*.sar /usr/sap/saprouter/
```

### 3. Package Extraction
Execute `SAPCAR` to uncar the core routing binaries and the cryptographic package dependencies into the working directory:

```bash
cd /usr/sap/saprouter

# Extract the SAP Router utilities and Crypto Library
./SAPCAR -xvf saprouter_*.sar
./SAPCAR -xvf sapcrypto_*.sar
```

