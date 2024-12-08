
![image](https://github.com/user-attachments/assets/288515b2-d7ad-4f6b-9fed-c7523cd6f437)

E.g: Confluence 01 is vulnwerable to RCE - CVE-2022-26134
- Payload to exploit Confluence01 from Kali
  - ![image](https://github.com/user-attachments/assets/85174bde-88e4-47b1-bd32-e295b4a05339)
  - ![image](https://github.com/user-attachments/assets/166b937a-565f-4b23-8de5-031c45ceef8e)
- Confluence have 2 network interfaces (ens192, ens224):
  - ![image](https://github.com/user-attachments/assets/1b754b5d-9567-4ca0-8312-6fb74f1bd007)
- ip route will shows routes:
  - ![image](https://github.com/user-attachments/assets/65b9adc4-cb59-4eca-afc1-180443990e06)
-  Found DB anme and pwd in a config file:
  -  ![image](https://github.com/user-attachments/assets/086125ed-e01e-447b-bc13-f0ed5036c1db)
  -  We can use credntails, But we dont have postgresql web client to access,
  -  We have psql in kali, but we cannot connect directly to DB. As DB is only accesseble from conf01 machine
  -  There is no Firewall between Conflu01 and DB, so we can connecte from Kali machine usinf port forwarding.

### Port forwarding using socat
- Conf01 should listen to port in WAN and forward all packets them to Pg DB in internal subnet.
- ![image](https://github.com/user-attachments/assets/4c6ac43b-f141-4b93-90cb-2ec532e8372b)
  - ![image](https://github.com/user-attachments/assets/79e18b0d-792d-4c01-a1bd-36e953ef7906)
  - ![image](https://github.com/user-attachments/assets/1635790f-dc34-4c12-82b8-2e9b1fb77bab)
  - List all tables
  - ![image](https://github.com/user-attachments/assets/5c79ca7d-d020-47b1-a496-6511e13d7ca0)
  - Lets see cwda user table, contains credentails for all confluence users
  - Connect to DB as below:
  - ![image](https://github.com/user-attachments/assets/5beefc92-c17e-40cd-b4e0-bb1fc05cc52c)
  - To review al details from cwd_user:
  - ![image](https://github.com/user-attachments/assets/9b85a7e6-7319-4a89-ab4b-c2b55d2b6071)
  - ![image](https://github.com/user-attachments/assets/e3fa4c3f-0bce-433a-ac63-823e47dd7ef7)
  - Use hashcat to decode the hashes
  - ![image](https://github.com/user-attachments/assets/902f0669-ed38-4097-aa55-33bb9ff16035)
  - ![image](https://github.com/user-attachments/assets/955a8872-18ab-4de8-bc90-350f47c69563)





