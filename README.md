# Enterprise-LDAP-Hardening
This repository provides detailed, practical instructions for enabling LDAP Channel Binding and LDAP Signing, critical security features that protect against man-in-the-middle attacks and ensure message integrity in Active Directory communications.


# LDAP Channel Binding and LDAP Signing Guide
> A comprehensive guide for implementing secure LDAP communications in Windows environments

## 📚 Overview

LDAP (Lightweight Directory Access Protocol) Channel Binding and Signing are essential security features that protect Active Directory communications. This guide walks through implementing these features step-by-step, making your directory services more secure against potential attacks.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Understanding LDAP Security](#understanding-ldap-security)
- [Implementation Steps](#implementation-steps)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Prerequisites

Before beginning the implementation, ensure you have:

```plaintext
- Windows Server with Domain Controller role
- Active Directory Domain Services installed
- Enterprise/Domain Admin rights
- Group Policy Management Console access
```

## Understanding LDAP Security

### Channel Binding
Channel Binding provides protection against man-in-the-middle attacks by securely binding the LDAP communication channel between client and server. When enabled, it ensures that the client connects directly to the intended domain controller.

### LDAP Signing
LDAP Signing adds digital signatures to all LDAP traffic, ensuring message integrity and preventing tampering during transmission. This feature guarantees that messages haven't been modified in transit.

## Implementation Steps

### 1. Check Current Settings

First, verify your current LDAP security configuration:

```powershell
# Launch PowerShell as Administrator
Get-ADOptionalFeature -Filter * | Where-Object {
    $_.Name -like "LDAP Channel Binding" -or 
    $_.Name -like "LDAP Signing"
} | Format-Table Name, EnabledScopes
```

### 2. 

```powershell
# Open Group Policy Management Console
Press Win + R, type gpmc.msc, and hit Enter.
```

![image](https://github.com/user-attachments/assets/5cbec07f-c607-434d-a4aa-28d6e54f1e88)

```powershell
# Create a new GPO or edit an existing one:
Right-click your domain and select "Create a GPO in this domain, and Link it here".
```

![image](https://github.com/user-attachments/assets/d55ae525-5efa-490c-af41-c78f6c166975)

```powershell
Name the GPO (e.g., LDAP Channel Binding and Signing) and click on OK.
```

![image](https://github.com/user-attachments/assets/8bc8a34a-f6c9-47f6-9b86-1f7df5c5f077)

```powershell
Right-click on the new GPO and select Edit
```

![image](https://github.com/user-attachments/assets/9a526b33-7b60-4439-a723-787268109e47)

```powershell
# Navigate through:
Computer Configuration  > Policies > Administrative Templates > System > KDC

# Enable setting:
1. Find and enable "KDC support for claims, compound authentication and Kerberos armoring"
Double-click it and set it to "Enabled"

2. Enable "Request compound authentication"
Double-click and set to "Enabled"
```

![image](https://github.com/user-attachments/assets/08e4be0e-b7ed-48fd-b043-e6df31b60616)

### 3. Configure LDAP Signing and Enable Channel Binding

```powershell

Right-click on the same GPO and select Edit.
# In the same GPO, navigate to:
Computer Configuration  > Policies > Windows Settings > Security Settings >Local Policies >Security Options

# Enable setting:
1. Double-click "Domain controller: LDAP server channel binding token requirements"
Check "Define this policy setting" (if not checked)
Select "Always" from the dropdown (which appears you've also done)
Click "Apply" first
Then click "OK"

2. Double-click "Domain controller: LDAP server signing requirements"
Check "Define this policy setting" (if not checked)
Select "Require signing" from the dropdown options
Click "Apply" first
Then click "OK"
```

![image](https://github.com/user-attachments/assets/d526e27b-0241-4d76-b5c3-70c62b8a6b89)

### 4. Client Configuration

Apply LDAP signing requirements to clients:

```powershell
# Create/Edit  the same same GPO
Computer Configuration  > Policies > Windows Settings > Security Settings >Local Policies >Security Options

# Set policy:
Double-click "Network security: LDAP client signing requirements"
Set to "Negotiate signing" or "Require signing" (Require signing is more secure but ensure your environment can support it)
Check "Define this policy setting" (if not checked)
Click "Apply" first
Then click "OK"
```

![p4](https://github.com/user-attachments/assets/c3ba08b4-2b27-4843-8280-748b641cef96)


```powershell
# Update Group Policy
gpupdate /force
```

![image](https://github.com/user-attachments/assets/ea441859-ba7a-4c5c-b4d4-fd6d9b8b3c91)

## Verification

After implementing the changes, verify your configuration:

```powershell
# Check Domain Controller settings
Get-ADDomainController -Filter * | Select-Object Name, LDAPChannelBinding, LDAPSigning

# Test LDAP connectivity 
Test-NetConnection -ComputerName 192.168.2.15 -Port 389

Test-NetConnection -ComputerName 192.168.2.15 -Port 636  
```
Replace **192.168.2.15** with your own IP address.

![image](https://github.com/user-attachments/assets/c8fe2958-b5a9-4f4b-85ed-52dfae3b44e5)


## Troubleshooting

### Common Issues and Solutions

```powershell
# Check Event Viewer for LDAP errors
Event ID 2886: LDAP signing errors
Event ID 2887: Channel binding errors

# Monitor LDAP connections
Get-Counter '\NTDS\LDAP Active Threads'
```

![image](https://github.com/user-attachments/assets/113ff1cd-e27b-4e84-8973-d2230015fe1d)

### Error Resolution Steps

1. **Policy Not Applying**
   ```powershell
   # Verify GPO application
   gpresult /R /Scope Computer
   ```

![p5](https://github.com/user-attachments/assets/8236e4a2-f598-4790-9f82-9632750a1b14)

## Best Practices

### Security Recommendations

1. **Regular Monitoring**
```powershell
   # Monitor LDAP events
Event ID 2886: LDAP signing is not enabled.
Event ID 2887: Number of insecure LDAP binds performed.
Event ID 4624: Logon events (filter for network logon type).
Event ID 1644: Expensive LDAP searche

  $ldapEventIDs = @(2886, 2887, 4624, 1644)
 Get-EventLog -LogName Security | Where-Object {$_.EventID -in $ldapEventIDs}
```
![image](https://github.com/user-attachments/assets/f06a2fad-24d4-41e2-b998-a830dd7fc45e)

2. **Backup Configuration**
   ```powershell
   # Export GPO settings
   
   New-Item -ItemType Directory -Path "C:\GPOBackup" -Force
   Backup-GPO -Name "LDAP Channel Binding and Signing" -Path "C:\GPOBackup"
   ```

![image](https://github.com/user-attachments/assets/9393be6b-2cc3-4dd3-9e53-b3c3e42a5389)

### Implementation Guidelines

- Test in non-production first
- Document current settings
- Prepare rollback plan
- Monitor application compatibility

---

## 📝 Notes

- Regular updates are recommended
- Monitor security logs
- Keep systems patched
- Document all changes

## 📚 Additional Resources

- [LDAP signing in Windows Server](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/enable-ldap-signing-in-windows-server)
- [Active Directory Best Practices](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)
- [Windows Server Documentation](https://learn.microsoft.com/en-us/windows-server/)

---

⭐ Found this helpful? Star the repository!

📢 For issues or questions, please open an Issue in the repository.

