## 🔧 Prerequisites
For the script to function correctly, the environment must meet the following requirements:
### 1. **Certificate Template Configuration**
* A certificate template (e.g. **“RDP Authentication”**) must exist in Active Directory Certificate Services (AD CS)
* The template must:
  * Include the **Remote Desktop Authentication EKU**
    * `1.3.6.1.4.1.311.54.1.2` (required)
  * Optionally include **Server Authentication**
    * `1.3.6.1.5.5.7.3.1` (recommended)
  * Allow **private key export/use on the machine**
  * Be configured for **computer accounts**
***

### 2. **Auto-Enrolment Enabled**
* Group Policy must enable **certificate auto-enrollment** for computer objects:
  * ✅ Automatically enroll certificates
  * ✅ Renew expired certificates
  * ✅ Update pending certificates
Typical policy location:
Computer Configuration  
 → Policies  
   → Windows Settings  
     → Security Settings  
       → Public Key Policies  
         → Certificate Services Client – Auto-Enrollment
***

### 3. **Automatic Certificate Renewal**
* The certificate template must be configured to:
  * **Auto-renew before expiry**
  * Have a reasonable validity period (e.g., 1 year)
This ensures the script always has a **replacement certificate available**.
***

### 4. **Domain-Joined Machines**
* Target systems must be:
  * Joined to the domain
  * Able to communicate with the **issuing Certificate Authority**
***

### 5. **Permissions**
* The computer account must have:
  * ✅ **Enroll** permission on the certificate template
  * ✅ **Auto-enroll** permission (if using auto-enrollment)
***

### 6. **RDP Enabled**
* Remote Desktop Services must be enabled:
  * The script uses:
    ```
    Win32_TSGeneralSetting (RDP-Tcp)
    ```
  * RDP listener must exist on the system
***

### 7. **Administrative Privileges**
* Script must run **elevated** to:
  * Bind the certificate to RDP
  * Modify registry (auto-enrollment policy)
  * Remove invalid certificates
***

## Script
This script automatically identifies, validates, and binds the most appropriate certificate for Remote Desktop Services (RDP), ensuring secure connectivity while maintaining compliance with certificate requirements—without disrupting the service.
***

## 🔄 Step-by-step process
### 1. **Prepare the environment**
* Ensures certificate **auto-enrollment is enabled** via registry.
* Optionally runs in **TestOnly mode** (no changes, just reporting).
* Checks **administrator privileges** for binding actions.
***

### 2. **Detect current RDP certificate**
* Queries WMI (`Win32_TSGeneralSetting`)
* Retrieves the **currently bound certificate thumbprint**
* Uses this later to avoid unnecessary changes or unsafe removal
***

### 3. **Cleanup invalid certificates (proactive hygiene)**
* Scans:
  * `LocalMachine\My`
  * `LocalMachine\RemoteDesktop`
* Targets only certificates from the **“RDP Authentication” template**
Removes certificates that:
* ❌ Are missing required EKUs (Remote Desktop Authentication)
* ❌ Are non-compliant
Safety controls:
* ✅ **Does NOT remove the currently bound cert** unless a valid replacement exists
* ✅ Leaves valid certs untouched
If cleanup occurs:
* Triggers **auto-enrollment (`certutil -pulse`)** to request replacements
***

### 4. **Discover candidate certificates**
* Searches the certificate store for:
  * Correct **template name**
  * Valid **date range**
  * Presence of **private key**
  * Required **EKUs**
***

### 5. **Validate certificates**
Each candidate is checked for:
* ✅ Remote Desktop Authentication EKU (required)
* ✅ Server Authentication EKU (optional/recommended)
* ✅ Private key presence
* ✅ Validity period (not expired or not yet valid)
Invalid certificates are excluded.
***

### 6. **Select the best certificate**
* Chooses the **newest valid certificate** (latest expiration date)
* This ensures:
  * Longest lifespan
  * Automatic rollover behaviour
***

### 7. **Compare with current binding**
* If the selected certificate is **already bound**:
  * ✅ No action taken
  * ✅ Logs and exits cleanly
***

### 8. **Bind certificate to RDP**
* Updates:
  Win32_TSGeneralSetting.SSLCertificateSHA1Hash
* Applies the new certificate **without restarting RDP services**
***

### 9. **Post-binding behaviour**
* Confirms the binding was successful
* Logs result to the **Windows Event Log**
* Notes:
  > The new certificate will take effect after the next reboot (GPO-safe approach)
***
## 🛡️ Key characteristics

### ✅ Non-disruptive
* No restart of `TermService`
* Safe for production / GPO environments

### ✅ Self-healing
* Removes bad certs
* Triggers re-enrollment automatically

### ✅ Idempotent
* No changes if already in correct state

### ✅ Safe cleanup logic
* Never removes the active cert unless a replacement exists
* 
### ✅ Auditable
* Writes all actions to Event Log


If you want a more “CV / documentation style” version or a super short summary for change records, I can tailor that too 👍
