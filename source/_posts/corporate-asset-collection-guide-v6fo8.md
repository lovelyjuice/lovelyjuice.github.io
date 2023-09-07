---
created: '2023-09-01 17:55:12'
updated: '2023-09-07 15:37:07'
title: 企业资产收集指南
permalink: siyuan://blocks/20230901175512-mbgjnw4
categories: []
---


# 企业资产收集指南

## 1. 主域名收集

### 1.1 搜索引擎

通过搜索引擎搜索企业名称，查询其公开在互联网上的网站，有手就行。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537300.png)

需要注意的是，必须通过脚注验证网站是否属于该企业名下，比如上图第四条`http://www.caihcom.com/`，看起来很像官网，但打开网页后查看脚注，会发现其实它是一个高仿网站 **CTI 论坛 - 融合通信专业资讯网**，并不属于 **中国 - 东盟信息港股份有限公司** 的资产。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537265.png)

### 1.2 企业信息查询

通过天眼查、企查查等企业信息查询平台查询企业的域名和邮箱，对于部分门户网站存在感较低的企业有良好效果，不过由于这类信息查询平台并没有验证域名的有效性，因此有时候会出现误报，需要进一步人工验证。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537968.png)

### 1.3 备案查询

对于运营在中国境内的公司，可以通过工信部的 [ICP/IP 地址 / 域名信息备案管理系统](https://beian.miit.gov.cn/)查询企业已备案的资产，以**广西西江开发投资集团有限公司**为例，通过搜索引擎搜索，仅能找到 `gxxijiang.com` 一个域名。但通过备案查询，可以查到另一个域名 `xjsy56.com` ，甚至还能找到仅有 IP 的资产，这些资产由于 robots 限制、存在感低、网站标题与企业名称无关等原因，并未被搜索引擎爬取到。

需要注意的是以企业名称查询备案信息时，需要**一字不差**地输入企业**全称**，至于企业全称去哪找？可以先用主域名查询备案企业全称。

​![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537828.png)​

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537174.png)

### 自动化收集主域名

Enscan_go提供了爱企查、企查查、天眼查、阿拉丁（测试未成功）、七麦数据（某些时候会导致程序崩溃）等多个平台的自动化查询方案。天眼查具有频率限制，多次频繁访问需要验证码，目前百度的爱企查无任何限制，仅需提供Cookie即可。

```shell
.\enscan-0.0.10-windows-amd64.exe -f 单位列表.txt -field icp,app,wx_app -invest 51 -type aqc,aldzs --is-branch --branch -is-merge -o hw2023 -hold -deep 4 --is-show
```

由于一些公司会将IP备案，因此需要通过正则表达式将IP格式的数据删除。

```
\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}
```

Enscan_go 搜索公司名称之后会把搜索到的第一条记录作为结果，所有就会出现搜索某个分支机构但又因为这个分支机构不是独立法人，所以搜不到这个分支机构，反而会模糊搜索到另一个名字和这个机构很像的单位。Enscan_go 无法处理这种情况，因此可以使用SQL语句删除这些错误目标。

但是要**注意**，如果资产清单中的公司名称不是全称而是缩写，则模糊匹配反而会查找到正确的目标，这时候如果删除这些模糊匹配到的数据**会导致漏掉目标！**

```sql
delete from enscan 
where NOT EXISTS(
	select
		 company.name
	from
		company 
	where
		enscan.company like concat('%', company.name, '%'));
```

## 2. 子域收集

