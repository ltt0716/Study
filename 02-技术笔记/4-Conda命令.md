# `Conda` 相关操作

### **核心功能**

Conda 的核心功能是包管理和环境管理，可以使用它来安装、更新和删除软件包，并创建、激活和管理相互隔离的虚拟环境。

### **minicoda** 安装
1. Winddows CMD
```bash 
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe -o .\miniconda.exe
start /wait "" .\miniconda.exe /S /D="D:\Miniconda3"
del .\miniconda.exe
```
- 打开 Anaconda Prompt (从开始菜单里找)。
- 运行初始化命令：这个命令会自动配置您的 Shell（cmd, PowerShell等），让 conda 命令可用。
```bash
conda init
```
2. Windows PowerShell
```bash
wget "https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe" -outfile ".\miniconda.exe"
Start-Process -FilePath ".\miniconda.exe" -ArgumentList "/S" -Wait
del .\miniconda.exe
```

3. Linux
```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```

### **环境管理**

管理虚拟环境是 Conda 最有用的功能之一，它允许您为不同的项目创建独立的、包含特定版本软件包的环境。

*   **创建环境:**
    *   创建一个名为 `myenv` 的新环境，并指定 Python 版本为 3.8：
        ```bash
        conda create --name myenv python=3.12
        ```
*   **激活环境:**
    *   要开始在特定环境中使用，您需要激活它：
        ```bash
        conda activate myenv
        ```
*   **退出环境:**
    *   当您完成在当前环境中的工作后，可以停用它以返回到基础环境：
        ```bash
        conda deactivate
        ```
*   **查看所有环境:**
    *   列出您创建的所有环境：
        ```bash
        conda env list
        ```
*   **删除环境:**
    *   删除不再需要的环境及其所有已安装的包：
        ```bash
        conda remove --name myenv --all
        ```
*   **从文件创建环境:**
    *   您可以根据一个 YAML 文件来创建一个环境，这对于与他人共享环境非常有用：
        ```bash
        conda env create -f environment.yml
        ```
*   **导出环境:**
    *   将当前环境的配置导出到一个 YAML 文件：
        ```bash
        conda env export > environment.yml
        ```

### **包管理**

Conda 简化了软件包的安装、更新和删除过程。

*   **安装包:**
    *   在当前激活的环境中安装一个包（例如 NumPy）：
        ```bash
        conda install numpy
        ```
    *   您也可以在创建环境时直接安装包：
        ```bash
        conda create --name myenv numpy
        ```
*   **查看已安装的包:**
    *   列出当前环境中所有已安装的包及其版本：
        ```bash
        conda list
        ```
*   **更新包:**
    *   更新一个特定的包：
        ```bash
        conda update numpy
        ```
    *   更新当前环境中所有的包：
        ```bash
        conda update --all
        ```
*   **删除包:**
    *   从当前环境中删除一个包：
        ```bash
        conda remove numpy
        ```
*   **搜索包:**
    *   搜索可用的包：
        ```bash
        conda search numpy
        ```

### **其他常用命令**

*   **查看 Conda 版本:**
    *   检查您安装的 Conda 的版本：
        ```bash
        conda --version
        ```
*   **更新 Conda 本身:**
    *   将 Conda 更新到最新版本：
        ```bash
        conda update -n base conda
        ```
*   **查看帮助信息:**
    *   获取某个命令的帮助信息（例如 `install` 命令）：
        ```bash
        conda install --help
        ```
*   **清理缓存:**
    *   清除 Conda 的索引缓存、锁定的包和未使用的包，以释放磁盘空间：
        ```bash
        conda clean --all
        ```
 --- 

## `Conda` 换源

### **临时使用镜像源**

如果您只是想临时在某一次安装中使用国内镜像源，可以在 `conda install` 命令后面加上 `-c` 参数指定源的地址。这个命令会从清华大学的镜像源来安装 NumPy。
```bash
conda install --channel https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main numpy
```

### **永久更换镜像源（推荐）**

通过修改 conda 的配置文件 `.condarc`，可以实现永久性的换源。这样以后每次使用 `conda` 命令都会默认从新的镜像源下载。

**操作步骤如下：**

**1. 生成或找到 `.condarc` 配置文件**

这个文件通常位于您的用户主目录下。

*   **Windows:** `C:\Users\YourUsername\.condarc`
*   **macOS / Linux:** `/Users/YourUsername/.condarc` 或 `~/.condarc`

如果这个文件不存在，您可以通过运行以下命令来生成它：

```bash
conda config --set show_channel_urls yes
```

**2. 添加清华大学镜像源**

打开终端（或 Anaconda Prompt），然后依次执行以下命令，将清华大学的镜像源地址添加到您的配置中：

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
```

```bash
# python永久换源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**3. 添加 `conda-forge` 镜像源 (非常重要)**

`conda-forge` 是一个包含大量 conda 官方源没有的包的社区渠道，强烈建议您也将其镜像源添加进去：

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
```

**4. 设置搜索时显示通道 URL**

执行以下命令，以确保在搜索和安装时能看到包的来源 URL：

```bash
conda config --set show_channel_urls yes
```

### **三、 查看和管理您的配置**

完成上述步骤后，您可以验证是否配置成功。

*   **查看当前的源配置：**
    ```bash
    conda config --show channels
    ```
    执行后，您应该能看到类似下面的输出，并且清华大学的源地址排在最前面：
    ```
    channels:
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
      - defaults
    ```

*   **清理索引缓存并测试：**
    为了让新的配置生效，建议清理一下 conda 的索引缓存：
    ```bash
    conda clean -i
    ```
    然后您可以尝试创建一个新的环境或安装一个包来测试速度是否变快：
    ```bash
    conda create -n test_env python=3.8
    ```

### **四、 如何恢复默认源**

如果您想恢复到 conda 的默认官方源，可以执行以下命令来移除您添加的镜像源：

```bash
conda config --remove-key channels
```

### **其他常用国内镜像源**

除了清华大学源，您也可以选择其他国内的镜像源，例如：

*   **中科大 (USTC) 源**
*   **北京外国语大学源**
*   **上海交通大学源**

更换这些源的方法与上面类似，只需将命令中的 URL 替换成相应镜像站提供的地址即可。

希望这份详细的指南能帮助您成功更换 conda 源，享受更快的包管理体验！