# Active Directory Lab

A hands-on lab for learning and practicing Microsoft Active Directory, including domain controller setup, user management, organizational units, group policies, and secure authentication. Built using Windows Server 2022 and Windows 11 Enterprise in VirtualBox.

---

## Skills & Competencies Demonstrated

This project showcases practical experience with:

| Area | What I Learned |
|------|----------------|
| **Active Directory** | Domain Controller promotion, forest and root domain creation, OU hierarchy design, AD Users and Computers |
| **User & Identity Management** | Creating domain users, security groups, cross-OU group membership, naming conventions, password policy awareness |
| **Network Configuration** | Static IP assignment, DNS configuration (loopback for DC vs. pointing to DC for clients), NAT networking in VirtualBox |
| **File Sharing & Permissions** | SMB shares, NTFS permissions, disabling inheritance, group-based access control, network drive mapping |
| **Group Policy (GPO)** | Creating and linking GPOs to OUs, user configuration (e.g., desktop wallpaper), computer configuration (account lockout), policy enforcement |
| **PKI & Certificate Services** | AD Certificate Services, Certification Authority (CA) role, SHA-256 for secure authentication |
| **Help Desk / IT Support** | Account lockout recovery, password reset workflow, searching AD for users, unlock procedures |
| **Windows Server Admin** | Server Manager, roles and features, promote to DC, best practices (e.g., avoid daily use of domain admin account) |
| **Virtualization** | VirtualBox, VM snapshots, Guest Additions, NAT networks for lab isolation |

---

## Overview

This lab walks through:

- **Domain Controller (DC)** setup and promotion
- **Active Directory Certificate Services** for secure client authentication
- **User & OU management** (Organizational Units)
- **Windows 11 client** domain join
- **Shared folders** with group-based access control
- **Group Policy Objects (GPO)** for wallpaper and account lockout policies
- **Password reset** workflow (lockout recovery)

---

## Lab Environment

| Component | Specification |
|-----------|---------------|
| Hypervisor | VirtualBox |
| Domain Controller | Windows Server 2022 (Desktop Experience) |
| Client | Windows 11 Enterprise |
| Domain | `mylab.local` |
| RAM | 8 GB (Server), 4 GB (Client) |
| CPU | 4 vCPUs |
| Storage | 50 GB |

---

## Prerequisites