由于子域名的搜集方法比较多，常见的有 DNS 解析记录、搜索引擎、爆破、网络空间搜索引擎等方式，人工收集效率极低，最好使用子域名收集工具进行自动化收集，推荐工具 [oneforall](https://github.com/shmilylty/OneForAll)。收集到的子域名对应 IP 有可能是 CDN 或虚拟主机，此时 oneforall 会在扫描结果中标注该 IP 是否为 CDN，但有时候会存在误报现象，因此建议通过 IP 反查域名来判断是否是 CDN.

大多数中小企业都会使用 CDN 服务商，这种情况下 CDN 不属于企业的资产。大型互联网企业会自建 CDN，但由于 CDN 仅作流量转发，攻击面较小，可忽略此类资产。

oneforall 默认不支持四级域名，比如 a.b.c.d，oneforall 会把 b.c.d 当作主域名，该问题在其官方 GitHub 仓库 issue 中多次被提及，但作者并没有直接修复，而是给出另一种解决办法：在 data/public_suffix_list.dat 添加一条 b.c.d 的记录，对于批量检测也可以添加 `*.c.d` 的记录，但要注意只支持一个星号，不支持 `*.*.d` 这样的方式。

### 自动化收集子域名

针对广西区内资产，需要在 `data/public_suffix_list.dat` 末尾加入 `"*.gxzf.gov.cn","*.gov.cn"`后才能进行进一步的搜集，因为很多县处级机关单位的主域名都是四级域名。不加这两条的话，oneforall会认为`gxzf.gov.cn`是主域名，导致所有广西的政府机构（厅局级、县处级、甚至乡科级）域名都会被收集。

```bash
python oneforall.py --targets main_domains.txt --brute True run
```

收集完成之后可以将得到的csv文件导入数据库（mysql 8.0以上，否则后续一些函数无法使用），需要注意的一点是csv的编码是GB18030,导入时需指定编码，否则地址、标题等字段会乱码。

​![cffc19fc22ea9b4f1dbb49a726c4a07e_MD5](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537865.png)​

oneforall本身通过 `data` 目录下的几个 `cdn_xxx.json` 文件判断CDN，但CNAME列表太老，且会通过响应包头部做出判断，因此不太准确。最好单独通过cname过滤下面的关键字。但是其实还有更简单的一个办法，那就是直接判断cname是否以子域名结尾，原因如下。

- 如果cname和子域名为不同公司的域名，则必定需要排除
- 如果cname和子域名指向同一公司的不同域名，则它们对应的IP都是同一个，端口扫描时也需要排除掉

至于CDN需不需要扫描，我认为是有必要的，当然仅仅扫描web服务就行，没有必要去扫非web的端口。然而如果非要区分开扫描web端口和非web端口的话，端口扫描的步骤会显得较为麻烦，还不如多一些脏数据，但是端口扫描一遍过。

### 去除泛解析

onefoall对域名是否开启泛解析的判断存在不稳定的情况，有时候某个域名开了泛解析，它会误认为没有开泛解析，但是你重新扫一遍它又能正常识别出泛解析开启了。这种不稳定的情况一旦出现，对最终结果的准确性会造成灾难性的影响。下面是泛解析判断失误的一个例子。

|IP|对应的域名数量|
| :------------------------------------------------------------| ----------------|
|221.\*.\*.137|19285|
|119.3.\*.135,119.3.\*.50|54244|
|119.3.\*.50,119.3.\*.135|54990|
|156.\*.\*.26|4335|
|156.\*.\*.138|6125|
|156.\*.\*.117|6814|
|156.\*.\*.142|3035|
|156.\*.\*.140|5953|
|156.\*.\*.27|4095|
|156.\*.\*.115|5129|
|223.\*.\*.162,223.\*.\*.159,223.\*.\*.160|3269|
|223.\*.\*.161,223.\*.\*.162,223.\*.\*.159|3321|
|223.\*.\*.159,223.\*.\*.160,223.\*.\*.161|3365|
|223.\*.\*.160,223.\*.\*.161,223.\*.\*.162|3232|

可以看出onefoall对存在负载均衡的泛解析域名无能为力，需要用下面的SQL语句去除泛解析域名。

```sql
# 筛选出超过一定数量的IP
select ip,count(*) as times from oneforall GROUP BY ip HAVING times > 2000;

# 删除超过一定数量的IP
DELETE FROM oneforall WHERE ip IN (
SELECT ip FROM (
	SELECT ip,COUNT(*) AS c FROM oneforall GROUP BY ip
	) as sq
	WHERE c>1000 
);
```

除此之外，不少公司的域名存在被博彩劫持的情况，这时候IP地理位置一般在国外。由此，可以得到筛选正常IP的SQL语句。

```sql
SELECT DISTINCT ip from oneforall 
WHERE 
addr like '%中国%'
and addr not like '%香港%'
and addr not like '%台湾%'
and addr not like '%澳门%'
and addr not like '%内网%';
```

导出结果后记得使用正则表达式 `,(.*?)\n` 替换掉存在负载均衡的CDN的IP。cname包含下面这些字段的一般都是dns解析到云服务去了，必须删除这些记录，不然会打错目标。

```
cdn.
waf.cn
yundun
cloudwaf
qcloud
163.com
aliyun
netease.com
qiniu.com
cloudfront
alibabadns
tdnsv
trafficmanager
tencent.com
alimail
yunyou.top
cdnhwc
kunluncan
mail.qq.com
queniusa
queniuqy
ivanlynetwork.cn
kunlunpi.com
vhostgo
cname
dnspod
```

```sql
DELETE FROM oneforall WHERE cname like 'cdn.%';
DELETE FROM oneforall WHERE cname like '%waf.cn%';
DELETE FROM oneforall WHERE cname like '%yundun%';
DELETE FROM oneforall WHERE cname like '%cloudwaf%';
DELETE FROM oneforall WHERE cname like '%qcloud%';
DELETE FROM oneforall WHERE cname like '%.163.com%';
DELETE FROM oneforall WHERE cname like '%.mail.qq.com%';
DELETE FROM oneforall WHERE cname like '%.netease.com%';
DELETE FROM oneforall WHERE cname like '%.tencent.com%';
DELETE FROM oneforall WHERE cname like '%alimail%';
DELETE FROM oneforall WHERE cname like '%vhostgo%';
DELETE FROM oneforall WHERE cname like '%qiniu.com%';
DELETE FROM oneforall WHERE cname like '%cloudfront%';
DELETE FROM oneforall WHERE cname like '%alibabadns%';
DELETE FROM oneforall WHERE cname like '%tdnsv%';
DELETE FROM oneforall WHERE cname like '%trafficmanager%';
DELETE FROM oneforall WHERE cname like '%yunyou.top%';
DELETE FROM oneforall WHERE cname like '%cdnhwc%';
DELETE FROM oneforall WHERE cname like '%kunluncan%';
DELETE FROM oneforall WHERE cname like '%aliyun%';
DELETE FROM oneforall WHERE cname like '%queniusa%';
DELETE FROM oneforall WHERE cname like '%queniuqy%';
DELETE FROM oneforall WHERE cname like '%ivanlynetwork.cn%';
DELETE FROM oneforall WHERE cname like '%kunlunpi.com%';
DELETE FROM oneforall WHERE cname like '%cname%';
DELETE FROM oneforall WHERE cname like '%dnspod%';
```

## 3. IP 收集

- 收集主域名时通过备案发现的IP:  使用CookText提取IP `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}`
- 百度、谷歌以及Fofa、360quake、鹰图等网络空间搜索引擎搜索公司全称、简称
- 根据域名对应的IP分布范围猜测 `/30` 或者 `/29` 段的IP，除非是成立时间比较久的大公司或者是有教育网IP的高校，其他单位很少有整个C段
- 对于教育网，比如 210.36 开头的，可以在vscode中用正则将`210\.36\.(\d{1,3}).*?\n` 替换为 `210.36.$1.0/24\n`，然后去重即可确定需要扫描的c段

搜集完成后将 ip 导入 pure_ip 数据表，列名为 ip 。

## 4. 端口扫描

在攻防演练期间，各类安全设备会对端口扫描速率造成极大的限制。对此，应当降低单位IP的扫描速率，提高扫描并发数。经过测试，masscan、nmap均具备此特性，但nmap需要额外指定参数启用单个IP的速率限制，否则依然是一个一个IP扫。masscan默认启用该特性。

### masscan

以下是masscan的一个配置文件。建议对单个IP的扫描速率为 1 秒钟 1-5 个请求包。即便是一秒钟只发一个包，65535秒——即18个小时就能扫完所有目标。

```ini
rate=4000
port=21-65535
output-format=list
output-filename = masscan_result.csv
open-only=true

range=1.1.1.1
range=2.2.2.2
range=3.3.3.3
```

扫描完成后，得到一个空格分隔的CSV文件，准备导入SQLite，如果是用navicat的话，导入的时候需要选择文本格式而不是CSV格式，否则无法自定义字段之间的分隔符。IP列的类型应该使用char(15)而不是下面截图里面的text，同时**最好针对IP建个索引**，该索引**按顺序**保存列值，不然后面进行大量数据查询的时候性能很低，尤其是使用 `group by` 子句的时候。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071537830.png)

