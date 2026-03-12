# Active Directory Home Lab Project

## Overview

I built a home lab with Active Directory to learn Windows Server administration and practice IT support skills. This project simulates a small business network with a domain controller and client computers.

## What I Built

- 1 Windows Server 2019 (Domain Controller)
- 2 Windows 10 computers (Client machines)
- Active Directory domain with users and groups
- Group policies to control user settings
- DHCP for automatic IP addresses

## Lab Setup

### Step 1: Install Proxmox

I downloaded VirtualBox from virtualbox.org and installed it on my computer. This lets me run multiple virtual machines.

### Step 2: Create the Domain Controller VM

**VM Settings:**
- Name: DC01
- OS: Windows Server 2019
- RAM: 4 GB
- Hard Drive: 60 GB
- Network: Internal Network

I installed Windows Server 2019 and set up a static IP address: 10.0.2.10

<img width="989" height="689" alt="image" src="https://github.com/user-attachments/assets/02e30a89-6ef4-44e5-ab7f-2506cfd60439" />


### Step 3: Install Active Directory

I used Server Manager to install Active Directory Domain Services:

1. Opened Server Manager
2. Clicked "Add Roles and Features"
3. Selected "Active Directory Domain Services"
4. After installation, promoted the server to Domain Controller
5. Created a new domain called "homelab.local"
6. Server automatically restarted

Now I have a working domain controller!

### Step 4: Set Up DHCP

I added the DHCP role so client computers could get IP addresses automatically:

1. Server Manager → Add Roles → DHCP Server
2. Created a scope: 10.0.2.100 - 10.0.2.200
3. Set DNS server to: 10.0.2.10 (the domain controller)

<img width="560" height="418" alt="image" src="https://github.com/user-attachments/assets/a385db0a-2803-42ed-847a-429dc9ce1a70" />


### Step 5: Create Organizational Units

I organized Active Directory with folders (Organizational Units):

```
homelab.local
├── IT Department
├── HR Department
├── Admin Users
└── Workstations
```

This keeps everything organized, just like folders on a computer.

### Step 6: Create User Accounts

I created test users for different departments:

**IT Department:**
- Username: john.smith
- Name: John Smith
- Password: TempPass123!

**HR Department:**
- Username: mary.jones
- Name: Mary Jones
- Password: TempPass123!

**Admin Account:**
- Username: admin.user
- Name: IT Admin
- Password: AdminPass123!

I set all accounts to "User must change password at next logon" for security.

### Step 7: Create Security Groups

I made groups to manage permissions easier:

- **IT_Team** - For IT department staff
- **HR_Team** - For HR department staff
- **Domain_Admins** - For administrators

Then I added users to their groups:
- john.smith → IT_Team
- mary.jones → HR_Team
- admin.user → Domain_Admins

<img width="1014" height="586" alt="image" src="https://github.com/user-attachments/assets/ba35984a-0663-414d-bf2d-7145fdf9dcf9" />


### Step 8: Set Up Windows 10 Client Computers

I created two Windows 10 VMs:

<img width="1010" height="613" alt="image" src="https://github.com/user-attachments/assets/1ed4a94d-0159-404b-ae84-570629cb55ec" />


**VM Settings:**
- Names: PC01 and PC02
- OS: Windows 10 Pro (needed for domain join)
- RAM: 8 GB each
- Hard Drive: 50 GB each
- Network: Same internal network as DC01

### Step 9: Join Computers to Domain

On each Windows 10 VM:

1. Went to Settings → System → About
2. Clicked "Rename this PC (advanced)"
3. Clicked "Change" button
4. Selected "Domain" and typed: homelab.local
5. Entered domain admin username and password
6. Computer restarted
<img width="1011" height="480" alt="image" src="https://github.com/user-attachments/assets/04fcdde8-6bc6-4bb2-b232-8db5c8756fae" />


Success! The computers are now part of the homelab.local domain.

### Step 10: Test Domain Login

I logged into PC01 using domain credentials:

1. At login screen, clicked "Other user"
2. Typed: homelab\john.smith
3. Entered password: TempPass123!
4. Windows prompted me to change password (as configured)
5. Set new password: NewPass123!
6. Successfully logged in!

