# 共识协议的简单应用
[TOC]

## 前言
区块链是目前最热门的技术之一，许多新的概念如Web3, 元宇宙, DeFi等都建立在区块链的基础之上。而区块链离不开共识协议, 共识协议为区块链提供了底层安全性保证。区块链是一个分布式账本，共识协议的作用是在所有的账本副本上保持相同的内容，保证区块链的不可篡改性，同时不同的共识协议也决定了整体区块链的性能。在目前的区块链共识协议中，可以基本分为以下种类（共识节点指的是参与共识协议的节点）：
 * 公链共识协议（共识节点数量未知）：
     * PoW: 工作量证明算法, 如Bitcoin[1]，Eth1.0[2]
     * PoS: 权益证明机制，如Dfinity[3], Algorand[4], Eth2.0[5]
     * 其他: PoX, DPoS等等 
 * 联盟链共识协议（共识节点数量已知）：
     * PBFT[6]: 基于leader的半同步拜占庭容错协议，经典的分布式共识协议
     * HotStuff[7]: 基于leader的半同步拜占庭协议，具有线性的通信复杂度以及可响应性，被Facebook(Meta)的区块链项目[Diem](https://github.com/diem/diem)所采用的
     * Honeybadger BFT[8]: 第一个异步拜占庭共识协议
     * 其他：Dumbo[9], Tusk[10]等等

上面针对共识协议仅进行了基本的分类，但是目前区块链共识协议发展十分迅速，并且呈现出多样化的趋势，对于其他的共识协议不再赘述。
## 实验目标
了解共识的运行过程，并动手实现一个共识协议。通过协议的运行的保证所有参与节点以相同的顺序对所有的请求进行提交。




## 实验准备
### 实验系统：linux
* 由于测试会在linux系统下进行，可以直接在linux环境下开发，Windows可使用[WSL2](https://learn.microsoft.com/zh-cn/windows/wsl/install)或者虚拟机软件（Vmware，Virtualbox）创建虚拟机环境，也可以在Windows下进行开发，保证最终的提交版本能在linux下编译运行即可。
### Go语言
#### Go 简介
* Go（又称golang）是Google开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言，由于实验环境是在go环境下开发，因此需要预先对go的语法规则有个基本的了解。
#### Go 环境配置
* [GO安装教程](https://go.dev/doc/install)
* [GO modules项目依赖管理](https://go.dev/blog/using-go-modules)
    * `go env -w GO111MODULE=on`
* GO更换国内代理
    * `go env -w GOPROXY="https://goproxy.io"`
* [GO VSCode插件](https://code.visualstudio.com/docs/languages/go)
![](https://notes.sjtu.edu.cn/uploads/upload_5c114a4e051e5436d469b6712bce1909.png)
#### Go 学习
* [GO之旅](https://tour.go-zh.org/welcome/1) 
* [GO文档](https://go.dev/doc/)
 

## 代码实现
### 基本定义
* 节点（node）：参与共识协议过程的一个进程
* 提议（propose）：节点提议一组交易内容，参与共识协议的过程，确定是否提交，如在实际区块链系统中，会选取一组客户发起的交易，在本实验不考虑实际的含义，选择随机的内容进行提议。
* 提交（commit）：对于共识确定的提议进行“记账”，成为链上不可篡改的内容。
### 实现目标
* 节点的每个"位置"（Sequence Number）代表一次共识的结果，通过实现的共识协议，保证以下性质：
    * 正确节点一个位置最多只有一个区块代表共识的结果
    ![](https://notes.sjtu.edu.cn/uploads/upload_385c05274ecf7ab689bfd0cbed1f1080.png)

    * 正确节点的一个位置可以没有区块
    ![](https://notes.sjtu.edu.cn/uploads/upload_b4d15629eb1cb331c2e104a5e1d153d3.png)

    * 所有正确节点相同的位置必须有保证相同的区块（如果没有区块，所有正确节点都没有）

### 代码模板
* 代码模板地址：[Github](https://github.com/Waynezee/SimpleConsensus)，[Gitee](https://gitee.com/xiangzhew/SimpleConsensus)
    * `mylogger`: 实现简单的日志记录功能
    * `myrpc`: 实现简单的rpc功能，节点间通过rpc进行相互通信
    * `core`: **待实现**的共识协议
    * `config`: 保存节点的配置文件
    * `log`: 保存节点的日志信息
    * `cmd`: 保存可执行文件
    * `scripts`: 测试和运行脚本
* 测试流程：
    1. `git clone https://github.com/Waynezee/SimpleConsensus`
    2. `cd SimpleConsensus/ && mkdir log`
    3. `cd cmd/ && make`
    4. `cd ../scripts/ && chmod +x *.sh` 
    5. `./run_nodes.sh [test time (seconds)]`
    6. 等待运行结束
    7. `python3 check.py 7`
* 当编写你的代码的时候需要注意：
    * 需要保证`cmd/node.go`的输入参数形式与原始的一致
    * 保证`core/block.go`中`getBlock()`和`commitBlock()`函数不变，并在合适的时候进行调用
        * `getBlock()`: 生成一个区块，同时会将相关信息记录到日志中
        * `commitBlock()` : 当协议确定性的完成共识后进行调用，同时会将相关信息记录到日志中
    * 主要需要需要实现的部分在`core/core.go`
    * 代码模板中其他的地方都可以在不影响测试的条件下进行修改。
* 调试：
    * 分布式程序一般debug的模式是通过日志进行分析(测试中也是通过日志来检测协议的正确性)，可以在关键步骤进行日志打印，逐渐缩小范围定位出问题的代码。
    * 在本地环境下，可使用[Vscode Debug](https://code.visualstudio.com/docs/editor/debugging#_multitarget-debugging)调试多个进程
## 评价标准

测试会在7节点规模下进行测试，通过运行共识协议一段时间之后对各个节点的日志进行分析，对以下指标进行检测:
* 一定的性能：在一定时间内完成提交的数量（>= 10 ops/s）
* 安全性（Safety）：节点之间提交的提议顺序是否一致


测试将分为三个场景下进行：
> crash指停机，一般认为节点不会重新启动
* （正常场景）所有节点都正常工作，测试如下：
    * `cd ../scripts/ && ./normal_test.sh`
* （crash场景）随机一个节点停机(进程被中止)：
    * 如 `cd ../scripts/ && ./crash1_test.sh 0`
* （crash场景）随机两个节点停机
    * 如 `cd ../scripts/ && ./crash2_test.sh 0 1`

## 学习路线
* 学习GO
* 选择一个共识协议（PoW, [Paxos](https://www.microsoft.com/en-us/research/publication/2016/12/paxos-simple-Copy.pdf), [Raft](https://raft.github.io/)等)，只需要关注与实验相关的部分即可。
    * [一个可能的实现思路](https://notes.sjtu.edu.cn/s/cf7nS_HGl)
* 了解代码模板和基本的测试模式
* 着手实现

## 讨论
> 如果发现代码模板有问题，可以随时指出，我会尽快修复
* [Github issue](https://github.com/Waynezee/SimpleConsensus/issues)
* 邮箱: xiangzhe_wang@163.com
* WX: wxz18568738360

## 参考文献
[1] Nakamoto S. Bitcoin: A peer-to-peer electronic cash system[J]. Decentralized business review, 2008: 21260.
[2] Ethash: https://ethereum.org/en/developers/docs/consensus-mechanisms/pow/mining-algorithms/ethash/
[3] Hanke T, Movahedi M, Williams D. Dfinity technology overview series, consensus system[J]. arXiv preprint arXiv:1805.04548, 2018.
[4] Gilad Y, Hemo R, Micali S, et al. Algorand: Scaling byzantine agreements for cryptocurrencies[C]//Proceedings of the 26th symposium on operating systems principles. 2017: 51-68.
[5] Buterin V, Hernandez D, Kamphefner T, et al. Combining GHOST and casper[J]. arXiv preprint arXiv:2003.03052, 2020.
[6] Castro M, Liskov B. Practical byzantine fault tolerance[C]//OsDI. 1999, 99(1999): 173-186.
[7] Yin M, Malkhi D, Reiter M K, et al. HotStuff: BFT consensus with linearity and responsiveness[C]//Proceedings of the 2019 ACM Symposium on Principles of Distributed Computing. 2019: 347-356.
[8] Miller A, Xia Y, Croman K, et al. The honey badger of BFT protocols[C]//Proceedings of the 2016 ACM SIGSAC conference on computer and communications security. 2016: 31-42.
[9] Guo B, Lu Z, Tang Q, et al. Dumbo: Faster asynchronous bft protocols[C]//Proceedings of the 2020 ACM SIGSAC Conference on Computer and Communications Security. 2020: 803-818.
[10] Danezis G, Kokoris-Kogias L, Sonnino A, et al. Narwhal and tusk: a dag-based mempool and efficient bft consensus[C]//Proceedings of the Seventeenth European Conference on Computer Systems. 2022: 34-50.
