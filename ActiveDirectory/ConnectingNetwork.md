![image](https://github.com/Vamckis/OSCP-Preparation/assets/71128825/65beb369-6cec-4bf1-ac81-585e4e9b7028)

- To configure DNS in host VM of the lab setup:
``` systemd-resolve --interface breachad --set-dns 10.200.26.101 --set-domain za.tryhackme.com ```
``` nslookup 10.200.26.101.za.tryhackme.com ```
