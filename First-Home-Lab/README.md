# Windows Active Directory & Group Policy Home Lab

A hands-on Windows infrastructure and cybersecurity home lab built to
simulate a small centralized organization using Windows Server 2025,
Windows 11 Pro, Active Directory Domain Services, DNS, Organizational
Units, Security Groups, Group Policy, scripts, and domain security
policies.

> **Author:** Mohamed Ali Mohamed\
> **Role:** Cybersecurity Graduate \| SOC Analyst\
> **LinkedIn:** https://www.linkedin.com/in/mohamed-ali-481a70387\
> **Email:** mo5665ali@gmail.com

------------------------------------------------------------------------

## Table of Contents

-   [Project Overview](#project-overview)
-   [Objectives](#objectives)
-   [Lab Environment](#lab-environment)
-   [Architecture](#architecture)
-   [Prerequisites](#prerequisites)
-   [1. Create the Virtual Machines](#1-create-the-virtual-machines)
-   [2. Prepare Windows Server 2025](#2-prepare-windows-server-2025)
-   [3. Install Active Directory Domain
    Services](#3-install-active-directory-domain-services)
-   [4. Configure the Domain
    Controller](#4-configure-the-domain-controller)
-   [5. Create Organizational Units](#5-create-organizational-units)
-   [6. Create Users and Security
    Groups](#6-create-users-and-security-groups)
-   [7. Prepare Windows 11](#7-prepare-windows-11)
-   [8. Join Windows 11 to the Domain](#8-join-windows-11-to-the-domain)
-   [9. Verify the Domain Environment](#9-verify-the-domain-environment)
-   [10. Configure Group Policy](#10-configure-group-policy)
-   [11. Configure Department
    Policies](#11-configure-department-policies)
-   [12. Configure Scripts](#12-configure-scripts)
-   [13. Configure Password and Account Lockout
    Policies](#13-configure-password-and-account-lockout-policies)
-   [14. Test the Policies](#14-test-the-policies)
-   [15. Administrative Commands](#15-administrative-commands)
-   [16. Troubleshooting](#16-troubleshooting)
-   [17. Security Considerations](#17-security-considerations)
-   [18. What I Learned](#18-what-i-learned)
-   [19. Future Improvements](#19-future-improvements)
-   [Project Summary](#project-summary)

------------------------------------------------------------------------

## Project Overview

This project is a practical Windows Active Directory and Group Policy
home lab.

The goal was to build a small centralized Windows domain environment
instead of managing every Windows workstation independently.

The lab contains a Windows Server 2025 Domain Controller and a Windows
11 Pro domain client. Active Directory provides centralized identity
management, DNS supports domain discovery and name resolution, and Group
Policy is used to apply security and usability settings to users and
computers.

The domain used in the lab is:

``` text
test.local
```

The documented server is:

``` text
ADMIN-Server
IP Address: 192.168.0.10
```

The Windows 11 client is:

``` text
PC-11
DNS: 192.168.0.10
```

The project was built as a hands-on learning environment for Windows
administration, Active Directory, Group Policy, basic Windows security,
and troubleshooting.

------------------------------------------------------------------------

## Objectives

The main objectives of the lab were to gain practical experience with:

-   Windows Server 2025
-   Active Directory Domain Services
-   Domain Controllers
-   DNS and its relationship with Active Directory
-   Organizational Units
-   Domain users
-   Security groups
-   Windows 11 domain joining
-   Group Policy Objects
-   Department-specific policies
-   Policy inheritance and enforcement
-   Group Policy scripts
-   Password policies
-   Account lockout policies
-   Windows administrative tools
-   Basic troubleshooting
-   Centralized Windows management

------------------------------------------------------------------------

## Lab Environment

### Virtual Machines

  -----------------------------------------------------------------------
  Machine                 Operating System        Role
  ----------------------- ----------------------- -----------------------
  `ADMIN-Server`          Windows Server 2025     Domain Controller, AD
                          Datacenter              DS, DNS, GPO management

  `PC-11`                 Windows 11 Pro          Domain-joined client
  -----------------------------------------------------------------------

Both machines were documented as virtual machines. The lab documentation
uses the machines shown in the original project and does not assume
additional servers or network devices.

### Core Configuration

  Item             Value
  ---------------- --------------------------------
  Domain           `test.local`
  Server           `ADMIN-Server`
  Server IP        `192.168.0.10`
  Client           `PC-11`
  Client DNS       `192.168.0.10`
  Server OS        Windows Server 2025 Datacenter
  Client OS        Windows 11 Pro
  Virtualization   VMware

> The exact Windows 11 client IP was documented only as a `192.168.0.1X`
> range in the project notes, so this README does not invent the final
> client address.

------------------------------------------------------------------------

## Architecture

``` text
                    Windows Server 2025
                       ADMIN-Server
                      192.168.0.10
                            |
             +--------------+--------------+
             |                             |
     Active Directory                    DNS
        test.local                 192.168.0.10
             |
       Organizational Units
             |
   +---------+---------+---------+
   |         |         |         |
  IT        HR      Finance     Sales
   |         |         |         |
 Users / Security Groups / GPOs
             |
             v
       Windows 11 Pro
          PC-11
       Domain Client
```

The main idea is simple:

``` text
User
  ↓
Security Group
  ↓
Organizational Unit
  ↓
Group Policy
  ↓
Windows Client
```

------------------------------------------------------------------------

# 1. Create the Virtual Machines

The first step was to prepare the virtual lab environment.

Two virtual machines were used:

1.  Windows Server 2025
2.  Windows 11 Pro

### Windows Server 2025

The Server VM acts as the central machine for the lab.

Its responsibilities are:

-   Active Directory Domain Services
-   DNS
-   Domain Controller functionality
-   Group Policy Management

### Windows 11 Pro

The Windows 11 VM acts as an end-user workstation.

It is later joined to the `test.local` domain so that users can
authenticate against Active Directory and receive Group Policies.

------------------------------------------------------------------------

# 2. Prepare Windows Server 2025

After installing Windows Server 2025, the server was prepared for its
role as a Domain Controller.

## 2.1 Configure the Server Name

The server was configured with the computer name:

``` text
ADMIN-Server
```

A meaningful server name makes the lab easier to manage and identify.

## 2.2 Configure a Static IP Address

The server was assigned:

``` text
192.168.0.10
```

The IP was configured through the Windows network adapter settings.

The `ncpa.cpl` utility can be used to quickly open Network Connections.

``` text
ncpa.cpl
```

### Why use a static IP?

A Domain Controller needs a predictable network address.

Clients rely on the Domain Controller for services such as DNS and
Active Directory authentication. If the server address changes
unexpectedly, clients may no longer be able to locate the domain
services correctly.

### Important note

The original project documentation does not explicitly record the final
subnet mask, gateway, or server date/time values, so they are not
invented here.

------------------------------------------------------------------------

# 3. Install Active Directory Domain Services

Active Directory Domain Services, or AD DS, provides centralized
identity and directory management for the Windows domain.

Instead of creating independent accounts on every workstation, users and
computers can be managed centrally.

## What was configured?

AD DS was installed on Windows Server 2025.

The server was then configured as the Domain Controller for:

``` text
test.local
```

The server also provides DNS for the lab.

## Why?

The Domain Controller provides the central directory that stores and
manages:

-   Users
-   Groups
-   Computers
-   Organizational Units
-   Authentication
-   Group Policy relationships

Without a Domain Controller, the Windows 11 client would not have an
Active Directory domain to join.

------------------------------------------------------------------------

# 4. Configure the Domain Controller

The final documented server configuration was:

``` text
Operating System: Windows Server 2025 Datacenter
Computer Name: ADMIN-Server
Domain: test.local
IP Address: 192.168.0.10
```

The Server Manager screenshots also showed:

-   Active Directory Domain Services
-   DNS
-   File and Storage Services

The documented VM screenshot showed VMware with 4 GB RAM.

## How DNS and Active Directory work together

The Windows 11 client uses:

``` text
DNS = 192.168.0.10
```

The client asks the DNS service on the Domain Controller where the
domain and its services are located.

The basic flow is:

``` text
PC-11
  |
  | DNS request
  v
192.168.0.10
  |
  v
DNS
  |
  v
Active Directory / Domain Controller
  |
  v
test.local
```

This is why using the correct DNS server is one of the most important
parts of the lab.

------------------------------------------------------------------------

# 5. Create Organizational Units

Organizational Units were created using:

``` text
dsa.msc
```

This opens Active Directory Users and Computers.

The documented OUs are:

``` text
IT
HR
Finance
Sales
Groups
```

## Why use OUs?

An OU is a logical container inside Active Directory.

OUs help administrators:

-   Organize users and computers
-   Separate departments
-   Apply different Group Policies
-   Keep the directory structured
-   Support delegated administration

In this lab, departmental OUs are especially important because Group
Policies are linked to specific departments.

For example:

``` text
HR OU
  ↓
HR GPOs

Sales OU
  ↓
Sales GPOs

Finance OU
  ↓
Finance GPOs
```

No IT-specific GPO is claimed here because the project documentation did
not contain a policy screenshot for the IT OU.

------------------------------------------------------------------------

# 6. Create Users and Security Groups

## 6.1 Create Users

Domain users were created for:

-   IT
-   HR
-   Finance
-   Sales

Users were placed inside their respective departmental OUs.

The original screenshots do not document individual usernames or
passwords, so this README does not list any.

### Why centralized users?

Centralized accounts make it possible to manage authentication from
Active Directory.

For example, an administrator can manage an account from one location
rather than creating separate local accounts on every workstation.

------------------------------------------------------------------------

## 6.2 Create Security Groups

The documented security groups are:

  Group            Department / Purpose
  ---------------- ----------------------
  `IT-Group`       IT
  `HR-Group`       HR
  `Fin-Group`      Finance
  `Sales-Group`    Sales
  `Admins-Group`   Admin

Groups allow permissions and administrative rights to be assigned to
collections of users instead of individual accounts.

The group names suggest their departmental relationships. This
relationship is based on the documented group names.

------------------------------------------------------------------------

# 7. Prepare Windows 11

Windows 11 Pro was installed as the client workstation.

The documented client configuration includes:

``` text
Operating System: Windows 11 Pro
Computer Name: PC-11
DNS: 192.168.0.10
IP Range: 192.168.0.1X
```

The original notes use `PC11`, while the screenshot shows `PC-11`.

The client date and time were changed, but the final value was not
recorded in the source documentation.

## Why configure DNS before joining?

The client needs to resolve the Active Directory domain.

The primary DNS server was therefore configured as:

``` text
192.168.0.10
```

This points the client to the Domain Controller's DNS service.

------------------------------------------------------------------------

# 8. Join Windows 11 to the Domain

The domain joining process followed these steps.

## Step 1 --- Configure the Client Network

Configure the Windows 11 network adapter and make sure the DNS server
is:

``` text
192.168.0.10
```

## Step 2 --- Open System Properties

Use:

``` text
sysdm.cpl
```

This opens System Properties.

## Step 3 --- Change the Domain

Select the computer name/domain change option and choose:

``` text
Domain
```

Enter:

``` text
test.local
```

## Step 4 --- Provide Domain Credentials

Provide appropriate domain credentials when Windows requests
authentication.

## Step 5 --- Restart the Client

Restart Windows 11 to complete the domain membership process.

## Step 6 --- Sign In

After restarting, sign in using a domain account.

## Step 7 --- Verify Active Directory

Open:

``` text
dsa.msc
```

and verify that:

``` text
PC-11
```

appears in Active Directory.

### Result

The documented screenshot shows:

``` text
PC-11.test.local
```

and the computer object appears in the Computers container.

------------------------------------------------------------------------

# 9. Verify the Domain Environment

The project used several checks to verify the environment.

### Check 1 --- Domain Membership

System Properties shows:

``` text
test.local
```

### Check 2 --- Active Directory Computer Object

In Active Directory Users and Computers:

``` text
Computers
  └── PC-11
```

### Check 3 --- Domain Authentication

A domain account can be used to sign in to the domain-joined Windows 11
client.

### Check 4 --- Group Policy

On the client:

``` cmd
gpupdate /force
```

Then test the expected behavior of the applicable policies.

------------------------------------------------------------------------

# 10. Configure Group Policy

Group Policy was used to centrally manage Windows settings.

Instead of configuring every workstation manually, policies can be
configured centrally and linked to the required domain or OU.

## User Configuration

User Configuration applies settings to the user.

Examples in this project include restrictions related to:

-   Control Panel
-   Task Manager
-   Desktop behavior
-   Other user experience settings

## Computer Configuration

Computer Configuration applies settings to the machine.

The project uses Computer Configuration for security settings and policy
elements such as the password policy.

The documented password policy path is:

``` text
Computer Configuration
    ↓
Windows Settings
    ↓
Security Settings
    ↓
Account Policies
```

------------------------------------------------------------------------

# 11. Configure Department Policies

The lab uses both domain-level policies and department-specific
policies.

## Domain-Level GPOs

The documented domain-level GPOs include:

``` text
Default Domain Policy
FULL-Control-Panel
ADD-Local-Admin
ADD-IT-Group
Allow-ICMP
PASSWORD-Policy
Disable-Control-Panel
Disable-RUN
Disable-CMD
```

The project documentation records 19 enabled GPOs in the domain console,
including restriction, Control Panel, desktop, access, security, and
default policy objects.

Exact GPO names are preserved from the console, including spellings such
as:

``` text
Enable-Conrol_panel
Creat-ShourtCut
```

------------------------------------------------------------------------

## 11.1 All Users / Domain-Level Policy

The documented linked GPO order includes:

  Order   GPO                     Enforced Link
  ------- ----------------------- ---------------
  1       Default Domain Policy   Yes
  2       FULL-Control-Panel      Yes
  3       ADD-Local-Admin         Yes
  4       ADD-IT-Group            Yes
  5       Allow-ICMP              Yes
  6       PASSWORD-Policy         Yes
  7       Disable-Control-Panel   No
  8       Disable-RUN             No
  9       Disable-CMD             No

The project notes clarify that GPOs with `Link = No` exist but are not
applied at the domain level.

### FULL-Control-Panel

This policy is documented as enforced.

Its purpose is to control Control Panel behavior and prevent lower-level
departmental policies from overriding the enforced configuration.

### ADD-Local-Admin

Runs the documented local administrator creation script.

### ADD-IT-Group

Adds the domain `IT-Group` to the local Administrators group.

### Allow-ICMP

Allows ICMP traffic through the client firewall so the machine can
respond to ping.

### PASSWORD-Policy

Contains the documented password and account lockout controls.

------------------------------------------------------------------------

# 11.2 HR Group Policy

The HR OU has the following documented GPOs:

``` text
Remove-Clock
Disable-Properties
Disable-USB
Enable-Conrol_panel
Disable-Task-Manager
Creat-ShourtCut
```

### Remove-Clock

Intended result:

``` text
Clock is removed from the taskbar.
```

### Disable-Properties

Restricts Properties dialogs.

The exact internal policy setting was not visible in the source
screenshots.

### Disable-USB

Intended to block USB storage.

### Enable-Conrol_panel

Allows Control Panel access for HR according to the GPO name and project
documentation.

### Disable-Task-Manager

Blocks access to Task Manager.

### Creat-ShourtCut

Creates a shortcut.

The exact target of the shortcut was not documented.

### Testing

Sign in as an HR user, run:

``` cmd
gpupdate /force
```

Then verify each expected behavior.

------------------------------------------------------------------------

# 11.3 Sales Group Policy

The Sales OU has:

``` text
Disable-USB
Disable-Task-Manager
Disable-Properties
Remove-Clock
Calc-ON
```

### Expected behavior

-   USB storage is restricted
-   Task Manager is restricted
-   Properties access is restricted
-   The taskbar clock is removed
-   Calculator automatically launches

The policies are tested by signing in as a Sales user and checking each
behavior after:

``` cmd
gpupdate /force
```

------------------------------------------------------------------------

# 11.4 Finance Group Policy

The Finance OU has:

``` text
Calc-ON
Disable-Properties
Disable-USB
Remove-Clock
Disable-Task-Manager
```

### Expected behavior

-   Calculator launches automatically
-   Properties access is restricted
-   USB storage is restricted
-   The taskbar clock is removed
-   Task Manager is restricted

The policies are tested by signing in as a Finance user and checking the
expected behavior.

------------------------------------------------------------------------

# 12. Configure Scripts

The lab uses scripts through Group Policy to demonstrate centralized
automation.

## 12.1 Calculator Script

The documented script is:

``` text
calc.exe
```

The GPO is:

``` text
Calc-ON
```

It is linked to:

``` text
Sales
Finance
```

### Purpose

Calculator auto-launch is a simple and harmless way to verify that a
Group Policy script is being executed on the client.

The original documentation does not explicitly state whether this was
configured as a startup or logon script.

------------------------------------------------------------------------

## 12.2 Create a Local Administrator

The documented commands are:

``` cmd
net user itadmin PASSWORD /add
net localgroup Administrators itadmin /add
```

### What they do

The first command creates a local user:

``` text
itadmin
```

The second command adds that user to the local:

``` text
Administrators
```

group.

The configuration is associated with:

``` text
ADD-Local-Admin
```

### Important Security Note

This is a **lab-only demonstration**.

A plaintext password inside a script is unsafe in a production
environment. Scripts stored in locations such as SYSVOL can potentially
be readable by authenticated domain users.

A production environment should use a managed local administrator
password solution rather than embedding a shared password in a script.

------------------------------------------------------------------------

## 12.3 Add IT-Group to Local Administrators

The documented command is:

``` cmd
net localgroup administrators IT-Group /add
```

The related GPO is:

``` text
ADD-IT-Group
```

### Purpose

This demonstrates group-based administration.

Instead of manually adding every IT employee to every workstation, the
domain group can be used to manage local administrator membership
centrally.

### Security consideration

In this lab, `IT-Group` receives local administrator rights on clients
within the GPO scope.

In production, this should be limited to the machines that actually
require it and reviewed regularly according to least-privilege
principles.

------------------------------------------------------------------------

# 13. Configure Password and Account Lockout Policies

The password policy was configured through:

``` text
Group Policy Management Editor
    ↓
Computer Configuration
    ↓
Windows Settings
    ↓
Security Settings
    ↓
Account Policies
```

The GPO is:

``` text
PASSWORD-Policy
```

and is linked at:

``` text
test.local
```

## Password Policy

  Setting                    Value
  -------------------------- -----------------------
  Enforce password history   1 password remembered
  Maximum password age       30 days
  Minimum password age       0 days
  Minimum password length    8 characters
  Complexity requirements    Enabled

## Account Lockout Policy

  Setting                               Value
  ------------------------------------- --------------------
  Account lockout threshold             5 invalid attempts
  Account lockout duration              10 minutes
  Reset lockout counter after           10 minutes
  Allow Administrator account lockout   Enabled

### Purpose

The lockout configuration is intended to slow repeated failed
authentication attempts.

For example, after five invalid logon attempts, the account is locked
for ten minutes according to the configured policy.

### Lab review

The project documentation notes that a history value of `1`, minimum age
of `0`, and a 30-day maximum age are configurations that would be worth
reviewing against current production guidance.

------------------------------------------------------------------------

# 14. Test the Policies

Testing was performed by signing in as users from affected OUs and
checking the expected behavior.

## General testing process

1.  Sign in as the target departmental user.
2.  Run:

``` cmd
gpupdate
```

or:

``` cmd
gpupdate /force
```

3.  Verify that the policy is applied.
4.  Test the expected restriction or action.
5.  Compare the workstation before and after the policy.
6.  If the policy does not apply, check DNS, OU placement, GPO link
    status, and refresh the policy.

## Examples

### HR

Test:

-   Control Panel
-   Task Manager
-   USB
-   Properties
-   Clock
-   Shortcut

### Sales

Test:

-   USB
-   Task Manager
-   Properties
-   Clock
-   Calculator

### Finance

Test:

-   Calculator
-   Properties
-   USB
-   Clock
-   Task Manager

------------------------------------------------------------------------

# 15. Administrative Commands

The following Windows tools were used during the lab.

  -----------------------------------------------------------------------
  Command                 Purpose                 Main Use
  ----------------------- ----------------------- -----------------------
  `ncpa.cpl`              Network Connections     Configure IP / DNS

  `sysdm.cpl`             System Properties       Computer name / domain
                                                  membership

  `dsa.msc`               Active Directory Users  Users, groups, OUs,
                          and Computers           computers

  `gpedit.msc`            Local Group Policy      Local policy
                          Editor                  configuration

  `gpmc.msc`              Group Policy Management Domain GPO management
                          Console                 

  `gpupdate`              Update Group Policy     Apply policy changes

  `gpupdate /force`       Force Group Policy      Reapply all policies
                          refresh                 
  -----------------------------------------------------------------------

> The original notes contained `cnpa.cpl`, but the standard Windows
> command is `ncpa.cpl`, which is used here.

------------------------------------------------------------------------

# 16. Troubleshooting

The following are common troubleshooting checks documented for this
architecture. They are general considerations and are not presented as
problems that necessarily occurred during the lab.

## Client cannot join the domain

### Possible cause

Incorrect DNS or network connectivity.

### Check

Open:

``` text
ncpa.cpl
```

Verify that the client DNS is:

``` text
192.168.0.10
```

### Solution

Correct DNS first, verify connectivity, then retry the domain join
through:

``` text
sysdm.cpl
```

------------------------------------------------------------------------

## Group Policy is not applying

### Possible causes

-   Wrong OU
-   Disabled GPO link
-   Policy has not been refreshed

### Check

Use:

``` text
gpmc.msc
```

Check the GPO link and OU.

Then run:

``` cmd
gpupdate /force
```

------------------------------------------------------------------------

## Computer does not appear in Active Directory

### Possible causes

-   Domain join did not finish
-   Wrong container
-   Join process was interrupted

### Check

Open:

``` text
dsa.msc
```

and check the Computers container.

### Solution

If required, rejoin the domain through:

``` text
sysdm.cpl
```

then move the computer object to the correct OU.

------------------------------------------------------------------------

## User cannot authenticate

Check:

-   Username and password
-   Account status
-   Password policy
-   Account lockout status
-   Domain connectivity

The documented account lockout duration is ten minutes.

------------------------------------------------------------------------

## Policy change is not visible

The client may not have refreshed its policy yet.

Run:

``` cmd
gpupdate
```

or:

``` cmd
gpupdate /force
```

A sign-out/sign-in may also be required for some user settings.

------------------------------------------------------------------------

# 17. Security Considerations

This project demonstrates several important Windows security concepts.

## Centralized Identity Management

Active Directory provides one central directory for accounts.

An account can be managed centrally instead of maintaining separate
local accounts on every workstation.

## Least Privilege

Administrative rights should only be granted where they are required.

The lab demonstrates why group-based administration is useful, but also
shows how easily broad administrator rights can become excessive.

## Group-Based Administration

The `IT-Group` example demonstrates managing administrator membership
through a domain group instead of adding individual users manually.

## Password Security

The lab includes:

-   Password history
-   Minimum password length
-   Complexity
-   Password expiration
-   Account lockout

## Policy Enforcement

The lab also demonstrates enforced GPO behavior, including:

``` text
FULL-Control-Panel
```

which is documented as enforced.

## Department Separation

Departmental OUs allow policies to be applied differently to:

``` text
HR
Sales
Finance
IT
```

------------------------------------------------------------------------

# Lab Configuration vs Production Practice

Some configurations were intentionally simple for demonstration.

  -----------------------------------------------------------------------
  Lab Configuration                   Production Consideration
  ----------------------------------- -----------------------------------
  Plaintext password in `itadmin`     Never store passwords directly in
  script                              scripts

  Domain-wide local admin creation    Restrict local admin management to
                                      required systems

  `IT-Group` administrator access     Apply least privilege and review
                                      membership

  Direct department GPO deployment    Test GPOs in a pilot OU first

  Password history = 1                Review against current
                                      organizational security guidance

  Minimum password age = 0            Review according to current policy
                                      requirements

  30-day password expiration          Review against current security
                                      guidance and authentication
                                      architecture
  -----------------------------------------------------------------------

These differences are important because a home lab is useful for
learning, but production environments need additional controls and
careful change management.

------------------------------------------------------------------------

# 18. What I Learned

Building this lab gave me practical experience with several areas.

## Windows Server

-   Installing Windows Server 2025
-   Server configuration
-   Static network configuration
-   Preparing a Domain Controller

## Active Directory

-   Domain Controllers
-   Users
-   Groups
-   Organizational Units
-   Domain management

## DNS

-   Client DNS configuration
-   DNS dependency for Active Directory
-   Domain discovery

## Group Policy

-   Creating GPOs
-   Linking GPOs
-   Department-specific policies
-   Policy inheritance
-   Enforced policies
-   Policy testing
-   Troubleshooting

## Windows Clients

-   Windows 11 configuration
-   Domain joining
-   Domain authentication
-   Centralized management

## Automation

-   Windows administrative commands
-   Group Policy scripts
-   Basic administrative automation

------------------------------------------------------------------------

# Project Reflection

Before building this lab, it was easy to think of each Windows computer
as an independent machine.

The project made the centralized model much clearer.

Instead of configuring every PC separately, the Domain Controller can
manage identity and policies centrally.

The relationship can be visualized as:

``` text
User
  ↓
Group
  ↓
OU
  ↓
Group Policy
  ↓
Windows Client
```

One policy change on the server can change the behavior of a client
after the policy is applied.

The lab also showed the other side of centralized administration: a
badly scoped policy or excessive administrative permission can affect
many machines at once.

That is one of the main lessons I took from the project.

------------------------------------------------------------------------

# 19. Future Improvements

The following are future ideas only. They are not part of the current
implementation.

## Add More Windows Clients

Create additional Windows 11 clients to simulate a larger organization.

## Add a File Server

Introduce a dedicated Windows file server and practice:

-   File shares
-   NTFS permissions
-   Share permissions
-   Department access

## Advanced Group Policy

Create more advanced policies for:

-   Windows security
-   User restrictions
-   Workstation hardening
-   Auditing

## Windows Auditing

Enable and monitor Windows security events.

## Centralized Logging

Forward Windows security logs to a central platform.

## SIEM Integration

Integrate Wazuh and use the Windows domain environment as a source of
security telemetry.

## Endpoint Security Monitoring

Deploy agents to monitor Windows activity and security events.

## Active Directory Attack and Defense

Build controlled lab scenarios to understand how common Active Directory
attacks generate logs and telemetry.

## PowerShell Automation

Replace repetitive manual administration with controlled PowerShell
automation.

## Backup and Recovery

Create backup and recovery scenarios for:

-   Active Directory
-   Server configuration
-   Important data

------------------------------------------------------------------------

# Project Summary

This project built a centralized Windows domain environment using:

-   Windows Server 2025
-   Windows 11 Pro
-   Active Directory Domain Services
-   DNS
-   Organizational Units
-   Security Groups
-   Group Policy
-   Windows scripts
-   Password and account lockout policies

The final lab used:

``` text
Domain Controller
    ADMIN-Server
    192.168.0.10
          |
       test.local
          |
    +-----+-----+-----+
    |     |     |     |
   IT    HR  Finance Sales
          |
       PC-11
    Windows 11 Pro
```

The main outcome was practical experience with centralized identity
management, Windows administration, Group Policy, DNS, domain joining,
policy testing, and basic Windows security.

The project also highlighted an important part of real security work:
configurations that are acceptable for a controlled lab may need
significant hardening before they are suitable for production.

------------------------------------------------------------------------

## Author

**Mohamed Ali Mohamed**

Cybersecurity Graduate \| SOC Analyst

-   LinkedIn: https://www.linkedin.com/in/mohamed-ali-481a70387
-   Email: mo5665ali@gmail.com

------------------------------------------------------------------------

## Disclaimer

This project is a personal educational home lab.

All security testing and administrative configurations were performed
for learning and demonstration purposes in a controlled virtual
environment.
