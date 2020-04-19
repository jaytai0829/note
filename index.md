Debian 10
====
1.設定網路
=====
設定IP
---
auto <網卡名稱>
iface <網卡名稱> inet static ->固定IP
iface <網卡名稱> inet DHCP   ->DHCP
address <ip位置>
netmask <子網路遮罩>
gateway <預設閘道>

更改網卡名稱(eth0開始)
---
位置:/etc/default/grub 
在 GRUB_CMDLINE_LINUX="" 加入net.ifnames=0 biosdevname=0
grub-mkconfig -o /boot/grub/grub.cfg
reboot

更改網卡為自定義名稱
---
:::danger
此指令開機後會失效
:::
ifrename -i eth0 -n Ethernet
註:ifrename 需要額外安裝

2.建立資料夾
=====
語法：mkdir "options" directoryname
|options|說明|
| -| -|
|-m |mode --mode=模式 設定權限 <模式> (類似 chmod)
|-p |--parents 需要時建立上層目錄，如目錄早已存在則不當作錯誤
|-v| --verbose 每次建立新目錄都顯示訊息。

mkdir <資料夾名稱>

-p： --parents 需要時建立上層目錄，如目錄早已存在則不當作錯誤
註:此方法會在當前的位置建立一個資料夾
如果是 mkdir -p 則會將全部資料夾建立出來

3.iptables
===
### 大小寫注意!
處理iptables規則:
---
|參數|說明|參數|說明|
| -| -| -| -|
|-F|清除所有的已訂定的規則|-L|列出目前的 table 的規則|
|-h|help information|-V|顯示 iptables 版本|
|-I|將規則插入至最前面 or 加上號碼插入指定處|-A|將規則插入至最後面|
|-R|取代指定的規則 (加上規則號碼)|-D|刪除指定的規則 (加上規則號碼)|
|-p|這是小寫！設定封包的協定|-s|--sport：來源封包的 port 號碼，也可以使用 port1:port2 如 21:23 同時通過 21,22,23 的意思|
|-d|目標主機的 IP 或者是 Network ( 網域 )|-j|動作|

**-L:**
-n：不使用 DNS 解析直接以 IP 位址顯示
-v：顯示目前 iptables 規則處理的封包數
-x：顯示完整封包數 (ex.顯示 1151519，而不是 12M)
**-j:**
ACCEPT ：接受該封包 
DROP　 ：丟棄封包 
LOG　　：將該封包的資訊記錄下來 (預設記錄到 /var/log/messages 檔案)
**-I:**
INPUT　：規則設定為 filter table 的 INPUT 鏈 
OUTPUT ：規則設定為 filter table 的 OUTPUT 鏈 
FORWARD：規則設定為 filter table 的 FORWARD 鏈 
**-p:**
tcp ：封包為 TCP 協定的封包 
upd ：封包為 UDP 協定的封包 
icmp：封包為 ICMP 協定 
all ：表示為所有的封包

處理iptales 規則鏈(chain):
---
|參數|說明|參數|說明|
| -| -| -| -|
|-N|建立新的規則鏈|-X|刪除指定的規則鏈|
|-E|更改指定的規則鏈(chain)名稱|-P|變更指定規則鏈(chain)的政策 (ex. policy for DROP、REJECT、ACCEPT)|
|-Z|將 iptables 計數器歸零|-i|設定『封包進入』的網路卡介面 |
|-o|設定『封包流出』的網路卡介面 |||
**-P:**
:::info
root@test root:~#iptables [-t tables] [-P] [INPUT,OUTPUT,FORWARD| PREROUTING,OUTPUT,POSTROUTING] [ACCEPT,DROP] 
:::
INPUT:封包為輸入主機的方向
OUTPUT:封包為輸出主機的方向
FORWARD:封包為不進入主機而向外再傳輸出去的方向
PREROUTING:在進入路由之前進行的工作
POSTROUTING:在進入路由之後進行的工作

iptables通用參數
---
|參數|說明|參數|說明|
| -| -| -| -|
|-i|用來比對封包是從哪片網卡進入|-t|參數用來指定規則表，內建的規則表有三個，分別是：nat、mangle 和 filter，當未指定規則表時，則一律視為是 filter。|


允許HTTP連接：80埠
---
:::info
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
:::
:::info
iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
:::