<img width="1013" height="629" alt="image" src="https://github.com/user-attachments/assets/12742fea-5453-4a8f-b362-ebf4a95cf6d8" />

The user profile was created automatically. Everything worked!

## Group Policies I Created

### Policy 1: Disable Control Panel

**Purpose:** Prevent regular users from changing system settings

**Steps:**
1. Opened Group Policy Management
2. Created new GPO: "Disable Control Panel"
3. Linked to "HR Department" OU
4. Edited policy:
   - User Configuration → Administrative Templates → Control Panel
   - Enabled "Prohibit access to Control Panel and PC settings"
5. On PC01, ran `gpupdate /force`

<img width="1016" height="548" alt="image" src="https://github.com/user-attachments/assets/292ffa85-73b3-4a21-a595-31600ee331f3" />


<img width="1016" height="630" alt="image" src="https://github.com/user-attachments/assets/0182467d-e32e-4672-a259-bc71354cb487" />


**Result:** Mary Jones (HR user) cannot open Control Panel anymore. Perfect for restricting non-technical users!

### Policy 2: Desktop Wallpaper

**Purpose:** Set company wallpaper for all users

**Steps:**
1. Created new GPO: "Company Wallpaper"
2. Linked to entire domain
3. Configured:
   - User Configuration → Administrative Templates → Desktop → Desktop
   - Set "Desktop Wallpaper" to company image path
4. Ran `gpupdate /force` on client computers
<img width="1006" height="485" alt="image" src="https://github.com/user-attachments/assets/61d1421e-0f53-463d-a874-3bea6b675d31" />
<img width="1005" height="483" alt="image" src="https://github.com/user-attachments/assets/bb956076-a913-4ab8-a1c0-2eb23bdb7c9e" />

**Result:** All users see the company wallpaper when they log in.

### Policy 3: Password Policy

**Purpose:** Enforce strong passwords

**Steps:**
1. Created new GPO: "Password Requirements"
2. Linked to domain
3. Settings:
   - Minimum password length: 8 characters
   - Password must meet complexity requirements: Enabled
   - Maximum password age: 90 days
<img width="1015" height="625" alt="image" src="https://github.com/user-attachments/assets/0dca9909-0649-4216-90b9-2a4fa46787bc" />

**Result:** Users must create strong passwords and change them every 90 days.

### Policy 4: Lock Screen After Inactivity

**Purpose:** Automatically lock computers after 10 minutes

**Steps:**
1. Created GPO: "Auto Lock Screen"
2. Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options
3. Enabled "Interactive logon: Machine inactivity limit" → 600 seconds

**Result:** Computers lock automatically if no one is using them. Good for security!
<img width="1012" height="638" alt="image" src="https://github.com/user-attachments/assets/99a43cd9-0065-4a3d-a889-b44d5ffb8e19" />

## File Permissions Setup

### Shared Folder with Different Access Levels

I created a shared folder on DC01 to practice permissions:

**Folder Structure:**
```
C:\SharedFiles
├── IT_Only (IT Team only)
├── HR_Only (HR Team only)
└── Everyone (All users)
```

**IT_Only Folder Permissions:**
- IT_Team: Full Control
- HR_Team: No access
- Everyone: No access

**HR_Only Folder Permissions:**
- HR_Team: Read & Write
- IT_Team: Read only (for support)
- Everyone: No access

**Everyone Folder Permissions:**
- Domain Users: Read & Write
- Everyone: Read only

**How I Set This Up:**
1. Right-clicked folder → Properties → Sharing → Advanced Sharing
2. Shared the folder
3. Clicked "Permissions" → Removed "Everyone"
4. Added specific groups (IT_Team, HR_Team, etc.)
5. Set NTFS permissions under "Security" tab
6. Made sure NTFS permissions matched share permissions
<img width="878" height="549" alt="image" src="https://github.com/user-attachments/assets/b0bd0916-1f9d-4bc3-8501-2cff5d3f5cab" />

