# PHP
1. \=\= 与 \=\=\=
	\=赋值 \=\=不会比对两边的类型(弱比较/强制转换类型) \=\=\=会比对两边的类型(强比较/不转换，比对类型)
	$a='1';
	a\=\=+1\=\=1.0\=\=1shift 成立
	a\=\=\=+1\=\=\=1.0不成立
	如果传入数组比较，其是不支持的，就会将数组识别为NULL(在两边参数可控且为强等于时可以使用这个绕过)

2. md5
	![[Pasted image 20240517204530.png]]
	第一种解法，使用两个md5值为0e开头的(PHP会识别为科学计数法)，\=\=就会把前面的0e比对成功，即可绕过。
	![[Pasted image 20240517204713.png]]
	第二种解法，利用上面PHP比较不支持数组的特点，传入两个数组，两边为NULL判相等，即可绕过。
	![[Pasted image 20240517205122.png]]
3. intval
	![[Pasted image 20240517205453.png]]
	![[Pasted image 20240517205727.png]]
4. strpos
	![[Pasted image 20240517205856.png]]
	![[Pasted image 20240517205911.png]]
	![[Pasted image 20240517210044.png]]
5. in_array
	![[Pasted image 20240517210227.png]]
	![[Pasted image 20240517210204.png]]
	strict不设置True时是弱比较与\=\=绕过同理。
	设置True还会比较类型。
6. preg_match
	![[Pasted image 20240517210437.png]]
	preg_match只能处理字符串，传数组等就会报错。
7. str_replace
	![[Pasted image 20240517210808.png]]
	它无法迭代检测
	如果传se==select==lect,中间的select会被替换，但是余下部分仍是select，无法再次检测。