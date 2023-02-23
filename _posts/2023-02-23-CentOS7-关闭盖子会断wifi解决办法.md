---
layout: post
tags: [config]
categories: [CentOS7]
---


## 笔记本装CentOS7打算做服务器，结果发现关上盖子后自动断WiFi


1. Open the `/etc/systemd/logind.conf` file for editing.
2. Find the `HandleLidSwitch=suspendline` in the file. If it is quoted out with the `#` character at the start, unquote it.   
If the line is not present in the file, add it.
3. Replace the default suspendparameter with
* `lock` for the screen to lock;
* `ignore` for nothing to happen;
* `poweroff` for the computer to switch off.
For example:
```
[Login]
HandleLidSwitch=lock
```
4. Save your changes and close the editor.
5. Run the following command so that your changes preserve the next restart of the system:
```
# systemctl restart systemd-logind.service
```