**Testing:**
- Logged in as john.smith (IT) - Could access IT_Only folder ✓
- Logged in as mary.jones (HR) - Could NOT access IT_Only folder ✓
- Both could access Everyone folder ✓
<img width="1009" height="628" alt="image" src="https://github.com/user-attachments/assets/08bfe51b-46fc-40f0-87da-d32b856c9ac4" />

## Testing Common IT Support Scenarios

### Scenario 1: User Forgot Password

**Problem:** Mary Jones forgot her password and can't log in.

**Solution Steps:**
1. Logged into DC01 as administrator
2. Opened "Active Directory Users and Computers"
3. Found mary.jones in HR Department OU
4. Right-clicked → "Reset Password"
5. Entered new temporary password: TempHR123!
6. Checked "User must change password at next logon"
7. Told user the temporary password
8. User logged in and was forced to create new password

**Time to resolve:** 2 minutes
<img width="1015" height="628" alt="image" src="https://github.com/user-attachments/assets/b3d21ed9-afe4-419e-aba1-8a33e376cafc" />

**Skills demonstrated:** Password reset, user support, security best practices

### Scenario 2: User Account Locked Out

**Problem:** John Smith entered wrong password 5 times. Account locked.

**Solution Steps:**
1. Opened Active Directory Users and Computers
2. Found john.smith
3. Right-clicked → Properties → Account tab
4. Checked "Unlock account" checkbox
5. Clicked OK
6. User could log in immediately

**Alternative method using PowerShell:**
```powershell
Unlock-ADAccount -Identity john.smith
```
<img width="1013" height="625" alt="image" src="https://github.com/user-attachments/assets/7297e196-d132-48be-85c3-23da00be0d69" />

**Time to resolve:** 1 minute

### Scenario 3: New Employee Needs Account

**Problem:** New HR employee "Sarah Johnson" needs domain account.

**Solution Steps:**
1. Opened Active Directory Users and Computers
2. Right-clicked "HR Department" OU → New → User
3. Filled in information:
   - First name: Sarah
   - Last name: Johnson
   - User logon name: sarah.johnson
4. Set temporary password: TempHR456!
5. Checked "User must change password at next logon"
6. Clicked Finish
7. Added sarah.johnson to HR_Team group
8. Verified permissions by logging in as Sarah
<img width="1194" height="753" alt="image" src="https://github.com/user-attachments/assets/f1fea798-bed9-4489-b7c0-75c07164af01" />

**Time to complete:** 5 minutes

### Scenario 4: User Can't Access Shared Folder

**Problem:** Mary Jones says "Access Denied" when opening IT_Only folder.

**Diagnosis:**
1. Checked what groups Mary is in: `HR_Team`
2. Checked IT_Only folder permissions: Only `IT_Team` has access
3. This is correct! HR shouldn't access IT files
<img width="1014" height="631" alt="image" src="https://github.com/user-attachments/assets/df776d64-f057-4767-b45d-6752d2d3f867" />

**Resolution:**
Explained to user that this folder is restricted to IT department only. Not a problem - security working as designed!

If she actually NEEDED access, I would:
1. Ask IT manager for approval
2. Add mary.jones to IT_Team group OR
3. Add HR_Team to folder permissions with appropriate access level
<img width="1016" height="642" alt="image" src="https://github.com/user-attachments/assets/3f45b63e-8b1e-4b97-bc12-c8d34df75a11" />

**Time to resolve:** 3 minutes

**Skills demonstrated:** Troubleshooting, checking permissions, security awareness

### Scenario 5: User Can't Log Into Domain

**Problem:** PC02 says "The trust relationship between this workstation and the primary domain failed."

**Cause:** Computer account password expired or corrupt.

**Solution Steps:**
1. Logged into PC02 with local administrator account
2. Removed computer from domain (joined workgroup temporarily)
3. Restarted computer
4. Rejoined computer to homelab.local domain
5. Restarted again
6. User could log in successfully

**Alternative PowerShell fix (on DC01):**
```powershell
Reset-ComputerMachinePassword -Server DC01 -Credential HOMELAB\Administrator
```

**Time to resolve:** 10 minutes

### Scenario 6: User Needs Access to New Shared Folder

**Problem:** Create new shared folder for Finance department.

