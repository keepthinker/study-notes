## gradle命令
```bash
# 帮助命令
gradle --help

# 查看版本
gradle -v

# 执行特定的任务
gradle [taskName]

# 构建
gradle build

# 跳过测试构建构建
gradle build -x test

# 继续执行任务而忽略前面失败的任务
gradle build --continue

# 试运行build
gradle -m build

# 产生build运行时间的报告
gradle build --profile
结果存储在build/report/profile目录，名称为build运行的时间。

# 显示任务间的依赖关系
gradlle tasks --all

# 查看testCompile的依赖关系
gradle -q dependencies --configuration testCompile

# 清空所有编译、打包生成的文件(即：清空build目录)
gradle clean

# 使用指定的Gradle文件调用任务
gradle -b [file_path] [task]

# 使用指定的目录调用任务
gradle -q -p [dir] helloWorld

# Gradle的图形界面
gradle --gui

# Gradle的命令日志输出有ERROR（错误信息）、QUIET（重要信息）、WARNGING（警告信息）、LIFECYLE（进程信息）、 INFO（一般信息）、DEBUG （调试信息）一共6个级别。在执行Gradle任务是可以适时地调整信息输出等级，以方便地观看执行结果。

-q/--quit 启用重要信息级别，改级别下只会输出自己在命令行下打印的信息及错误信息。
-i/--info 会输出除DEBUG以外的所有信息。
-d/--dubug 会输出所有日志信息。
-s/--stacktrace 会输出详细的错误堆栈。
```

## 参考
[Gradle常用命令](https://www.cnblogs.com/three-fighter/p/12347208.html)