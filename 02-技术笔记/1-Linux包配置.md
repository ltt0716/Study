# 一、Linux安装包、工具
## 1.安装 apt-rdepends 工具
使用 apt-rdepends 这个工具来递归地查找所有依赖，然后用` apt-get download ` 来下载它们
```  
sudo apt update  
sudo apt install apt-rdepends  
```  
## 2.列出所有必需包及其全部依赖项
使用` apt-rdepends `来生成一个完整的、包含所有层级依赖的包名列表。
```  
apt-rdepends build-essential cmake libcurl4-openssl-dev python3-dev python3-pip | grep -v "^ " > packages.txt  

**命令解析：**
* ` apt-rdepends ... `: 递归地查找这几个核心包的所有依赖项，并打印出来。
* ` | grep -v "^ " `: 这是一个反向过滤。apt-rdepends 的输出中，依赖项都不是以空格开头的，而一些附加信息是以空格开头的。-v "^ " 的意思就是“不要那些以空格开头的行”，从而得到一个纯净的包名列表。
* ` > packages.txt `: 将这个纯净的包名列表保存到 packages.txt 文件中。
```   
## 3.根据列表，强制下载所有包
现在我们有了完整的包名列表，就可以使用 apt-get download 来下载它们了。
```  
# 创建一个文件夹来存放所有 .deb 文件，并进入
mkdir system_debs && cd system_debs

# 使用我们刚刚生成的列表，下载所有包
apt-get download $(cat ../packages.txt)

**命令解析：**
*   `apt-get download`: 专门用于下载 `.deb` 包，即使它已经安装过。
*   `$(cat ../packages.txt)`: 读取上一级目录的 `packages.txt` 文件，并将其中的所有包名作为参数传给 `apt-get download`。  
```
