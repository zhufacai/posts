# 2022-10-3-在CentOS上使用rclone挂载OneDrive网盘
---
title:  在CentOS上使用rclone挂载OneDrive网盘

slug:  CentOS-rclone-OneDrive

categories:
  - Codes

tags:

author:  Joker

date:  2022-10-03 14:11:00

timezone: UTC+8
---

申请了微软的 E5 开发者账号，OneDrive 有 5T 的容量，于是就想着挂载到自己的 VPS 上，本文记录一下使用 Rclone 挂载 OneDrive 到 Linux 的过程。

# 一、安装 Rclone

CentOS 下安装命令：

```
curl https://rclone.org/install.sh | sudo bash
```

# 二、安装 fuse

挂载位本地硬盘必须有 fuse 支持，否则运行挂载命令会报错

```
yum -y install fuse
```

# 三、配置

## （一）自建 API

1.打开[此链接](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)注册新应用（已注册过的可以跳过），其中**“受支持的帐户类型”** 选择第三项-**任何组织目录(任何 Azure AD 目录 - 多租户)中的帐户和个人 Microsoft 帐户(例如，Skype、Xbox)**，重定向 URL 为 具体看下面：

**重定向 url：**

- rclone 类：[http://localhost:53682](http://localhost:53682)
- SharePoint：[http://localhost](http://localhost)
- 其他：可以写自己的域名，注意一定要是 https。（或者根据程序要求。）

  2.切换到**“API 权限”**选项卡， 访问**Microsoft API >> Microsoft Graph**。

搜索到并勾选以下权限：

```
1. Files.Read
2. Files.Read.All
3. Files.ReadWrite
4. Files.ReadWrite.All
5. offline_access
6. User.Read
```

3.切换到**“证书和密码”**选项卡，创建一个`Client secret`，复制**机密值(Client Secret)**并保存。

## （二）客户端授权

1.在 Windows 电脑上安装 rclone 并在安装目录运行 cmd

2.运行下面命令获取授权

```
rclone authorize "onedrive"
```

3.这时会自动打开浏览器，然后登陆 OneDrive，按照提示操作完之后会在 cmd 终端中返回如下信息：

```
C:\Users\ZhuXiaopeng\Software\rclone-v1.59.2-windows-amd64>rclone authorize "onedrive"
2022/10/03 12:46:58 NOTICE: If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth?state=6csp-lQNGJxT6-ATzwuOQQ
2022/10/03 12:46:58 NOTICE: Log in and authorize rclone for access
2022/10/03 12:46:58 NOTICE: Waiting for code...
2022/10/03 12:47:18 NOTICE: Got code
Paste the following into your remote machine --->
{"access_token":"xxxxxx"}
<---End paste
```

4.复制整个`{"access_token":"xxxxxx"}`内容并保存。

## （三）配置 rclone

```
rclone config
```

1.选择 n 并输入任意名称，如下：输入名称为 onedrive

```
[root@Joker ~]# rclone config
2022/10/03 12:28:12 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found, make a new one?
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n

Enter name for new remote.
name> onedrive
```

2.输入名称后，找到 OneDrive，如下：

```
Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
 2 / Akamai NetStorage
   \ (netstorage)
 3 / Alias for an existing remote
   \ (alias)
 4 / Amazon Drive
   \ (amazon cloud drive)
 5 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, China Mobile, Cloudflare, ArvanCloud, Digital Ocean, Dreamhost, Huawei OBS, IBM COS, IDrive e2, Lyve Cloud, Minio, Netease, RackCorp, Scaleway, SeaweedFS, StackPath, Storj, Tencent COS and Wasabi
   \ (s3)
 6 / Backblaze B2
   \ (b2)
 7 / Better checksums for other remotes
   \ (hasher)
 8 / Box
   \ (box)
 9 / Cache a remote
   \ (cache)
10 / Citrix Sharefile
   \ (sharefile)
11 / Combine several remotes into one
   \ (combine)
12 / Compress a remote
   \ (compress)
13 / Dropbox
   \ (dropbox)
14 / Encrypt/Decrypt a remote
   \ (crypt)
15 / Enterprise File Fabric
   \ (filefabric)
16 / FTP
   \ (ftp)
17 / Google Cloud Storage (this is not Google Drive)
   \ (google cloud storage)
18 / Google Drive
   \ (drive)
19 / Google Photos
   \ (google photos)
20 / HTTP
   \ (http)
21 / Hadoop distributed file system
   \ (hdfs)
22 / HiDrive
   \ (hidrive)
23 / Hubic
   \ (hubic)
24 / In memory object storage system.
   \ (memory)
25 / Internet Archive
   \ (internetarchive)
26 / Jottacloud
   \ (jottacloud)
27 / Koofr, Digi Storage and other Koofr-compatible storage providers
   \ (koofr)
28 / Local Disk
   \ (local)
29 / Mail.ru Cloud
   \ (mailru)
30 / Mega
   \ (mega)
31 / Microsoft Azure Blob Storage
   \ (azureblob)
32 / Microsoft OneDrive
   \ (onedrive)
33 / OpenDrive
   \ (opendrive)
34 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ (swift)
35 / Pcloud
   \ (pcloud)
36 / Put.io
   \ (putio)
37 / QingCloud Object Storage
   \ (qingstor)
38 / SSH/SFTP
   \ (sftp)
39 / Sia Decentralized Cloud
   \ (sia)
40 / Storj Decentralized Cloud Storage
   \ (storj)
41 / Sugarsync
   \ (sugarsync)
42 / Transparently chunk/split large files
   \ (chunker)
43 / Union merges the contents of several upstream fs
   \ (union)
44 / Uptobox
   \ (uptobox)
45 / WebDAV
   \ (webdav)
46 / Yandex Disk
   \ (yandex)
47 / Zoho
   \ (zoho)
48 / premiumize.me
   \ (premiumizeme)
49 / seafile
   \ (seafile)
Storage> 32
```

3.选择 32，继续，按照下面示例填入`client_id` (概述页的`应用程序(客户端) ID`)和`client_secret` ，并依次选择`1-OneDrive国际版`

```
Storage> 32

Option client_id.
OAuth Client Id.
Leave blank normally.
Enter a value. Press Enter to leave empty.
client_id> 你的client_id

Option client_secret.
OAuth Client Secret.
Leave blank normally.
Enter a value. Press Enter to leave empty.
client_secret> 你的client_secret

Option region.
Choose national cloud region for OneDrive.
Choose a number from below, or type in your own string value.
Press Enter for the default (global).
 1 / Microsoft Cloud Global
   \ (global)
 2 / Microsoft Cloud for US Government
   \ (us)
 3 / Microsoft Cloud Germany
   \ (de)
 4 / Azure and Office 365 operated by 21Vianet in China
   \ (cn)
region> 1
```

4.`Edit advanced config`和`Use auto config`均选择 n

```
Edit advanced config?
y) Yes
n) No (default)
y/n>

Use auto config?

 * Say Y if not sure
 * Say N if you are working on a remote or headless machine

y) Yes (default)
n) No
y/n> n

Option config_token.
For this to work, you will need rclone available on a machine that has
a web browser available.
For more help and alternate methods see: https://rclone.org/remote_setup/
Execute the following on the machine with the web browser (same rclone
version recommended):
        rclone authorize "onedrive" "eyJjbGllbnRfaWQiOiI1YjI1MTdhZi01ZWE0LTQzZTgtOWZkNS01ODgxMDY2NDNhMTMiLCJjbGllbnRfc2VjcmV0IjoifmJYOFF+alBYdWJyRGpRbzEuUnBmNncuYVl5N2RpQnJsOTRkfmRifiJ9"
Then paste the result.
Enter a value.
config_token>
```

5.这里要求填入`config_token` ，输入上一步获取的 token，并选择 1，个人/商业版，如需挂载 sharepoint 就选择 2

```
config_token> {"access_token":"xxxx"}

Option config_type.
Type of connection
Choose a number from below, or type in an existing string value.
Press Enter for the default (onedrive).
 1 / OneDrive Personal or Business
   \ (onedrive)
 2 / Root Sharepoint site
   \ (sharepoint)
   / Sharepoint site name or URL
 3 | E.g. mysite or https://contoso.sharepoint.com/sites/mysite
   \ (url)
 4 / Search for a Sharepoint site
   \ (search)
 5 / Type in driveID (advanced)
   \ (driveid)
 6 / Type in SiteID (advanced)
   \ (siteid)
   / Sharepoint server-relative path (advanced)
 7 | E.g. /teams/hr
   \ (path)
config_type> 1


Option config_driveid.
Select drive you want to use
Choose a number from below, or type in your own string value.
Press Enter for the default (b!ok2fyBMiXUe8Hk4pBurTd6xzMuxSarNEh0nX4VvC1VLKpExsMolbTqfIA-KVfvxX).
 1 / OneDrive (business)
   \ (b!ok2fyBMiXUe8Hk4pBurTd6xzMuxSarNEh0nX4VvC1VLKpExsMolbTqfIA-KVfvxX)
config_driveid> 1
```

6.选择 y

```
Drive OK?

Found drive "root" of type "business"
URL: https://1pvgbd-my.sharepoint.com/personal/joker_1pvgbd_onmicrosoft_com/Documents

y) Yes (default)
n) No
y/n>

Configuration complete.
Options:

- type: onedrive
- client_id: xxx
- client_secret: xxx
- token: {"access_token":"xxxxx"}
- drive_id: b!ok2fyBMiXUe8Hk4pBurTd6xzMuxSarNEh0nX4VvC1VLKpExsMolbTqfIA-KVfvxX
- drive_type: business
  Keep this "onedrive" remote?
  y) Yes this is OK (default)
  e) Edit this remote
  d) Delete this remote
  y/e/d> y

Current remotes:

Name                 Type
====                 ====
onedrive           onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

然后选择 q 退出，至此，rclone 配置完成。

## （四）挂载

1.新建本地文件夹，路径自己定，这里以根目录下为例。

```
mkdir /OneDrive
```

2.挂载为磁盘，下面的 DriveName、Folder、LocalFolder 参数根据说明自行替换

```
rclone mount DriveName:Folder LocalFolder --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
```

`DriveName`为初始化配置填的`name`，`Folder`为`OneDrive`里的文件夹，`LocalFolder`为`VPS`上的本地文件夹。

这里选择挂载 OneDrive 根目录，如下：

```
rclone mount onedrive:/ /OneDrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
```

如果挂载过程中出现`NOTICE: One drive root 'test': poll-interval is not supported by this remote`错误，可以无视该错误。

至此，挂载完成。

如果要卸载，可以使用`rclone config`命令来完成。

# 四、开机自启

如果使用宝塔面板，则直接安装`Supervisor管理器`添加守护进程，启动命令为之前的挂载命令。或者按照下面进行配置。

1.运行以下命令

```
command=" mount onedrive:/ /OneDrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000"
```

此命令后面为上面临时挂载命令去掉开头的 rclone。

2.再使用命令：

```
cat > /etc/systemd/system/rclone.service <<EOF
[Unit]
Description=Rclone
After=network-online.target

[Service]
Type=simple
ExecStart=$(command -v rclone) ${command}
Restart=on-abort
User=root

[Install]
WantedBy=default.target
EOF
```

此为一条完整的 shell 命令，需要全部复制，一次执行。

开始启动：

```
systemctl start rclone
```

设置开机自启：

```
systemctl enable rclone
```

其他命令：

```
重启：systemctl restart rclone
停止：systemctl stop rclone
状态：systemctl status rclone
```

如果你想挂载多个网盘，那么将`systemd`配置文件的`rclone.service`改成`rclone1.service`即可，重启动什么的同样换成`rclone1`。

> Rclone 常用功能选项

- 命令格式

```
# 本地到网盘
rclone [功能选项] <本地路径> <网盘名称:路径> [参数] [参数] ...

# 网盘到本地
rclone [功能选项] <网盘名称:路径> <本地路径> [参数] [参数] ...

# 网盘到网盘
rclone [功能选项] <网盘名称:路径> <网盘名称:路径> [参数] [参数] ...
```

- 举例

```
同步内容到网盘
/usr/bin/rclone sync -v /root/aria2-downloads/ onedrive:/ > ~/drive.log 2>&1
同步内容到本地
/usr/bin/rclone sync -v onedrive:/ /root/aria2-downloads/ > ~/drive.log 2>&1
```

- 其他命令

```
rclone copy - 复制
rclone move - 移动，如果要在移动后删除空源目录，请加上 --delete-empty-src-dirs 参数
rclone sync - 同步：将源目录同步到目标目录，只更改目标目录。
rclone size - 查看网盘文件占用大小。
rclone delete - 删除路径下的文件内容。
rclone purge - 删除路径及其所有文件内容。
rclone mkdir - 创建目录。
rclone rmdir - 删除目录。
rclone rmdirs - 删除指定灵境下的空目录。如果加上 --leave-root 参数，则不会删除根目录。
rclone check - 检查源和目的地址数据是否匹配。
rclone ls - 列出指定路径下的所有的文件以及文件大小和路径。
rclone lsl - 比上面多一个显示上传时间。
rclone lsd 列出指定路径下的目录
rclone lsf - 列出指定路径下的目录和文件
```

# 五、离线下载

Aria2 有一个配置项 `on-download-complete`，即在下载完后执行一个脚本或命令。当下载完成后 Aria2 会给脚本传递分别为 GID 、文件数量、文件路径的 3 个变量。利用这个配置项和这些变量就可以实现诸如下载完成后调用 Rclone 进行上传的操作。整个过程简单来说就是，Aria2 下载文件到 VPS ，完成后告诉 Rclone 将文件上传到网盘。理论上只要是 Rclone 支持的网盘，都可以按照这个思路来实现~~伪~~离线下载。

## （一）安装 aria2

这里使用 [Aria2 一键安装管理脚本 增强版]([P3TERX/aria2.sh: Aria2 一键安装管理脚本 增强版 (github.com)](https://github.com/P3TERX/aria2.sh))，执行下面的代码下载并运行脚本，出现脚本操作菜单输入 `1` 开始安装。

```
wget -N git.io/aria2.sh && chmod +x aria2.sh && ./aria2.sh
```

## （二）安装前端 AriaNg

```
wget https://github.com/mayswind/AriaNg/releases/download/1.2.3/AriaNg-1.2.3.zip
unzip AriaNg-1.2.3.zip
```

然后配置密钥跟 aria2.conf 中的密钥相同的话应该就能正常连接了。

## （三）配置自动上传

1.打开`/root/.aria2c/aria2.conf`文件，找到“下载完成后执行的命令”，把`clean.sh`替换为`upload.sh`。

2.打开/root/.aria2c/script.conf 文件，修改网盘名称`drive-name=OneDrive(RCLONE 配置时填写的 name)`

3.重启 Aria2

```
service aria2 restart
```

4.执行`upload.sh`脚本，提示`success`即表示上传脚本能正常被调用，否则请检查与 RCLONE 有关的配置。

```none
/root/.aria2c/upload.sh
```

注意：success 成功但不能上传的话可以先安装 jq 再配置上传脚本后重启系统。

```
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum repolist
yum install jq
```
