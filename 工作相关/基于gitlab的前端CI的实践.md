---
title: 基于gitlab的前端CI的实践
url: https://www.yuque.com/endday/blog/cg9eeg
---

<a name="GSJcz"></a>

# 前言

<a name="TrWir"></a>

# 正文

<a name="q2ih6"></a>

## gitlab-runner

<a name="W0zcA"></a>

### 安装

环境：阿里云服务器，Ubuntu

执行脚本

```shell
# For Debian/Ubuntu/Mint
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash

# For RHEL/CentOS/Fedora
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

安装

```shell
# For Debian/Ubuntu/Mint
sudo apt-get install gitlab-runner

# For RHEL/CentOS/Fedora
sudo yum install gitlab-runner
```

 安装[nvm](https://github.com/nvm-sh/nvm)（因为没有使用docker，所以需要nvm控制node版本）

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

重启shell，测试nvm

```shell
command -v nvm
```

nvm 速度问题，更换镜像

```shell
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
```

<a name="D66GI"></a>

### 注册

<a name="pH0Dn"></a>

#### 填写 Gitlab URL

gitlab实例地址如果要用https，需要配置证书，所以填写http和内网ip地址，这里是支持ip地址加端口的 <a name="AQ7vD"></a>

#### 填写 token

<a name="PraxH"></a>

#### runner 描述信息

名称，建议写个有含义，别乱填，做提示用，用于`gitlab-runner list`命令，区分不同runner用，也用来`gitlab-runner unregister`时指定runner的名称。 <a name="FL6qO"></a>

#### 填写 tag

在项目构建时，可以指定带有某个tag的runner来执行构建。
可以不填，就要注意在这个runner中配置下面选项，不然没有指定tag（指标记）的项目（指作业）无法使用此runner构建。
![image.png](..\assets\cg9eeg\1643357119095-098ac812-acd8-4ff3-9faf-6e5bfba8b9ca.png) <a name="mluWm"></a>

#### 执行器

选择 shell，这里看公司支持了，下次迭代试试docker <a name="LgfGH"></a>

## nvm安装

由于使用的gitlab-runner不是docker模式，是shell，需要自己整个nvm来控制node版本
注意安装时要配置环境变量etc/enviroment，如$NVMDIR路径

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
# 建议指定安装目录
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash | NVM_DIR="/example/nvm"
```

```bash
# cli中使用
source $NVM_DIR/nvm.sh; #nvm初始化
# 我是带上了source这个才能用，对linux还是不熟悉，没有深究为何
nvm use ${nodeVersion};
node -v;
npm -v;
```

```yaml
#yml的配置
variables:
  NODE_VERSION_DEFAULT: 10 #默认node版本

before_script:
  - NODE_VERSION=${NODE_VERSION:=$NODE_VERSION_DEFAULT}
  - source ${NVM_DIR}/nvm.sh #nvm初始化，nvm是一个脚本不是指令
  - nvm use ${NODE_VERSION}
  - node -v
```

<a name="mM08u"></a>

## gitlab-ci.yml

<a name="ApEIj"></a>

### 配置

```yaml
default:
  before_script:
    - whoami
    - echo "当前项目为 【$CI_PROJECT_NAME】"
    - pwd
    - node -v
    - export NODE_MODULES_VERSION=`sha1sum package.json`

  cache:
    key: '$NODE_MODULES_VERSION'
    paths:
      - node_modules/

stages:
  - npm-install
  - build

# 安装依赖
npm-install:
  stage: npm-install
  interruptible: true
  rules:
    - if: '$CI_COMMIT_BRANCH =~ /^[alpha|beta|master]+$/'
    - changes:
        - package.json
  cache:
    key: '$NODE_MODULES_VERSION'
    paths:
      - node_modules/
  script:
    - echo 'npm-install 开始'
    - echo "node_modules版本=$NODE_MODULES_VERSION"
    #   判断node_modules文件夹是否存在
    - |
      if [ -d "./node_modules/" ]; then
        echo -e "\e[42m 缓存存在,跳过npm i \e[0m"
      else
        echo -e "\e[43m node_modules不存在, 执行npm i \e[0m"
        npm i
      fi
    - echo "npm-install 结束"
#  only:
#    variables:
#      - $CI_COMMIT_TAG == 'npm-install'
#      - $CI_COMMIT_MESSAGE =~ /npm-install/

build:
  stage: build
  interruptible: true
  rules:
    - if: '$CI_COMMIT_BRANCH =~ /^[alpha|beta|master]+$/'
  cache:
    key: '$NODE_MODULES_VERSION'
    # 下面的配置指示，我们当前只拉取缓存，不上传，这样会节省不少时间
    policy: pull
    # 指定要缓存的文件/文件夹
    paths:
      - node_modules/
  script:
    - echo 'npm run build 开始'
    - npm run dist
    - echo 'npm run build 结束'
  artifacts:
    paths:
      - ./dist
    expire_in: 3 days # 构建产物过期，无法下载
```
