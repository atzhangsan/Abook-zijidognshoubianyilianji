```
如何调用so库文件
96  飞扬小米 
2017.07.11 13:19* 字数 837 阅读 31评论 3喜欢 1
制作so文件
首先先制作制作so文件：libadd_c.so
[ add.c]

int add(int a, int b) {  
    return a + b;  
}  
编译：

gcc -shared -fpic -o libadd_c.so add.c
-shared 生成共享目标文件，通常用在建立共享库时
-fpic 作用于编译阶段，告诉编译器产生与位置无关代码(Position-Independent Code)，则产生的代码中，没有绝对地址，全部使用相对地址，故而代码可以被加载器加载到内存的任意位置，都可以正确的执行。
这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。
参考：gcc-4.9.4命令文档

执行编译任务，输出相应的libadd_c.so文件。

编写测试函数
[ test.cpp]

#include <stdio.h>  
#include <dlfcn.h>  
#include <stdlib.h>  
#include <iostream>  
using namespace std;  
int main()  
{  
    int a = 0;  

    void *handle = dlopen("./libadd_c.so", RTLD_LAZY);  

    if(!handle)  {   
        printf("open lib error\n");  
        cout<<dlerror()<<endl;  
        return -1;  
    }   

    typedef int (*add_t)(int a, int b);  
    add_t add = (add_t) dlsym(handle, "add");  
    if(!add)  {   
        cout<<dlerror()<<endl;  
        dlclose(handle);  
        return -1;  
    }   

    a = add(4, 4);  
    printf("a = %d\n",a);  
    dlclose(handle);  
    return 0;  
}
编译：

g++ test.cpp -ldl -o test
运行

./test
输出为：8
注意：typedef int (*add_t)(int a, int b);声明一个函数指针。
首先你要明白函数指针的概念
int *p(int ,int );//声明一个函数
int (*p)(int ,int);//声明一个函数指针
typedef int(*add_t)(int, int);

就是把这个类型的函数指针的声明变为add_t

在使用动态链接库libadd_c.so文件的时候，主要使用到了四个函数：
(1) dlopen()

函数原型：void *dlopen(const char *libname,int flag);
功能描述：dlopen必须在dlerror，dlsym和dlclose之前调用，表示要将库装载到内存，准备使用。如果要装载的库依赖于其它库，必须首先装载依赖库。如果dlopen操作失败，返回NULL值；如果库已经被装载过，则dlopen会返回同样的句柄。
参数中的libname一般是库的全路径，这样dlopen会直接装载该文件；如果只是指定了库名称，在dlopen会按照下面的机制去搜寻：
a.根据环境变量LD_LIBRARY_PATH查找
b.根据/etc/ld.so.cache查找
c.查找依次在/lib和/usr/lib目录查找。
flag参数表示处理未定义函数的方式，可以使用RTLD_LAZY或RTLD_NOW。RTLD_LAZY表示暂时不去处理未定义函数，先把库装载到内存，等用到没定义的函数再说；RTLD_NOW表示马上检查是否存在未定义的函数，若存在，则dlopen以失败告终。

(2) dlerror()

函数原型：char *dlerror(void);
功能描述：dlerror可以获得最近一次dlopen，dlsym或dlclose操作的错误信息，返回NULL表示无错误。dlerror在返回错误信息的同时，也会清除错误信息。
(3) dlsym()

函数原型：void *dlsym(void *handle, const char *symbol);
功能描述：在dlopen之后，库被装载到内存。dlsym可以获得指定函数(symbol)在内存中的位置(指针)。如果找不到指定函数，则dlsym会返回NULL值。但判断函数是否存在最好的方法是使用dlerror函数。
注意：个人感觉从dlsym()寻找到内存中指针位置这个操作还是很麻烦的。

(4) dlclose

函数原型：int dlclose(void *);
功能描述：将已经装载的库句柄减一，如果句柄减至零，则该库会被卸载。如果存在析构函数，则在dlclose之后，析构函数会被调用。
```
