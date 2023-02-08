---
title: npm7的新特性引发的问题
url: https://www.yuque.com/endday/blog/ksc9uz
---

正如大家所期待， npm CLI 7 现在已经可以使用了

除了一些新特性和不兼容更改之外。与npm 6相比，我们对npm 7的性能方面产生了一些重要的影响，其中包括：

依赖包数量上减少了54%（npm 7 67个，npm 6 123个）
代码测试覆盖率增加了54%(npm 7 94% vs npm 6 77%)
在各种示例中的各种benchmarks中看到了显着的性能提升
注意，npm 7现在已发布到npm仓库的最新版本，执行npm install --global 时将默认安装npm 7。如果要安装npm 6，请执行npm install --global npm @6

不兼容改动
尽管对npm内部进行了较大的修改，但我们仍在努力，以确保对大多数工作流的破坏最小。也就是说，必须进行一些破坏性更改才能改善开发者的体验。可以在博客中查阅不兼容的改动。

会修改lockfile
一个需要注意的改动是新的lockfile格式，该格式会向后兼容npm 6用户

在以前的版本中，yarn.lock文件被忽略，npm CLI现在可以使用yarn.lock作为package元数据和依赖的来源。如果存在yarn.lock，则npm还将使它与package的内容保持最新。

使用npm 7并且在有v1 的lockfile的项目中执行npm install，则会把lock file文件的内容取代成v2的格式。如果想避免这种行为，可以通过执行npm install --no-save

peer dependencies
npm 7中引入的一项新功能是自动安装peer dependencies。在npm的之前版本（4-6）中，peer dependencies冲突会有版本不兼容的警告，但仍会安装依赖并不会抛出错误。在npm 7中，如果存在无法自动解决的依赖冲突，将会阻止安装。

可以通过使--force选项重新安装来绕过冲突，或者选择--legacy-peer-deps选项peer dependencies的依赖关系（类似于npm版本4-6）。

由于许多包都依赖宽松的peer dependencies解析，npm 7将打印警告并解决包依赖树中存在的大多数同级冲突，因此这些冲突不能手动处理。要在所有层级强制执行严格正确的peer dependencies依赖关系，请使用--strict-peer-deps选项。

感谢
最后，我们要向感谢那些提交了更改、参加了RFC讨论、提供了反馈和作为早期采用者的社区成员。之后我们仍致力于继续改进npm CLI，因此，如果你将来有任何反馈，请使用npm/feedback仓库来讨论。
————————————————
版权声明：本文为CSDN博主「flytam」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：<https://blog.csdn.net/flytam/article/details/113760841>

1. **自动安装 peerDependencies导致一些依赖相互覆盖的bug出现**
2. **yarn.lock的支持会让用户一脸懵逼**

**结论，不要随便升级npm，需要和node版本一样，保持稳定**
