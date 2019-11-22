### 国内使用Homebrew 非常卡

#### 建议使用清华镜像

#### 详情查看 https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/

#### 1.  步骤一

替换现有上游

```shell
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

brew update
```



#### 2. 步骤二

```shell
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

如果是zsh 写到==.zshrc==里面



### 备注：还原方式

```shell
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git

brew update
```



删除在.bash_profile（或.zshrc）中关于**export HOMEBREW_BOTTLE_DOMAIN**那一行