**Solution Steps:**
1. Created folder: C:\SharedFiles\Finance_Only
2. Shared the folder with share permissions:
   - Finance_Team: Full Control
3. Set NTFS security permissions:
   - Finance_Team: Modify
   - Domain Admins: Full Control
   - Removed all other permissions
4. Created shortcut in user's Group Policy mapped drives
5. Tested access with Finance user account
<img width="1016" height="639" alt="image" src="https://github.com/user-attachments/assets/da5142c8-c0ae-45c0-8f6c-c1ff48a75a98" />
<img width="1019" height="637" alt="image" src="https://github.com/user-attachments/assets/341e804a-a123-4455-b980-e11bb0f61c10" />

**Time to complete:** 8 minutes

### Scenario 7: Apply Group Policy Not Working

**Problem:** Desktop wallpaper policy not applying to PC01.

**Troubleshooting Steps:**
1. On PC01, opened Command Prompt as administrator
2. Ran: `gpupdate /force` to force policy refresh
3. Ran: `gpresult /r` to see applied policies
4. Checked if policy was actually linked to correct OU
5. Verified computer was in correct OU in Active Directory
6. Restarted PC01

**Result:** Policy applied successfully after restart!

**Lessons learned:** 
- Group policies need `gpupdate` or restart to apply
- Always check policy is linked to correct OU
- Use `gpresult` to troubleshoot policy issues

## Key Skills Demonstrated

**Windows Server Administration:**
- Active Directory installation and configuration
- Domain controller setup
- DNS and DHCP configuration
- User and group management

**Group Policy Management:**
- Creating and linking GPOs
- User restrictions and security policies
- Desktop management policies
- Troubleshooting policy application

**Security & Permissions:**
- NTFS permissions configuration
- Share permissions setup
- Security group management
- Principle of least privilege

**IT Support Skills:**
- Password resets
- Account unlocking
- New user creation
- Access troubleshooting
- Domain join issues
- Group policy troubleshooting

**PowerShell:**
- Basic AD commands
- User management automation
- Troubleshooting commands

## Challenges I Overcame

**Challenge 1:** VM wouldn't connect to domain
- **Solution:** Checked DNS settings - client was using wrong DNS server. Changed to point to DC (10.0.2.10)
<img width="1013" height="499" alt="image" src="https://github.com/user-attachments/assets/3548fd2a-6502-47d2-b3de-787cc8d283c1" />

**Challenge 2:** Group Policy not applying
- **Solution:** Ran `gpupdate /force` and restarted computer. Learned policies don't apply instantly.

**Challenge 3:** NTFS permissions weren't changing
- **Solution:** NTFS permissions were allowing inheritance from parent. Learned that this setting is configured separately.
<img width="1015" height="634" alt="image" src="https://github.com/user-attachments/assets/2d500915-1ad5-4d48-957d-f49758ba6f14" />

**Challenge 4:** Forgot DSRM password for DC
- **Solution:** Documented all passwords in secure location. Learned importance of password management.

## What I Learned

1. **Active Directory is the backbone** of Windows networks - everything connects to it
2. **Group Policy is powerful** - you can control almost anything on user computers
3. **Permissions are tricky** - both share AND NTFS permissions matter
4. **Documentation is critical** - I documented every step and it saved me many times
5. **Testing is important** - Always test from a user's perspective
6. **Troubleshooting methodology** - Check basics first (DNS, connectivity), then dig deeper

## Future Improvements

- Add file server with home drives for each user
- Set up certificate services for SSL certificates
- Create automated backup solution
- Add Linux client and integrate with Active Directory
- Implement Remote Desktop Services
- Set up VPN access for remote users
- Create disaster recovery plan and test it

## Resources Used

- Microsoft Documentation
- KevTech's YouTube Videos
- Claude
- My own trial and error!


## Contact

Questions about this project? Feel free to reach out!

- Email: keikoyoter@gmail.com
- LinkedIn: [your-linkedin-profile](https://www.linkedin.com/in/koyo-keiter-20962b37a/)
- GitHub: Koyo-dotcom

---

**Project Status:** Complete ✓  
**Date Completed:** November 2024  
**Time Invested:** ~10 hours over 2 weeks
