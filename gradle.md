## gradle命令
### 帮助命令
gradle --help

### 查看版本
gradle -v

### 执行特定的任务
gradle [taskName]

### 构建
gradle build

### 跳过测试构建构建
gradle build -x test

### 继续执行任务而忽略前面失败的任务
gradle build --continue

### 试运行build
gradle -m build

### 产生build运行时间的报告
gradle build --profile  
结果存储在build/report/profile目录，名称为build运行的时间。

### 显示任务间的依赖关系
gradle tasks --all

### 查看app子工程下的任务
./gradlew :app:tasks

### 查看testCompile的依赖关系
gradle -q dependencies --configuration testCompile

- `->` 标识代表 依赖冲突，也是在 dependencies graph 中最常见的一种标识。比如 1.3.2 -> 1.6.0，表示当前依赖树中依赖的版本是 1.3.2，但由于全局的依赖冲突，最终被升级到了 1.6.0 版本。gradle 处理依赖冲突的总体原则是取冲突中的最高版本，但有很多特例。
- `(*)` 在前面的 demo 里，(*) 这个标识已经出现过了，这个标识由于跟常规语境下的含义不太一样，所以也具有一定的迷惑性。(*) 符号字面意思是省略显示，仅仅是展示层面的删减。
- `(c)` 这个标识对应 dependecy constraint，这部分逻辑的具体解释可以参考 这个章节，它对应 gradle 中的 constraints
- `(n)` - Not resolved (configuration is not meant to be resolved)


> 参考： [读懂 gradle dependencies](https://juejin.cn/post/7176466452558872613)

### 清空所有编译、打包生成的文件(即：清空build目录)
gradle clean

### 使用指定的Gradle文件调用任务
gradle -b [file_path] [task]

### 使用指定的目录调用任务
gradle -q -p [dir] helloWorld

### Gradle的图形界面
gradle --gui

Gradle的命令日志输出有ERROR（错误信息）、QUIET（重要信息）、WARNGING（警告信息）、LIFECYLE（进程信息）、 INFO（一般信息）、DEBUG （调试信息）一共6个级别。在执行Gradle任务是可以适时地调整信息输出等级，以方便地观看执行结果。

-q/--quit 启用重要信息级别，改级别下只会输出自己在命令行下打印的信息及错误信息。
-i/--info 会输出除DEBUG以外的所有信息。
-d/--dubug 会输出所有日志信息。
-s/--stacktrace 会输出详细的错误堆栈。

### 查看子工程app的jar包依赖关系
gradle :app:dependencies

### 先 clean 后 build
./gradlew :app:clean :app:build

### 查看有多少子工程
gradle projects



## dependency Configuration

Configuration Name	| Description |	Used to:
| --- | --- | --- | 
| api | Dependencies required for both compilation and runtime, and included in the published API. | Declare Dependencies |
| implementation | Dependencies required for both compilation and runtime. | Declare Dependencies |
| compileOnly | Dependencies needed only for compilation, not included in runtime or publication. | Declare Dependencies |
| compileOnlyApi | Dependencies needed only for compilation, but included in the published API. | Declare Dependencies |
| runtimeOnly | Dependencies needed only at runtime, not included in the compile classpath. | Declare Dependencies |
| testImplementation | Dependencies required for compiling and running tests. | Declare Dependencies |
| testCompileOnly | Dependencies needed only for test compilation. | Declare Dependencies |
| testRuntimeOnly | Dependencies needed only for running tests. | Declare Dependencies |

## Part 1: Initializing the Project

```bash

mkdir authoring-tutorial
cd authoring-tutorial
# Run gradle init with parameters to generate a Java application:
gradle init --type java-application  --dsl groovy

```




## 参考
[Gradle常用命令](https://www.cnblogs.com/three-fighter/p/12347208.html)
[Gradle tutorial](https://docs.gradle.org/current/userguide/partr1_gradle_init.html)
[Gradle’s Reference Materials](https://docs.gradle.org/current/userguide/part7_gradle_refs.html)