# 系统调用
```c
//文件描述符,当打开一个文件时,会有一个文件描述符返回
//#define STDIN_FILENO   0  //标准输入的文件描述符
//#define STDOUT_FILENO  1  //标准输出的文件描述符
//#define STDERR_FILENO  2  //标准错误的文件描述符
1. int fork()
//创建进程, 在子进程和父进程中都会有返回值,在子进程中返回0,在父进程中返回子进程的pid
2. int exit(int status)
//退出当前进程,status会被返回给父进程,如果status=0表示正常退出,非0表示异常退出,等效于main里面的return 0
3. int wait(int *status)
//在父进程等待子进程退出,返回子进程pid
4. int kill(int pid)
//终止某个进程,成功返回0,否则返回-1
5. int getpid()
//获取当前进程的pid
6. int sleep(int n)
//以阻塞方式暂停进程n哥ticks
7. int exec(char *file, char *argv[])
//加载某个文件,并以指定参数执行它
8. char *sbrk(int n)
//申请n个字节的内存空间,返回地址
9. int open(char *file, int flags)
//打开某个文件,file文件名,flag代表读或者写[O_RDONLY, O_WRONLY, O_RDWR],返回文件描述符
10. int write(int fd, char *buf, int n)
//向指定文件中写入n个字节的数据,返回写入的字节个数
11. int read(int fd, char *buf, int n)
//从指定文件中读取n个字节的数据,返回读取到的字节个数,返回0代表读到了文件尾部
12. int close(int fd)
//关闭已经打开的文件
13. int dup(int fd)
//为同一个文件起另一个文件描述符
14. int pipe(int p[])
//创造一个管道,输入为二大小的数组,其中[0]存取读的文件描述符,[1]存取写的文件描述符,只能单向
15. int chdir(char *dir)
//更改当前目录
16. int mkdir(char *dir)
//创建一个新目录
17. int mknod(char *file, int, int)
//创建一个设备文件
18. int fstat(int fd, struct stat *st)
//获取文件信息,使用文件描述符来访问
19. int stat(char *file, struct stat *st)
//获取文件状态信息,使用文件名来访问
20. int link(char *file1, char *file2)
//为文件创建一个硬链接,成功返回0,失败返回-1
21. int unlink(char *file)
//删除一个文件(软硬链接文件)
//软链接和硬链接的区别
//软链接:软链接会创建一个链接文件,该文件的inode与源文件不同,但里面会存储源文件的inode,对软链接文件的操作也会指向源文件,软链接可以指向不同的文件系统
//硬链接:硬链接创建一个具有相同inode的文件.
```
# Lab 1
## sleep
```c
#include "kernel/types.h"
#include "user/user.h"
//包含必要的头文件以调用系统调用
int main(int argc, char *argv[])
{//argc传递的参数个数,默认第一个参数为执行文件
	if (argc != 2)
	{
		fprintf(2, "Usage: sleep tick\n");
	}
	sleep(atoi(argv[1])); //atoi 将字符串转换为整数

	exit(0);//equal to return 0
}
```
## pingpong
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char *argv[])
{//在两个进程间进行通过pipe进行通信
	int fd1[2]; //创建一个数组来存储读写文件描述符号
	int fd2[2];
	char data = 'a'; //就以单字节形式进行交流

	pipe(fd1);
	pipe(fd2);

	if (fork() == 0)
	{//在子进程中,fd2
		close(fd1[1]); //只读
		close(fd2[0]); //只写
		if (read(fd1[0], &data, sizeof(data)))
		{
			printf("received ping.\n");
			write(fd2[1], &data, 1);
		}
		else
		{
			exit(1);
		}

		close(fd1[1]);
		close(fd2[0]);
		exit(0);
	}
	else
	{//在父进程中
		close(fd1[0]); //只写
		close(fd2[1]); //只读

		write(fd1[1], &data, 1);
		read(fd2[0], &data, 1);
		printf("received pong.\n");
	}

	exit(0);
}
```
## primes
```c
#include "kernel/types.h"
#include "user/user.h"

int SubProcessHandler(int *leftFd);

int main(int argc, char *argv[])
{
	int fd[2];
	int i = 2;
	pipe(fd);
	if (fork() == 0)
	{
		SubProcessHandler(fd);
	}
	else
	{
		close(fd[0]);
		for (;i < 36;i++)
		{
			write(fd[1], &i, 4);
		}
		close(fd[1]);
		wait(0);
	}
	exit(0);
}

