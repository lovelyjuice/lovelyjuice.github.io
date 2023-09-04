---
title: GraalVM 打包 Java ShellcodeLoader 为可执行文件
date: '2023-09-01 17:56:34'
updated: '2023-09-04 18:48:41'
permalink: /post/graalvm-package-java-shellcodeloader-is-executable-file-z1sjqca.html
comments: true
toc: true
---


# GraalVM 打包 Java ShellcodeLoader 为可执行文件

## 打包成 Jar 包

先上项目地址：https://github.com/yzddmr6/Java-Shellcode-Loader

下载后用 IDEA 打开，在项目结构里把 lib 目录添加到依赖库里面。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848101.png)

之后构建 Artifacts，打包成 jar 包。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848998.png)

## 安装 GraalVM

接下来开始把 jar 包打包成 exe 可执行文件。按照[这篇文章](https://hosch3n.github.io/2022/09/16/%E5%88%A9%E7%94%A8GraalVM%E5%AE%9E%E7%8E%B0%E5%85%8D%E6%9D%80%E5%8A%A0%E8%BD%BD%E5%99%A8/)里面的方法，我这里选择下载 Oracle 企业版的 GraalVM（貌似企业版的运行快一点？）链接如下：
https://www.oracle.com/downloads/graalvm-downloads.html
总共需要下载两个文件，第一个是 GraalVM 本体，下载后解压就能用。第二个 native image 就是用来打包 exe 的插件，下载后是个 jar 包。

**2023年8月8日更新：GraalVM JDK 20 之后的版本已经自带 native-image 插件，不需要额外下载了。**

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848727.png)

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848712.png)

下载之后，参考[官方教程](https://docs.oracle.com/en/graalvm/enterprise/22/docs/reference-manual/graalvm-updater/)离线安装 native image 插件，网速好的话也可以在线安装。

```
graalvm-ee-java11-22.3.1\bin\gu.cmd -L install native-image-installable-svm-svmee-java11-windows-amd64-22.3.1.jar
```

安装完成后 bin 目录下面会多出来 native-image 的一些 cmd 脚本，后面打包成 exe 就需要用到这些脚本。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848397.png)

到了这一步，就能够用 native-image 打包一些简单的 java 项目了，但是由于 ShellcodeLoader 的依赖库 tinyjna.jar 用到了不少反射相关的技术，直接打包的话 GraalVM 是不知道要把哪些动态加载的类打包进去的，直接打包的话会提示由于使用了反射且未指定配置文件，无法生成 stand-alone 形式的 image，而是会生成一个 fallback 的 image。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848064.png)

因此需要帮助 GraalVM 找到需要在运行时动态加载的类。这里的步骤参考 [Build a Native Executable with Reflection](https://docs.oracle.com/en/graalvm/enterprise/22/docs/reference-manual/native-image/guides/configure-with-tracing-agent/)，通过运行时加载 agent 的方式监控反射用到的类，然后把这些类的名称保存到一个配置文件中，打包的时候再根据配置文件将这些类打包到 exe 里面。

在 ShellcodeLoader 的同级目录新建文件夹`META-INF\native-image`，然后运行 ShellcodeLoader，这里的 shellcode 可以随便填，比如我填的 abcdef，执行不了也无妨，只是为了让 agent 知道反射调用了哪些类。

```
graalvm-ee-java11-22.3.1/bin/java -agentlib:native-image-agent=config-output-dir=META-INF/native-image -jar ShellcodeLoader.jar abcdef
```

运行之后就会在上面新建的目录里面生成配置文件。

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309041848211.png)

接下来就是通过加载配置文件打包成 exe 了，在此之前需要注意一下，由于打包成 exe 需要使用 msvc 生成工具，否则会提示找不到 cl.exe。所以需要安装 msvc 并且通过开始菜单启动专用的控制台。

PS: 第一次打包我没用专用控制台，而是把 msvc 的 bin 目录加到环境变量里面，虽然找到了 cl.exe 但后续还是报错了。

**2023年8月8日更新： GraalVM JDK 20 之后的版本可以在 Windows 上可以自动设置构建环境，不再需要在 x64 Native Tools Command Prompt 中运行。但是打包的时候需要添加额外参数**​**`-H:-CheckToolchain`**

[GraalVM JDK 20 更新日志](https://www.oschina.net/news/245539/graalvm-jdk20-summer-2023-)

![](https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309040911063.png)

启动专用控制台之后切换到 ShellcodeLoader.jar 所在的目录，执行打包命令，这里必需添加 `-cp .` 参数，表示将当前目录添加到 classpath，否则会找不到配置文件。

```
graalvm-ee-java11-22.3.1/bin/native-image.cmd -jar ShellcodeLoader.jar -cp .
```

**如果使用  GraalVM JDK 20 以上版本**，且不在x64 Native Tools Command Prompt 中运行，则需要添加额外参数：

```
graalvm-ee-java11-22.3.1/bin/native-image.cmd -jar ShellcodeLoader.jar -H:-CheckToolchain -cp .
```

打包完成后会生成如下三个文件，那个 dll 不知道有什么用，删除了也不影响 exe 正常运行。

```
--------------------------------------------------------------
Produced artifacts:
 D:\Java-Shellcode-Loader\out\artifacts\ShellcodeLoader_jar\ShellcodeLoader.build_artifacts.txt (txt)
 D:\Java-Shellcode-Loader\out\artifacts\ShellcodeLoader_jar\ShellcodeLoader.exe (executable)
 D:\Java-Shellcode-Loader\out\artifacts\ShellcodeLoader_jar\sunmscapi.dll (jdk_lib)
=====================================
Finished generating 'ShellcodeLoader' in 45.8s.
```

免杀效果还是不错的，VT 全过，就是生成的 exe 太大了，足足 15M，甚至比[精简过的 JRE](https://yzddmr6.com/posts/litejre-for-shellcode-loader/) 还大。

还试了下将 msf 的`java/meterpreter/reverse_tcp`生成的 jar 包转 exe，结果也是 15M 的大小，而且还运行报错

> Meterpreter session 4 is not valid and will be closed

后续或许可以参考精简 JRE 的思路把 GraalVM 的几个 jar 包也精简一下。
Oracle 官方也提供了减少镜像大小的一种方式：https://docs.oracle.com/en/graalvm/enterprise/22/docs/reference-manual/native-image/guides/use-graalvm-dashboard/

参考：
https://blog.csdn.net/xianzhan_/article/details/108181095
https://github.com/yzddmr6/Java-Shellcode-Loader
[精简JRE,打造无依赖的Java-ShellCode-Loader](精简JRE,打造无依赖的Java-ShellCode-Loader.md)

‍
