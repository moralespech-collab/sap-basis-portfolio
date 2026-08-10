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

## 🔐 Phase 2: Environment Variables Configuration (SNC Setup)

For the cryptographic library to function correctly and establish a Secure Network Communication (SNC) tunnel with SAP, you must define the system environment variables. Centralizing all components (executables, libraries, and certificates) in the same directory (`C:\usr\sap\saprouter`) is the standard practice for streamlined management.

Execute the following PowerShell commands to permanently configure `SECUDIR` and `SNC_LIB` at the system level:

```powershell
# Define the SECUDIR variable pointing to the SAP Router working directory
[Environment]::SetEnvironmentVariable("SECUDIR", "C:\usr\sap\saprouter", "Machine")

# Define the SNC_LIB variable pointing directly to the cryptographic DLL
[Environment]::SetEnvironmentVariable("SNC_LIB", "C:\usr\sap\saprouter\sapcrypto.dll", "Machine")

# Refresh the environment variables in the current PowerShell session without restarting the console
$env:SECUDIR = [Environment]::GetEnvironmentVariable("SECUDIR", "Machine")
$env:SNC_LIB = [Environment]::GetEnvironmentVariable("SNC_LIB", "Machine")

# Verify that the variables were successfully set and mapped
Get-ChildItem Env:SECUDIR, Env:SNC_LIB
```

## 🔑 Phase 3: SNC Certificate Request Generation

With the environment variables successfully configured, the crypto library is now able to handle SSL/SNC communications. The next step is to generate a Personal Security Environment (PSE) file (`local.pse`) and create a certificate request.

### 1. Execute sapgenpse to Generate the Request
Run the `sapgenpse` binary to initialize the local cryptographic environment. 

> [!IMPORTANT]
> Replace the Distinguished Name (`-r` parameter) with the official criteria provided for your specific system under the SAP Support Portal registration.

```powershell
# Navigate to the workspace (if not already there)
cd "C:\usr/sap\saprouter"

# Generate the local.pse file and the certificate request text file (certreq)
# Format: CN=<host_name>, OU=<customer_number>, OU=SAProuter, O=SAP, C=DE
.\sapgenpse.exe get_pse -v -r certreq -p local.pse "CN=your_server_name, OU=0001234567, OU=SAProuter, O=SAP, C=DE"
```

### 2. Enter and Confirm the PIN
During execution, the console will prompt you to enter a PIN twice. 
*   Choose a secure PIN code.
*   **Document or securely store this PIN**, as it will be required to grant the SAP Router service permanent access to the cryptographic keys.

This command will output two crucial elements in your `C:\usr\sap\saprouter` directory:
1.  `local.pse`: The encrypted file hosting your private key container.
2.  `certreq`: A plain text file containing the certificate signing request (CSR).



