---
title: 或许我们在 JavaScript 中不需要 this 和 class （示例代码搬运）
url: https://www.yuque.com/endday/blog/bg9pfy
---

```javascript
// 写一个 machine 函数达到如下效果

function machine() {}
machine('ygy').execute();
// start ygy
machine('ygy')
  .do('eat')
  .execute();
// start ygy
// ygy eat
machine('ygy')
  .wait(5)
  .do('eat')
  .execute();
// start ygy
// wait 5s（这里等待了5s）
// ygy eat
machine('ygy')
  .waitFirst(5)
  .do('eat')
  .execute();
// wait 5s
// start ygy
// ygy eat
```

```javascript
// 基于首次答案有微调

const defer = sec => new Promise(resolve => setTimeout(resolve, sec * 1000));

function machine(name) {
  const tasks = [];
  const initTask = () => {
    console.log(`start ${name}`);
  };
  tasks.push(initTask);

  function _do(str) {
    const task = () => {
      console.log(`${name} ${str}`);
    };
    tasks.push(task);
    return this;
  }

  function wait(sec) {
    const task = async () => {
      console.log(`wait ${sec}s`);
      await defer(sec);
    };
    tasks.push(task);
    return this;
  }

  function waitFirst(sec) {
    const task = async () => {
      console.log(`wait ${sec}s`);
      await defer(sec);
    };

    tasks.unshift(task);
    return this;
  }

  function execute() {
    tasks.reduce(async (promise, task) => {
      await promise;
      await task();
    }, undefined);
  }

  return {
    do: _do,
    wait,
    waitFirst,
    execute,
  };
}
```

```javascript
const defer = sec => new Promise(resolve => setTimeout(resolve, sec * 1000));

function machine(name) {
  const context = {};
  const tasks = [];
  const initTask = () => {
    console.log(`start ${name}`);
  };
  tasks.push(initTask);

  function _do(str) {
    const task = () => {
      console.log(`${name} ${str}`);
    };
    tasks.push(task);
    return context;
  }

  function wait(sec) {
    const task = async () => {
      console.log(`wait ${sec}s`);
      await defer(sec);
    };
    tasks.push(task);
    return context;
  }

  function waitFirst(sec) {
    const task = async () => {
      console.log(`wait ${sec}s`);
      await defer(sec);
    };

    tasks.unshift(task);
    return context;
  }

  function execute() {
    tasks.reduce(async (promise, task) => {
      await promise;
      await task();
    }, undefined);
  }

  // 用 Object.freeze 来防止调用者修改内部函数，保障安全
  return Object.freeze(
    Object.assign(context, {
      do: _do,
      wait,
      waitFirst,
      execute,
    })
  );
}
```

作者：serialcoder

链接：<https://juejin.im/post/5c92dfe7f265da60d428f119>

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
