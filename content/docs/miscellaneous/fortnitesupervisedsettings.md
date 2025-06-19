---
weight: 999
title: "Fortnite - Unable to Download Supervised Settings"
description: "How to fix \"unable to download supervised settings\" issue."
icon: "article"
date: "2024-05-30T14:49:49-05:00"
lastmod: "2024-05-30T14:49:49-05:00"
draft: false
toc: true
---

## 🔧 Fixing SSL Certificate Verification Failure for Epic Online Services (EOS)

### 📄 Issue Overview

While launching a game using Epic Online Services, I encountered repeated login failures in the logs, with errors like:

```
[OnlineAccount.Logout:0][operation_succeeded] ...
Result=[ELoginResult::SupervisedSettingsDownloadFailed (45)], 
FailedStep=[DownloadingSupervisedSettings], 
DisplayError=[Failed to download Supervised Settings.], 
DetailedError=[[1.1.1-3.4.1] no_connection (EOS_NoConnection)]
```

Further down, the **underlying issue became clear**:

```
libcurl error: 60 (SSL peer certificate or SSH remote key was not OK)
SSL certificate problem: unable to get local issuer certificate
```

---

## 🔍 Root Cause Summary

The game client could not establish a secure connection to the Epic backend API:

```
https://api.kws.ol.epicgames.com
```

This was caused by a **missing root CA certificate** in the local system trust store. Specifically, the server’s certificate chain was signed by:

* `Amazon RSA 2048 M02` → `Amazon Root CA 1` → **`Starfield Services Root Certificate Authority - G2`**

The final root CA (`Starfield Services Root CA - G2`) was **not trusted locally**, triggering SSL verification failures in `libcurl` (used internally by EOS SDK).

---

## ✅ Solution Steps

### Step 1: Inspect the Certificate Chain

Run the following command to view the full chain:

```bash
openssl.exe s_client -showcerts -connect api.kws.ol.epicgames.com:443
```

The output showed the chain terminating at:

```
CN=Starfield Services Root Certificate Authority - G2
```

---

### Step 2: Download the Missing Root CA

```bash
curl -LO https://www.amazontrust.com/repository/SFSRootCAG2.pem
```

---

### Step 3: Convert PEM to CRT Format (Optional)

If needed, convert the `.pem` file to a binary `.crt` format:

```bash
openssl x509 -outform der -in SFSRootCAG2.pem -out StarfieldRootG2.crt
```

Alternatively, for Windows MMC installation, **renaming the `.pem` to `.crt` or `.cer` is usually enough**.

---

### Step 4: Install the Certificate in Windows

#### 1. Launch Certificate Manager

* Press `Win + R`, type `mmc`, and press Enter.
* File → **Add/Remove Snap-in**
* Select **Certificates** → Click *Add* → Choose **Computer account** → *Local computer*

#### 2. Import Certificate

* In MMC, navigate to:

  ```
  Certificates (Local Computer) → Trusted Root Certification Authorities → Certificates
  ```
* Right-click `Certificates` → **All Tasks → Import**
* Follow the wizard:

  * Choose your `.crt` or `.cer` file (converted or renamed)
  * Import it to the **Trusted Root Certification Authorities** store

---

### Step 5: Test

After installing the certificate:

* SSL connections to `api.kws.ol.epicgames.com` should now succeed
* Login and supervised settings download should complete normally

---

# Result

After installing the missing root CA, the game successfully connected to the Epic backend services. The `SupervisedSettingsDownloadFailed` error was resolved, and login proceeded without SSL verification errors.
