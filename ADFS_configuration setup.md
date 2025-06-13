# ADFS cofiguration


## 🛠️ ✅ **Full Lab Script: Domain Controller + ADFS Setup for goat.com**

> 👨‍💻 Domain: `goat.com`
> 🖥 DC Hostname: `dc-1`
> 🖥 ADFS Hostname: `adfs-1`
> 🗂 OS: Windows Server 2019/2022 (any supported)

---

## 🟩 **1. Domain Controller Setup (dc-1)**

### 📦 Install AD DS Role

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

---

### 🏗️ Promote to Domain Controller

```powershell
Install-ADDSForest `
 -DomainName "goat.com" `
 -DomainNetbiosName "GOAT" `
 -SafeModeAdministratorPassword (ConvertTo-SecureString "Goat@123" -AsPlainText -Force)
```

> 🔄 Server will reboot and become `goat.com` DC.

---

## 🟨 **2. ADFS Server Setup (adfs-1)**

---

### 📛 Rename Computer & Join Domain

```powershell
Rename-Computer -NewName "adfs-1" -Restart
```

Once rebooted, then join domain:

```powershell
Add-Computer -DomainName goat.com -Credential goat\Administrator -Restart
```

---

### 🔐 Install ADFS Role

```powershell
Install-WindowsFeature ADFS-Federation -IncludeManagementTools
```

---

## 🟧 **3. SSL Certificate Configuration (Self-Signed for Lab)**

```powershell
New-SelfSignedCertificate `
 -DnsName "fs.goat.com" `
 -CertStoreLocation "cert:\LocalMachine\My"
```

### 🧭 Copy to Trusted Root (Manual Step)

1. Run `certlm.msc`
2. Go to **Personal > Certificates**
3. Right-click on `fs.goat.com` → **Copy**
4. Paste into **Trusted Root Certification Authorities**
5. 📋 Note the **Thumbprint**

---

## 🟥 **4. Install ADFS Farm**

```powershell
$adminConfig = @{
    FederationServiceDisplayName = "Goat Lab ADFS"
    FederationServiceName = "fs.goat.com"
    GroupServiceAccountIdentifier = ""
}

Install-AdfsFarm `
 -CertificateThumbprint "YOUR_THUMBPRINT_HERE" `
 -ServiceAccountCredential (Get-Credential) `
 -AdminConfiguration $adminConfig
```

📝 **Use a domain user with local admin rights.**

---

## 🟦 **5. Configure DNS (on client machine)**

Edit the client’s `hosts` file to resolve `fs.goat.com`:

```plaintext
<PRIVATE_IP_OF_adfs-1>    fs.goat.com
```

Use Notepad as Administrator to edit:

```powershell
notepad C:\Windows\System32\drivers\etc\hosts
```

---

## 🟫 **6. Verify Federation Service Metadata**

In browser (client or ADFS):

```url
https://fs.goat.com/federationmetadata/2007-06/federationmetadata.xml
```

✅ If you see XML, your ADFS is live.

---

## 🟪 **7. Enable IdP Initiated Sign-On**

```powershell
Set-AdfsProperties -EnableIdpInitiatedSignonPage $true
```

Now test at:

```url
https://fs.goat.com/adfs/ls/IdpInitiatedSignon.aspx
```

Sign in with a domain user like `aravindh@goat.com`

---

## 🟫 **8. Create Relying Party Trust (Dummy App)**

1. Open **ADFS Management**
2. Relying Party Trusts → Add Relying Party Trust
3. Choose: **Enter data manually**
4. Name: `Dummy App`
5. Identifiers: `https://dummyapp.goat.com/sso`
6. Endpoints: `https://dummyapp.goat.com/sso` (Optional for lab)

---

## 🟪 **9. Add Claim Rule**

In ADFS Management:

* Click your Relying Party Trust → Edit Claim Issuance Policy → Add Rule
* **Template**: Send LDAP Attributes as Claims
* **Name**: Send UPN
* **Attribute Store**: Active Directory
* **Mapping**:

  * LDAP Attribute: `User-Principal-Name`
  * Outgoing Claim Type: `Name ID`

---

## ✅ 🎯 You’re Done!

Your ADFS lab with `goat.com` domain is fully operational. You can:

* Add real apps (like AWS, Salesforce, etc.)
* Link with Entra ID for federation
* Or test login flows using the built-in IdP page.