int SubProcessHandler(int *leftFd)
{
	int rightFd[2];
	int firstNum = 0;
	int lastNum = 0;
	close(leftFd[1]);
	if (read(leftFd[0], &firstNum, 4))
	{
		printf("prime %d\n", firstNum);
		pipe(rightFd);
		if (fork() == 0)
		{
			SubProcessHandler(rightFd);
		}
		else
		{
			close(rightFd[0]);
			while (read(leftFd[0], &lastNum, 4))
			{
				if (lastNum % firstNum != 0)
				{
					write(rightFd[1], &lastNum, 4);
				}
			}
			close(rightFd[1]);
			wait(0);
		}
	}
	exit(0);
}
```
## find
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "kernel/fs.h"
#include "user/user.h"

/*

伪代码
find（path， file）
    if (path)
    {
        1.file 直接退出
        2.path
        {
            遍历每个目录项
            {
                1.file 判断是不是想要的文件
                2.path 继续递归
            }
        }  
    }
*/

  

void find(char *pathName, char *fileName)
{
    char buf[512]; //用来缓存路径名
    char *p;
    int fd, fd1;
    struct dirent de; //用来保存目录信息
    struct stat st, st1; //用来记录文件信息
  
    if ((fd = open(pathName, 0)) < 0)
    {//如果打开路径失败
        fprintf(2, "path error.\n");
        exit(1);
    }
  

    if (fstat(fd, &st) < 0)
    {//如果文件信息录入错误,有可能是无效文件
        fprintf(2, "stat error.\n");
        close(fd);
        exit(1);
    }
 

    switch (st.type)
    {
        case T_FILE://如果指向了文件
            fprintf(2, "path error.\n");
            exit(1);
        case T_DIR:
            strcpy(buf, pathName); //加载路径名称
            p = buf + strlen(buf); //定位到当前路径末尾；
            *p++ = '/'; //在末尾加上文件切换符号

            while (read(fd, &de, sizeof(de)) == sizeof(de))
            {//遍历搜索目录
                if (de.inum == 0)
                {//如果读到了无效文件，继续往下遍历
                    continue;
                }

                if (!strcmp(de.name, ".") || !strcmp(de.name, ".."))
                {//如果是. 或者 ..则忽略
                    continue;
                }

                memmove(p, de.name, DIRSIZ); //将遍历到的文件名加上去
                if ((fd1 = open(buf, 0)) >= 0)
                {//打开新的文件
                    if (fstat(fd1, &st1) >= 0)
                    {//获取新的文件信息
                        switch (st1.type)
                        {
                            case T_FILE:
                                if (!strcmp(de.name, fileName))
                                {
                                    printf("%s\n", buf);//如果文件名一致输出
                                } 

                                close(fd1);
                                break;
                            case T_DIR:
                                close(fd1);
                                find(buf, fileName);//如果还是目录递归
                                break;
                            default:
                                close(fd1);
                        }
                    }
                }
            }
    }
    close(fd);
}

int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        fprintf(2, "Usage: find path file\n");
    }

    find(argv[1], argv[2]); 
    exit(0);
}
```
## xargs
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/param.h"
  
//将标准输入中的数据当作参数执行
int main(int argc, char *argv[])
{
    char stdIn[512];
    int size = read(0, stdIn, sizeof(stdIn));
    int i = 0, j = 0;
    int line = 0;

    for (int k = 0;k < size;k++)
    {
        if (stdIn[k] == '\n')
        {
            line++; //统计标准输入中的行数
        }
    }  

    char output[line][64];
    for (int k = 0;k < size;k++)
    {//将每行分开存储
        output[i][j++] = stdIn[k];
        if (stdIn[k] == '\n')
        {
            output[i][j-1] = '\0';
            i++;
            j = 0;
        }
    }  

    char *arguments[MAXARG]; //定义数组暂存参数
    for (j = 0;j < argc-1;j++)
    {
        arguments[j] = argv[j+1]; //先复制已有参数
    }  

    i = 0;
    while (i < line)
    {
        arguments[j] = output[i++]; //把每行参数贴到命令最后面
        if (fork() == 0)
        {
            exec(argv[1], arguments); //exec需要可执行文件名 和 参数数组
            exit(0);
        }
        wait(0);//创建子进程执行每个命令
    }
    exit(0);
}
```
