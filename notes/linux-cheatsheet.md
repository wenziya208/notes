Linux 常用命令速查
文件与目录
bash
ls -lht
按修改时间倒序列出文件，人类可读大小。查日志谁最新改的，这个最快。
bash
find . -name "*.log" -mtime +7 -delete
删掉 7 天前的日志。清理磁盘前先不加 -delete 跑一遍确认，血泪教训。
bash
rsync -avz --progress src/ user@host:/data/src/
同步目录到远程机器。结尾的 / 很讲究，有斜杠是同步目录内容，没斜杠是把 src 目录整个拷过去。
bash
du -sh * | sort -rh | head -10
看看当前目录下谁最占空间。每次磁盘报警都靠它。
bash
mkdir -p a/b/c
一次建多层目录，父目录不存在也能建。这个老是记不住加 -p，然后报错。
文本处理
bash
grep -rn "timeout" src/ --include="*.py"
递归搜代码，只搜 py 文件并显示行号。找配置项散落在哪个文件时非常好用。
bash
tail -f app.log | grep --color -i "error"
实时盯着日志里的 error。-i 忽略大小写，有些日志大小写很随意。
bash
awk -F: '{print $1,$3}' /etc/passwd
按冒号切列，取用户名和 UID。awk 记不全语法没关系，记住 -F 和 $1 就够应付大半场景。
bash
sed -i 's/8080/9090/g' config.yaml
批量替换文件里的端口。改之前建议先去掉 -i 预览一遍。
bash
sort ips.txt | uniq -c | sort -rn
统计重复行出现次数并倒排。分析 access log 里哪些 IP 访问最多就靠这一套。
进程与端口
bash
lsof -i :8080
看谁占了 8080 端口。“Address already in use” 出现时第一反应就是它。
bash
ps aux | grep java | grep -v grep
找 java 进程，grep -v grep 把自己那行去掉。
bash
kill -9 $(pgrep -f "myapp.jar")
按命令行匹配杀进程，比手动复制 PID 快。
bash
nohup ./app > app.log 2>&1 &
后台跑服务，输出全进 app.log。2>&1 的顺序不能反，反了 stderr 还是会打到屏幕。
bash
top -Hp <pid>
看某个进程里各线程的 CPU 占用。排查某个线程打满 CPU 时用。
磁盘
bash
df -h
看各分区使用率。报警时先看这个。
bash
df -i
看 inode 使用量。小文件把 inode 耗光但磁盘没满的情况，很坑，一般人想不到。
bash
dd if=/dev/zero of=test.img bs=1M count=1024
造一个 1G 的测试文件。测磁盘写入速度或者占位测试都行。
bash
mount -o ro /dev/sdb1 /mnt
只读方式挂载。怀疑盘有问题时，先只读挂上，别二次伤害数据。
网络
bash
curl -sv http://localhost:8080/health -o /dev/null
测接口通不通，-v 能看到完整请求过程和 TLS 细节。
bash
ss -tlnp
看本机监听了哪些端口。比老式 netstat 快，新系统上 netstat 可能都没有了。
bash
mtr 8.8.8.8
ping 和 traceroute 合体，能持续看到每一跳的丢包。排查网络是哪个节点出问题时特别直观。
bash
ssh -L 3306:db.internal:3306 user@jump
通过跳板机把内网数据库端口转发到本地。连不上内网库时的救命招。
bash
dig +short example.com
快速查域名解析，+short 只出结果不啰嗦。改完 DNS 想确认生效就用它。
权限
bash
chmod -R 755 /var/www
目录批量改成 755。注意别把密钥文件也放进去了，那些要 600。
bash
chown -R www:www /data/site
递归改属主属组。Web 目录权限问题十有八九是这个没改。
bash
sudo -u www ls /tmp
以指定用户身份执行命令。排查"我这里好好的，线上用户跑不了"的权限问题时必备。
bash
chmod 600 ~/.ssh/id_rsa
SSH 私钥权限必须是 600，否则 ssh 直接拒绝使用，报错还很含蓄。