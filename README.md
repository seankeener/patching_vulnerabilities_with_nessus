## SOP: Scan a Virtual Machine with Nessus Essentials and Remediate TLS Vulnerabilities

### Objective

Use Nessus Essentials to scan a virtual machine for vulnerabilities, identify TLS-related findings, and apply remediation so outdated TLS versions are disabled and the system is re-scanned to confirm the issue is resolved.

### Key Steps

 

**1. Set up Nessus Essentials and target the virtual machine** [0:01](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=1)

![generated-image-at-00:00:01](https://loom.com/i/79ddab374c1940369ae795a136c214bc?workflows_screenshot=true)

- Download and install **Nessus Essentials**.
- Identify the **IP address of the virtual machine** you want to assess.
- Confirm the VM is reachable from the scanning system.
- Use this setup to perform a **basic network scan** against the VM.

 

**2. Review the initial scan results and focus on TLS findings** [0:09](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=9)

![generated-image-at-00:00:09](https://loom.com/i/f212022383c54212ae415b1043556137?workflows_screenshot=true)

- Run the scan and review the discovered vulnerabilities.
- For this procedure, focus specifically on **TLS-related issues**.
- Note that the scan may also return SSL or other findings, but those are outside the scope of this lab.
- Record the affected TLS versions and severity levels for follow-up.

 

**3. Identify the vulnerable TLS versions** [0:33](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=33)

![generated-image-at-00:00:33](https://loom.com/i/2f275b94e7c84d1eb0712418051a5ee7?workflows_screenshot=true)

- Confirm whether **TLS 1.0** and **TLS 1.1** are enabled.
- Document that these versions should be disabled because they are considered insecure.
- Prioritize remediation based on the severity reported by Nessus.

 

**4. Use Nessus details to understand the remediation path** [0:42](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=42)

![generated-image-at-00:00:42](https://loom.com/i/9714533cc9db4fc8aff3442cd9be6932?workflows_screenshot=true)

- Open one of the TLS findings in Nessus.
- Read the **description** to understand why the issue is flagged.
- Review the **solution/remediation guidance** provided by Nessus.
- Use the finding details to determine the required configuration change on the VM.

 

**5. Apply the remediation to disable outdated TLS and enable secure versions** [1:00](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=60)

![generated-image-at-00:01:00](https://loom.com/i/80bcdcbbb9ce4ea7b98086f68bc04b77?workflows_screenshot=true)

- Disable support for **TLS 1.0** and **TLS 1.1** on the virtual machine.
- Enable support for **TLS 1.2** and **TLS 1.3**.
- If needed, use a script or automation tool to make the configuration change consistently.
- Verify the script or change set addresses all outdated TLS versions.

 

**6. Validate severity and prioritize the fix appropriately** [1:13](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=73)

![generated-image-at-00:01:13](https://loom.com/i/d95238474be743e2a3ff6f444651d7ef?workflows_screenshot=true)

- Review the **CVSS score** shown in Nessus.
- Treat medium-to-high severity findings as items that should be addressed promptly.
- Ensure the vulnerability is not left exposed after remediation planning.

 

**7. Create and run a remediation script if manual changes are inefficient** [1:23](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=83)

![generated-image-at-00:01:23](https://loom.com/i/8267934c6f864e10833a0c0284503e89?workflows_screenshot=true)

- Use an AI assistant or scripting method to generate a patching script if appropriate.
- Test the script carefully before applying it broadly.
- Make sure the script disables the vulnerable TLS versions and enables the approved versions.
- Keep a copy of the script for repeatable remediation.

 

**8. Correct any missed TLS settings and rerun the fix** [1:33](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=93)

![generated-image-at-00:01:33](https://loom.com/i/9abae019eed24b3aa0536c4e2a1c1546?workflows_screenshot=true)

- Compare the first scan results with the changes applied.
- If a TLS version was missed, update the script or configuration.
- Re-run the remediation so all insecure TLS versions are disabled.
- Confirm the final configuration matches the intended security standard.

 

**9. Restart the virtual machine and rescan to confirm resolution** [2:11](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=131)

![generated-image-at-00:02:11](https://loom.com/i/3f20b414728142b5889cfeb2b618aa93?workflows_screenshot=true)

- Restart the virtual machine after applying the TLS changes.
- Run Nessus again against the same IP address.
- Confirm the TLS 1.0 and 1.1 findings no longer appear.
- Document the final scan results as evidence that the vulnerability was patched.

 

**10. Close out the lab and document the process** [2:43](https://loom.com/share/594defb3d5a94791b5c23279c31a8a62?t=163)

![generated-image-at-00:02:43](https://loom.com/i/c38f8500047940b994e9f991ccf538f7?workflows_screenshot=true)

- Summarize the scan, remediation, and validation steps completed.
- Note any mistakes or corrections made during the process.
- Record the final outcome: the TLS vulnerability was successfully patched.
- Save the SOP, script, and scan evidence for future reference.

### Cautionary Notes

- **Do not rely on a single scan result**; always rescan after remediation to confirm the issue is resolved.
- **Verify script changes carefully** before running them on production systems.
- **Disable only the intended TLS versions** and avoid breaking required services.
- In a real-world environment, **review all severity findings**, not just TLS-related issues.
- Restarting the VM may cause temporary service interruption; plan accordingly.

### Tips for Efficiency

- Use Nessus findings to guide remediation instead of manually guessing the fix.
- Automate repetitive configuration changes with a script to reduce human error.
- Keep a baseline scan report so you can compare before-and-after results quickly.
- If a change is missed, update the script once and reuse it for future systems.
- Document the exact TLS settings applied so future audits are faster.

### Link to Loom

<https://loom.com/share/594defb3d5a94791b5c23279c31a8a62>
