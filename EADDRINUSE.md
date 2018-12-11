#Error: listen EADDRINUSE#

node app.js 时报错：
```
Error: listen EADDRINUSE :::3456
    at Server.setupListenHandle [as _listen2] (net.js:1360:14)
    at listenInCluster (net.js:1401:12)
    at Server.listen (net.js:1485:7)
    at Function.listen (/root/mock-test-bank-page/node_modules/express/lib/application.js:618:24)
    at Object.<anonymous> (/root/mock-test-bank-page/app.local.js:6:18)
    at Module._compile (module.js:653:30)
    at Object.Module._extensions..js (module.js:664:10)
    at Module.load (module.js:566:32)
    at tryModuleLoad (module.js:506:12)
    at Function.Module._load (module.js:498:3)
```

后来google到解决办法：

‘EADDRINUSE’应该是‘error address in use’的缩写。也就是说你监听的端口已经被使用了!

查询什么进程占用了3456端口：
```
sudo fuser -n tcp 3456
3456/tcp:            23673
```
kill掉这个进程：
``
sudo kill 23673
``

哒哒~~解决！

今天用这个办法不灵，因为：fuser 不能 -n？？？

4c32759b87a1:MockBankOTPWebApp fayefei$ sudo fuser -n tcp 3000
Password:
Unknown option: n
fuser: [-cfu] file ...
    -c  file is treated as mount point
    -f  the report is only for the named files
    -u  print username of pid in parenthesis

然后其他的方法查到线程ID，但是kill不了：

4c32759b87a1:MockBankOTPWebApp fayefei$ lsof -i tcp:3000
COMMAND   PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
node    69578 fayefei   13u  IPv6 0xf15ed80625ac167b      0t0  TCP *:hbci (LISTEN)
4c32759b87a1:MockBankOTPWebApp fayefei$ kill -15 69578
======
4c32759b87a1:MockBankOTPWebApp fayefei$ sudo kill 69578
======
4c32759b87a1:MockBankOTPWebApp fayefei$ pgrep node
69578
4c32759b87a1:MockBankOTPWebApp fayefei$ pkill node
4c32759b87a1:MockBankOTPWebApp fayefei$ pgrep node
69578



最后还是用 -9 kill的
4c32759b87a1:MockBankOTPWebApp fayefei$ lsof -PiTCP -sTCP:LISTEN
COMMAND   PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Python   4358 fayefei    4u  IPv4 0xf15ed80609e5bc13      0t0  TCP localhost:9217 (LISTEN)
idea    13753 fayefei  187u  IPv4 0xf15ed8061c6b631b      0t0  TCP localhost:6942 (LISTEN)
idea    13753 fayefei  518u  IPv4 0xf15ed8062019231b      0t0  TCP localhost:63342 (LISTEN)
java    22366 fayefei  268u  IPv6 0xf15ed806272c2bbb      0t0  TCP localhost:2014 (LISTEN)
node    69578 fayefei   13u  IPv6 0xf15ed80625ac167b      0t0  TCP *:3000 (LISTEN)
4c32759b87a1:MockBankOTPWebApp fayefei$ kill -9 69578
4c32759b87a1:MockBankOTPWebApp fayefei$ lsof -PiTCP -sTCP:LISTEN
COMMAND   PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Python   4358 fayefei    4u  IPv4 0xf15ed80609e5bc13      0t0  TCP localhost:9217 (LISTEN)
idea    13753 fayefei  187u  IPv4 0xf15ed8061c6b631b      0t0  TCP localhost:6942 (LISTEN)
idea    13753 fayefei  518u  IPv4 0xf15ed8062019231b      0t0  TCP localhost:63342 (LISTEN)
java    22366 fayefei  268u  IPv6 0xf15ed806272c2bbb      0t0  TCP localhost:2014 (LISTEN)
[1]+  Killed: 9               node app.js
4c32759b87a1:MockBankOTPWebApp fayefei$ node app.js
MockBankOTPWebApp is listening on http://:::3000

