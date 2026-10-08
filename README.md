# BT02 - Cấu hình phân quyền Nhóm và sudoers bằng visudo (Group Permissions and sudoers)

## 1. Tạo nhóm người dùng devops-admin

```bash
devops@0310263186:~$ sudo groupadd devops-admin
devops@0310263186:~$ getent group devops-admin
devops-admin:x:1002:
```

## 2. Tạo tài khoản deployer và đưa vào nhóm

```bash
devops@0310263186:~$ sudo adduser deployer
info: Adding user `deployer' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `deployer' (1003) ...
info: Adding new user `deployer' (1003) with group `deployer (1003)' ...
info: Creating home directory `/home/deployer' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for deployer
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y

devops@0310263186:~$ sudo usermod -aG devops-admin deployer
devops@0310263186:~$ id deployer
uid=1003(deployer) gid=1003(deployer) groups=1003(deployer),100(users),1002(devops-admin)
```

## 3. Cấu hình /etc/sudoers bằng visudo

```bash
devops@0310263186:~$ sudo visudo
```

Thêm dòng sau vào cuối tệp `/etc/sudoers` (dấu `%` biểu thị cho Group):

```
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Kiểm tra cú pháp và vị trí dòng vừa thêm:

```bash
devops@0310263186:~$ sudo visudo -c
/etc/sudoers: parsed OK

devops@0310263186:~$ sudo grep -n "devops-admin" /etc/sudoers
32:%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

## 4. Kiểm tra quyền của user deployer

```bash
devops@0310263186:~$ su - deployer
deployer@0310263186:~$ sudo -l
Matching Defaults entries for deployer on 0310263186:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deployer may run the following commands on 0310263186:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

## 5. Chạy thử lệnh quản lý dịch vụ không cần mật khẩu

```bash
deployer@0310263186:~$ sudo systemctl restart cron
deployer@0310263186:~$ sudo systemctl status cron --no-pager
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-10-06 09:20:41 UTC; 3s ago
       Docs: man:cron(8)
   Main PID: 1245 (cron)
      Tasks: 1 (limit: 1113)
     Memory: 1.0M (peak: 1.1M)
        CPU: 12ms
     CGroup: /system.slice/cron.service
             └─1245 /usr/sbin/cron -f -P
```

## 6. Kiểm tra giới hạn quyền (lệnh ngoài danh sách bị từ chối)

```bash
deployer@0310263186:~$ sudo systemctl daemon-reload
[sudo] password for deployer:
Sorry, user deployer is not allowed to execute '/usr/bin/systemctl daemon-reload' as root on 0310263186.

deployer@0310263186:~$ exit
logout
devops@0310263186:~$
```