运行下面的SQL语句筛选掉开放端口数量大于等于50的，因为部分防火墙（比如阿里云）会将所有端口都显示为开放。

```sql
SELECT ip,port 
FROM masscan 
WHERE ip IN 
( SELECT ip FROM 
  ( SELECT ip, COUNT( * ) AS ports_num FROM masscan GROUP BY ip ) temp
  WHERE temp.ports_num < 50 
)
```

如果要继续下一步的话，建议备份原数据库后直接删除开放端口数量超过特定值的记录：

```sql
DELETE 
FROM masscan 
WHERE ip IN 
( SELECT ip FROM 
  ( SELECT ip, COUNT( * ) AS ports_num FROM masscan GROUP BY ip ) temp
  WHERE temp.ports_num > 300 
);
```

### Nmap

> Nmap 具有并行扫描多主机端口或版本的能力，Nmap 将多个目标 IP 地址空间分成组，然后在同一时间对一个组进行扫描。通常，大的组更有效。缺点是只有当整个组扫描结束后才会提供主机的扫描结果。如果组的大小定义为 50，则只有当前 50 个主机扫描结束后 才能得到报告(详细模式中的补充信息除外)。
> 默认方式下，Nmap 采取折衷的方法。开始扫描时的组较小，最小为 5，这样便于尽快产生结果；随后增长组的大小，最大为 1024。确切的大小依赖于所给定的选项。

