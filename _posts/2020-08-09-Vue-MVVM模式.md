---
layout:     post
title:      vue keep-alive
subtitle:   vue的keepalive是如何实现的，具体缓存的时生命？
date:       2020-08-09
author:     BY CHU GAO
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - vue
    - keep-alive
    - object
    - 开源框架
---

# keep-alive
## props
- include 字符串或正则表达式，只有名称匹配的组件会被匹配
- exclude 字符串或正则表达式，任何名称匹配的组件都不会被缓存
- max 数字。最多可以缓存多少组件实例
> keep-alive 包裹动态组件时，会缓存不活动的组件实例
## 主要流程
1. 判断组件name，不在 include 或者在 exclude 中，直接返回vnode， 说明该组件不被缓存。<br/>
2. 获取组件实例key，如果有获取实例的key， 否则重新生成。<br/>
3. key生成规则，<abbr>cid + "::" + tag</abbr> ，仅靠cid是不够的，因为相同的构造函数可以注册为不同的本地组件。<br/>
4. 如果缓存对象内存在，则直接从缓存对象中获取组件实例给vnode，不存在则直接添加到缓存对象中。<br/>
5. 最大缓存数量，当缓存组件数量超过max值时，清楚keys组件内第一个组件。
# keep-alive 实现
```
    const patternTypes: Array<Function> = [String, RegExp, Array] // 接受字符串、正则、数组

    export default {
        name: 'keep-alive',
        abstract: true, // 抽象组件，是一个抽象组件，它自身不会渲染一个DOM元素，也不会出现在父组件链中。

        props: {
            include: patternTypes, // 匹配组件，缓存
            exclude: patternTypes, // 不去匹配的组件，不缓存 
            max: [String, Number], // 缓存组件的最大实例数
        },

        created() {
            // 用于缓存虚拟DOM数组和vnode的key
            this.cache = Object.create(null)
            this.key = []
        },

        destroyed() {
            // 销毁缓存cache组件实例
            for(const key in this.cache) {
                pruneCacheEntry(this.cache, key, this.keys)
            }
        },

        mounted() {
            // prune 消减精简[v.]
            // 去监控include和exclude的改变，根据最新的include和exclude的内容，来实时消减缓存的内容
            this.$watch('include', val => {
                pruneCache(this, name => matches(val, name))
            })
            this.$watch('exclude', val => {
                pruneCache(this, name => !matches(val, name))
            })
        }
    }
```
    


