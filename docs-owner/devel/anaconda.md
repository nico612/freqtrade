
## python 环境管理工具 Anaconda

### 安装
mac 安装：
```shell
brew update
brew install --cask anaconda
```

设置环境变量
```shell
echo 'export PATH="$PATH:/usr/local/anaconda3/bin"' >> ~/.zshrc
source ~/.zshrc
```

[windows 安装](https://conda.io/projects/conda/en/latest/user-guide/install/windows.html)


验证是否安装成功

```shell
conda --version
```

查看python版本
```shell
python --version
```

列出可用环境，默认会安装一个base环境
```shell
conda env list
```
激活环境

```shell
conda activate <环境名称>

# 较新的版本先执行下面命令激活，后面可使用上面命令直接激活
source activate my_env

```

退出当前环境

```shell
conda deactivate

```

创建新的环境

```shell
conda create --name myenv python=3.11
```

创建一个带有特定包的环境：
```shell
conda create --name myenv numpy pandas
```

删除环境
```shell
conda env remove --name myenv
```

安装包
```shell
conda install numpy
```

取消没错终端自动进入环境

```shell
conda config --set auto_activate_base false
conda config --set auto_activate_base true // 修改为自动进入base环境
```


### 可能错线的错误处理
在执行conda命令时，conda无法识别当前使用什么终端类型，会报错让你初始化终端，如果没有则不管
```shell
conda init zsh
```

如何判断终端是那种类型：

在shell脚本中，可以通过判断环境变量SHELL的值来确定当前终端的类型。下面是一些常见终端类型的环境变量值：

- Bash: `/bin/bash`
- Zsh: `/bin/zsh`
- Fish: `/usr/local/bin/fish`
- Tcsh: `/bin/tcsh`
- Xonsh: `/usr/local/bin/xonsh`
- PowerShell: `/usr/local/microsoft/powershell/7/pwsh`

你可以使用`echo $SHELL`命令来查看当前终端的环境变量值。

在shell脚本中，你可以使用条件语句来判断终端类型。以下是一个示例：

```shell
shellCopy code
#!/bin/bash

if [ "$SHELL" = "/bin/bash" ]; then
    echo "Bash"
elif [ "$SHELL" = "/bin/zsh" ]; then
    echo "Zsh"
elif [ "$SHELL" = "/usr/local/bin/fish" ]; then
    echo "Fish"
elif [ "$SHELL" = "/bin/tcsh" ]; then
    echo "Tcsh"
elif [ "$SHELL" = "/usr/local/bin/xonsh" ]; then
    echo "Xonsh"
elif [ "$SHELL" = "/usr/local/microsoft/powershell/7/pwsh" ]; then
    echo "PowerShell"
else
    echo "Unknown shell"
fi
```
根据当前终端的环境变量值，上述脚本将输出对应的终端类型。


