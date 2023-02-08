---
title: webpack的hash引发的问题
url: https://www.yuque.com/endday/blog/uzayee
---

<https://webpack.docschina.org/configuration/optimization/#optimizationmoduleids>
<https://blog.csdn.net/weixin_44800563/article/details/119735888>
<https://github.com/jiangjiu/blog-md/issues/49>
<https://github.com/HexMox/blog/issues/6>
[手摸手，带你用合理的姿势使用webpack4（上）](https://segmentfault.com/a/1190000015919863)

> 在v4.16.0版本中废除了optimization.occurrenceOrder、optimization.namedChunks、optimization.hashedModuleIds、optimization.namedModules 这几个配置项，替换成了optimization.moduleIds 和 optimization.chunkIds，但文档完中全没有任何体现，所以你在新版本中还按照文档那样配置其实是没有任何效果的。

<https://github.com/webpack/webpack/issues/9520>

[webpack4 完整配置json](https://github.com/webpack/webpack/blob/webpack-4/schemas/WebpackOptions.json)

| hash类型 | 区别 |
| --- | --- |
| hash	 | hash是根据整个项目构建，只要项目里有文件更改，整个项目构建的hash值都会更改，并且全部文件都共用相同的hash值 |
| chunkhash	 | chunkhash根据不同的入口文件(Entry)进行依赖文件解析、构建对应的代码块（chunk），生成对应的哈希值，某文件变化时只有该文件对应代码块（chunk）的hash会变化 |
| contentHash	 | 每一个代码块（chunk）中的js和css输出文件都会独立生成一个hash，当某一个代码块（chunk）中的js源文件被修改时，只有该代码块（chunk）输出的js文件的hash会发生变化 |

————————————————
版权声明：本文为CSDN博主「Phil01」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：<https://blog.csdn.net/qq_33594101/article/details/107981866>

```typescript
module.exports = {
    configureWebpack: (config)=>{
        const plugins = config.plugins;
        for(let i=0;i<plugins.length;i++){
            if(plugins[i] instanceof webpack.HashedModuleIdsPlugin){
                plugins.splice(i,1)
            }else if(plugins[i] instanceof webpack.NamedChunksPlugin){
                plugins.splice(i,1)
            }
        }
        plugins.push(new webpack.HashedModuleIdsPlugin({//生成稳定的模块id
            hashDigest: 'hex',
            hashDigestLength:8
        }));
        plugins.push(new webpack.NamedChunksPlugin((chunk) => {//生成稳定的chunk id
            if (chunk.name) {
                return chunk.name;
            }
            return "W"+(++id)
        }));
        return  void 0
    }
};
```

<https://blog.csdn.net/qq_29456953/article/details/106431477>

还比如在v4.16.0版本中废除了optimization.occurrenceOrder、optimization.namedChunks、optimization.hashedModuleIds、optimization.namedModules 这几个配置项，替换成了optimization.moduleIds 和 optimization.chunkIds，但文档完中全没有任何体现，所以你在新版本中还按照文档那样配置其实是没有任何效果的。
