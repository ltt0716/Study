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