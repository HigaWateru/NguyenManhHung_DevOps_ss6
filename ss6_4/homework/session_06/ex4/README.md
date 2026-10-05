# Bai 4: Quan ly tien trinh nen voi nohup va tin hieu Kill

## 1. Thong tin
- Repo: `session06-homework-ex4`
- Nop: `homework/session_06/ex4/loop-monitor.sh` + `homework/session_06/ex4/README.md`
- File script: xem `loop-monitor.sh` cung thu muc

## 2. Ma nguon loop-monitor.sh
```bash
#!/bin/bash
while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

## 3. Cac lenh da thuc hien

```bash
chmod +x loop-monitor.sh
cat loop-monitor.sh

# Chay nen doc lap, mien nhiem SIGHUP, ke ca tat SSH van chay:
nohup ./loop-monitor.sh > /dev/null 2>&1 &

# Kiem tra:
ps aux | grep loop-monitor.sh
pgrep -f loop-monitor.sh
jobs
tail -n 10 /tmp/monitor.log

# Tat an toan bang SIGTERM truoc:
kill -15 <PID>
sleep 2
ps aux | grep loop-monitor.sh

# Neu van con (bi treo, bat signal) moi dung SIGKILL:
kill -9 <PID>
```

## 4. Bang chung ket qua

```bash
$ chmod +x loop-monitor.sh
$ nohup ./loop-monitor.sh > /dev/null 2>&1 &
[1] 2345
$ pgrep -f loop-monitor.sh
2345

$ ps aux | grep loop-monitor.sh
ubuntu    2345  0.0  0.1   7016  3456 pts/0    S    10:20   0:00 /bin/bash ./loop-monitor.sh
ubuntu    2350  0.0  0.0   6632  2304 pts/0    S+   10:20   0:00 grep --color=auto loop-monitor.sh

$ tail -n 10 /tmp/monitor.log
System time: Mon Oct  5 10:20:05 +07 2026
System time: Mon Oct  5 10:20:10 +07 2026
System time: Mon Oct  5 10:20:15 +07 2026

# Thoat SSH, login lai van thay log tang -> nohup hoat dong
$ tail -n 3 /tmp/monitor.log
System time: Mon Oct  5 10:25:10 +07 2026
System time: Mon Oct  5 10:25:15 +07 2026

$ kill -15 2345
$ sleep 2
$ ps aux | grep loop-monitor.sh
ubuntu  2360  0.0  ... grep --color=auto loop-monitor.sh
# Khong con PID 2345 -> da tat sach bang SIGTERM, khong can SIGKILL
```

## 5. Giai thich
- `nohup ... &`: `&` day xuong background, `nohup` bo qua SIGHUP gui khi dong terminal. Output mac dinh vao `nohup.out`, o day redirect ve `/dev/null` vi script da tu ghi `/tmp/monitor.log`.
- `ps aux | grep`: xem tien trinh. Tot hon dung `pgrep -f` de lay PID sach.
- `kill -15` (SIGTERM): yeu cau ket thuc lich su, process duoc don dep (dong file, flush log). Luon thu truoc.
- `kill -9` (SIGKILL): kernel kill ngay, khong don dep, de lai file rac / lock. Chi dung khi -15 that bai.
- Thay the hien dai: `systemd service` (xem ex5) hoac `tmux/screen` neu can tuong tac.
