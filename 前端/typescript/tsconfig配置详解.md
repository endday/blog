---
title: tsconfig配置详解
url: https://www.yuque.com/endday/blog/nx90fs
---

<a name="OHpgp"></a>

## 使用tsconfig.json

- 不带任何输入文件的情况下调用tsc，编译器会从当前目录开始去查找tsconfig.json文件，逐级向上搜索父目录。
- 不带任何输入文件的情况下调用tsc，且使用命令行参数--project（或-p）指定一个包含tsconfig.json文件的目录。

当命令行上指定了输入文件时，tsconfig.json文件会被忽略。

```json
{
  "include": [
    "./index.ts"
  ], //只编译的文件
  "exclude": [
    "./test.ts"
  ], //除了[]中的文件，编译其他所有文件
  // 编译过程中一些编译的属性或者编译的配置
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */
    /* Projects */
    // "incremental": true,                              /* 启用增量编译 */
    // "composite": true,                                /* 启用允许将类型脚脚本项目与项目引用一起使用的约束 */
    // "tsBuildInfoFile": "./",                          /* 指定.tsbuildinfo增量编译文件的文件夹。 */
    // "disableSourceOfProjectReferenceRedirect": true,  /* 在引用复合项目时，将禁用首选的源文件，而不是声明文件 */
    // "disableSolutionSearching": true,                 /* 在编辑时，选择一个项目退出多项目引用检查。 */
    // "disableReferencedProjectLoad": true,             /* 减少通过类型脚本自动加载的项目数量。 */
    /* Language and Environment */
    "target": "es2016", /* 为发出的JavaScript设置JavaScript语言版本，并包含兼容的库声明 */
    // "lib": [],                                        /* 指定一组描述目标运行时环境的捆绑库声明文件 */
    // "jsx": "preserve",                                /* 指定生成的JSX代码 */
    // "experimentalDecorators": true,                   /* 为TC39第二阶段的草稿装饰器提供实验支持。 */
    // "emitDecoratorMetadata": true,                    /* 为源文件中的修饰声明的设计类型元数据 */
    // "jsxFactory": "",                                 /* 指定针对ReactJSX发射时使用的JSX工厂函数，例如“React.createElement”或“h” */
    // "jsxFragmentFactory": "",                         /* 指定用于片段的JSX片段引用例如：React.Fragment' or 'Fragment. */
    // "jsxImportSource": "",                            /* 指定用于在使用`jsx时导入JSX工厂函数的模块说明符：反应-jsx*`.`  */
    // "reactNamespace": "",                             /* 指定为`创建元素`调用的对象。这只适用于针对`，`JSX emit。*/
    // "noLib": true,                                    /* 禁用包括任何库文件，包括默认的lib.d.ts。 */
    // "useDefineForClassFields": true,                  /* emit ECMAScript-符合标准的类字段。 */
    /* Modules */
    "module": "commonjs", /* 指定所生成的模块代码。 */
    // "rootDir": "./",                                  /* 在源文件中指定根文件夹。 */
    // "moduleResolution": "node",                       /* 指定TypeScript如何从给定的模块指定符中查找文件。 */
    // "baseUrl": "./",                                  /* 指定要解析非相对模块名称的基本目录。 */
    // "paths": {},                                      /* 指定一组将导入重新映射到其他查找位置的条目。 */
    // "rootDirs": [],                                   /* 在解析模块时，允许将多个文件夹视为一个文件夹。*/
    // "typeRoots": [],                                  /* 指定多个类似于`./node_modules/@的`类型的文件夹。 */
    // "types": [],                                      /* 指定要在源文件中引用的类型包名。 */
    // "allowUmdGlobalAccess": true,                     /* 允许从模块访问UMD全局文件。 */
    // "resolveJsonModule": true,                        /* 启用导入.json文件 */
    // "noResolve": true,                                /* 不允许`import`，`require`或`reference`来扩展类型应该添加到项目的文件数量。*/
    /* JavaScript Support */
    // "allowJs": true,                                  /* 允许JavaScript文件成为您的程序的一部分。使用`检查JS`选项从这些文件中获取错误。 */
    // "checkJs": true,                                  /* 在已检查类型的JavaScript文件中启用错误报告。 */
    // "maxNodeModuleJsDepth": 1,                        /* 指定用于从`node_modules`中检查JavaScript文件的最大文件夹深度。仅适用于`允许的Js`。 */
    /* Emit */
    // "declaration": true,                              /* 从项目中的typeScript和JavaScript文件生成.d.ts文件。 */
    // "declarationMap": true,                           /* 为d.ts文件创建源集映射。 */
    // "emitDeclarationOnly": true,                      /* 只输出d.ts文件，而不输出JavaScript文件。 */
    // "sourceMap": true,                                /* 为发出的JavaScript文件创建源映射文件。 */
    // "outFile": "./",                                  /* 指定一个将所有输出捆绑到一个JavaScript文件中的文件。如果`声明`为true，则还指定一个捆绑所有.d.ts输出的文件。 */
    // "outDir": "./",                                   /* 为所有发出的文件指定一个输出文件夹。 */
    //去除注释
    // "removeComments": true,                           /* 禁用注释 */
    // "noEmit": true,                                   /* 从编译中禁用emit文件。 */
    // "importHelpers": true,                            /* 允许从每个项目的tslib中导入一次助手函数，而不是为每个文件包含它们。 */
    // "importsNotUsedAsValues": "remove",               /* 为仅用于类型的导入指定发射/检查行为 */
    // "downlevelIteration": true,                       /* Emit 更兼容，但冗长和性能较差的JavaScript的迭代。 */
    // "sourceRoot": "",                                 /* 为调试器指定根路径以查找引用源代码的位置。 */
    // "mapRoot": "",                                    /* 指定调试器应该定位映射文件的位置，而不是生成的位置。 */
    // "inlineSourceMap": true,                          /* 在发出的JavaScript中包含源源映射文件。*/
    // "inlineSources": true,                            /* 在发出的JavaScript中的源映射中包含源代码。 */
    // "emitBOM": true,                                  /* 在输出文件的开头发出UTF-8字节顺序标记(BOM)。 */
    // "newLine": "crlf",                                /* 设置发射文件的换行符。 */
    // "stripInternal": true,                            /* 禁用在其JSDoc注释中具有`@internal`的发射声明。 */
    // "noEmitHelpers": true,                            /* 禁用在编译输出中生成自定义助手函数。 */
    // "noEmitOnError": true,                            /* 如果报告了任何类型检查错误，则禁用发射文件。 */
    // "preserveConstEnums": true,                       /* 禁用擦除生成的代码中的`常数枚举`声明。 */
    // "declarationDir": "./",                           /* 为生成的声明文件的输出目录。 */
    // "preserveValueImports": true,                     /* 在JavaScript输出中保留未使用的导入值，否则将被删除。 */
    /* Interop Constraints */
    // "isolatedModules": true,                          /* 确保每个文件都可以安全地传输，而不依赖于其他导入。 */
    // "allowSyntheticDefaultImports": true,             /* 当模块没有默认导出时，允许“从y导入x”。 */
    "esModuleInterop": true, /* 发出额外的JavaScript，以简化对导入CommonJS模块的支持。这使得`允许合成默认导入`以实现类型兼容性。    */
    // "preserveSymlinks": true,                         /* 禁用对符号链接到其实际路径的解析。这与节点中的同一标志相关联。 */
    "forceConsistentCasingInFileNames": true, /* 确保进口时外壳正确。 */
    /* Type Checking */
    //strict为true,代表下面的都是true
    "strict": true,
    //                                    /* Enable all strict type-checking options. */
    //不要求必须显示的设置any
    // "noImplicitAny": true,                            /*为隐含`任何`类型的表达式和声明启用错误报告  */
    //不强制进行null监测
    // "strictNullChecks": true,                         /* 在类型检查时，考虑`null`和`未定义`。 */
    // "strictFunctionTypes": true,                      /* 在分配函数时，请检查以确保参数和返回值与子类型兼容 */
    // "strictBindCallApply": true,                      /* 检查`绑定`、`调用`和`应用`方法的参数是否与原始函数匹配。 */
    // "strictPropertyInitialization": true,             /* 检查在构造函数中已声明但未设置的类属性。 */
    // "noImplicitThis": true,                           /* 当`this`具有类型`any`时，启用错误报告。*/
    // "useUnknownInCatchVariables": true,               /* 将catch子句变量类型化为“unknown”，而不是“any”。 */
    // "alwaysStrict": true,                             /* 确保始终发出“严格使用”信号。 */
    // "noUnusedLocals": true,                           /* 在未读取局部变量时启用错误报告。*/
    // "noUnusedParameters": true,                       /* 在未读取函数参数时引发错误 */
    // "exactOptionalPropertyTypes": true,               /* 将可选的属性类型解释为已写入的，而不是添加“未定义的”。*/
    // "noImplicitReturns": true,                        /* 为不在函数中显式返回的代码路径启用错误报告。 */
    // "noFallthroughCasesInSwitch": true,               /* 为开关语句中的故障情况启用错误报告。*/
    // "noUncheckedIndexedAccess": true,                 /* 在索引签名结果中包含“未定义的”*/
    // "noImplicitOverride": true,                       /* 确保在派生类中标记覆盖成员。 */
    // "noPropertyAccessFromIndexSignature": true,       /* 对使用索引类型声明的密钥强制使用索引访问器 */
    // "allowUnusedLabels": true,                        /* 禁用对未使用的标签的错误报告。 */
    // "allowUnreachableCode": true,                     /* 禁用对不可达代码的错误报告。 */
    /* Completeness */
    // "skipDefaultLibCheck": true,                      /* 跳过类型检查。d.ts类型脚本中包含的ts文件。 */
    "skipLibCheck": true /* 跳过类型检查所有。d.ts文件。 */
  }
}

作者：Geek喜多川
链接：https://juejin.cn/post/7078666410339565576
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
