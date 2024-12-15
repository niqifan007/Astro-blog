---
title: solidity动态数组的增删改查
published: 2023-09-17
description: 'solidity 内存(memory) 可变数组(动态数组) 的 增删改查'
image: 'https://s2.loli.net/2023/09/17/gLucRQZtHqVbNaA.png'
tags: [solidity, 区块链]
category: 'Guides'
draft: false 
lang: ''
---
## 数组类型

Solidity支持两种数组: 静态数组和动态数组。

又分`storage`与`memory`型数组

```
uint[] memory list;
//变长memory数组，使用前必须确定长度
list =new uint[](a);
//然后再使用，一般用在函数内
memory型数组不能使用pop,push也不能修改length
```

### 静态数组

```
// 固定长度为2的静态数组定义
uint[2] fixedArray;
//定长数组实例化
fixedArray = [4, 6];
```

静态数组不可新增元素，但可修改现有元素的值。

### 动态数组

动态数组，长度不固定，可以动态添加元素。

```
//声明
uint[] dynamicArray;
//初始化，这里实例化一个长度为2的数组，值为0。
dynamicArray = new uint[](2);
```
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

library Array {
    function push(uint256[] memory _nums, uint256 _num) internal pure {
        assembly {
            // 在可变数组的 末尾追加 一个value (_num)
            mstore(add(_nums, mul(add(mload(_nums), 1), 0x20)), _num)
            
            // 可变数组的length 加 1
            mstore(_nums, add(mload(_nums), 1))
            
            // 0x40 是空闲内存指针的预定义位置 (value 为 空闲指针开始位)
            mstore(0x40, add(mload(0x40), 0x20))
        }
    }

    function pop(uint256[] memory _nums) internal pure returns (uint256 num_) {
        assembly {
            // 取出可变数组的最后一个value
            num_ := mload(add(_nums, mul(mload(_nums), 0x20)))
            // length - 1
            mstore(_nums, sub(mload(_nums), 1))
        }
    }

    function del(uint256[] memory _nums, uint256 _index) internal pure {
        assembly {
            // 下标位置需 小于 数组.length
            if lt(_index, mload(_nums)) {
                // 将最后一个value 移到 _index 下标处 
                mstore(
                    add(_nums, mul(add(_index, 1), 0x20)),
                    mload(add(_nums, mul(mload(_nums), 0x20)))
                )
                // length - 1
                mstore(_nums, sub(mload(_nums), 1))
            }
        }
    }

    function update(
        uint256[] memory _nums,
        uint256 _index,
        uint256 _num
    ) internal pure {
        _nums[_index] = _num;
    }

    function get(uint256[] memory _nums, uint256 _index)
        internal
        pure
        returns (uint256)
    {
        return _nums[_index];
    }
}

contract testArr {
    using Array for uint256[];

    function push(uint256[] memory _nums, uint256 num)
        external
        pure
        returns (uint256[] memory)
    {
        _nums.push(num);
        return _nums;
    }

    function pop(uint256[] memory _nums)
        external
        pure
        returns (uint256[] memory, uint256)
    {
        uint256 num_ = _nums.pop();
        return (_nums, num_);
    }

    function del(uint256[] memory _nums, uint256 _index)
        external
        pure
        returns (uint256[] memory)
    {
        _nums.del(_index);
        return _nums;
    }

    function update(
        uint256[] memory _nums,
        uint256 _index,
        uint256 _num
    ) external pure returns (uint256[] memory) {
        _nums.update(_index, _num);
        return _nums;
    }

    function get(uint256[] memory _nums, uint256 _index)
        external
        pure
        returns (uint256)
    {
        return _nums.get(_index);
    }
}


```