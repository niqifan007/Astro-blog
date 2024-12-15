---
title: 在超级账本Hyperledger Fabric中开发智能合约遇到的问题
published: 2023-09-17
description: '在超级账本Hyperledger Fabric中开发智能合约遇到的问题'
image: 'https://s2.loli.net/2023/09/17/sK6adboOkR3gliW.png'
tags: [区块链, Hyperledger Fabric]
category: 'Guides'
draft: false 
lang: ''
---
## 1\. 不要尝试获取节点本地的时间作为key或value的值

即使每个节点的本地的时间系统是经过校对一致的，但同一个交易在每个节点上执行的时间点不一定是相同的。如果使用time.now().unix()来取节点本地的时间作为key或value的值，会导致每个节点执行同一个交易的结果不一致，从而造成交易失败。正确的做法是使用函数GetTxTimestamp()来获取交易的时间。每个节点通过函数GetTxTimestamp()来获取到的交易时间是相同的  
例子：  
t, err := stub.GetTxTimestamp()  
t.Seconds // 交易的时间，unix时间戳，精确到秒  
t.Nanos// 交易的时间，unix时间戳，精确到纳秒  
time.Unix(t.Seconds, int64(t.Nanos)).UnixNano() / 1e6 // 交易的时间，unix时间戳，精确到毫秒

但需要注意的是，每个节点通过函数GetTxTimestamp()来获取到的交易时间是相同的，但它取到的时间是客户端创建交易时的本地时间。如果客户端的本地时间没有校准的话，GetTxTimestamp()获取到的交易时间也是不准确的。

## 2\. 在同一个区块里多笔交易对同一键进行更新

对同一键进行更新的一个或多笔交易打包到同一块中，只有第1笔交易会成功，后面的交易会失败。相关的原因是：在一个区块里的多笔交易，只有在区块被提交后才会真正生效。而每一个键值都有版本号，假设一个键“key\_1”在本个区块提交前的版本号为9，然后在本个区块里有2笔交易分别更新键"key\_1"的值为"123"和"456"。那么当第1笔交易更新键“key\_1”的值时，键"key\_1"的版本号在本个区块里将更新为10，但在本区块未提交前，真正生效的版本号还是9。而当执行第2笔交易时，该交易读取到键"key\_1"的版本号还是9，这样第2笔交易还是把键"key\_1"的版本号更新为10，这样系统会判断为“双花”行为，从而引起冲突，导致只有第1笔交易会执行成功，第2笔交易执行失败。具体可以参考这篇文章 [https://learnblockchain.cn/article/620](https://learnblockchain.cn/article/620)  
简单地说，在同一个区块里多笔交易对同一键进行PutState()更新时，只有一笔交易是成功的，其它的交易会出现 MVCC\_READ\_CONFLICT报错。  
因此，试图在fabric合约里创建全局的计数器是一件不可靠的事情。

## 3\. 在同一个区块里多笔交易先后对同一个键进行更新和读取

假设在一个区块里有2笔交易，第1笔交易对键"key\_1"值更新为"123"，第2笔交易尝试读取键"key\_1"的值，结果第2笔交易读键"key\_1"的值结果为空。原因上面已经提及到，一个区块里的多笔交易，只有在区块被提交后才会真正生效。

## 4\. 在一个方法内对同一个键先写后读

在一个方法内对同一个键先进行写键值（putState()），然后再进行读键值（getState()），会发现前面的putState操作没有生效，原因也是同前面一样的道理

## 5\. 尝试通过合约使用加密算法对信息进行加密

有些加密算法对同样内容的信息进行加密，有可能每次加密的结果是不一样的。例如：RSA公钥对同一数据加密，每次的结果都不一样。这样会导致不同的节点执行加密的结果不一样，从而导致提交的内容不一致，造成交易失败

## 6\. 没有了解query和invoke的合约函数调用方式

超级账本调用合约函数的方式有2种：query和invoke  
通过query方式调用合约函数，只能查询链上数据，不能向链上写数据，并且不产生交易  
通过invoke方式调用合约函数，可以向链上写数据，并产生交易

如果一个合约函数里调用了PutState()操作，但是通过query方式来调用该合约函数，虽然不会报错，但PutState()操作实际没有生效，世界状态不会被改变

如果一个合约函数里没有调用PutState()操作，但是通过invoke方式来调用该合约函数，虽然不会报错，但却会产生交易

## 7\. 使用CouchDB，在使用select语句时没有对select的字段建立索引

超级账本可以使用Leveldb或CouchDB。如果要支持富查询，一般会选择使用CouchDB。CouchDB可以支持类似于SQL的select语句来实现富查询。但是CouchDB的效率并不高，在不建立索引的情况下，可能对2-3万条的数据使用select语句就会导致查询非常耗时，从而导致合约函数的执行超时而失败。而对于传统的关系型数据库，2-3万条的数据量，在不建立索引的情况下，对查询的影响不是很明显。

例如下面的select语句：  
"{"selector":{"StudentAge":{"$lte": 10}, "sort":\[{"RegisterTime": "asc"}\]}}"

最好就需要对字段StudentAge和RegisterTime建立索引。

想知道CouchDB在执行一条查询语句究竟有没有使用索引，用了哪些索引，可以通过向CouchDB发送post请求的方法，通过返回的response来查看使用了哪些索引。具体可参考下面的链接  
[https://docs.couchdb.org/en/latest/api/database/find.html#db-explain](https://docs.couchdb.org/en/latest/api/database/find.html#db-explain)  
1.3.8. /db/\_explain 这一节的内容

具体的操作可以通过postman来实现，  
post的地址如[http://127.0.0.1:5984/member\_goods/\_explain](http://127.0.0.1:5984/member_goods/_explain)  
发送的body就是查询select语句的内容，  
注意header里需要配置一个KeyValue Content-Type:application/json

## 8\. 使用CouchDB，在使用select语句时尽量避免使用sort进行排序

sort排序会严重消耗查询时间，容易造成查询超时，应尽量避免使用。使用sort语句，必须对sort语句中涉及的字段创建索引，否则会报错。

## 9.搭建Fabric区块链浏览器的坑

搭建Fabric区块链浏览器的时，如果之前曾经在同一台机器上使用不同的用户证书搭建过Fabric区块链浏览器，那么在同一台机器上使用新的用户证书搭建Fabric区块链浏览器时，一定要运行下面的命令清除之前的容器服务里的volume残留值：  
docker-compose down -v

详细可以参考这篇文章  
[https://blog.csdn.net/lyz19961221/article/details/118576529](https://blog.csdn.net/lyz19961221/article/details/118576529)

## 10.避免Fabric的智能合约函数陷入死循环

合约函数陷入死循环，不但会导致本次合约函数执行超时，而且后面会导致无法执行其它合约函数的问题，后果非常严重，详细可以参考这篇文章  
[https://zhuanlan.zhihu.com/p/572782459](https://zhuanlan.zhihu.com/p/572782459)
<!--more-->