5.apache2 虛擬站台
===
1.把檔案建立在 /etc/apache2/sites-available/自訂名稱.conf
2.```.conf```內容
~~~
<VirualHost *:80>
ServerName   網站名稱 
ServerAlias  網站別名
DocumentRoot 網站的根目錄
ErrorLog     網站的錯誤日誌檔存放的位置與檔名
CustomLog    網站日誌檔存放的位置與檔名+combined
</VirtualHost>
~~~
3.mkdir -p DocumentRoot的位置
  mkdir log的位置
  
建立server-status 頁面(含認證):
----
在VirualHost中加入
![](https://i.imgur.com/khXY9Vx.png)
~~~
<location />
setHandler server-status \\不照打會404NOT FOUND
Authtype basic
AuthName "status" \\認證名稱
AuthUserFile \\認證帳號的檔案位置[註1]
Require valid-user
</location>
~~~
輸出結果:
![](https://i.imgur.com/3cKMhZX.png)


註1:建立帳號的指令
htpasswd -c /etc/apache2/.htpasswd username
輸入<密碼>
-c:當需要建立檔案時加入

https (mod ssl)
---
![](https://i.imgur.com/yrFOAbg.png)
1.啟用ssl
```
a2enmod ssl
```
產生憑證要求:
```
openssl req -new -nodes -keyout server.key -out server.csr
```
:::    danger
HTTPS 跟 HTTP 的 virtualhost 不要放在一起
:::

:::    danger
在申請憑證時要選擇base64
:::
location
---
ex : 題目要求在www.skills39.com/file 顯示網頁
![](https://i.imgur.com/hab0rOW.png)
然後在documentroot的位置底下新增一個/file的資料夾
放入題目要求的東西
:::danger
Require valid-user 的 valid一定要小寫 否則會報錯!!
:::

proxy
---
![Uploading file..._c212bvwre]()

6.bind9
=== 
*nslook指令:apt-get install dnsutils*

#### 1.設定Named.conf.default-zones
![](https://i.imgur.com/ExPbO4K.png)

#### 2.產生db檔
cp /etc/bind/db.local /etc/bind/db.XXX 
註:XXX = 網域名稱
![](https://i.imgur.com/ECALSvu.png)
設定本機的DNS 伺服器
---
路徑 /etc/resolv.conf
nameserver 8.8.8.8

註:resolv.conf 消失時
```
apt-get install --reinstall resolvconf
service networking restart
```
![](https://i.imgur.com/6zKBgju.png)

userdir(使用者個人頁面)
---
a2enmode userdir
修改userdir.conf
UserDir <目錄名稱>
Userdir enable

接下來就會在 www.skills39.com/~USERNAME 看到個人頁面

7.使用者和群組設定
===
增加使用者:
---
#### useradd（常常搭配迴圈使用）
:::info
```
uesradd 使用者名稱$i -m/-d 家目錄 -s/bin/bash -p `openssl passwd -1 密碼`
```
註:$i是for的變數
:::
|參數|說明|
| -| -|
|-u|後面接的是 UID ，是一組數字。直接指定一個特定的 UID 給這個帳號|
|-g|後面接的那個群組名稱就是我們上面提到的 initial group 該群組的 GID 會被放置到 /etc/passwd 的第四個欄位內|
|-G|後面接的群組名稱則是這個帳號還可以加入的群組。這個選項與參數會修改 /etc/group 內的相關資料|
|-M|強制！不要建立使用者家目錄！(系統帳號預設值)|
|-m|強制！要建立使用者家目錄！(一般帳號預設值)|
|-c|這個就是 /etc/passwd 的第五欄的說明內容啦～可以隨便我們設定的啦～|
|-d|指定某個目錄成為家目錄，而不要使用預設值。務必使用絕對路徑！|
|-r|建立一個系統的帳號，這個帳號的 UID 會有限制 (參考 /etc/login.defs)|
|-s|後面接一個 shell ，若沒有指定則預設是 /bin/bash |
|-e|後面接一個日期，格式為『YYYY-MM-DD』此項目可寫入 shadow 第八欄位，亦即帳號失效日的設定項目囉|
|-f|後面接 shadow 的第七欄位項目，指定密碼是否會失效。0為立刻失效，-1 為永遠不失效(密碼只會過期而強制於登入時重新設定而已。)|
#### adduser
~~~
adduser 使用者名稱
輸入密碼
~~~
剩下部份看需求

刪除使用者:
---
userdel 使用者名稱

管理使用者
---
usermod 使用者 <參數>

群組設定
---
1.增加群組
groupadd <名稱>

2.刪除群組
groupdel <名稱>

在使用者家目錄下自動建立資料夾
---
在/etc/skel目錄下新增檔案
當建立使用者時會自動複製/skel下的內容


8.迴圈
===
for ((i=初始值;i<限制值;增加值(ex:i++)))
>do
>指令
>done

9.服務Port 
===
位置:/etc/services 

10.SSH
===
#### 1.更改/etc/ssh/sshd_config
允許限定使用者登入
---
~~~
Allowusers <Username>
~~~
允許Root登入
---
~~~
PermitRootLogin yes
~~~
更改Port
---
~~~
Port <Num>
~~~
登入前顯示訊息
---
~~~
Banner <路徑> ex:/etc/aaa(文字檔)
~~~
ssh key 
---
1.在clinet端產生ssh key:
```
ssh-keygen
```
2.將key複製到server上
```
ssh-copy-id -i ~/.ssh/id_rsa.pub USER@HOST
```
停用密碼認證
---
![](https://i.imgur.com/h4NqMjz.png)

允許特定來源的規則
--
![](https://i.imgur.com/6N1oi7H.png)


11.設定預設文字編輯器
===
update-alternatives --config editor
找到 vim.basic 
輸入它的編號 ->按下確定

12.Samba            
===
設定檔: /etc/samba/smb.cfg

設定分享資料夾

chmod
原始權限:chmod 664
最大權限:chmod 777
:::warning
如果要在windows 下特定使用者免登入存取
就要產生跟windows 一樣的使用者
:::
限制群組
---
valid users = @GROUPNAME

大量建立samba使用者
---
:::info
samba專用的使用者需要在系統內本來就存在的使用者
:::
```
for ((i=1;i<10;i++))
do
(echo 密碼;echo 密碼) | smbpasswd -a user0$i 
done
```
掛載資料夾
---
```
mount -t cifs -o username=使用者名稱,password=密碼 //ip/目標資料夾 /目的地資料夾
```

限制檔案類型
---
![](https://i.imgur.com/1xcHPUQ.png)
限制.exe .dll .txt 無法上傳

限制IP
---
![](https://i.imgur.com/S3Kyp6N.png)
```
hosts allow = 192.168.10.20 \\允許 192.168.10.20存取
hosts deny = all \\除了hosts allow 的IP或網段都拒絕
```

13.telnet
===
安裝server端
```
apt-get install telnet
```
更改telnet port
位置:
```
/etc/services
```

14.在一般使用者登入時，不顯示上次登入時間訊息
===
/etc/pam.d/login
![](https://i.imgur.com/jljNcE2.png)
註解掉 pam_lastlog.so

15.ROOT登入時 記錄訊息在syslog裡
====
![](https://i.imgur.com/UiyRhEt.png)

16.安裝vmtool
====
1.掛載資料夾
```
mkdir /cdrom
mount /dev/cdrom /cdrom
```
2.切換目錄(ex:/etc/)
```
cd /etc
```
3.解壓縮&安裝
```
解壓縮>tar zxpf VmwareTools-10.2.0-7259539.tar.gz(按tab就會出現)
安裝>./vmware-tools-distrib/vmware-install.pl
```

17.服務被mask
===
ex: ssh 被 mask
```
systemctl umask ssh.service 
service ssh start
```

18.DHCP
===
```
apt-get install isc-dhcp-server
```
更改 /etc/dhcp/dhcpd.conf的內容
![](https://i.imgur.com/nV8oR1d.jpg)

指定網卡
/etc/default/isc-dhcp-server
```
INTERFACESv4="網卡名稱"
```

19.開機bash 
===
先複製init.d裡隨便一個bash
```
cp /etc/init.d/apache2 /etc/init.d/test
```
確認default-Start & default-Stop
![](https://i.imgur.com/y0EZNNp.png)
:::success
這邊使用預設的 也可以自訂
:::
將內部東西全部刪掉
:::warning
exit 0 要留下來
:::

更改權限
```
chmod /etc/init.d/test
```

使用 update-rc.d 更新開機 script
```
update-rc.d test defaults
```

參考資料:http://felix-lin.com/linux/debianubuntu-%E6%96%B0%E5%A2%9E%E9%96%8B%E6%A9%9F%E8%87%AA%E5%8B%95%E5%9F%B7%E8%A1%8C%E7%A8%8B%E5%BC%8F/

20.linux當作router
===
vim /etc/sysctl.conf
sysctl -p

ifrend
===
![](https://i.imgur.com/vyqkFoK.png)