nmap 通过 `--min-hostgroup` 选项控制并行扫描组的大小，如果需要同时对所有服务器进行扫描，则需要将这个参数设置为服务器数量。

除此之外，还需要通过 `--scan-delay` 参数控制 nmap 的发包间隔，单位是毫秒。

> 这个选项用于 Nmap 控制针对一个主机发送探测报文的等待时间(毫秒)，在带宽控制的情 况下这个选项非常有效。Solaris 主机在响应 UDP 扫描探测报文报文时，每秒只发送一个 ICMP 消息 因此 Nmap 发送的很多数探测报文是浪费的 `--scan-delay` 设为 1000 使 Nmap 低速运行。

**注意：暂未实现将nmap扫描结果导出为ip:port格式的脚本**

## 5. Web资产收集

由于部分网站需要域名才能访问，因此最好将 ip:port 格式的数据转换为 domain:port 形式的数据，可以通过SQL实现。某些IP没有域名，无法反查到域名，因此需要单独处理。

```sql
# domain:port 形式的数据
SELECT subdomain, masscan.port from oneforall, masscan  WHERE find_in_set(masscan.ip,oneforall.ip);
```

```sql
# ip:port 形式的数据
SELECT ip, port
FROM masscan
WHERE ip NOT IN (SELECT ip FROM oneforall);
```

参考：[Mysql字符串字段判断是否包含某个字符串的3种方法](https://www.cnblogs.com/lcngu/p/6200491.html)

### fscan

#### 导入数据库

fscan支持 IP/domain:Port 格式的文件作为输入进行快速扫描，可以使用grep从扫描结果中提取URL。

```
grep WebTitle fscan_result.txt > fscan_web.txt
grep -v 跳转url fscan_web.txt > fscan_no_redirect.txt
```

然后使用notepad3的 `编辑 - 选择 - 压缩空白字符`  功能把多个空格合并成一个，防止导入数据库时出错。由于标题可能包含空格，还需要将标题用引号包起来。

1. 删除 `[*] WebTitle: ​`
2. `title:` 替换为 `"`
3. `\n` 替换为 `"\n`
4. 删除 `len:` `code:` 等字符串

最终得到下图中的格式，可以直接导入数据库进一步处理。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309071534871.png)

#### 排除异常数据

扫描完成后可能会存在以下几种异常数据：

1. 扫描到CDN但是没有域名绑定到特定端口，比如 https://www.example.com/ 能正常，但是访问 https://www.example.com:7001/ 的时候CDN会显示示域名未绑定的页面。
2. 域名对应的协议和端口未接入阿里云Web应用防火墙，原因同上

