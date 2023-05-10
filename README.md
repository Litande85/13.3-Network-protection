# Домашнее задание к занятию 13.3. «Защита сети» - `Елена Махота`

- [Ответ к Заданию 1](#1)
- [Ответ к Заданию 2](#2)

------

### Подготовка к выполнению заданий

1. Подготовка защищаемой системы:

- установите **Suricata**,
- установите **Fail2Ban**.

2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

------

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### *<a name = "1"> Ответ к Заданию 1 </a>*

Установка `suricata` Debian 11

```bash
sudo apt-get install suricata
# sudo suricata-update
sudo systemctl enable suricata
sudo systemctl start suricata.service
sudo systemctl status suricata.service
```
![suricata](img/img%202023-05-10%20210535.png)



```bash
sudo nano /etc/suricata/suricata.yaml
# меняем значение параметра EXTERNAL_NET на "any"
sudo systemctl restart suricata
#Лог-файлы:
sudo tail /var/log/suricata/suricata.log
sudo tail /var/log/suricata/stats.log
#запуск
sudo suricata -c /etc/suricata/suricata.yaml -s signatures.rules -i enp0s9
#файл логов с предупреждениями
sudo tail -f /var/log/suricata/fast.log
```

Разведка

```bash
┌──(kali㉿makhota-kali)-[~]
└─$ ping 192.168.56.110 -c1     
PING 192.168.56.110 (192.168.56.110) 56(84) bytes of data.
64 bytes from 192.168.56.110: icmp_seq=1 ttl=64 time=0.414 ms

--- 192.168.56.110 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.414/0.414/0.414/0.000 ms
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo nmap -sA 192.168.56.110
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 15:18 EDT
Nmap scan report for 192.168.56.110
Host is up (0.0018s latency).
All 1000 scanned ports on 192.168.56.110 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
MAC Address: 08:00:27:05:29:C1 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 13.49 seconds
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo nmap -sT 192.168.56.110
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 15:19 EDT
Nmap scan report for 192.168.56.110
Host is up (0.0053s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
873/tcp  open  rsync
3389/tcp open  ms-wbt-server
MAC Address: 08:00:27:05:29:C1 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 13.66 seconds
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo nmap -sS 192.168.56.110
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 15:20 EDT
Nmap scan report for 192.168.56.110
Host is up (0.00068s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
873/tcp  open  rsync
3389/tcp open  ms-wbt-server
MAC Address: 08:00:27:05:29:C1 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 13.65 seconds
                                                                                                                                                                      
┌──(kali㉿makhota-kali)-[~]
└─$ sudo nmap -sV 192.168.56.110
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 15:20 EDT
Nmap scan report for 192.168.56.110
Host is up (0.0058s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp   open  http          Apache httpd 2.4.54 ((Debian))
873/tcp  open  rsync         (protocol version 31)
3389/tcp open  ms-wbt-server xrdp
MAC Address: 08:00:27:05:29:C1 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.28 seconds

```
Изменения в логах `suricata`

```bash
makhota@nodeone:~$ sudo tail /var/log/suricata/suricata.log
01~10/5/2023 -- 22:15:35 - <Info> - Using unix socket file '/var/run/suricata-command.socket'
10/5/2023 -- 22:15:35 - <Notice> - all 1 packet processing threads, 4 management threads initialized, engine started.
10/5/2023 -- 22:15:35 - <Info> - All AFP capture threads are running.
10/5/2023 -- 22:17:03 - <Notice> - This is Suricata version 6.0.1 RELEASE running in SYSTEM mode
10/5/2023 -- 22:17:03 - <Info> - CPUs/cores online: 1
10/5/2023 -- 22:17:03 - <Info> - Found an MTU of 1500 for 'enp0s9'
10/5/2023 -- 22:17:03 - <Info> - Found an MTU of 1500 for 'enp0s9'
10/5/2023 -- 22:17:04 - <Info> - fast output device (regular) initialized: fast.log
10/5/2023 -- 22:17:04 - <Info> - eve-log output device (regular) initialized: eve.json
10/5/2023 -- 22:17:04 - <Info> - stats output device (regular) initialized: stats.log
makhota@nodeone:~$ sudo tail -f /var/log/suricata/fast.log
05/10/2023-22:18:07.674725  [**] [1:2100366:8] GPL ICMP_INFO PING *NIX [**] [Classification: Misc activity] [Priority: 3] {ICMP} 192.168.56.108:8 -> 192.168.56.110:0
05/10/2023-22:19:29.083667  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:41852 -> 192.168.56.110:3306
05/10/2023-22:19:29.109060  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:45902 -> 192.168.56.110:5901
05/10/2023-22:19:29.171454  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:41836 -> 192.168.56.110:1521
05/10/2023-22:19:29.190927  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:46540 -> 192.168.56.110:1433
05/10/2023-22:19:29.201715  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:35578 -> 192.168.56.110:5432
05/10/2023-22:19:29.271032  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:32802 -> 192.168.56.110:5810
05/10/2023-22:20:17.997212  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:3306
05/10/2023-22:20:17.997211  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:3306
05/10/2023-22:20:18.076775  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:5911
05/10/2023-22:20:18.089011  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:5432
05/10/2023-22:20:18.089010  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:5432
05/10/2023-22:20:18.118340  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:1521
05/10/2023-22:20:18.118340  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:1521
05/10/2023-22:20:18.145441  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:5815
05/10/2023-22:20:18.156132  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:1433
05/10/2023-22:20:18.156133  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:50434 -> 192.168.56.110:1433
05/10/2023-22:21:05.094905  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:3306
05/10/2023-22:21:05.094906  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:3306
05/10/2023-22:21:05.149054  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:5915
05/10/2023-22:21:05.180812  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:5432
05/10/2023-22:21:05.180813  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:5432
05/10/2023-22:21:05.259254  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:1521
05/10/2023-22:21:05.267521  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:1433
05/10/2023-22:21:05.249278  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted 
Information Leak] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:5802
05/10/2023-22:21:05.259255  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:1521
05/10/2023-22:21:05.267522  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.56.108:58436 -> 192.168.56.110:1433
05/10/2023-22:21:11.882456  [**] [1:2001330:10] ET INFO RDP - Response To External Host [**] [Classification: Misc activity] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:33834
05/10/2023-22:21:11.882455  [**] [1:2001330:10] ET INFO RDP - Response To External Host [**] [Classification: Misc activity] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:33834
05/10/2023-22:21:16.878356  [**] [1:2036252:1] ET SCAN RDP Connection Attempt from Nmap [**] [Classification: Detection of a Network Scan] [Priority: 3] {TCP} 192.168.56.108:33834 -> 192.168.56.110:3389
05/10/2023-22:21:16.878363  [**] [1:2036252:1] ET SCAN RDP Connection Attempt from Nmap [**] [Classification: Detection of a Network Scan] [Priority: 3] {TCP} 192.168.56.108:33834 -> 192.168.56.110:3389
05/10/2023-22:21:16.880301  [**] [1:2001330:10] ET INFO RDP - Response To External Host [**] [Classification: Misc activity] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:33834
05/10/2023-22:21:16.880300  [**] [1:2001330:10] ET INFO RDP - Response To External Host [**] [Classification: Misc activity] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:33834
05/10/2023-22:21:16.923114  [**] [1:2230002:1] SURICATA TLS invalid record type [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.923114  [**] [1:2230010:1] SURICATA TLS invalid record/traffic [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.923114  [**] [1:2260000:1] SURICATA Applayer Mismatch protocol both directions [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.923105  [**] [1:2230002:1] SURICATA TLS invalid record type [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.923105  [**] [1:2230010:1] SURICATA TLS invalid record/traffic [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.923105  [**] [1:2260000:1] SURICATA Applayer Mismatch protocol both directions [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:40196 -> 192.168.56.110:3389
05/10/2023-22:21:16.931979  [**] [1:2230002:1] SURICATA TLS invalid record type [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:40196
05/10/2023-22:21:16.931979  [**] [1:2230010:1] SURICATA TLS invalid record/traffic [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:40196
05/10/2023-22:21:16.931987  [**] [1:2230002:1] SURICATA TLS invalid record type [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:40196
05/10/2023-22:21:16.931987  [**] [1:2230010:1] SURICATA TLS invalid record/traffic [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:3389 -> 192.168.56.108:40196
05/10/2023-22:21:16.980160  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41986 -> 192.168.56.110:80    
05/10/2023-22:21:16.980166  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41986 -> 192.168.56.110:80    
05/10/2023-22:21:16.980166  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41986 -> 192.168.56.110:80
05/10/2023-22:21:16.980160  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41986 -> 192.168.56.110:80
05/10/2023-22:21:16.980173  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41998 -> 192.168.56.110:80    
05/10/2023-22:21:16.980172  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41998 -> 192.168.56.110:80    
05/10/2023-22:21:16.980173  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41998 -> 192.168.56.110:80
05/10/2023-22:21:16.980172  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:41998 -> 192.168.56.110:80
05/10/2023-22:21:16.983372  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42008 -> 192.168.56.110:80    
05/10/2023-22:21:16.983372  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42008 -> 192.168.56.110:80
05/10/2023-22:21:16.983371  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42008 -> 192.168.56.110:80    
05/10/2023-22:21:16.983371  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42008 -> 192.168.56.110:80
05/10/2023-22:21:16.990448  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42020 -> 192.168.56.110:80    
05/10/2023-22:21:16.990448  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42020 -> 192.168.56.110:80
05/10/2023-22:21:16.990447  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42020 -> 192.168.56.110:80    
05/10/2023-22:21:16.990447  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.56.108:42020 -> 192.168.56.110:80
```

Трафик при сканировании `nmap -sA` `suricata` не посчитала подозрительным, присвоив ему приоритет 3.
Трафик при сканировании `nmap -sS` `suricata`  посчитала подозрительным, потенициально вредоносным, присвоив ему приоритет 2.
Трафик при сканировании `nmap -sT` `suricata` не посчитала подозрительным, присвоив ему приоритет 3.
Трафик при сканировании `nmap -sV` `suricata`   классифицировала как web-атаку, посчитала вредоносным, присвоив ему приоритет 1.

`fail2ban` не зафиксировал трафик при сканировании `nmap`, изменений в логах не было.

---

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.



*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### *<a name = "2"> Ответ к Заданию 2 </a>*

1. Атака на подбор пароля для службы SSH

![Alt text](img/img%202023-05-10%20224919.png)

```bash
makhota@nodeone:~$ sudo tail -f /var/log/suricata/fast.log                                                          
[sudo] пароль для makhota: 
05/10/2023-22:47:55.221913  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:22 -> 192.168.56.108:42512
05/10/2023-22:47:55.221912  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:22 -> 192.168.56.108:42512
05/10/2023-22:47:55.235225  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:42546 -> 192.168.56.110:22
05/10/2023-22:47:55.235224  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.108:42546 -> 192.168.56.110:22
05/10/2023-22:47:59.057488  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.56.108:42574 -> 192.168.56.110:22
05/10/2023-22:47:59.057485  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.56.108:42574 -> 192.168.56.110:22
05/10/2023-22:47:59.234264  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.56.108:42594 -> 192.168.56.110:22
05/10/2023-22:47:59.234263  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.56.108:42594 -> 192.168.56.110:22
05/10/2023-22:47:59.333794  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:22 -> 192.168.56.108:42614
05/10/2023-22:47:59.333796  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.56.110:22 -> 192.168.56.108:42614
```

2. Установка `sudo apt install fail2ban`

![Alt text](img/img%202023-05-10%20230113.png)


Атака на подбор пароля для службы SSH

![Alt text](img/img%202023-05-10%20231412.png)

```bash
makhota@nodeone:~$ sudo tail /var/log/auth.log
May 10 23:10:52 localhost sudo: pam_unix(sudo:session): session opened for user root(uid=0) by makhota(uid=1000)
May 10 23:11:39 localhost sudo: pam_unix(sudo:session): session closed for user root
May 10 23:11:43 localhost polkitd(authority=local): Registered Authentication Agent for unix-process:6889:825699 (system bus name :1.138 [/usr/bin/pkttyagent --notify-fd 5 --fallback], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale ru_RU.UTF-8)
May 10 23:11:46 localhost polkitd(authority=local): Operator of unix-process:6889:825699 successfully authenticated 
as unix-user:makhota to gain ONE-SHOT authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.139 [systemctl restart fail2ban.service] (owned by unix-user:makhota)
May 10 23:11:46 localhost polkitd(authority=local): Unregistered Authentication Agent for unix-process:6889:825699 (system bus name :1.138, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale ru_RU.UTF-8) (disconnected from bus)
May 10 23:11:57 localhost sudo:  makhota : TTY=pts/2 ; PWD=/home/makhota ; USER=root ; COMMAND=/usr/bin/tail /var/log/auth.log
May 10 23:11:57 localhost sudo: pam_unix(sudo:session): session opened for user root(uid=0) by makhota(uid=1000)    
May 10 23:11:57 localhost sudo: pam_unix(sudo:session): session closed for user root
May 10 23:12:51 localhost sudo:  makhota : TTY=pts/2 ; PWD=/home/makhota ; USER=root ; COMMAND=/usr/bin/tail /var/log/auth.log
May 10 23:12:51 localhost sudo: pam_unix(sudo:session): session opened for user root(uid=0) by makhota(uid=1000)    

makhota@nodeone:~$ sudo cat /var/log/fail2ban.log
--------------------------------------------------  
2023-05-10 23:11:47,352 fail2ban.server         [6900]: INFO    Starting Fail2ban v0.11.2
2023-05-10 23:11:47,353 fail2ban.observer       [6900]: INFO    Observer start...
2023-05-10 23:11:47,361 fail2ban.database       [6900]: INFO    Connected to fail2ban persistent database '/var/lib/fail2ban/fail2ban.sqlite3'
2023-05-10 23:11:47,362 fail2ban.jail           [6900]: INFO    Creating new jail 'sshd'
2023-05-10 23:11:47,396 fail2ban.jail           [6900]: INFO    Jail 'sshd' uses pyinotify {}
2023-05-10 23:11:47,416 fail2ban.jail           [6900]: INFO    Initiated 'pyinotify' backend
2023-05-10 23:11:47,422 fail2ban.filter         [6900]: INFO      maxLines: 1
2023-05-10 23:11:47,477 fail2ban.filter         [6900]: INFO      maxRetry: 5
2023-05-10 23:11:47,478 fail2ban.filter         [6900]: INFO      findtime: 600
2023-05-10 23:11:47,478 fail2ban.actions        [6900]: INFO      banTime: 600
2023-05-10 23:11:47,478 fail2ban.filter         [6900]: INFO      encoding: UTF-8
2023-05-10 23:11:47,480 fail2ban.filter         [6900]: INFO    Added logfile: '/var/log/auth.log' (pos = 56548, hash = 8ef661b06198e6098cd0aabf834475fc25dd71a6)
2023-05-10 23:11:47,488 fail2ban.jail           [6900]: INFO    Jail 'sshd' started
2023-05-10 23:11:47,515 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:36   
2023-05-10 23:11:47,516 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,517 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,518 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,518 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,519 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,520 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,521 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,522 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,523 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,525 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,525 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,526 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,527 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,527 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,528 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,529 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38
2023-05-10 23:11:47,531 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,534 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,535 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,535 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:38   
2023-05-10 23:11:47,537 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,538 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,538 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,539 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,539 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,541 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,541 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,542 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,543 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,543 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,544 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,544 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,545 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:39   
2023-05-10 23:11:47,546 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:40   
2023-05-10 23:11:47,547 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:40   
2023-05-10 23:11:47,550 fail2ban.filter         [6900]: INFO    [sshd] Found 192.168.56.108 - 2023-05-10 23:08:42   
2023-05-10 23:11:47,688 fail2ban.actions        [6900]: NOTICE  [sshd] Ban 192.168.56.108
2023-05-10 23:18:40,444 fail2ban.actions        [6900]: NOTICE  [sshd] Unban 192.168.56.108
```

Комментарии результата:

`suricata` зафиксировала, но не предотвратила утечку информации, пароль был подобран.
`fail2ban` зафиксировал незарегистрированную авторизацию и предотвратил утечку информации, пароль не был подобран.