[一文弄懂 container_of 的原理和实际应用](https://mp.weixin.qq.com/s/GX-QZ-a-bVCC6Z8P_Ue_XQ)

通过结构体中的某个**成员变量地址**找到该**结构体的首地址**

```c
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:    the pointer to the member.
 * @type:   the type of the container struct this is embedded in.
 * @member: the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({          \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```
