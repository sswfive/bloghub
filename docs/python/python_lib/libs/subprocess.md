# subprocess -> 子进程管理


> Reference：[官方文档](https://docs.python.org/zh-cn/3.12/library/subprocess.html)

## 模块概述
### 介绍：
subprocess是Python内置库中用于管理子进程的模块，支持在Windows、Linux、MacOS平台中使用； 

该模块通过创建启动一个子进程来执行系统级命令、调用其他的可执行文件或脚本、与其他进程进行交互等等；其主要通过管道技术连接进程间的输入/输出/错误管道，从而获得返回值。

在subprocess模块发布后，官方建议使用该模块去替换标准库中的os.system()、os.spawn*()等方法去执行系统命令；

### 版本更新

1. subprocess是python2.4中新增的模块；
2. 在python3.5之前，可以通过subprocess.call()，subprocess.getoutput()、subprocess.check_output()等等方法实现模块功能；
3. 在python3.5之后，新增subprocess.run()方法，并且官方文档建议通过subprocess.run()或者subprocess.Popen()来替换原有的老版本的功能函数；
## 应用场景
### 执行系统命令

- 在程序内部执行系统级命令，如ls、kill、mkdir、awk等，可以使用subprocess实现；
### 调用其他的可执行文件或脚本

- 在程序内部执行其他的执行文件或脚本，可以使用subprocess实现；
### 与其他的进程交互

- 在需要与其他进程进行交互，比如向某个进程发送数据、从某个进程读取数据等，可以使用subprocess实现；
### 实现异步任务

- 在程序内部中需要实现异步任务，可以使用subprocess实现；
### 替代os.system()

- 不想使用os.system()或os.system()无法实现需求时，可以subprocess实现；  

## 常用接口
### 接口总览
![Alt text](image.png)

### 重要接口
| **接口类别** | **接口方法名** | **功能描述** | **返回结果** |
| --- | --- | --- | --- |
| 高阶接口 | subprocess.run() | 执行命令并等待命令完成，返回一个对象，包含了命令的输出信息 | 阻塞等待；返回一个对象，包含状态码、输出信息（stdin/stdout/stderr） |
| 等层接口 | subprocess.Popen() | 执行命令不等待完成，返回一个对象 | 非阻塞等待；返回一个对象，包含状态码、输出信息（stdin/stdout/stderr） |
| 低版本接口（基本已被高阶的run()方法替代了） | subprocess.call() | 执行命令，返回命令的结果和执行状态 | 阻塞等待；0或者非0 |
|  | subprocess.check_call() | 执行命令，返回结果和状态 | 阻塞等待；正常为0 ，执行错误则抛出异常 |
|  | subprocess.getstatusoutput() | 接受字符串形式的命令，返回 一个元组形式的结果， | 阻塞等待；第一个元素是命令执行状态，第二个为执行结果。 |
|  | subprocess.getoutput() | 接收字符串形式的命令，返回结果 | 阻塞等待；返回执行结果 |
|  | subprocess.check_output() | 执行命令，如何执行状态码为0则返回命令的结果，否则抛出异常 | 阻塞等待；0或抛出异常 |

## 核心接口分析之subprocess.run()
### 函数特点：

   - 输入：默认情况下，子进程会继承父进程的设置，会将输出显示在终端上
   - 输出：阻塞等待，执行成功returncode为0， 非0，表示执行异常
```python
subprocess.run(
    args,   # 要执行的shell命令，默认应该是一个字符串序列，如[‘df’, ‘-Th’]或(‘df’, ‘-Th’)，也可以是一个字符串，如’df -Th’，但是此时需要把shell参数的值置为True；
    *, 
    stdin=None, # 子进程的标准输入。有三个参数可选：subprocess.PIPE 创建一个管道，允许与子进程进行通信；subprocess.DEVNULL 特殊的文件对象，可以将其用于丢弃子进程的输出；一个打开的文件对象，将内容写入文件
    input=None, # 将参数传递给 Popen.communicate() 以及子进程的 stdin,允许将字节或字符串传递给子进程的标准输入(stdin);
    stdout=None, # 子进程的标准输出(stdout);
    stderr=None, # 子进程的标准错误(stderr);
    capture_output=False, #参数控制是否捕获外部命令的标准输出(stdout)和标准错误(stderr)。如果将其设置为True，run()函数将返回一个CompletedProcess对象，该对象具有stdout和stderr属性，分别存储了命令的标准输出和标准错误输出。如果设置为False，标准输出和标准错误将被发送到控制台。默认为False
    shell=False,  # 指定是否通过shell来执行命令。如果为True，命令将在shell中执行；如果为False，则直接调用可执行文件；
    cwd=None,  # 设置子进程的工作目录。默认为None，表示使用当前工作目录；
    timeout=None, #设置子进程的超时时间（秒）。如果子进程在指定的时间内没有运行完成，则会引发TimeoutExpired异常；
    check=False, # 设置是否检查子进程的返回码。如果为True，并且子进程的返回码不为零，则会引发CalledProcessError异常；
    encoding=None, # 默认是字节数据，如果指定了编码格式，则输出为字符串；
    errors=None, # 该参数定义在解码输出时如何处理编码错误，常用的值包括"strict" (默认值，抛出异常)、"ignore" (忽略错误字符) 和 "replace" (用替代字符代替错误字符);
    text=None,  # 指定是否将输出结果以文本形式返回。如果为True，则结果以字符串形式返回，同时input或者stdin参数也需要输入String；如果为False，则返回字节流。默认为False。
    env=None, # 该参数允许您为子进程指定环境变量。它可以接受一个字典类型的对象，其中键是环境变量的名称，值是环境变量的值。通过设置env参数，可以在子进程中使用特定的环境变量。
    universal_newlines=None, # 该参数影响的是输入与输出的数据格式，比如它的值默认为False，此时stdout和stderr的输出是字节序列；当该参数的值设置为True时，stdout和stderr的输出是字符串。
    **other_popen_kwargs
)
```
### 重要参数：args

   - 可以接收两种方法：字符串或列表。
   - 使用列表形式subprocess.run(["ls", "-al"])
   - 使用字符串形式 subprocess.run("ls -al", shell=True)。使用字符串形式必须设置参数shell=True
### 重要参数：stdin、stdout、sterr

   - 用来设置标准输入，标准输出，标准错误的。默认情况下，子进程会继承父进程的设置，会将输出显示在控制台；
   - subprocess.PIPE 创建一个管道，允许与子进程进行通信；
   - subprocess.DEVNULL 特殊的文件对象，可以将其用于丢弃子进程的输出
   - 一个打开的文件对象，将内容写入文件
### 使用示例
```python
In [1]: import subprocess

In [3]: subprocess.run("ls -al", shell=True)
total 16
drwxr-xr-x   3 root root 4096 2024-01-12 16:18:55 .
drwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..
-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log
-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh
drwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo
Out[3]: CompletedProcess(args='ls -al', returncode=0)

In [4]: subprocess.run(["ls","-al"])
total 16
drwxr-xr-x   3 root root 4096 2024-01-12 16:18:55 .
drwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..
-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log
-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh
drwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo
Out[4]: CompletedProcess(args=['ls', '-al'], returncode=0)

In [5]: subprocess.run(["ls", "-l", "/dev/null"], capture_output=True)
Out[5]: CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0, stdout=b'crw-rw-rw- 1 experiment root 1, 3 2023-07-19 15:53:04 /dev/null\n', stderr=b'')

In [6]: res = subprocess.run(["ls","-al"])
total 16
drwxr-xr-x   3 root root 4096 2024-01-12 16:18:55 .
drwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..
-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log
-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh
drwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo

In [7]: res.returncode
Out[7]: 0

In [8]: res = subprocess.run(["ls","-al","./cmd_output"], stdout=subprocess.PIPE)
ls: cannot access ./cmd_output: No such file or directory

In [9]: res = subprocess.run(["ls","-al","./"], stdout=subprocess.PIPE)

In [10]: res.stdout
Out[10]: b'total 16\ndrwxr-xr-x   3 root root 4096 2024-01-12 16:18:55 .\ndrwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..\n-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log\n-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh\ndrwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo\n'

In [11]: res = subprocess.run(["ls","-al","./"], stdout=subprocess.PIPE, text=True)

In [12]: res.stdout
Out[12]: 'total 16\ndrwxr-xr-x   3 root root 4096 2024-01-12 16:18:55 .\ndrwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..\n-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log\n-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh\ndrwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo\n'

In [14]: with open("cmd_output.txt", "a+") as f:
    ...:     res = subprocess.run(["ls","-al","./"], stdout=f, text=True)
    ...:     print(f"code:{res.returncode}")
    ...:     print(f"stdout: {res.stdout}")
    ...: 
code:0
stdout: None

In [15]: ls
cmd_output.txt  find_big_file.log  find_big_file.sh*  shell_demo/

In [16]: res = subprocess.run(["cat","cmd_output.txt"])
total 16
drwxr-xr-x   3 root root 4096 2024-01-12 16:27:11 .
drwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..
-rw-r--r--   1 root root    0 2024-01-12 16:27:11 cmd_output.txt
-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log
-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh
drwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo
total 20
drwxr-xr-x   3 root root 4096 2024-01-12 16:27:11 .
drwxr-xr-x. 14 root root 4096 2024-01-12 16:17:37 ..
-rw-r--r--   1 root root  375 2024-01-12 16:27:11 cmd_output.txt
-rw-r--r--   1 root root    0 2024-01-12 16:18:08 find_big_file.log
-rwxr-xr-x   1 root root 1453 2024-01-12 16:18:00 find_big_file.sh
drwxr-xr-x   2 root root 4096 2024-01-12 16:18:56 shell_demo
```
## 核心接口分析之subprocess.Popen()
### 函数特点：

- **run()方法底层调用的是Popen()方法，当run方法无法满足需求是，使用Popen()方法去实现；**
- 输入：执行的命令
- 输出：非阻塞输出，执行后立即返回，执行结果通过返回对象获取
```python
subprocess.Popen(
    args,  #同run()方法
    bufsize=- 1, # 定义子进程的缓冲大小，默认值-1表示使用系统默认的缓冲大小；
    executable=None, # 指定要执行的程序路径，若未指定，则通过PATH环境变量来确定可执行文件位置；
    stdin=None, # 同run()方法
    stdout=None, # 同run()方法
    stderr=None, # 同run()方法
    preexec_fn=None, # 指定在子进程启动之前要执行的函数，该函数在fork()调用成功，在exec()调用之前被调用；
    close_fds=True, # 指定是否关闭所有文件描述符，
    shell=False, # # 同run()方法
    cwd=None, # 同run()方法
    env=None, # 同run()方法
    universal_newlines=None, # 同run()方法
    startupinfo=None, # 一个可选的subprocess.STARTUPINFO对象，用于指定子进程的启动信息，如窗口大小、窗口标题等
    creationflags=0, # 用于指定子进程的创建标志，控制子进程的各种行为。可以使用subprocess.CREATE_NEW_CONSOLE、subprocess.CREATE_NEW_PROCESS_GROUP等常量进行设置
    restore_signals=True, # 用于确定是否在子进程中恢复信号处理程序的默认行为
    start_new_session=False, # 用于代替使用 preexec_fn 的代码来在子进程中调用 os.setsid() 或 os.setpgid()；如果 start_new_session 为真值则 setsid() 系统调用将在执行子进程之前在子进程中执行
    pass_fds=(), # 是一个可选的在父子进程间保持打开的文件描述符序列。提供任何 pass_fds 将强制 close_fds 为 True
    *, 
    group=None, #（仅 POSIX）: 如果 group 不为 None，则 setregid() 系统调用将于子进程执行之前在下级进程中进行。 如果所提供的值为一个字符串，将通过 grp.getgrnam() 来查找它，并将使用 gr_gid 中的值。 如果该值为一个整数，它将被原样传递
    extra_groups=None, # （仅 POSIX）: 如果 extra_groups 不为 None，则 setgroups() 系统调用将于子进程之前在下级进程中进行。 在 extra_groups 中提供的字符串将通过 grp.getgrnam() 来查找，并将使用 gr_gid 中的值。 整数值将被原样传递
    user=None, # 仅 POSIX）: 如果 user 不为 None，则 setreuid() 系统调用将于子进程执行之前在下级进程中进行。 如果所提供的值为一个字符串，将通过 pwd.getpwnam() 来查找它，并将使用 pw_uid 中的值。 如果该值为一个整数，它将被原样传递
    umask=- 1, # 如果 umask 不为负值，则 umask() 系统调用将在子进程执行之前在下级进程中进行
    encoding=None,  # 同run()方法
    errors=None,  # 同run()方法
    text=None,  # 同run()方法
    pipesize=- 1, # 当 PIPE 被用作 stdin, stdout 或 stderr 时 pipesize 可被用于改变管道的大小。 管道的大小仅会在受支持的平台上被改变（当撰写本文档时只有 Linux 支持）
    process_group=None # 用于代替使用 preexec_fn 的代码来在子进程中调用 os.setsid() 或 os.setpgid()；如果 start_new_session 为真值则 setsid() 系统调用将在执行子进程之前在子进程中执行
)
```
### 重要属性-Popen对象
![image.png](https://cdn.nlark.com/yuque/0/2024/png/2130981/1705050956201-09e1da8f-a825-401c-ae20-e67a90f633e8.png#averageHue=%23474645&clientId=u0bfed3c2-e7cc-4&from=paste&height=150&id=u92fe8a26&originHeight=150&originWidth=850&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9909&status=done&style=none&taskId=ude3c6d9e-c5fa-4a36-b297-495f76a1fd4&title=&width=850)

- Popen.**communicate**(_input=None_, _timeout=None_)
   - 与进程交互：将数据发送到 stdin。 从 stdout 和 stderr 读取数据，直到抵达文件结尾。 等待进程终止并设置 [returncode](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.Popen.returncode) 属性，可选的 _input_ 参数应为要发送到下级进程的数据，或者如果没有要发送到下级进程的数据则为 None。 如果流是以文本模式打开的，则 _input_ 必须为字符串。 在其他情况下，它必须为字节串。
   - [communicate()](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.Popen.communicate) 返回一个 (stdout_data, stderr_data) 元组。如果文件以文本模式打开则为字符串；否则字节。
- Popen.**poll**()
   - 检查子进程是否已被终止。设置并返回 [returncode](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.Popen.returncode) 属性。否则返回 None。
- Popen.**wait**(_timeout=None_)
   - 等待子进程被终止。设置并返回 [returncode](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.Popen.returncode) 属性。
- Popen.**send_signal**(_signal_)
   - 将信号 _signal_ 发送给子进程，如SIGINT、SIGTERM等
### 使用示例

- 基本操作
```python
In [1]: import subprocess

In [2]: res = subprocess.Popen("sleep 10 && pwd", shell=True)

In [3]: res
Out[3]: <Popen: returncode: None args: 'sleep 10 && pwd'>

In [4]: res.stdout

In [5]: res.stdout

In [7]: /data/sswang/subprocess_demo
In [7]: 
```

- 案例
```python
'''
使用subprocess的Popen来执行shell命令，这种方式会创建一个新的进程来执行shell
使用要放在后台一直执行的shell进程，比如通过shell启动某种服务，如果通过上面
的方式，就会遇到当python被关闭时，shell启动的程序也被关闭
需要注意的是close_fds参数，默认为false，表示继承fb，即文件资源符，如果继承fb，那么当父进程关闭时
shell启动的子进程会去继承父进程相应的资源，如Flask web服务中通过shell启动了某个服务，此时Flask关闭了
但原本Flask监听的端口依旧会被shell子进程占用，这样就导致Flask无法再次启动。
将close_fds设置为false可以避免这种情况
'''
def execshell2(self,shell):
    try:
        p = subprocess.Popen(shell, shell=True,stdout=subprocess.PIPE ,stderr=subprocess.STDOUT, close_fds=True)
        output = p.stdout.read()
        # p.terminate()
        code = 0
        print('%s execute success!' % shell)
    except subprocess.CalledProcessError as e:
        code = e.returncode
        output = e.output
        print('%s execute error: exit_status [%s] err [%s]' % (shell, str(code), output))
        exit()
    return code, output
```
## subprocess常用异常类
### SubprocessError

- 此模块的其他异常
### TimeoutExpired

- [SubprocessError](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.SubprocessError) 的子类，等待子进程的过程中发生超时时被抛出
### CalledProcessError

- [SubprocessError](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.SubprocessError) 的子类，当一个由 [check_call()](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.check_call), [check_output()](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.check_output) 或 [run()](https://docs.python.org/zh-cn/3.12/library/subprocess.html#subprocess.run) (附带 check=True) 运行的进程返回了非零退出状态码时将被引发

## 实践案例

- **需求：使用typer+subprocess实现了一个用于管理项目依赖和执行一些常见任务的脚本的工具。**
```python
import sys
from pathlib import Path
import subprocess
import click

class UpgradeDependencies:
    @classmethod
    def _build_args(cls, dependencies):
        # 解析依赖项并构建参数
        packs, specials = [], {}
        for dep in dependencies:
            if ":" in dep:
                k, v = dep.split(":")
                specials[k.strip()] = v.strip().split()
            else:
                packs.append(dep.strip())
        return packs, specials

    @classmethod
    def _parse_dependencies(cls, text, title):
        # 从文本中解析依赖项
        main, dev = text.split(title)
        devs = dev.split("[tool.")[0].strip().splitlines()
        mains = main.strip().splitlines()
        prod_packs, specials = cls._build_args(mains)
        dev_packs, specials = cls._build_args(devs)
        return prod_packs, dev_packs, specials

    @classmethod
    def get_args(cls):
        # 获取依赖项参数
        dev_title = "[tool.poetry.group.dev.dependencies]"
        dev_flag = "--group dev"
        main_args, dev_args, others = [], [], []
        if dev_title in text:
            main_title = "[tool.poetry.dependencies]"
        else:
            dev_flag = "--group dev"
            dev_title = "[tool.poetry.group.dev.dependencies]"
        main_args, dev_args, others = cls._parse_dependencies(text, main_title, dev_title, dev_flag)
        return main_args, dev_args, others, dev_flag

    @classmethod
    def gen_cmd(cls):
        # 生成升级依赖项的命令
        main_args, dev_args, others, dev_flag = cls.get_args()
        command = f"poetry add {' '.join(main_args)} && poetry add {dev_flag} {' '.join(dev_args)}"
        for packages in others:
            command += f" && poetry add {' '.join(packages)}"
        return command

def exit_if_run_failed(cmd):
    # 如果运行命令失败，则退出
    try:
        subprocess.run(cmd, check=True, shell=True)
    except subprocess.CalledProcessError as e:
        print(f"Error: {e}")
        sys.exit(1)

@click.command()
def update():
    """升级所有依赖包到最新版"""
    exit_if_run_failed(UpgradeDependencies.gen_cmd())

@click.command()
def lint():
    """格式化加静态检查"""
    remove_imports = "autoflake --in-place --remove-all-unused-imports"
    cmd = ""
    paths = "."
    if args := sys.argv[1:]:
        if "-r" in args:
            args.remove("-r")
            if all(Path(i).is_file() for i in args):
                cmd = f"{remove_imports} {paths} && " + cmd
        paths = " ".join(args)
    tools = ("isort", "black", "ruff", "mypy")
    cmd += f"poetry run {tools[0]} {paths} && poetry run {tools[1]} {paths} && poetry run {tools[2]} {paths} && poetry run {tools[3]} {paths}"
    exit_if_run_failed(cmd)

@click.command()
def dev():
    """启动服务：相当于django的runserver"""
    cmd = "poetry run python main.py"
    if args := sys.argv[1:]:
        cmd += " " + " ".join(args)
    exit_if_run_failed(cmd)

@click.command()
def makemigrations():
    """生成数据库迁移文件，类似Django的./manage.py makemigrations"""
    exit_if_run_failed("aerich migrate")

@click.command()
def migrate():
    """更新数据库表结构：相当于django的./manage.py migrate"""
    exit_if_run_failed("aerich upgrade")

if __name__ == "__main__":
    cli()  # Assuming you have defined `cli` somewhere

```
