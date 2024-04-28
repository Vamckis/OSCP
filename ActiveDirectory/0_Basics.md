Active Directory (AD)
Domain Controller (DC)
Active Directory Domain Service (AD DS)
Organizational unit (OU) - Used to apply polocies on users and machines
Security Groups - Used to grant privileges to users and machines
GPO (Group Policy Objects) - collection of settings that can be applied to OUs.


### Types of Accounts 
- Users 
  - People 
  - Services : will have privilegs required to run specific service.
- Machines / security principals : For each compuet joining AD will have a machine onject created
- Security Groups
  - Domain Admins
  - Server Operators
  - Backup Operators
  - Accout operators
  - Domain Users
  - Domain Computers
  - Domain Controllers : Includes all DCs on 


### Delegation 
- Process of granting privileges to a user
#### Windows Machine
- Flow: 'Active Directory Users and Computers' --> Right click on a OU (Sales) --> Delegate Control --> Add --> Check Names (phillip) --> Check box the  privilege to reset password --> Apply
- To reset password: C:\Users\phillip>Set-ADAccountPassword sophie -Reset -NewPassword (Read-Hpst -AsSecureString -Prompt 'New Password') -Verbose 

### Edit GPO:
- Flow: 'Group Policy Mangement' --> right click a policy --> edit --> GPM Editor  --> Polocies --> Windows settings --> Security Settings --> Account Policy
- GPO are distributed to network via network share called SYSVOL, it is stored in DC.
- To get all GPU polocies take effect: gpupdate /force 

### Windows Authentication Methods
- All credentails are stored in Domain Controllers
- Two protocaols for Authentication:
  - Kerberos: Mostly used in all windows
  - NetNTLM: Old Protocol, kept for compatability purposes
#### Kerberos
- KDC (Key Distribution Center) - Accepts username, Timestamp encrypted with Pasword in Domain Controller
- TGT (Ticket Granting Ticket) - KDC send TGT+Session Key, 
  - TGT allows users to request more tickets to access services. It acts as a form of SSO login for future ticket requests. 
  - TGT is encrypted with krbtgt password hash, so user wont be able to retrieve the contents.This hash alos includes session key.
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/4b64ac88-e9f7-496c-96ba-8e0b059b485d)
  
- When user want to access a service,They will use TGT to ask KDC for TGS (Ticket Granting Service).
  - TGS grants access to specific service.
  - For TGS, user sends username + timestamp + TGT + Service principalname.
- In response KDS sends TGS + Service Session Key
  - TGS is encrypted with service owner hash.
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/0d9975e1-43f6-465c-a138-c083eeacb94d)
- Finally, user sends TGS to authenticate and establish connection with desired service. Service will decrypt TGS and validate service session key.
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/3b4145e1-9523-47a1-a840-a1e176870d99)

#### NetNTLM Authentication
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/e93ab9a5-b276-4759-a244-cfaee740982b)
- the user's password (or hash) is never transmitted through the network for security.
- Above described process applies when using a domain account. If a local account is used, the server can verify the response to the challenge itself without requiring interaction with the domain controller since it has the password hash stored locally on its SAM.

### Trees, Forests
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/1f9910ca-6771-4893-8d1c-3b3ef98b9a1c)

![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/d15fd33e-9e0d-437b-8d46-f75c1d4c4c7e)

- trust relationship: In a one-way trust, if Domain AAA trusts Domain BBB, this means that a user on BBB can be authorised to access resources on AAA:
![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/68c5b7a9-c56b-4e9a-ad14-80f6fc670c6e)
