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