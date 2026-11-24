# Networkwalks-Internship-Week4
# 🏥 Mediroza General Hospital — Web Application Penetration Test

**NetworkWalks — Week 04 Cybersecurity Project**

![Type](https://img.shields.io/badge/Assessment-Black--box-0f2f4c)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-557C94)
![Focus](https://img.shields.io/badge/Focus-Web%20App%20Security-14b8a6)
![Risk](https://img.shields.io/badge/Overall%20Risk-CRITICAL-b4232a)

A full black-box penetration test of the Mediroza General Hospital web application, carried out as part of the NetworkWalks Week 04 project. Starting from zero credentials, the assessment chained several vulnerabilities to access confidential patient records, crack encrypted lab reports, and recover exposed staff and shareholder data.

> ### ⚠️ Disclaimer
> This project was conducted in a **controlled, authorized training environment** provided by NetworkWalks, using a **simulated hospital** with **fictional data**. These techniques must **never** be used against any system without explicit written permission from its owner. For education only.

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Tools Used](#-tools-used)
- [Milestone 1 — Initial Access](#-milestone-1--initial-access)
- [Milestone 2 — Cracking the Encryption](#-milestone-2--cracking-the-encryption)
- [Milestone 3 — Critical Data Exposure](#-milestone-3--critical-data-exposure)
- [Milestone 4 — Reporting](#-milestone-4--reporting)
- [Findings Summary](#-findings-summary)
- [Recommendations](#-recommendations)
- [Author](#-author)

---

## 🎯 Project Overview

| | |
|---|---|
| **Client** | Mediroza General Hospital |
| **Target** | `https://medirozahospital.com` |
| **Assessment Type** | Black-box Web Application Penetration Test |
| **Duration** | 5 Days |
| **Authorization** | Written permission granted by NetworkWalks |

**Objectives (milestones):**
1. **Initial Access** — attack the website and retrieve 3 confidential patient PDF lab reports.
2. **Data Extraction** — crack the encryption on all 3 retrieved files.
3. **Critical Data Exposure** — find the staff salaries and shareholder details of the hospital.
4. **Reporting** — produce a professional penetration testing report.

---

## 🧰 Tools Used

- **Kali Linux** (VirtualBox VM) — testing platform
- **Nmap** — port and service discovery
- **dirb** — directory and content brute-forcing
- **Firefox + `curl`** — manual application testing and raw HTTP inspection
- **NetworkWalks Hash Calculator** — PDF hash extraction
- **NetworkWalks Password Cracker** — dictionary attack on PDF hashes

---

## 🚪 Milestone 1 — Initial Access

### 1. Reconnaissance
Confirmed the target was live and ran a service scan.

```bash
ping medirozahospital.com
nmap -sV -sC medirozahospital.com
```

The host resolved to shared web hosting (LiteSpeed / OpenResty), confirming the attack surface was the **web application itself** — ports 80 (HTTP) and 443 (HTTPS).

### 2. Content Discovery
Brute-forced directories and files to map hidden areas.

```bash
dirb https://medirozahospital.com /usr/share/wordlists/dirb/common.txt
```

Interesting hits: `robots.txt`, `sitemap.xml`, `/staff/`, and `/old/`.

### 3. Information Disclosure via `robots.txt`
```
User-agent: *
Disallow: /patient/
Disallow: /staff/
Disallow: /old/
```

`robots.txt` revealed the exact paths the admin was trying to hide. Browsing `/patient/` exposed a **directory listing** containing `login.php`, `portal.php`, `download.php`, and a `reports/` folder.

### 4. Authentication Analysis
The patient portal login (`/patient/login.php`) returned **different error messages** for invalid users ("username not found") versus valid users with a wrong password ("incorrect password") — enabling **username enumeration**. This confirmed `admin` as a valid account.

### 5. SQL Injection — Authentication Bypass
Testing the `username` field with a single quote produced a verbose MySQL error, confirming the input was unsanitised:

```
Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that
corresponds to your MySQL server version for the right syntax to use near ''' at line 1
```

<img width="481" height="495" alt="Screenshot 2026-11-24 143713" src="https://github.com/user-attachments/assets/fc0f2ab5-4cc1-40d4-b02b-9c3658349773" />


Using a MySQL comment to bypass the password check logged the session straight in as `admin`:

```
Username:  admin'#
Password:  (anything)
```

<img width="960" height="540" alt="Screenshot 2026-11-24 144951" src="https://github.com/user-attachments/assets/b79649df-88fe-457c-9906-a7b389dd7437" />


### 6. Retrieving the Reports
The portal listed three confidential lab reports, served by ID through `download.php`:

```
https://medirozahospital.com/patient/download.php?id=1
https://medirozahospital.com/patient/download.php?id=2
https://medirozahospital.com/patient/download.php?id=3
```

All three encrypted PDFs were downloaded via **right-click → Save Link As**.

✅ **Milestone 1 complete** — authenticated access gained, 3 patient PDFs retrieved.

---

## 🔓 Milestone 2 — Cracking the Encryption

The three PDFs were password-protected. For each file, a crackable hash was extracted with the **NetworkWalks Hash Calculator**:

<img width="960" height="540" alt="Screenshot 2026-11-24 152218" src="https://github.com/user-attachments/assets/31be895d-cffe-41a0-9a3d-b9f7b620beea" />


The hash was then recovered with the **NetworkWalks Password Cracker** (dictionary attack). All three used trivial, top-of-wordlist passwords:

| File | Patient | Password |
|------|---------|----------|
| `patient_report_1.pdf` | S. Dlamini | `123456` |
| `patient_report_2.pdf` | P. Reddy | `password` |
| `patient_report_3.pdf` | E. Thompson | `!@#$%^&` |

<img width="960" height="540" alt="4 - Copy" src="https://github.com/user-attachments/assets/f34f7853-15c7-4ab2-b946-5c2a24909508" />
<img width="960" height="540" alt="5" src="https://github.com/user-attachments/assets/e13082aa-17e8-42aa-bf77-c7d3055f50d8" />



Opening the PDFs with the recovered passwords revealed the full pathology reports:

<img width="960" height="540" alt="6" src="https://github.com/user-attachments/assets/47e7663e-60c5-4399-bbca-5b38f51a8894" />

<img width="960" height="540" alt="7 - Copy" src="https://github.com/user-attachments/assets/8d460e73-fe97-407e-aaba-e702b519102e" />


✅ **Milestone 2 complete** — all 3 reports decrypted.

---

## 🧨 Milestone 3 — Critical Data Exposure

The `/old/` directory (disclosed earlier in `robots.txt`) had directory listing enabled and contained a complete, **unauthenticated database backup**:

```
https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

This single file exposed the hospital's HR database. The tables were enumerated and extracted with `grep`:

```bash
# list the tables in the backup
grep -i "CREATE TABLE" mediroza_db_backup_2019.sql

# extract all staff records (each row contains a company email)
grep "@medirozahospital" mediroza_db_backup_2019.sql

# extract the shareholder register
grep -i -A 30 "shareholder" mediroza_db_backup_2019.sql
```

### Exposed Staff Salaries

<img width="960" height="540" alt="9" src="https://github.com/user-attachments/assets/6ac90dbf-cc85-44a4-89e7-8cabffca1a97" />

The `staff` table exposed **30 employees'** salaries, job titles, and **South African national ID numbers**. Total monthly payroll ≈ **R2,027,000** (≈ R24.3M/year). *(Full table in the report.)*

### Exposed Shareholder Register

<img width="960" height="540" alt="10" src="https://github.com/user-attachments/assets/935148c0-f540-42dc-8216-a3b69f04d68d" />

| Shareholder | Share % | Shares | Class |
|-------------|---------|--------|-------|
| Dr. Rajesh Naidoo | 18.0 | 180,000 | Ordinary |
| Cedar Health Holdings (Pty) Ltd | 15.0 | 150,000 | Ordinary |
| Dr. Johan van der Merwe | 12.0 | 120,000 | Ordinary |
| Reddy Family Trust | 11.0 | 110,000 | Ordinary |
| Thabo Molefe | 10.0 | 100,000 | Ordinary |
| Sarah Botha | 9.0 | 90,000 | Ordinary |
| Dr. Ahmed Kara | 8.0 | 80,000 | Preferential |
| Naledi Zulu | 7.0 | 70,000 | Ordinary |
| Michael Roberts | 6.0 | 60,000 | Ordinary |
| Dr. Vikram Chetty | 4.0 | 40,000 | Preferential |

✅ **Milestone 3 complete** — salaries and shareholder data recovered.

---

## 📝 Milestone 4 — Reporting

A full professional penetration testing report was produced, covering the Executive Summary, Scope & Methodology, Findings & Proof of Exploitation, Risk Ratings, and Recommendations.



---

## 🔍 Findings Summary

| # | Finding | Severity |
|---|---------|----------|
| 1 | SQL injection — authentication bypass (`/patient/login.php`) | 🔴 **Critical** |
| 2 | Public database backup exposing staff & shareholder data (`/old/`) | 🔴 **Critical** |
| 3 | Weak PDF encryption — trivial passwords | 🟠 **High** |
| 4 | Information disclosure — `robots.txt` & verbose SQL errors | 🟡 **Medium** |
| 5 | Username enumeration via distinct error messages | 🟡 **Medium** |
| 6 | Directory listing enabled | 🟡 **Medium** |
| 7 | Missing security headers & self-signed TLS certificate | 🟢 **Low** |

---

## 🛡️ Recommendations

- **Use parameterised / prepared statements** for all database queries and validate user input (fixes the SQL injection).
- **Remove the database backup** from the web root; store backups in an encrypted, access-controlled location off the web server.
- **Re-issue PDFs with strong, unique passwords** delivered out-of-band; enforce a password policy.
- **Serve patient documents through authenticated, server-side authorization checks** (fix the `download.php?id=` direct object reference).
- **Disable directory indexing**, return generic authentication errors, and suppress verbose database errors in production.
- **Add security headers** (HSTS, CSP, X-Content-Type-Options) and install a valid TLS certificate.

---

## 📂 Repository Structure

```
.
├── README.md
├── Mediroza_Pentest_Report.pdf      # Full professional report
└── images/                          # Evidence screenshots
    ├── 01-sqli-error.png
    ├── 02-portal-access.png
    ├── 03-pdf-hash.png
    ├── 04-crack-report1.png
    ├── 05-crack-report2.png
    ├── 06-report1-dlamini.png
    ├── 07-report2-reddy.png
    ├── 08-staff-salaries.png
    └── 09-shareholders.png
```

---

## 👤 Author

**Takudzwa Chatikobo**
🔗 [LinkedIn](https://www.linkedin.com/in/takudzwa-chatikobo-b1597b436)

*NetworkWalks — Week 04 Cybersecurity Project · Web Application Penetration Testing*
