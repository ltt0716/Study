# first create
``` git
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/ltt0716/dd.git
git push -u origin main
```
# later pull
```git
#commit 
git add .
git commit -m ""
git remote add origin https://github.com/ltt0716/dd.git
git branch -M main
git push -u origin main
```

# other cads
``` git   
#显示已暂存但尚未提交的更改(当前目录)

git diff --staged

#如果想查看所有已暂存和未暂存的更改，用：
git status
```

# init
```git 
git config --global user.name "ltt0716"
git config --global user.email "2509336485@qq.com"

git config --list
```

# 撤销上次提交
```git 
git log 

#copy commit id

git revert id
```



# 克隆不同分支
```git 
# 语法: git clone -b <分支名> <仓库地址> <本地文件夹名>
# 后端分支
git clone -b backend git@github.com:ltt0716/SSICSS.git Backend

#前端分支
git clone -b frontend git@github.com:ltt0716/SSICSS.git Frontend

```

# 身份验证 clone 私有仓库
1. **查看密钥文件**
    ```git 
    ls -al ~/.ssh
    ```
    >如果看到了 `id_rsa.pub` 或 `id_ed25519.pub` 这样的文件，说明本地已经有了密钥，可以直接跳到 **第 3 步**。

2. **生成SSH 密钥**  
    如果你没有密钥，就需要创建一个。推荐使用 `ed25519` 算法，需要**替换邮箱**。
    ```bash
    ssh-keygen -t ed25519 -C "2509336485@qq.com"
    ```
    >接下来会有一系列提示：  
  `Enter a file in which to save the key...`：直接按回车，使用默认路径即可。  
  `Enter passphrase (empty for no passphrase):`：输入一个密码。这个密码用来保护私钥文件。如果设置密码，以后每次使用这个密钥时都需要输入这个密码。可以直接按回车，表示不设置密码。  
  `Enter same passphrase again:`：再次输入密码，或者直接回车。  

3. 将 SSH 公钥添加到GitHub 账户  
这一步是把“锁”（公钥）安装到 GitHub 上。
   1. 复制公钥文件的内容。  
   请注意，是复制 `.pub` 结尾的公钥。(复制屏幕上所有以 `ssh-ed25519` 开头，以邮箱结尾的内容)。
        >```bash  
        >cat ~/.ssh/id_ed25519.pub
        >```
    2. 在GitHub上添加SSH密钥
      - 登录GitHub 账户。
      - 点击头像，选择 **Settings** (设置)。
      - 选择 **SSH and GPG keys** (SSH 和 GPG 密钥)。
      - 点击 **New SSH key** (新建 SSH 密钥) 按钮。
      - **Title** (标题) 栏里随便填一个名称。
      - **Key** (密钥) 栏里，粘贴所复制的公钥内容。
      - 点击 **Add SSH key** (添加 SSH 密钥)。

4. 测试 SSH 连接  
终端里测试一下与 GitHub 的连接：
    ```bash
    ssh -T git@github.com
    ```