使用下面的SQL语句找出一些异常数据

```sql
# 根据特征查找异常数据  
SELECT count(url) as c,group_concat(url),status_code,length,title  
FROM fscan  
GROUP BY status_code,length,title  
ORDER BY c DESC ;
```

已知异常数据，由于这些页面的特征可能会更新，最好自己找特征：

```sql
# 360网站卫士未绑定域名  
DELETE FROM fscan WHERE status_code=403 AND length=294 AND title='403 Forbidden';  
  
# 阿里云web防火墙未绑定域名  
DELETE FROM fscan WHERE status_code=410 AND length=10925 AND title='阿里云 Web应用防火墙';  
  
# 安恒云防护未绑定域名  
DELETE FROM fscan WHERE status_code=422 AND length=63967 AND title='None';  
  
# 某防火墙  
DELETE FROM fscan WHERE status_code=404 AND length=27569 AND title='404 Not Found!';  
  
# 阿里邮箱  
DELETE FROM fscan WHERE title LIKE '%阿里邮箱%';  
  
# 杂七杂八的重定向  
DELETE FROM fscan  
WHERE url LIKE '%qq.com%'  
OR url LIKE '%baidu.com%'  
OR url LIKE '%aliyun.com%'  
OR url LIKE '%tencent.com%'  
OR url LIKE '%163.com%'  
OR url LIKE '%netease.com%'  
OR url LIKE '%cnki.net%'  
OR url LIKE '%chaoxing.com%'  
OR url LIKE '%campushoy.com%'  
OR url LIKE '%microsoft%'  
OR url LIKE '%amazon%'  
OR url LIKE '%bilibili%'  
OR url LIKE '%seewo.com%'  
OR url LIKE '%vhostgo%';
```

给 fscan 数据表添加一个名为IP数据类型为 `varchar(300)` 的列，将 oneforall 解析出来的 IP 也作为添加到 fscan 数据表中，用于筛选重复页面，排除掉因为没有绑定域名而导致同一 web 服务对应多个域名的情况。注意，这一步会执行非常长的时间，十几万的数据量需要40分钟左右。

但在此之前需要先处理oneforall 里面没有的形如 http://ip:port 格式的记录，用正则提取IP：

```sql
UPDATE fscan SET ip=substr(regexp_substr(url,':\\/\\/\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}',1,1),4); 
```

```sql
update fscan
set fscan.ip =
    (select distinct oneforall.ip
     from oneforall
     where oneforall.subdomain != ''
       and fscan.url LIKE concat('%', oneforall.subdomain, '%')
     limit 1);
```

给 fscan 数据表添加一个 port 列。

```sql
# 提取端口（regexp_substr 函数需要 MySQL 8.0 以上） 
UPDATE fscan SET port=substr(regexp_substr(url,':\\d{2,5}',8,1),2); 
# 上面正则无法提取端口的就是80或者443
UPDATE fscan SET port=80 WHERE port is NULL AND instr(url,'http://')=1;  
UPDATE fscan SET port=443 WHERE port is NULL AND instr(url,'https://')=1;
```

#### 去重

根据（title, status_code, ip）去重得到结果

```sql
# 去重查询所有列  
SELECT DISTINCT f1.*
FROM fscan f1
 JOIN (SELECT min(url) AS min_url
       FROM fscan
       GROUP BY title, status_code, ip) f2
WHERE f1.url = f2.min_url
ORDER BY url;
```

## 6. 漏洞扫描

### xray

xray支持在一次扫描中同时导出html和json格式的报告。
扫描完成后，如果是html格式的报告，可以通过以下脚本转换成专用json格式，然后用navicat导入数据库。

[xray_Html2Json1.py](assets/xray_Html2Json1-20230901175717-gm1vrwd.py)

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309012023694.png)

```sql
#更新公司名称
UPDATE xray_report 
SET company =(
	SELECT enscan.company FROM enscan 
	WHERE xray_report.target LIKE CONCAT( '%', enscan.addr, '%' ) 
	LIMIT 1 
	);
```

‍
