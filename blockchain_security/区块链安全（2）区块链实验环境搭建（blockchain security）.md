# 简介

这篇文章将教大家如何搭建区块链环境和区块链安全的工具，最好先看一下我之前写的区块链安全介绍

# VMware

```
下载地址：https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html
```

安装包教程可以参考百度上的教程，不是很难，一直next就好了

# CloudBreach虚拟机

```
下载地址：https://breakforgearchive.s3.amazonaws.com/cloudbreach-linux.zip
```

我们的环境将在这台虚拟机上部署
下载完后解压缩，进入目录

![image.png](https://image.3001.net/images/20221217/1671258318_639d60ce1713395fae448.png!small)

![image.png](https://image.3001.net/images/20221217/1671258332_639d60dc710a895f12cbb.png!small)

![image.png](https://image.3001.net/images/20221217/1671258345_639d60e94bd62d8592963.png!small)

或者双击直接打开.ovf文件，设置虚拟机名字和保存的文件夹就好了

# Fananche安装

Ganache 是一款允许我们创建本地区块链的软件，我们可以使用它来测试智能合约的部署和交互。

```
虚拟机的账号和密码都是fog，进入虚拟机
ganache下载地址：https://www.trufflesuite.com/ganache
```

然后赋予下载文件最高权限

```
chmod 777 ganache-2.6.0-beta.3-linux-x86_64.AppImage
./ganache-2.6.0-beta.3-linux-x86_64.AppImage
```

点击“New Workspace”
给它一个名字

![image.png](https://image.3001.net/images/20221217/1671258375_639d6107d599a9541b763.png!small)

单击 Accounts & Keys 并设置您要测试的帐户数量和余额 100 eth 和 10 个帐户就可以了
点击“Save”即可

![image.png](https://image.3001.net/images/20221217/1671258390_639d611697f9b4abd7b24.png!small)

# Metamask安装

Metamask 是一个非常流行的加密货币钱包，但也非常适合与 Web3 资源交互。我们将使用它来签署交易，例如我们部署智能合约的交易。

```
下载地址：https://metamask.io/download.html
```

或者火狐插件里搜索Metamask然后添加

![image.png](https://image.3001.net/images/20221217/1671258410_639d612a0369eee36bda9.png!small)

单击地址栏中的 Metamask 图标，然后单击“network”下拉菜单，选择“自定义 RPC”

![image.png](https://image.3001.net/images/20221217/1671258422_639d613670ff5ec0c703d.png!small)

![image.png](https://image.3001.net/images/20221217/1671258431_639d613f01dccdac11f80.png!small)

![image.png](https://image.3001.net/images/20221217/1671258438_639d6146e81d27d89e105.png!small)

```
随意输入名字
RPC 网址：http://127.0.0.1:7545
chain ID：1337（Ganache 使用 1337 作为其默认链 ID）
currency symbol ：ETH
点击save
```

如果保存不了，点击右边的networks，然后把多余的那个默认设置删除即可

![image.png](https://image.3001.net/images/20221217/1671258453_639d6155f09287b633436.png!small)

要导入 Ganache 钱包的私钥，请单击 Metamask 中的“帐户”图标，然后单击“导入帐户”。粘贴要导入的帐户的私钥。

![image.png](https://image.3001.net/images/20221217/1671258466_639d616257ab1032406a1.png!small)

回到Ganache程序内
随意选择一个账户

![image.png](https://image.3001.net/images/20221217/1671258478_639d616e235ea8f96ff7e.png!small)

![image.png](https://image.3001.net/images/20221217/1671258485_639d6175f1e5f1e94320d.png!small)

复制账户密钥，回到metamask

![image.png](https://image.3001.net/images/20221217/1671258497_639d6181a87b1299c3447.png!small)

输入账户的密钥，点击import
然后就大功告成了

# Remix IDE

Remix IDE 是快速开始编写、分析、编译和部署智能合约的绝佳选择。

```
使用网址为：https://remix.ethereum.org/
```

我们之后再介绍如何使用这个网站

# Truffle

Truffle 是一个用于开发、编译和部署智能合约的框架。
安装环境教程：

```
进入终端，输入：
wget https://nodejs.org/dist/v16.14.2/node-v16.14.2-linux-x64.tar.xz   ##下载软件
tar -xvf node-v16.14.2-linux-x64.tar.xz    ##解压缩
mv node-v10.16.3-linux-x64  nodejs      ## 重命名文件夹
建立软连接，变为全局
ln -s /home/fog/nodejs/bin/npm /usr/local/bin/ 
ln -s /home/fog/nodejs/bin/node /usr/bin/ 
最后检查是否配置成功
node -v
npm -v
如果没有成功，可以看文章顶部，加我qq
```

安装truffle：

```
sudo npm install -g truffle
mkdir blockHAX   ##创建一个名叫blockhax的文件夹
cd blockHAX      ##进入文件夹
truffle init     ##初始化truffle
```

它将创建以下目录和文件

```
├── contracts
│   └── Migrations.sol
├── migrations
│   └── 1_initial_migration.js
├── test
└── truffle-config.js
```

文件夹解释：

```
contracts - 您将存储智能合约 Solidity (.sol) 文件的目录
migrations - 存储文件的目录，告诉 Truffle 如何部署智能合约（即部署它们的顺序，任何构造函数等）
test - 存储测试脚本以测试 Solidity 功能的位置
truffle-config.js - 配置文件，告诉 Truffle 在哪里部署合约（本地实例、测试网、主网等）以及使用什么编译器版本。
```

配置文件：

```
进入blockhax的文件夹
安装vim：apt install vim
vim  truffle-config.js
```

添加以下内容

```
module.exports ={
    compilers:{
      solc: {
        version: "^0.7.5",
      }
    },
    networks: { 
        "development": { 
            network_id: "*", 
            host: "127.0.0.1", 
            port: 7545 
        }, 
    } 
};
```

![image.png](https://image.3001.net/images/20221217/1671258528_639d61a0417429b37ddda.png!small)

将第50行版本号改为“0.6.4”
保存，大功告成，然后打开metamake

![image.png](https://image.3001.net/images/20221217/1671258542_639d61aece5f88178b1ba.png!small)

![image.png](https://image.3001.net/images/20221217/1671258554_639d61bad8d2d8ccda6b5.png!small)

# 部署智能合约

按照我们之前的步骤进行操作后，您现在应该可以编译和部署智能合约了。本节将简要介绍使用 Truffle 和 Remix IDE 编译和部署智能合约的过程
智能合约下载地址：

```
https://github.com/ethereumbook/ethereumbook/blob/develop/code/Solidity/Faucet.sol
```

复制源代码
在contracts文件夹下创建一个新的文件

```
touch faucet.sol    ##创建文件
vim faucet.sol    ##进入文件
然后粘贴源码
truffle compile  ##开始编译
如果有什么问题，看文章顶部，加我qq
```

成功编译合约后，我们将使用“迁移”功能将智能合约部署到我们本地的 Ganache 区块链

```
truffle migrate
```

![image.png](https://image.3001.net/images/20221217/1671258572_639d61cc7af80bc189ed5.png!small)

大功告成
Remix IDE 智能合约部署，网站为：

```
https://remix.ethereum.org/
```

然后删除左边全部的默认文件，只保留文件夹

![image.png](https://image.3001.net/images/20221217/1671258587_639d61db48d34176b3b2d.png!small)

![image.png](https://image.3001.net/images/20221217/1671258593_639d61e16c369c11ccb02.png!small)

选择第一个文件夹，右击，新建文件

![image.png](https://image.3001.net/images/20221217/1671258602_639d61ea1dca8758c313a.png!small)

文件名叫faucet.sol

![image.png](https://image.3001.net/images/20221217/1671258612_639d61f4b560e71eaebf7.png!small)

然后点击左边任务栏最下面的插件按钮

![image.png](https://image.3001.net/images/20221217/1671258624_639d6200bff00e68a1234.png!small)

![image.png](https://image.3001.net/images/20221217/1671258633_639d6209f07426e27f4da.png!small)

添加如下插件，如果没有solidity compiler logic，则可以不用添加
回到主页面，打开faucet.sol文件，将之前的源码复制进去

```
// SPDX-License-Identifier: CC-BY-SA-4.0

// Version of Solidity compiler this program was written for
pragma solidity 0.6.4;

// Our first contract is a faucet!
contract Faucet {
    // Accept any incoming amount
    receive() external payable {}

    // Give out ether to anyone who asks
    function withdraw(uint withdraw_amount) public {
        // Limit withdrawal amount
        require(withdraw_amount <= 100000000000000000);

        // Send the amount to the address that requested it
        msg.sender.transfer(withdraw_amount);
    }
}
```

![image.png](https://image.3001.net/images/20221217/1671258653_639d621d34fff55caf428.png!small)

![image.png](https://image.3001.net/images/20221217/1671258659_639d6223aaf90fd42b072.png!small)

选择版本号后运行，然后连接metamask，他会自己弹出来的

![image.png](https://image.3001.net/images/20221217/1671258669_639d622d241803d5f971e.png!small)

![image.png](https://image.3001.net/images/20221217/1671258675_639d6233b8ccad9d8491c.png!small)

大功告成

# 区块链安全工具安装

进入终端

```
sudo pip3 install mythril
myth analyze     ##你要扫描的文件
```

![image.png](https://image.3001.net/images/20221217/1671258687_639d623f9b8c30afd5e39.png!small)

成功运行

# 总结

写了大概两个小时，这篇文章只是部署区块链安全的实验环境，下一篇文章，将教大家如何利用漏洞，偷走池子里全部的比特币
