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

> [!IMPORTANT]
> All subsequent commands must be executed within an elevated PowerShell instance ("Run as Administrator").

```powershell
# Create the dedicated SAP Router directory structure
New-Item -ItemType Directory -Force -Path "C:\usr\sap\saprouter"

# Move the downloaded .SAR packages to the target installation path
# (Replace C:\tmp with your actual download or staging directory)
Move-Item -Path "C:\tmp\saprouter*.sar" -Destination "C:\usr\sap\saprouter\"
Move-Item -Path "C:\tmp\sapcrypto*.sar" -Destination "C:\usr\sap\saprouter\"
```

### 3. Package Extraction
Navigate to the installation directory and utilize the `SAPCAR.exe` utility to unpack the core routing binaries and cryptographic dependencies.

> [!NOTE]
> It is usually more convenient to rename the downloaded SAPCAR binary to simply `SAPCAR.exe` for easier command execution.

```powershell
# Navigate to the SAP Router working directory
cd "C:\usr/sap\saprouter"

# Unblock the SAPCAR executable if downloaded from the internet (prevents OS security blocks)
Unblock-File -Path ".\SAPCAR.exe"

# Extract the SAP Router utilities and Crypto Library
# (Replace the filename with the exact version number you downloaded)
.\SAPCAR.exe -xvf saprouter_XXXX-XXXX.sar
.\SAPCAR.exe -xvf sapcrypto_XXXX-XXXX.sar
```



