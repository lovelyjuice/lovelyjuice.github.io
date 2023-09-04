<h1>无视杀软使用远控工具进行横向移动</h1>
<p>在有杀软拦截，CS 无法上线的情况下，经常用到 todesk 和向日葵这两个远控工具进行横向移动。不过这两个工具现在好像不怎么好用了。不过无所谓，用其他的就是了，听说最近 GotoHTTP 很火，可以试试看。</p>
<h2><strong>1、GoToHTTP 的优点和缺点</strong></h2>
<p><strong>优点：</strong></p>
<ul>
<li>B2C 模式，无需安装控制端软件，有浏览器就可以远控。</li>
<li>流量走 https 协议，只要目标放行 443 端口出口就可以实现内网穿透。</li>
<li>在低带宽也可以使用，运行占用内存极低，控制时占用 CPU 仅为 0%-3%。</li>
<li>被控端在类 Linux 系统上支持图形界面（GUI）和字符界面（CLI）。</li>
</ul>
<p><strong>缺点：</strong></p>
<ul>
<li>需要管理员权限运行</li>
<li>一些行为（网络唤醒远程主机）需要加载驱动，导致运行时 360 安全卫士会拦截这行为，其他杀软则不会拦截。</li>
</ul>
<h2><strong>2、GoToHTTP 使用</strong></h2>
<p>下载：<a href="http://www.gotohttp.com/goto/download.12x">http://www.gotohttp.com/goto/download.12x</a><br />
GoToHTTP 提供了多种系统的被控端，可以根据情况选用。</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855650.png" alt="" /></p>
<p>在本地虚拟机演示一下</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855972.png" alt="" /></p>
<p>将 gotohttp.exe 上传到指定目录</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855203.png" alt="" /></p>
<p>这里可以看到靶机直接弹出连接框了</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855452.png" alt="" /></p>
<p>在哥斯拉中，gotohttp.exe 目录下直接生成了配置文件</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855432.png" alt="" /></p>
<p>读取配置文件，可以获取电脑 ID 和控制码</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855295.png" alt="" /></p>
<p>直接连接</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855205.png" alt="" /></p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855558.png" alt="" /></p>
<h2><strong>1、RustDesk 介绍</strong></h2>
<p>看了精灵师傅的文章发现了另一个巨 TM 好用的远控工具，按照精灵师傅的话就是：它不仅有 GotoHTTP 的优点，还是免费开源的，而且还支持自建服务器，简直业界良心。</p>
<p>RustDesk 相比于 GoToHTTP 最大的优点就是：<strong>普通权限即可运行，而且支持纯内网环境</strong>。<br />
RustDesk 地址：<a href="http://rustdesk.com/zh/">http://rustdesk.com/zh/</a></p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855520.png" alt="" /></p>
<p>注意：RuskDesk 控制端和被控制端是同一个应用</p>
<h2><strong>2、RustDesk 使用</strong></h2>
<p>通过 webshell 上传 rustdesk</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855641.png" alt="" /></p>
<p>运行 rustdesk</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855235.png" alt="" /></p>
<p>靶机会弹框，这个不需要管，我们直接去找配置文件</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855003.png" alt="" /></p>
<p>配置文件地址：C:Users 用户名 AppDataRoamingRustDeskconfig</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855118.png" alt="" /></p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855546.png" alt="" /></p>
<p>获取 id 和密码</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855738.png" alt="" /></p>
<p>成功连接</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855824.png" alt="" /></p>
<h2><strong>3、踩坑实录</strong></h2>
<p>（1）要下载 RustDesk 的 portable 版本，不要下载 puts 版本。portable 版本免安装，puts 版本会弹框提示安装<br />
个版本下载地址：<a href="https://gitee.com/rustdesk/rustdesk/releases">https://gitee.com/rustdesk/rustdesk/releases</a></p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855722.png" alt="" /></p>
<p>（2）不知道是不是 RustDesk 的 bug 还是出于安全考虑，如果是第一次在目标上运行 RustDesk，RustDesk 不会立即将密码保存到配置文件，而此时你把鼠标放在显示密码时则会保存到文件，遇到这种情况我们只需运行 RustDesk，生成配置文件后结束 RustDesk 进程，然后修改配置文件里 password 为指定密码，再运行 RustDesk 即可。<br />
修改配置文件 password</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855499.png" alt="" /></p>
<p>重新启动 RustDesk</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855626.png" alt="" /></p>
<p>成功修改密码</p>
<h2><strong>4、RustDesk 在纯内网环境中的优势</strong></h2>
<p>有时候我们打点控了一台跳板机，在内网有一台服务器有 webshell 权限，但是这台机器又不出网，而且上面装了多个杀软，防护开到最严，你又不知道目标账号密码，这时候你许多操作都会受限，这种场景还是挺常见的。这时候如果能远程过去把杀软退掉可能是一种捷径，而 RustDesk 就支持在内网使用 IP 进行直连。</p>
<p>在 RustDesk 默认设置中，允许 IP 直接访问功能未开启，如下图：</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855442.png" alt="" /></p>
<p>针对这个问题，我们可以修改配置文件 RustDesk2.toml，在 options 下添加一行</p>
<p>然后重启 RustDesk 即可。</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855444.png" alt="" /></p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855187.png" alt="" /></p>
<p>RustDesk 的实现 IP 直接访问功能监听的默认端口是 21118，这个端口也是可以修改的，同样的步骤修改配置文件 RustDesk2.toml，在 options 下添加一行 direct-access-port = '8443'，然后重启 RustDesk 即可。</p>
<p>注意：这里修改默认端口是因为在一些场景下被控端防火墙需要放行入站方向的指定端口。</p>
<p>在控制端输入目标地址（如果是默认端口直接输入 IP，如果是自定义端口输入 IP:PORT 格式），然后输入密码即可远控。</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855551.png" alt="" /></p>
<p>成功连接</p>
<p><img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041855421.png" alt="" /></p>
<hr />
<p>来源：<a href="https://xz.aliyun.com/t/11557">https://xz.aliyun.com/t/11557</a></p>
<p>原创作者：C0ldCash</p>
<p>如有侵权，请联系删除</p>
<p><strong>本文作者：<strong>​</strong></strong></p>
<p>**本文为安全脉搏专栏作者发布，转载请注明：**​<a href="https://www.secpulse.com/archives/187087.html"><strong>https://www.secpulse.com/archives/187087.html</strong></a></p>
