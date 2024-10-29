- [linux C 编程：重写 memcpy 函数-CSDN 博客](https://blog.csdn.net/wangquan1992/article/details/108348864)
- [C 语言面试重写 memcpy 函数-CSDN 博客](https://blog.csdn.net/buhuidage/article/details/121368266)

# 一、memcpy()源码实现

### 1.1 函数原型

拷贝 src 所指向地址存储内容的前 n 个字节到 dst 指向地址上
变量存储的地址为低地址，往高处存储(如 32 位的 int 型变量 a 地址为 0x0000，所占用地址为`0x0000 ~ 0x0003`)

```c
void *memcpy(void *dst, const void *src, size_t n);
```

注意：

- 需复制的字节数 n 大于 src 或 dst 存储的内容大小，该函数不会进行判断
- 该函数不检查 dst 和 src 指向的数组或其他类型是否具有同样的空间大小
- 若 src 和 dst 存在地址重叠(地址向前重叠/向后重叠)，则该函数也会出现问题
  - 向前重叠：`src > dst && dst + n > src`(可以正确拷贝，但是 dst 的内容会覆盖 src 的内容) --> **src 地址高于 dst 地址，==src 前面会被重叠==，从前向后拷贝**
  - 向后重叠：`dst > src && src + n > dst`(无法正确拷贝，因为 src 后面的数据被 dst 存储的数据覆盖掉了) -> 此时可使用 memmove()代替 memcpy() --> **dst 地址高于 src 地址，==src 后面会被重叠==，从后向前拷贝**

![](https://s2.loli.net/2024/07/26/lqWtHDMCxIPv1Xh.png)
![](https://s2.loli.net/2024/07/26/wPMaWgRxmbjQcsS.png)

### 1.2 源码实现

```c
/**
 * memcpy - Copy one area of memory to another
 * @dest: Where to copy to
 * @src: Where to copy from
 * @count: The size of the area.
 *
 * You should not use this function to access IO space, use memcpy_toio()
 * or memcpy_fromio() instead.
 */
void *memcpy(void *dest, const void *src, size_t count)
{
	char *tmp = dest;
	const char *s = src;

	while (count--)
		*tmp++ = *s++;
	return dest;
}

```

# 二、重写 memcpy()

```cpp
void *memcpy_func(void *dst, const void *src, size_t n)
{
	if(dst == NULL || src == NULL || n <= 0)
	{
		return NULL;
	}

	char *tmp = (char *)dst;
	char *s = (char *)src;

	if((s + n > tmp) && (tmp > s)) // 地址向后重叠 从高地址往低地址复制
	{
		tmp += n;
		s += n;
		while(n--)
		{
			*tmp-- = *s--;
		}
	}
	else // 没有地址重叠或存在向前重叠 从低地址往高地址复制
	{
		while(n--)
		{
			*tmp++ = *s++;
		}
	}
	return dst;
}
```

- 重写 memcpy()函数可以==避免地址向后重叠导致从前到后无法正确拷贝==的问题
- 重写 memcpy()函数与 Linux C 自带 memcpy()函数也无法避免地址向前重叠导致的 src 存储内容被覆盖的问题