- [VirtualBox](https://www.virtualbox.org/) installed
- Windows Server 2022 evaluation ISO ([download here](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022))
- Windows 11 Enterprise ISO (available through Microsoft evaluation or Volume Licensing)

---

## Part 1: Windows Server 2022 Setup

1. Download Windows Server 2022 (64-bit, English) from the Microsoft evaluation site.
2. Create a new VM in VirtualBox:
   - 8 GB RAM, 4 CPUs, 50 GB disk
   - Desktop Experience installation
   - Custom install on the virtual drive
3. Set the administrator password (e.g., `Password!`).
4. Install VirtualBox Guest Additions for better resolution and integration.
5. Take a snapshot of the clean VM before further changes (Machine → Snapshot).

![Server Setup](screenshots/Screenshot%202026-02-17%20181922.png)

---

## Part 2: Domain Controller Promotion

1. Rename the VM to **DomainController**.
2. Open **Server Manager** → Manage → **Add Roles and Features**.
3. Select the DomainController server as the destination.
4. Install **Active Directory Domain Services** (keep defaults).
5. Promote the server to a domain controller:
   - Choose **Add a new forest**
   - Root domain: `mylab.local`
   - Complete the wizard and install (server will restart).

![Domain Controller](screenshots/Screenshot%202026-02-17%20211653.png)  
![Forest Configuration](screenshots/Screenshot%202026-02-17%212530.png)

---

## Part 3: Active Directory Certificate Services

Certificate Services enables secure authentication between clients and the domain controller (e.g., LDAPS, Kerberos over TLS). The **Certification Authority (CA)** issues and manages digital certificates for domain members.

1. Add **Active Directory Certificate Services** via Add Roles and Features.
2. Configure AD CS:
   - Select **Certification Authority**
   - Use **SHA-256** for certificate signing (industry standard, more secure than SHA-1).

---

## Part 4: User & Organizational Unit Management

1. **Server Manager** → Tools → **Active Directory Users and Computers**.
2. View the directory structure under `mylab.local`.
3. Create an OU named **Groups** (right-click `mylab.local` → New → Organizational Unit).
4. Move security group objects into the Groups OU; leave only user accounts in Users.
5. Create new users with a naming scheme (e.g., first initial + last name). For this lab, simple passwords and "never expire" were used; in production, enforce password complexity and "must change at next logon."

![AD Users and Computers](screenshots/Screenshot%202026-02-18%20090613.png)  
![OU Structure](screenshots/Screenshot%202026-02-18%20090853.png)

---

## Part 5: Windows 11 Client & Network Setup

### Client VM Setup

- Install Windows 11 Enterprise and VirtualBox Guest Additions.
- Use the same NAT network as the Domain Controller (see below).

### NAT Network (VirtualBox)

1. **File** → **Host Network Manager** → **NAT Network** → Create.
2. Name it **ActiveDirectory**.
3. In both VM settings, set **Attached to: NAT Network** and select **ActiveDirectory**.

### Static IP Configuration

**Domain Controller (DC):**

- Use a static IP so clients always know where to find DNS and authentication.
- `ipconfig` to see the current address, then set:
  - **IP:** `10.0.2.15`
  - **Subnet mask:** `255.255.255.0`
  - **Default gateway:** `10.0.2.1`
  - **DNS:** `127.0.0.1` (loopback; the DC hosts DNS, so it points to itself)

**Windows 11 Client:**

- Use a sequential IP (e.g., `10.0.2.16`).
- **DNS:** Use the DC’s IP (`10.0.2.15`), not loopback—the client must resolve names via the DC.

![Network Settings](screenshots/Screenshot%202026-02-20%20130627.png)

### Join Domain

1. On the client: **Settings** → **Access work or school** → **Connect**.
2. Choose **Join this device to a local Active Directory domain**.
3. Enter `mylab.local` and the DC administrator credentials.
4. Restart when prompted.
5. Log in as **Other user** using one of the domain accounts.
6. Verify the client appears under `mylab.local` → Computers in AD Users and Computers.

![Domain Join](screenshots/Screenshot%202026-02-20%20131908.png)  
![Domain Login](screenshots/Screenshot%202026-02-20%20132257.png)

---

## Part 6: Shared Folders & Groups

1. Create OUs: **Engineering**, **Marketing**, **Sales**.
2. Inside Engineering, create group **Engineeringshare** and add Barney Stinson (Engineering) and Ted Mosby (Sales) to test cross-OU membership.
3. Create a shared folder via **Server Manager** → File and Storage Services → Shares.
4. Configure NTFS permissions:
   - Disable inheritance
   - Allow only: SYSTEM, Administrators, Creator Owner, and **Engineeringshare** group (Read, Write, Execute)
5. On the client, map the share in File Explorer (e.g., `\\DomainController\sharename`) or via **This PC** → Map network drive.
6. Log in as Ted Mosby, create a file in the share; log in as Barney Stinson and confirm the file is visible.

![Shared Folder](screenshots/Screenshot%202026-02-20%20134221.png)  
![Share Permissions](screenshots/Screenshot%202026-02-20%20134344.png)  
<!-- ![Mapped Drive](screenshots/Screenshot%202026-02-20%20135737.png) -->

---

## Part 7: Group Policy Objects (GPO)

### Wallpaper GPO (Engineering OU)

1. Create a custom wallpaper and copy it to **\\\\DomainController\\NETLOGON** (via Server Manager → File and Storage Services → Shares).
2. **Server Manager** → Tools → **Group Policy Management**.
3. Create a new GPO (e.g., **Engineering Wallpaper**) and link it to the Engineering OU.
4. Edit the GPO: **User Configuration** → **Policies** → **Administrative Templates** → **Desktop** → **Desktop Wallpaper**.
5. Enter the UNC path to the wallpaper.
6. Engineering users (e.g., Barney Stinson) receive the new wallpaper after policy refresh.

![GPO Editor](screenshots/Screenshot%202026-02-20%20142427.png)  
![Wallpaper Policy](screenshots/Screenshot%202026-02-20%20164439.png)

### Account Lockout GPO

1. Create a GPO (e.g., **Account Lockout Policy**).
2. Edit: **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Account Policies** → **Account Lockout Policy**.
3. Set **Account lockout threshold** to 3 invalid attempts.
4. Right-click the GPO and **Enforce** it.
5. Intentionally lock out an account (e.g., wrong password 3 times).
6. Unlock and reset: **AD Users and Computers** → Search for user → Unlock account → Reset password with a secure temporary password.

<!-- ![Account Lockout Policy](screenshots/Screenshot%202026-02-20%20163545.png)   -->
![Password Reset](screenshots/Screenshot%202026-02-20%20163622.png)

---

## Key Takeaways

- **Enterprise-ready skills:** This lab mirrors real-world tasks—domain joins, shared folder access, GPO deployment, and password reset tickets—common in help desk and junior sysadmin roles.
- **Security awareness:** Understood why DCs need static IPs (consistent DNS/auth), why SHA-256 is used over SHA-1, and why domain admin should be used sparingly.
- **Troubleshooting mindset:** Practiced account lockout recovery end-to-end (search user in AD → unlock → reset password with secure temp password), a typical IT support workflow.

---

## Notes

- Domain admin should only be used when required; use delegated accounts for day-to-day tasks.
- Lab uses `.local` and simple passwords for learning; production environments require proper domain names and stronger password policies.

---

## License

This project is for educational purposes only.
