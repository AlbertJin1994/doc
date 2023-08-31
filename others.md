# Windows端口被占用

```bash
# 查询端口
netstat -ano
# 查询指定端口
netstat -ano | findstr "端口号"
# 根据进程PID查询进程名称
tasklist | findstr "进程PID号"
# 根据PID杀死任务
taskkill /F /PID "进程PID号"
# 根据进程名称杀死任务，任务名称可能对应多个，假如现在有多个服务，都用进程名称"java.exe"的话，就会将所有服务都关闭
taskkill -f -t -im "进程名称"
```

