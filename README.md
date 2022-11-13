# linWinPwn

## Description

linWinPwn is a bash script that automates a number of Active Directory Enumeration and Vulnerability checks. The script uses a number of tools and serves as wrapper of them. Tools include: impacket, bloodhound, crackmapexec, ldapdomaindump, lsassy, smbmap, kerbrute, adidnsdump, certipy, and others. 

linWinPwn is particularly useful when you have access to an Active Directory environment for a limited time only, and you wish to automate the enumeration process and collect evidence efficiently.
In addition, linWinPwn can replace the use of enumeration tools on Windows in the aim of reducing the number of created artifacts (e.g., PowerShell commands, Windows Events, created files on disk), and bypassing certain Anti-Virus or EDRs. This can be achieved by performing remote dynamic port forwarding through the creation of an SSH tunnel from the Windows host (e.g., VDI machine or workstation or laptop) to a remote Linux machine (e.g., Pentest laptop or VPS), and running linWinPwn with proxychains.

On the Windows host, run using PowerShell:
```
ssh kali@<linux_machine> -R 1080 -NCqf
```
On the Linux machine, first update `/etc/proxychains4.conf` to include `socks5 127.0.0.1 1080`, then run:
```
proxychains ./linWinPwn.sh -t <Domain_Controller_IP>
```

## Setup

Git clone the repository and make the script executable

```bash
git clone https://github.com/lefayjey/linWinPwn
cd linWinPwn; chmod +x linWinPwn.sh
```

Install requirements on Kali machines using the `install.sh` script
```bash
chmod +x install.sh
sudo ./install.sh
```

On non-Kali machines, run the `install_nonkali.sh` script instead
```bash
chmod +x install_nonkali.sh
sudo ./install_nonkali.sh
```

If you're having DNS issues or time sync errors, run the `configure.sh` script with `-d` for DNS update and `-n` for NTP sync

*WARNING: The script will update /etc/resolv.conf*

```bash
chmod +x configure.sh
sudo ./configure.sh -t <DC_IP> -d -n
```

## Usage

### Modules
The linWinPwn script contains 6 modules that can be used either separately or simultaneously.

**Default: interactive** - Open interactive menu to run checks separately  

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>]
```

**User modules: ad_enum,kerberos,scan_shares,vuln_checks,mssql_enum**

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M user 
```

**All modules: ad_enum,kerberos,scan_shares,vuln_checks,mssql_enum,pwd_dump**

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M all 
```

**Module ad_enum:** Active Directory Enumeration

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M ad_enum 
```

**Module kerberos:** Kerberos Based Attacks

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M kerberos 
```

**Module scan_shares:** Network Shares Scan

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M scan_shares 
```

**Module vuln_checks:** Vulnerability Checks

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M vuln_checks
```

**Module mssql_enum:** MSSQL Enumeration

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M mssql_enum
```

**Module pwd_dump:** Password Dump

```bash
./linWinPwn.sh -t <Domain_Controller_IP> [-d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -o <output_dir>] -M pwd_dump
```

## Demos
- HackTheBox Forest

Interactive Mode:
[![asciicast](https://asciinema.org/a/499893.svg)](https://asciinema.org/a/499893)

Automated Mode:
[![asciicast](https://asciinema.org/a/464904.svg)](https://asciinema.org/a/464904)

- TryHackme AttacktiveDirectory

[![asciicast](https://asciinema.org/a/464901.svg)](https://asciinema.org/a/464901)

## Use cases

For each of the cases described, the linWinPwn script performs different checks as shown below.

**Case 1: Unauthenticated**
- Module ad_enum
    - rid bruteforce
    - user enumeration
    - ldapdomaindump anonymous enumeration
    - Check if ldap-signing is enforced, check for LDAP Relay
- Module kerberos
    - kerbrute user spray
    - ASREPRoast using collected list of users (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - Blind Kerberoast
    - CVE-2022-33679 exploit
- Module scan_shares
    - SMB shares anonymous enumeration on identified servers
- Module vuln_checks
    - Enumeration for WebDav, dfscoerce, shadowcoerce and Spooler services on identified servers
    - Check for ms17-010, zerologon, petitpotam, nopac, smb-sigining, ntlmv1, runasppl weaknesses

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -M user
```

**Case 2: Standard Account (using password, NTLM hash or Kerberos ticket)**
- DNS extraction using adidnsdump
- Module ad_enum
    - BloodHound data collection
    - ldapdomaindump enumeration
    - crackmapexec user=pass enumeration
    - Delegation information extraction
    - GPP Passwords extraction
    - Extract ADCS information using certipy
    - Check if ldap-signing is enforced, check for LDAP Relay
    - Extraction of MachineAccountQuota of user, Password Policy and users' descriptions containing "pass"
- Module kerberos
    - kerbrute user=pass enumeration
    - ASREPRoasting (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - Kerberoasting (and cracking hashes using john-the-ripper and the rockyou wordlist)
- Module scan_shares
    - SMB shares enumeration on all domain servers
    - KeePass files and processes discovery on all domain servers
- Module vuln_checks
    - Enumeration for WebDav, dfscoerce, shadowcoerce and Spooler services on all domain servers
    - Check for ms17-010, zerologon, petitpotam, nopac, smb-sigining, ntlmv1, runasppl weaknesses
- Module mssql_enum
    - Check mssql privilege escalation paths

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -M user
```

**Case 3: Administrator Account (using password, NTLM hash or Kerberos ticket)**
- All of the "Standard User" checks
- Module pwd_dump
    - LAPS and gMSA dump
    - secretsdump on all domain servers
    - Dump lsass on all domain servers using: procdump, lsassy, nanodump, handlekatz, masky 
    - Extract backup keys using DonPAPI

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -d <AD_domain> -u <AD_user> -p <AD_password_or_hash[LM:NT]_or_kerbticket[./krb5cc_ticket]> -M all
```

## TO DO

- Add more enumeration and exploitation tools...

## Credits

- [S3cur3Th1sSh1t](https://github.com/S3cur3Th1sSh1t) - WinPwn
- [SecureAuth](https://github.com/SecureAuthCorp) - impacket
- [byt3bl33d3r, mpgn and all contributors](https://porchetta.industries/) - crackmapexec
- [Fox-IT](https://github.com/fox-it) - bloodhound-python
- [dirkjanm](https://github.com/dirkjanm/) - ldapdomaindump, adidnsdump
- [zer1t0](https://github.com/zer1t0) - certi.py
- [ly4k](https://github.com/ly4k) - Certipy
- [ShawnDEvans](https://github.com/ShawnDEvans) - smbmap
- [ropnop](https://github.com/ropnop) - windapsearch, kerbrute
- [login-securite](https://github.com/login-securite) - DonPAPI

## Legal Disclamer

Usage of linWinPwn for attacking targets without prior mutual consent is illegal. It's the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program. Only use for educational purposes.
