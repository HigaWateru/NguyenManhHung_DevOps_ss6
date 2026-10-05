# Bai 2: Cau hinh phan quyen Nhom va sudoers bang visudo

## 1. Thong tin
- Repo: `session06-homework-ex2` | Duong dan: `homework/session_06/ex2/README.md`
- Moi truong: Ubuntu 22.04, user quan tri co sudo
- Boi canh: cap quyen restart dich vu cho `deployer` ma khong cap full root

## 2. Cac lenh da thuc hien

### Buoc 1: Tao group + user
```bash
sudo groupadd devops-admin
sudo adduser deployer
# tra loi password + info khi duoc hoi
sudo usermod -aG devops-admin deployer
id deployer
getent group devops-admin
```

Ket qua mong doi:
```
uid=1001(deployer) gid=1001(deployer) groups=1001(deployer),1002(devops-admin)
devops-admin:x:1002:deployer
```

### Buoc 2: Cau hinh sudoers an toan bang visudo
```bash
sudo visudo
# hoac chi dinh file drop-in (khuyen dung de tranh vo /etc/sudoers):
sudo visudo -f /etc/sudoers.d/devops-admin
```

Them dong (dau % = group):
```
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Giai thich:
- `%devops-admin`: ap dung cho group
- `ALL=(ALL)`: tu moi host, chay voi quyen moi user (chu yeu root)
- `NOPASSWD:`: khong hoi password
- Chi gioi han 4 subcommand `systemctl start/stop/restart/status`, co wildcard `*` cho ten service. Khong cho `enable/disable/edit` de tranh leo thang.

Kiem tra cu phap (bat buoc):
```bash
sudo visudo -c
# /etc/sudoers: parsed OK
# /etc/sudoers.d/devops-admin: parsed OK
sudo cat /etc/sudoers.d/devops-admin
```

### Buoc 3: Kiem tra voi user deployer
```bash
su - deployer
sudo -l
sudo systemctl status cron --no-pager
sudo systemctl restart cron
echo $?
```

## 3. Bang chung ket qua

```bash
$ su - deployer
$ sudo -l
Matching Defaults entries for deployer on ubuntu-vps:
    env_reset, mail_badpass, ...

User deployer may run the following commands on ubuntu-vps:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *

$ sudo systemctl restart cron
$ echo $?
0
# Khong hoi password -> dat

$ sudo cat /etc/shadow
# Sorry, user deployer is not allowed to execute '/usr/bin/cat /etc/shadow' as root -> bi chan dung (least privilege)
```

## 4. Luu y an ninh
- Luon dung `visudo`, khong `nano /etc/sudoers` truc tiep vi visudo check syntax truoc khi luu. Sai 1 dau phay co the mat sudo toan may.
- Dung `/etc/sudoers.d/` thay vi sua file goc de de rollback (`sudo rm` la xong).
- Khong dung `ALL=(ALL) NOPASSWD: ALL` cho deployer.
- Neu muon log lai: mac dinh sudo da log vao `/var/log/auth.log`, xem bang `sudo grep sudo /var/log/auth.log | tail`.
