Reverse shell listen:
nc -nlvp \[PORT]
	 -l 用于告诉 netcat 这将是一个侦听器
    -v 用于请求详细输出
    -n 告诉 netcat 不要解析主机名或使用 DNS。
    -p 表示将遵循端口规范。
Netcat Shell Stabilisation：

