###### tags: `教育訓練`
# CYBERSEC2022 - Practical Threat Hunting

course info: https://cyber.ithome.com.tw/2022/en/lab-page/816

---
## Resource
- slide
    - https://speakerdeck.com/will03/practical-threat-hunting-with-osquery
- vm, Task
    - contact me

## Answer
### osquery101

https://osquery.io/schema/4.5.1

```
SELECT version, build, platform FROM os_version;

SELECT * FROM kernel_info;

PRAGMA table_info(routes);

select * from users where uid=0 OR uid=33 OR uid=1000;

SELECT pid, name, path FROM processes WHERE euid!=0;

SELECT * FROM process_open_files 
WHERE (path NOT LIKE "/dev/%" AND path NOT LIKE "/memfd%");

SELECT path,type,uid ,mode ,datetime(atime,'unixepoch') 
FROM file WHERE directory="/usr/bin" order by atime;

```

### Task1

```
SELECT pid, name, path, cmdline from processes
WHERE path like "%python%"
OR path like "%bash%"
OR path like "%perl%"
OR path like "%php%"
OR path like "%ruby%";

SELECT pid, fd, local_address, remote_address, local_port, remote_port
FROM process_open_sockets
WHERE pid=8782;

SELECT  p.pid, p.name, p.path, p.cmdline, s.remote_address, s.remote_port
FROM processes AS p
JOIN process_open_sockets  AS s
USING(pid)
WHERE s.remote_address != ""
AND (p.path like "%python%"
OR p.path like "%bash%"
OR p.path like "%perl%"
OR p.path like "%php%"
OR p.path like "%ruby%");

SELECT * FROM processes
WHERE parent=<pid>;

SELECT path,type from file WHERE path=="/proc/<pid>/fd/1";
    
```

### Task2-1

```
<vm ip>/simple_webshell.php/?cmd=cat+/etc/passwd
```

- config file

```
{
  "options": {
    "worker_threads": "8",
    "disable_events": "false",
    "disable_audit": "false",
    "audit_allow_config": "true",
    "verbose": "false",
    "audit_allow_fim_events": "true",
    "audit_allow_sockets": "true"
  },
  "file_paths": {
    "webshell": [
      "/var/www/html/Online_Shopping/%%"
    ]
  }
}
```

```
select target_path,category from file_events where category="webshell";
```

### Task2-2

```
select * from users where uid=0 OR uid=33 OR uid=1000;

select pid, path, cwd 
FROM process_events WHERE uid=33;

SELECT path, datetime(atime,'unixepoch')
FROM file
WHERE directory="/var/www/html/Online_Shopping/images/item_images/m"
order by atime DESC;

```

### Task3
- config file

```
{
  "options": {
    "worker_threads": "8",
    "disable_events": "false",
    "disable_audit": "false",
    "audit_allow_config": "true",
    "verbose": "false",
    "audit_allow_fim_events": "true",
    "audit_allow_sockets": "true"
  },
  "file_paths": {
    "webshell": [
        "/var/www/html/%%"
    ],
    "systemd": [
       "/etc/systemd/system/%%"
    ]
  }
}
```

```
SELECT pid, name, cmdline, uid FROM processes WHERE parent = 1;

select target_path,category from file_events where category="systemd";

```

### Task4

```
kill -63 0
rmmod diamorphine
```