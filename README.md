# Active Directory Lab

A hands-on lab I completed to learn and practice Microsoft Active Directory, including domain controller setup, user management, organizational units, group policies, and secure authentication. I built this using Windows Server 2022 and Windows 11 Enterprise in VirtualBox.

---

## Skills & Competencies Demonstrated

This project showcases practical experience I gained with:

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

Here’s what I did in this lab:

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

1. I downloaded Windows Server 2022 (64-bit, English) from the Microsoft evaluation site.
2. I created a new VM in VirtualBox:
   - 8 GB RAM, 4 CPUs, 50 GB disk
   - Desktop Experience installation
   - Custom install on the virtual drive
3. I set the administrator password (e.g., `Password`).
4. I installed VirtualBox Guest Additions for better resolution and integration.
5. I took a snapshot of the clean VM before further changes (Machine → Snapshot).

![Server Setup](screenshots/Screenshot%202026-02-17%20181922.png)

---

## Part 2: Domain Controller Promotion

1. I renamed the VM to **DomainController**.
2. I opened **Server Manager** → Manage → **Add Roles and Features**.
3. I selected the DomainController server as the destination.
4. I installed **Active Directory Domain Services** (kept defaults).
5. I promoted the server to a domain controller:
   - I chose **Add a new forest**
   - Root domain: `mylab.local`
   - I completed the wizard and installed (server restarted).

![Domain Controller](screenshots/Screenshot%202026-02-17%20212048.png)  
![Forest Configuration](screenshots/Screenshot%202026-02-17%20212530.png)

---

## Part 3: Active Directory Certificate Services

Certificate Services enables secure authentication between clients and the domain controller (e.g., LDAPS, Kerberos over TLS). The **Certification Authority (CA)** issues and manages digital certificates for domain members.

1. I added **Active Directory Certificate Services** via Add Roles and Features.
2. I configured AD CS:
   - I selected **Certification Authority**
   - I used **SHA-256** for certificate signing (industry standard, more secure than SHA-1).

![AD CS Role Services](screenshots/Screenshot%202026-02-17%20214148.png)  
![AD CS Installed](screenshots/Screenshot%202026-02-17%20214416.png)

---

## Part 4: User & Organizational Unit Management

1. I opened **Server Manager** → Tools → **Active Directory Users and Computers**.
2. I viewed the directory structure under `mylab.local`.
3. I created an OU named **Groups** (right-click `mylab.local` → New → Organizational Unit).
4. I moved security group objects into the Groups OU and left only user accounts in Users.
5. I created new users with a naming scheme (e.g., first initial + last name). For this lab, I used simple passwords and "never expire"; in production, I would enforce password complexity and "must change at next logon."

![AD Users and Computers](screenshots/Screenshot%202026-02-18%20090613.png)  
![OU Structure](screenshots/Screenshot%202026-02-18%20090853.png)

---

## Part 5: Windows 11 Client & Network Setup

### Client VM Setup

- I installed Windows 11 Enterprise and VirtualBox Guest Additions.
- I used the same NAT network as the Domain Controller (see below).

### NAT Network (VirtualBox)

1. I went to **File** → **Host Network Manager** → **NAT Network** → Create.
2. I named it **ActiveDirectory**.
3. In both VM settings, I set **Attached to: NAT Network** and selected **ActiveDirectory**.

### Static IP Configuration

**Domain Controller (DC):**

- I used a static IP so clients always know where to find DNS and authentication.
- I ran `ipconfig` to see the current address, then set:
  - **IP:** `10.0.2.15`
  - **Subnet mask:** `255.255.255.0`
  - **Default gateway:** `10.0.2.1`
  - **DNS:** `127.0.0.1` (loopback; the DC hosts DNS, so it points to itself)

**Windows 11 Client:**

- I used a sequential IP (e.g., `10.0.2.16`).
- **DNS:** I set it to the DC’s IP (`10.0.2.15`), not loopback—the client must resolve names via the DC.

![Network Settings](screenshots/Screenshot%202026-02-20%20130627.png)

### Static IP on Domain Controller

![DC Static IP](screenshots/Screenshot%202026-02-20%20131908.png)

### Join Domain

1. On the client, I went to **Settings** → **Access work or school** → **Connect**.
2. I chose **Join this device to a local Active Directory domain**.
3. I entered `mylab.local` and the DC administrator credentials.
4. I restarted when prompted.
5. I logged in as **Other user** using one of the domain accounts.
6. I verified the client appears under `mylab.local` → Computers in AD Users and Computers.

