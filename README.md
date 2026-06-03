# Tomghost - TryHackMe Write-up

## Enumeration

The first step was performing reconnaissance against the target using Nmap in order to identify exposed services and potential attack vectors.
<br><br>
<img width="942" height="482" alt="image" src="https://github.com/user-attachments/assets/3ab912d0-562e-4dd8-84e7-2976ca6e7ded" />
<br><br>

The scan revealed several open ports, including SSH on port 22, an HTTP service running Apache Tomcat on port 8080, and the AJP13 service on port 8009. The presence of AJP immediately stood out as a potential attack vector due to known vulnerabilities affecting Apache Tomcat.

## Web Enumeration

With the initial scan completed, the next step was to investigate the web application exposed on port 8080.
<br><br>
<img width="1299" height="820" alt="image" src="https://github.com/user-attachments/assets/c1de1ecd-7e4f-4001-b1b0-2c2677f58692" />
<br><br>

Accessing the service revealed the default Apache Tomcat page, confirming the version in use and providing additional information about the environment.

Attempts were made to access the Manager App, Host Manager, and Server Status interfaces.
<br><br>
<img width="1913" height="482" alt="image" src="https://github.com/user-attachments/assets/b55520ad-7262-48f7-9dd6-595d8982c4e8" />
<br><br>
However, access to these administrative endpoints was restricted, requiring further investigation into the Tomcat installation.

## Vulnerability Research

Given the exposed AJP service and the identified Tomcat version, research was conducted to identify publicly known vulnerabilities affecting the target.

The investigation confirmed the presence of a vulnerability associated with the AJP protocol, commonly known as Ghostcat (CVE-2020-1938), which allows arbitrary file reads under specific conditions.

The vulnerability details confirmed that the exposed AJP connector could potentially be leveraged to access sensitive files from the Tomcat installation.
<br><br>
<img width="793" height="289" alt="image" src="https://github.com/user-attachments/assets/36ed4c63-ae26-4aac-85fc-1a86028195c0" />
<br><br>
## Exploiting Ghostcat

After confirming the vulnerability, Metasploit was launched to simplify the exploitation process.
<br><br>
<img width="783" height="422" alt="image" src="https://github.com/user-attachments/assets/174a5aec-0044-4dc6-8fbd-cbff25c5444f" />
<br><br>

The Ghostcat module was selected and configured against the target.
<br><br>
<img width="931" height="38" alt="image" src="https://github.com/user-attachments/assets/8b33b173-ad2d-4076-a518-88467cd32909" />
<br><br>

Successful exploitation allowed access to internal application files, revealing credentials embedded within the application's configuration.
<br><br>
<img width="658" height="507" alt="image" src="https://github.com/user-attachments/assets/b7db2ac0-36d6-488b-b126-076400025c09" />
<br><br>

## Initial Access

During the enumeration phase, an SSH service had already been identified on port 22. The recovered credentials were therefore tested against SSH authentication.
<br><br>
<img width="772" height="386" alt="image" src="https://github.com/user-attachments/assets/52b3218e-ac90-476f-84c8-9416f833f60a" />
<br><br>
The credentials were valid, providing initial access to the target system as the user `skyfuck`.

## User Enumeration

After obtaining shell access, the home directories were explored to identify additional users and sensitive files.
<br><br>
<img width="519" height="158" alt="image" src="https://github.com/user-attachments/assets/b495cceb-f009-4f99-bdc4-ecf4f0af0432" />
<br><br>

During this process, the user flag was located within Merlin's home directory.

## Privilege Escalation Enumeration

Several privilege escalation checks were performed, including SUID enumeration and sudo permission checks. Initially, no obvious escalation path was identified.
<br><br>
<img width="589" height="273" alt="image" src="https://github.com/user-attachments/assets/36205c6a-a0a9-4434-b2c7-83aed0c58f7c" />
<br><br>

As a result, attention shifted toward files located in Skyfuck's home directory.

## PGP Key Discovery

Two files immediately attracted attention: `credential.pgp` and `tryhackme.asc`.
<br><br>
<img width="271" height="39" alt="image" src="https://github.com/user-attachments/assets/9aa11ba1-a88f-41fd-8ad7-96be54dacfbe" />
<br><br>

Inspecting the `tryhackme.asc` file revealed a PGP private key.
<br><br>
<img width="587" height="474" alt="image" src="https://github.com/user-attachments/assets/1b553f5d-d697-485a-ba4d-f46740a3c611" />
<br><br>

The key was copied to the attack machine for further analysis.

## Cracking the PGP Passphrase

The PGP key hash was extracted in preparation for password cracking.
<br><br>
<img width="365" height="85" alt="image" src="https://github.com/user-attachments/assets/6befe03f-0688-4e6a-b8d4-308f1e36b223" />
<br><br>

