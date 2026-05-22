.SYNOPSIS
 Binds the newest valid "RDP Authentication" certificate to RDP (no service restart) and logs to Event Log.
.DESCRIPTION
 - Searches LocalMachine\My for certificates issued from the "RDP Authentication" template.
 - Validates EKUs (Remote Desktop Authentication + Server Authentication), private key, and time validity.
 - Selects the newest valid certificate and binds it to RDP (Win32_TSGeneralSetting.SSLCertificateSHA1Hash).
 - Skips restarting TermService (safer for GPO). Notes that the new cert will take effect after next reboot.
 - Logs outcomes to the Application event log (creates source if missing).

 Added:
 - Cleanup step: removes ONLY "RDP Authentication" template certs missing required EKUs (Remote Desktop Authentication),
   without touching the currently bound cert unless another valid replacement exists. Triggers auto-enrollment if needed.
.PARAMETER TestOnly
 Perform all checks and display details, but do not bind or change system state.
.EXAMPLE
 .\Bind-RdpCert.ps1
.EXAMPLE
 .\Bind-RdpCert.ps1 -TestOnly
