### 							     	strtok的用法

1. strtok的头文件

   str函数头文件基本都为<string.h>

   2.strtok的用法

```c
char *strtok(char *str, const char *delim)

```

​	str为要被分解的字符串，delim为你想要的分隔符

​	strtok函数是将分隔符的位置改为\0，然后返回被分割出的部分，并在记录当前的位置，便于下一次调用继续分割，如要继续分割，此时传入的strtok函数的第一个参数为空指针，并在上一次的位置开始分割



```c
#include<stdio.h>
#include<string.h>
int main()
{
char email[] = "1439986523@qq.com";
char cp[30] = {0};
char sep[] = "@.";
strcpy(cp, email);//由于strtok函数会对原本的字符串进行修改，所以要单独拷贝一份作为修改。
char* ret = NULL;

ret =strtok(cp, sep);
printf("%s\n", ret);

ret=strtok(NULL, sep);
printf("%s\n", ret);

ret=strtok(NULL, sep);
printf("%s\n", ret);
}
```

演示如下

![image-20241024225236686](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20241024225236686.png)

上述代码就有些单一，因为我们在使用时也不知道要切割几次，有以下改进

```c
#include<stdio.h>
#include<string.h>
int main()
{
	char email[] = "1439986523@qq.com";
	char cp[30] = {0};
	char sep[] = "@.";
	strcpy(cp, email);
	char* ret = NULL;
	for (ret = strtok(cp, sep); ret != NULL; ret = strtok(NULL, sep))
	{//for循环初始化第一次传入参数，如果ret不为空指针，则继续循环，传递空指针给strtok函数，直到最后全部分割完毕，退出循环
		printf("%s\n", ret);
	}

		return 0;
}
```

![image-20241024225236686](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20241024225236686.png)

这是效果。
