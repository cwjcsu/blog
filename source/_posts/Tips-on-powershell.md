title: 我用Powershell遇到的那些坑
date: 2018-02-05 15:18:20
tags:
- powershel
categories:
- 编程
description: 使用powershell遇到的坑

---

行云管家的Proxy和Agent在windows下的安装脚本是用powershell编写的，到现在已经28个小版本了。Proxy的bug，40%是来自powershell，40%来自bat,剩下20%来自java代码。
<!--more-->

# 下载文件的进度条把下载速度降低了10倍
**要求：**用powershell下载一个文件到指定位置，并且显示进度条。
    网上去搜索应该有一大堆代码，我也是参考了其中一些。代码片段如下：
```
function DownloadFile($url, $targetFile){
   $uri = New-Object "System.Uri" "$url"
   $request = [System.Net.HttpWebRequest]::Create($uri)
   $request.set_Timeout(10000) #10 second timeout
   $response = $request.GetResponse()
   ...
     $sw = [System.Diagnostics.Stopwatch]::StartNew()
   $delta=0
   while ($count -gt 0){
       $targetStream.Write($buffer, 0, $count)
       $count = $responseStream.Read($buffer,0,$buffer.length)
       $downloadedBytes = $downloadedBytes + $count
       $delta = $delta+$count
       if (($sw.Elapsed.TotalMilliseconds -ge 500) -or ($count -le 0) ) {#【1】
        #make it fast,avoid write-progress too often
           $percent=(([System.Math]::Floor($downloadedBytes/1024)) / $totalLength)  * 100
           $percentStr='{0:n1}' -f $percent
           $speed='{0:n2}' -f ($delta*1000/1024/$sw.Elapsed.TotalMilliseconds)
           Write-Progress -activity "Downloading file '$fileName'" -status "Downloaded ($([System.Math]::Floor($downloadedBytes/1024))K of $($totalLength)K):$percentStr% $speed KB/s" -PercentComplete $percent
           $sw.Reset(); $sw.Start();$delta=0
       }
   }
}
```
刚开始的代码【1】是没有`Stopwatch`计时器判断的，意味着每下载一个$buffer的数据之后，就会调用一次`Write-Progress`，而这个调用会在屏幕上移动一步进度条（如果精度够的话），这是一个UI操作，可以想象到这个过程必然是同步的。调用一两次耗时并无感知，但是当下载一个几十MB的文件，就要调用成千上万次（>100MB/32KB），而且跟IO操作的线程是串行的，所以会明显降低下载速度。
当初调试的时候，发现下载速度怎么也达不到该有的速度（100兆带宽11MB/s,千兆带宽30~80MB/s），最后找到罪魁祸首是`Write-Progress`调用过于频繁。
如上代码所示，降到500毫秒（或者1s）打印一次，就可以忽略`Write-Progress`带来的延迟，而界面显示的信息对人眼来说，也是实时更新的。
参见[How to improve the performance of Write-Progress?](https://stackoverflow.com/questions/21304282/how-to-improve-the-performance-of-write-progress)

# 递归删除非空目录偶尔失败且一直阻塞
先看代码：
```
if(test-path "$cloudGateway_folder\YunAgent"){
    Write-Host "Removing the old remaining directory $cloudGateway_folder\YunAgent"
    Remove-Item "$cloudGateway_folder\CloudGateway" -Recurse -Force
}
```
问题出在函数`Remote-Item`的参数`-Recurse` 上面。

这个bug有点奇葩：powershell的文档说`-Recurse `参数在这里无法正常工作，应该使用`Get-Childitem`加管道来替代。既然知道是bug，你倒是修复啊！经过测试发现，powershell 2.0~5.0都存在这个问题，微软不屑于修复这个bug。

另外，增加参数`-ErrorAction SilentlyContinue`并不能解决问题，但是会忽略错误，然后继续进行，很多时候这并不算你想要的。

我的替代方式是用cmd命令，应该足够稳定靠谱了吧。
```
&cmd.exe /c rd /s /q $cloudGateway_folder\YunAgent
```
这个bug不是必现的，所以你在开发测试阶段不一定能发现问题，这是最坑人的地方。
参见[Force-remove files and directories in PowerShell fails sometimes, but not always](https://serverfault.com/questions/199921/force-remove-files-and-directories-in-powershell-fails-sometimes-but-not-always)

# 未使用windows换行符导致脚本无法运行
众所周知，widnows、linux、mac三个大平台下的文件换行符都是各自一套的，powershell脚本(xxx.ps1)的换行符必须是windows换行符，即`CRLF`或者`\r\n`。这个看似一个很愚蠢的错误，其实不经意就会犯。

我们公司用idea做开发，我的开发及其是windows平台的。我编写powershell的时候，很多时候都是直接用idea打开修改，然后用git提交。git在windows下有时候配置为这样：
```
$ cat .git/config
[core]
        autocrlf=true
        ...
```
这样，代码checkout下来的时候，会自动添加CRLF换行符，而提交推送的时候，会去掉CR只保留LF。
在Linux下有一个简单的方法检测处理Windows换行符问题。使用vi -b 打开可以看到是否添加了CR字符。下面的示例`^M`表示CR字符
```
#:vi -b installGateway.ps1 
param(^M
[string]$importKey=$(throw "Parameter missing: -importKey key"),^M
[string]$repo=$(throw "Parameter missing: -repo repo"),^M
[string]$download_prefix=$(throw "Parameter missing: -download_prefix download_prefix"),^M
[int]$isAdmin=$(0)^M
)^M
$appName ="Cloudbility Gateway Service"^M
...
```
反过来，如果你的shell脚本以`\r\n`结尾，也是无法在linux下执行的。

# 变量`${env:ProgramFiles(x86)}`可能不存在
某些情况下，你需要使用ProgramFiles目录，通常是
```
C:\Program Files (x86)
or
C:\Program Files
```
在64位的windows下访问32位程序安装目录，用变量：`${env:ProgramFiles(x86)}`（注意必须用${}符号包裹， 因为变量名中有特殊字符`()`)，比如在我的64位windows下：
```
PS C:\> ${env:ProgramFiles(x86)}
C:\Program Files (x86)
PS C:\> ${env:ProgramFiles}
C:\Program Files
```
但是在32位下是没有变量`${env:ProgramFiles(x86)}`的，你的代码会出错。
这个坑一般要把你的代码放到32位机器上跑才会暴露，所以有时候也很难发现（现在32位windows已经越来越少了）。

# TODO more