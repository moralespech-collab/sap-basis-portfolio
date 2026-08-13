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

### 1. Go to the [SAProuter application](https://me.sap.com/app/saproutercertificate) and from the list of SAProuters registered to your installation, choose the relevant SAProuter.

### 2. Enter and Confirm the PIN
During execution, the console will prompt you to enter a PIN twice. 
*   Choose a secure PIN code.
*   **Document or securely store this PIN**, as it will be required to grant the SAP Router service permanent access to the cryptographic keys.

This command will output two crucial elements in your `C:\usr\sap\saprouter` directory:
1.  `local.pse`: The encrypted file hosting your private key container.
2.  `certreq`: A plain text file containing the certificate signing request (CSR).

## 🔐 Phase 4: Certificate Import and Credentials Setup

Once you receive the signed certificate text from the SAP Support Portal, you must import it into your local environment and grant permanent execution permissions to the system.

### 1. Import the Signed Certificate
Create a file named `srcert` in `C:\usr\sap\saprouter`, paste the certificate text inside, and run:
```powershell
.\sapgenpse.exe import_own_cert -c srcert -p local.pse
```

### 2. Generate the Credential File (cred_v2)
This step is mandatory so the Windows Service can open the `local.pse` file automatically without asking for a PIN code during server reboots.
```powershell
# Create credentials for the local Administrator or SAP service user
.\sapgenpse.exe seclogin -p local.pse -O Administrator
```

---

## 🌐 Phase 5: Routing Table Policy Configuration (saprouttab)

You must define the security policies to restrict or allow traffic. Create a plain text file named `saprouttab` (without any extension) inside `C:\usr\sap\saprouter`.

### Example Production Rules:
```properties
# ------------------------------------------------------------------
# SAPROUTTAB - Secure Routing Policy Table
# ------------------------------------------------------------------
# Syntax: P/D/C <Source-Host> <Dest-Host> <Dest-Service> <Password>

# 1. Allow SAP Support Portal to access your local Solution Manager (SolMan)
P  194.39.131.34      10.0.1.50      3201

# 2. Allow your local SolMan to send data out to SAP Support Services
P  10.0.1.50          194.39.131.34  3299

# DENY ALL OTHER TRAFFIC (Implicit at the end, but good practice to state)
D  *                  *              *
```

---

## 🚀 Phase 6: Windows Service Registration

To ensure high availability, the SAP Router must run as an automatic Windows Background Service instead of a manual command-line prompt.

Execute the following command using `sc.exe` in an elevated console:

```powershell
sc.exe create SAPRouter binPath= "C:\usr\sap\saprouter\saprouter.exe -r -R C:\usr\sap\saprouter\saprouttab -W 60000" start= auto obj= "LocalSystem" DisplayName= "SAP Router Service"
```

---

## 🔍 Phase 7: Verification and Handover Checklist

Before delivering the project to the client, perform the following validation tests to ensure the tunnel is fully operational.

### 1. Local Service Check
Verify that the Windows service is running and listening on port `3299`:
```powershell
# Check service status
Get-Service SAPRouter

# Verify active port binding
Get-NetTCPConnection -LocalPort 3299
```

### 2. Handover Deliverables for the Client
Provide the client's network and Basis teams with the following technical data to mark the project as completed:
*   **External SAP Router String:** `/H/<Public_IP_or_FQDN>/S/3299` (This is what SAP or external vendors will use to reach them).
*   **Firewall rule validation:** Confirm that external port `3299` (TCP) is bi-directionally open between the SAP Router host and SAP's public routers.
*   **Connection Test via SM59:** Create or test an RFC destination of type `G` or `H` in the customer's SAP system to confirm data reaches `://sap.com` successfully.


