## MAC homebrew无法安装

### 1.问题描述

```pro
Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

### 2.解决方案

运行下面自动脚本(已经全部替换为国内地址):

```pro
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### 3.查询nginx

```pro
brew search nginx
```

### 4.安装nginx

```pro
brew install nginx
```

### 5.测试nginx

```pro
nginx -v
```

## 6.修改配置文件

```pro
cd /usr/local/etc/nginx/
```

## 7.

```pro
ps -ef | grep nginx
```