The extracted hash was successfully cracked using John the Ripper, revealing the passphrase protecting the private key.
<br><br>
<img width="925" height="233" alt="image" src="https://github.com/user-attachments/assets/8a76ada6-c890-4054-aa4d-95ad25cc4df9" />
<br><br>

## Decrypting Stored Credentials

Returning to the target machine, the private key was imported into GPG.
<br><br>
<img width="654" height="166" alt="image" src="https://github.com/user-attachments/assets/d66bce59-23b6-4497-9279-03022e2a5f22" />
<br><br>

Verification confirmed that the import was successful and that the key belonged to the TryHackMe user.
<br><br>
<img width="463" height="108" alt="image" src="https://github.com/user-attachments/assets/425f5e68-3382-4480-8136-ef2816799176" />
<br><br>

Using the recovered passphrase, the encrypted file `credential.pgp` was decrypted.
<br><br>
<img width="714" height="195" alt="image" src="https://github.com/user-attachments/assets/868ad0d1-a26a-40c6-93a9-f25fddadf363" />
<br><br>

The decrypted content revealed credentials belonging to the user `merlin`.

## Pivoting to Merlin

With valid credentials recovered, it was possible to switch from the `skyfuck` account to the `merlin` account.
<br><br>
<img width="563" height="60" alt="image" src="https://github.com/user-attachments/assets/11734069-f262-413e-89e9-627e987143fe" />
<br><br>

This provided access to a new user context and opened additional privilege escalation opportunities.

## Sudo Misconfiguration

Enumerating sudo permissions revealed a critical misconfiguration.
<br><br>
<img width="728" height="114" alt="image" src="https://github.com/user-attachments/assets/7318263c-7d5e-4c8b-b5b0-25ea6b06de54" />
<br><br>

The user `merlin` was allowed to execute the `zip` binary as root without requiring a password.

## Privilege Escalation via GTFOBins

A search through GTFOBins revealed that the `zip` binary could be abused to obtain a privileged shell.
<br><br>
<img width="879" height="271" alt="image" src="https://github.com/user-attachments/assets/c6cbd872-9e27-484d-88d3-f31556d6342d" />
<br><br>

Executing the provided technique successfully spawned a root shell.
<br><br>
<img width="643" height="125" alt="image" src="https://github.com/user-attachments/assets/83091ff9-43d3-46a4-8787-844b20151557" />
<br><br>

## Root Access

With root privileges obtained, navigation to the root user's directory was possible.
<br><br>
<img width="399" height="96" alt="image" src="https://github.com/user-attachments/assets/a242d9be-ca02-40a2-a0db-f0124e33e5f1" />
<br><br>

The root flag was successfully retrieved, completing the machine.

## Key Learning Points

* Nmap Enumeration
* Apache Tomcat Enumeration
* AJP Protocol Enumeration
* Ghostcat (CVE-2020-1938)
* Metasploit Exploitation
* File Disclosure Vulnerability
* Credential Harvesting
* SSH Access
* Linux Enumeration
* PGP Key Analysis
* John the Ripper
* GPG Decryption
* User Pivoting
* Sudo Enumeration
* GTFOBins
* Privilege Escalation
* Root Compromise

## Security Recommendations

To mitigate the issues identified during this assessment, the following security measures should be implemented:

* Keep Apache Tomcat updated to a version not affected by CVE-2020-1938 (Ghostcat).
* Disable the AJP connector if it is not required by the application.
* Restrict access to the AJP service using firewall rules and network segmentation.
* Avoid storing plaintext credentials within application configuration files.
* Enforce strong password policies and unique credentials for all users.
* Protect sensitive information using secure secret management solutions.
* Regularly audit user directories for sensitive files and credentials.
* Use strong passphrases to protect PGP private keys.
* Restrict SSH access to authorized users and implement key-based authentication whenever possible.
* Periodically review sudo permissions and apply the principle of least privilege.
* Avoid granting root execution privileges to binaries that can be abused for shell execution.
* Conduct regular vulnerability assessments and security audits to identify misconfigurations before they can be exploited.

## Conclusion

Tomghost was an excellent machine for practicing web application enumeration, vulnerability research, credential harvesting, and Linux privilege escalation.

The attack chain started with the identification of an exposed Apache Tomcat instance and an accessible AJP connector. Further investigation led to the discovery and exploitation of the Ghostcat vulnerability (CVE-2020-1938), allowing sensitive information disclosure and the recovery of valid credentials.

After obtaining initial access via SSH, additional enumeration revealed encrypted credentials protected by a PGP private key. By extracting and cracking the key's passphrase, it was possible to decrypt the stored credentials and pivot to the user `merlin`.

The final stage involved identifying a sudo misconfiguration that allowed the execution of the `zip` binary as root. By leveraging a GTFOBins technique, full administrative access was obtained and the machine was successfully compromised.

Overall, this room provided valuable hands-on experience with Ghostcat exploitation, credential recovery, PGP key analysis, Linux enumeration, privilege escalation techniques, and the importance of secure service configuration and access control.
