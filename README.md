# BD Alaris 8015 Firmware Security Analysis

**Firmware Security Analysis of the BD Alaris 8015 PC Unit: Unencrypted Wi-Fi Credential Storage and Compound Vulnerabilities (CVE-2016-9355)**

[![CVE](https://img.shields.io/badge/CVE-2016--9355-red)](https://nvd.nist.gov/vuln/detail/CVE-2016-9355)
[![CVSS](https://img.shields.io/badge/CVSS-4.6%20(Medium)-orange)](https://nvd.nist.gov/vuln/detail/CVE-2016-9355)
[![Course](https://img.shields.io/badge/Course-Medical%20Device%20Cybersecurity-blue)]()

## Overview

This project presents a firmware security analysis of the BD Alaris 8015 Point-of-Care (PC) infusion pump, focusing on **CVE-2016-9355** — a vulnerability that allows extraction of unencrypted Wi-Fi credentials from the device's NAND flash memory.

**Course:** CY 7790 - Medical Device Cybersecurity  
**Professor:** Kevin Fu (Former Acting Director, Medical Device Cybersecurity at FDA)  
**Institution:** Northeastern University  
**Team:** Kalyan, Varun, Abisath, Arafat

## Key Findings

### Primary Vulnerability (CVE-2016-9355)
The device stores wireless network credentials in **plaintext XML files** on NAND flash, including:
- WPA2-PSK passphrases
- 802.1X enterprise domain credentials (PEAP-MSCHAPv2)
- EAP-TLS private key passphrases
- AES session keys

### Compound Vulnerabilities Discovered
| Vulnerability | Severity | Description |
|---------------|----------|-------------|
| HTTP Firmware Updates | High | Update channel uses unencrypted HTTP with MD5-only integrity |
| Disabled CRC Verification | High | OSE.ELF (main app) boots without integrity check (CRC=0xFFFFFFFF) |
| ProFTPD 1.3.3g (3 CVEs) | Medium | CVE-2011-4130, CVE-2010-4221, CVE-2010-3867 |
| Unauthenticated PPP | Medium | PPP configured without PAP/CHAP authentication |
| ValidateServerCert=false | Medium | PEAP profiles vulnerable to evil-twin RADIUS attacks |
| DNS Hijack Vector | Medium | Unqualified hostname 'natasha' enables firmware redirect |

## Methodology

### Analysis Approach
- **Track:** Hardware Analysis (Static Firmware Analysis)
- **Target:** Firmware version 9.33 (September 2017)
- **Environment:** Kali Linux

### Tools Used
- binwalk - Firmware extraction
- UBI Reader - UBIFS filesystem reconstruction
- strings / xxd - Binary analysis
- Python 3 - Custom UBIFS traversal scripts
- Entropy analysis tools

### Analysis Steps
1. Firmware package extraction and hash verification
2. UBIFS filesystem reconstruction from NAND image
3. Credential path identification (/etc/summit/)
4. XPath extraction from OSE.ELF binary (WlanProfileParser class)
5. Entropy analysis confirming absence of encryption
6. Compound vulnerability identification

## Technical Details

### Device Specifications
| Component | Details |
|-----------|---------|
| Processor | Atmel AT91SAM9G20 (ARM926EJ-S @ 400MHz) |
| Memory | 32 MB SDRAM, 64 MB NAND Flash |
| Wi-Fi Module | Laird WB40N (Broadcom BCM4329) |
| RTOS | ENEA OSE 4.5.2/4.6.1 |
| Main Binary | OSE.ELF (7.86 MB ARM ELF) |

### Credential Storage Location
/etc/summit/[ProfileName].xml

### Vulnerable XPaths (from OSE.ELF)
/PSK/Passphrase/text()
/Password/text()
/PEAP/Identity/text()
/EAPTLS/PrivateKeyPassphrase/text()
/EAPTLS/UserCertPassphrase/text()
/AES/Key/text()

## Threat Model (STRIDE)

| Category | Threat | Impact |
|----------|--------|--------|
| Spoofing | Stolen Wi-Fi PSK used to join hospital network | Network compromise |
| Tampering | HTTP firmware channel allows malicious updates | Device compromise |
| Repudiation | Disabled CRC leaves no audit trail | Undetectable tampering |
| Info Disclosure | CVE-2016-9355 plaintext credential extraction | Credential theft |
| Denial of Service | ProFTPD stack overflow (CVE-2010-4221) | Service crash |
| Elevation of Privilege | Domain credentials enable lateral movement | Hospital-wide breach |

## Mitigations

| Threat | Mitigation | Status |
|--------|------------|--------|
| Plaintext credentials | AES encryption with hardware-bound key (HKDF from GUID) | Fixed in v9.5+ |
| HTTP firmware updates | HTTPS + RSA/ECDSA code signing | Not addressed |
| MD5 integrity | SHA-256 + signed manifest | Not addressed |
| Disabled OSE.ELF CRC | SHA-256 verification at boot | Not addressed |
| ProFTPD vulnerabilities | Update to 1.3.8 or disable service | Not addressed |

## References

- ICS-CERT Advisory ICSMA-16-333-01: https://www.cisa.gov/news-events/ics-medical-advisories/icsma-20-317-01
- CVE-2016-9355 (NVD): https://nvd.nist.gov/vuln/detail/CVE-2016-9355
- MITRE Playbook for Threat Modeling Medical Devices

## Disclaimer

This research was conducted for academic purposes under the Medical Device Cybersecurity course at Northeastern University. All analysis was performed on firmware images only — no physical devices were accessed. Credential samples shown are fictional and for demonstration purposes only.

---

**Author:** Mohammed Arafat Nadaf  
**GitHub:** github.com/nadafarafat  
**LinkedIn:** linkedin.com/in/arafatnadaf
