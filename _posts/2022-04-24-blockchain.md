---
layout:post
title:"区块链共识协议"
description:"web测试"
tag："test"
---





# 区块链共识协议

**岳奥翔**    **PB19030883**

## 实验目的及要求


* 实现区块链上的POW证明算法

* 理解区块链上的难度调整的作用

## 实验原理

### 区块链共识协议

​	共识协议是为了决定谁可以写入区块的问题，区块链的决定是通过最长链来表示的，这个是因为最长的区块对应有最大的工作量投入在其中，保证区块链的安全性和一致性。相应地，为了保证区块链的出块保持在一个相对比较稳定的值，对应地，对进行区块链共识难度的调整来保证出块速度大致保持一致。

### Pow 工作量证明

​	工作量证明最早是为防止服务和资源滥用，或者拒绝服务攻击等场景而提出的一种经济对策。一般要求证明方在使用服务或资源之前，首先完成具有一定难度或者适当工作量的复杂运算，这种工作量对于证明方是“昂贵的”且“没有捷径”的，但对于验证方是快速和简单的。
​	PoW共识机制通过引入分布式节点的算力竞争来作为工作量证明，利用其算力来完成大量的哈希函数计算工作，以便选出每个10分钟时间窗口的唯一“记账人”，从而保证区块链账本数据的一致性和共识的安全性。

### 哈希函数

​	哈希函数具有以下特性：
1. 原始数据不能直接通过哈希值来还原，哈希值是没法解密的。
2. 特定数据有唯一确定的哈希值，并且这个哈希值很难出现两个输入对应相同哈希输出的情况。
3. 修改输入数据一比特的数据，会导致结果完全不同。
4. 没有除了穷举以外的办法来确定哈希值的范围。

​	可以通过使用哈希函数实现Pow共识协议：

1. 构建一个由上⼀个区块哈希值，当前区块数据对应哈希（区块数据的merkle根），时间戳，区块难度，计数器组成的区块头
2. 计算区块头产生的哈希值
3. 测试哈希值是否满足条件，即哈希值的前Bits位为0，Bits为调控区块计算难度的难度值
4. 满足条件则产生区块，否则返回2重新计算哈希值

## 实验平台

​		vscode环境下go语言。

## 实验步骤

### 完成Run()函数

​	函数功能：生成一个符合条件的区块
​	实现思路：将区块中的数据转换成Bytes型，计算数据的SHA256值，然后使用Validate()函数进行验证。迭代至验证成功。
​	函数实现：

```go
func (pow *ProofOfWork) Run() (int, []byte) {
	var BlockHead, TimeBytes, HashD, bitsByte, NonBytes []byte
	var d [32]byte
    //数据类型转换
	nonce := 0
	TimeBytes = IntToHex(pow.block.Timestamp)
	HashD = NewMerkleTree(pow.block.Data).RootNode.Data
	bitsByte = IntToHex((int64)(pow.block.Bits))
	//数据连接
	BlockHead = append(BlockHead, pow.block.PrevBlockHash...)
	BlockHead = append(BlockHead, HashD...)
	BlockHead = append(BlockHead, TimeBytes...)
	BlockHead = append(BlockHead, bitsByte...)
	NonBytes = IntToHex((int64)(nonce))
    //进行第一次计算
	d = mySha256(BlockHead)
	pow.block.Hash = d[:]
	for !pow.Validate() {
        //判断是否符合条件并继续计算
        nonce++
		pow.block.Nonce = nonce
		NonBytes = IntToHex((int64)(nonce))
		//BlockHead := (pow.block.PrevBlockHash + HashD + TimeBytes + bitsByte + NonBytes)
		BlockHead1 = append(BlockHead, NonBytes...)
		d = mySha256(BlockHead1)
		pow.block.Hash = d[:]
	}
	return nonce, pow.block.Hash
}

```

### 完成Validate()函数

​	函数功能：验证生成的SHA256值是否符合条件
​	函数思路：比较Hash的前Bits为是否为0
​	函数实现：

```go
func (pow *ProofOfWork) Validate() bool {
	j := 0
	for i := 0; i < (int)(pow.block.Bits); i++ {
		j = 7 - (i % 8)
		//验证第i为是否为0
		if ((pow.block.Hash[i/8]) & (1 << j)) != 0 {
			//不为0返回错误
			return false
		}
	}
	return true//验证前Bits为均为0，验证成功
}
```

## 实验结果

<img src="images\blockchain\image-20220423181557002.png" alt="image-20220423181557002" style="zoom:75%;" />

<img src="images\blockchain\image-20220423181645984.png" alt="image-20220423181645984" style="zoom:75%;" />

<img src="images\blockchain\image-20220423181719938.png" alt="image-20220423181719938" style="zoom:75%;" />

​		实验结果：每一次哈希的前TargetBits位均为0，区块产生正确。随着TargetBits的增长，计算次数也随着增加，从图中可见，每增加一点难度，计算次数大概增加一倍，符合理论预期。

## 实验总结

​		通过本次实验，实现了基于Pow的区块链共识协议。加深对Pow协议，哈希函数以及区块，区块头，区块链结构的了解。
