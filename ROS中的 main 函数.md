# 问题：main 函数形参代表什么意思
在写ros的c++代码的时候用到的 main 函数（int main( int argc, char* argv[] )）

带形参的main函数，如 main( int argc, char* argv[], char **env ) ，是UNIX、Linux以及Mac OS操作系统中C/C++的main函数标准写法，并且是血统最纯正的main函数写法。

# 参数含义

 argc和argv参数在用`命令行编译程序`时有用
 第一个参数，int型的argc，为整型，用来统计程序运行时发送给main函数的`命令行参数的个数`，在VS中默认值为1。   
 第二个参数，char* 型的 argv[]，为字符串数组，用来`存放指向的字符串参数的指针数组`，`每一个元素指向一个参数`。
	  argv[0]指向程序运行的全路径名   
	  argv[1]指向在DOS命令行中执行程序名后的第一个字符串   
	  argv[2]指向执行程序名后的第二个字符串   
	  argv[3]指向执行程序名后的第三个字符串   
	  argv[argc]为NULL