![Domain Join](screenshots/Screenshot%202026-02-20%20134221.png)  
![Domain Login](screenshots/Screenshot%202026-02-20%20134344.png)  
![Client in AD](screenshots/Screenshot%202026-02-20%20134722.png)

---

## Part 6: Shared Folders & Groups

1. I created OUs: **Engineering**, **Marketing**, **Sales**.
2. Inside Engineering, I created group **Engineeringshare** and added Barney Stinson (Engineering) and Ted Mosby (Sales) to test cross-OU membership.
3. I created a shared folder via **Server Manager** → File and Storage Services → Shares.
4. I configured NTFS permissions:
   - I disabled inheritance
   - I allowed only: SYSTEM, Administrators, Creator Owner, and **Engineeringshare** group (Read, Write, Execute)
5. On the client, I mapped the share in File Explorer (e.g., `\\DomainController\sharename`) or via **This PC** → Map network drive.
6. I logged in as Ted Mosby, created a file in the share; then logged in as Barney Stinson and confirmed the file was visible.
  
![Share Permissions](screenshots/Screenshot%202026-02-20%20140635.png) 
![Shared Folder](screenshots/Screenshot%202026-02-20%20142341.png) 
![Mapped Drive](screenshots/Screenshot%202026-02-20%20142427.png)

---

## Part 7: Group Policy Objects (GPO)

### Wallpaper GPO (Engineering OU)

1. I created a custom wallpaper and copied it to **\\\\DomainController\\NETLOGON** (via Server Manager → File and Storage Services → Shares).
2. I opened **Server Manager** → Tools → **Group Policy Management**.
3. I created a new GPO (e.g., **Engineering Wallpaper**) and linked it to the Engineering OU.
4. I edited the GPO: **User Configuration** → **Policies** → **Administrative Templates** → **Desktop** → **Desktop Wallpaper**.
5. I entered the UNC path to the wallpaper.
6. Engineering users (e.g., Barney Stinson) received the new wallpaper after policy refresh.

![Wallpaper Policy](screenshots/Screenshot%202026-02-20%20144543.png)
![GPO Editor](screenshots/Screenshot%202026-02-20%20145139.png)  

### Account Lockout GPO

1. I created a GPO (e.g., **Account Lockout Policy**).
2. I edited it: **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Account Policies** → **Account Lockout Policy**.
3. I set **Account lockout threshold** to 3 invalid attempts.
4. I right-clicked the GPO and chose **Enforce**.
5. I intentionally locked out an account (e.g., wrong password 3 times) to test the policy.
6. I unlocked and reset: **AD Users and Computers** → Search for user → Unlock account → Reset password with a secure temporary password.

![Account Lockout Policy](screenshots/Screenshot%202026-02-20%20163622.png)  
![Account Locked Out](screenshots/Screenshot%202026-02-20%20164439.png)  
![Password Reset](screenshots/Screenshot%202026-02-20%20164601.png)

---

## What I Learned

Walking through this lab taught me several important concepts and workflows:

**Active Directory and Domain Design** — I learned how a forest and root domain are created, why OUs matter for organizing users and computers, and how AD Users and Computers fits into day-to-day administration. The distinction between users, groups, and OUs became much clearer once I had to move objects and apply permissions.

**Network and DNS** — The difference between DC and client DNS config was a big takeaway. The DC points DNS to itself (127.0.0.1) because it hosts DNS; clients must point to the DC’s IP so they can resolve domain names and reach authentication services. Static IPs on the DC are necessary so clients always know where to find it.

**File Sharing and Permissions** — I got hands-on practice with NTFS permissions, inheritance, and group-based access. Disabling inheritance and explicitly assigning rights to a group (like Engineeringshare) reinforced how Windows combines share and NTFS permissions. Testing cross-OU group membership (Ted in Sales accessing an Engineering share) showed how groups can cross organizational boundaries.

**Group Policy** — I saw how GPOs apply differently: user configuration (e.g., wallpaper) vs. computer configuration (account lockout). Using Enforce to prioritize a GPO and testing the lockout policy end-to-end helped me understand how policy flows down the OU hierarchy.

**Security and Operations** — I learned why SHA-256 is preferred over SHA-1 for certificates, why domain admin accounts should be used sparingly, and how to handle a real help desk scenario: finding a user in AD, unlocking their account, and resetting their password with a secure temporary value.

**Virtualization** — Using VirtualBox, NAT networks, and snapshots gave me a repeatable, isolated lab that mirrors how enterprises segment and test environments.

---

## Notes

- Domain admin should only be used when required; use delegated accounts for day-to-day tasks.
- This lab uses `.local` and simple passwords for learning; production environments require proper domain names and stronger password policies.

---

## License

This project is for educational purposes only.
