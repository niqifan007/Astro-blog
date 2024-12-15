---
title: 默克尔树(Merkle tree)的Go实现
published: 2023-08-23
description: '使用go语言实现默克尔树'
image: 'https://esay.579878700.xyz/i/2023/08/23/1227pg2.jpg'
tags: [go]
category: 'Guides'
draft: false 
lang: ''
---

### Merkle trees

___

Merkle树是区块链技术的基本组成部分。它是由不同数据块的散列组成的数学数据结构，用作块中所有交易的摘要。

它还允许对大量数据中的内容进行有效和安全的验证。此结构有助于验证数据的一致性和内容。比特币和以太坊都使用Merkle树结构。Merkle树也被称为哈希树。

从根本上说，Merkle树是数据结构树，其中每个叶节点都用数据块的哈希标记，非叶节点用加密标记 其子节点标签的哈希值。叶节点是树中的最低节点。

![图片.png](https://img.learnblockchain.cn/attachments/2023/02/R9Yl2Xvj63e26ec555cc7.png)

### 原理

区块链中每个区块都会有一个 Merkle 树，它从叶子节点（树的底部）开始，一个叶子节点就是一个交易哈希。叶子节点的数量必须是双数，但是并非每个块都包含了双数的交易。如果一个块里面的交易数为单数，那么就将最后一个叶子节点（也就是 Merkle 树的最后一个交易，不是区块的最后一笔交易）复制一份凑成双数。

从下往上，两两成对，连接两个节点哈希，将组合哈希作为新的哈希。新的哈希就成为新的树节点。重复该过程，直到仅有一个节点，也就是树根。根哈希然后就会当作是整个块交易的唯一标示，将它保存到区块头，然后用于工作量证明。

___

### 代码实现

```golang
package main

import (
	"bufio"
	"crypto/sha256"
	"fmt"
	"os"
	"strconv"
)

//默克尔树节点结构体
type Node struct {
	Index    int
	Value    string
	RootTree *MHTree
}

//默克尔树结构体
type MHTree struct {
	Length   int
	Nodes    []Node
	rootHash string
}

//获得默克尔树根节点哈希值
func (t *MHTree) GetRootHash() string {
	//不管是否存储，都重新计算哈希值
	t.rootHash =   t.Nodes[1].getNodeHash()                            
	return t.rootHash
}

//计算默克尔树中某个节点的哈希值
func (n *Node) getNodeHash() string {
	//叶子节点，则直接计算该节点Value的哈希值
	if n.Value != "" {
		return calDataHash(n.Value)
	}
	//非叶子节点，则递归计算哈希值,其为2个子节点哈希值的哈希值123123123123
    return calDataHash(n.RootTree.Nodes[n.Index*2].getNodeHash() + n.RootTree.Nodes[n.Index*2+1].getNodeHash()
        )                                                         

//计算数据的哈希值
func calDataHash(data string) string {
	hash := sha256.New()
	hash.Write([]byte(data))
	return string(hash.Sum(nil))
}

//从结构化文件创建默克尔树
func CreateMHTree(fileName string) MHTree {
	var tree MHTree
	//打开文件
	file, err := os.Open(fileName)
	if err != nil {
		panic(err)
	}
	defer file.Close()
	//获取读取器
	buf := bufio.NewReader(file)
	//读取首行，获得叶子节点数目（要求叶子节点数目为2的整数次幂
	dataCountStr, _, _ := buf.ReadLine()
	dataCount, _ := strconv.Atoi(string(dataCountStr))

	//判断幂次
	level := 0
	for i := 1; ; i++ {                                          
		if 2<<i == dataCount {
			level = i
			break
		}
	}
	//创建默克尔树

	//给非叶子节点赋值
	for i := 1; i <= 2<<level-1; i++ {                               
		tree.Nodes[i].Index = i
		tree.Nodes[i].RootTree = &tree
	}
	//读取文件数据，给叶子节点赋值
	for i := 2 << level; i < tree.Length; i++ {                  
		str, _, _ := buf.ReadLine()
		tree.Nodes[i].Index = i
		tree.Nodes[i].RootTree = &tree
		tree.Nodes[i].Value = string(str)
	}
	return tree
}

func main() {
	var fileName string

	fmt.Println("请输入原始数据文件名称")
	fmt.Scanln(&fileName)
	mhTree1 := CreateMHTree(fileName)
	fmt.Println("请输入比对数据文件名称")
	fmt.Scanln(&fileName)
	mhTree2 := CreateMHTree(fileName)

	hash1 := mhTree1.GetRootHash()
	hash2 := mhTree2.GetRootHash()

	if hash1 == hash2 {
		fmt.Println("用户没有改变数据")
	} else {
		fmt.Println("用户改变了数据")
	}

}

```
