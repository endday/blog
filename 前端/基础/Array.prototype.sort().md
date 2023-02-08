---
title: Array.prototype.sort()
url: https://www.yuque.com/endday/blog/rzoagm
---

面试问了sort的相关问题，我完全答不出，真的很惭愧，所以做个笔记

sort() 方法用就地（ in-place ）的算法对数组的元素进行排序，并返回数组。 sort 排序不一定是稳定的。默认排序顺序是根据字符串Unicode码点。

<a name="262a2599"></a>

### 稳定性

排序算法的稳定性大家应该都知道，通俗地讲就是能保证排序前2个相等的数其在序列的前后位置顺序和排序后它们两个的前后位置顺序相同。在简单形式化一下，如果A\[i]= A\[j]，A\[i]原来在位置前，排序后A\[i]还是要在A\[j]位置前。

    var fruit = ['cherries', 'apples', 'bananas'];
    fruit.sort(); 
    // ['apples', 'bananas', 'cherries']

    var scores = [1, 10, 21, 2]; 
    scores.sort(); 
    // [1, 10, 2, 21]
    // 注意10在2之前,
    // 因为在 Unicode 指针顺序中"10"在"2"之前

    var things = ['word', 'Word', '1 Word', '2 Words'];
    things.sort(); 
    // ['1 Word', '2 Words', 'Word', 'word']
    // 在Unicode中, 数字在大写字母之前,
    // 大写字母在小写字母之前.

> arr.sort()
>
> arr.sort(compareFunction)

compareFunction

可选。用来指定按某种顺序进行排列的函数。如果省略，元素按照转换为的字符串的各个字符的Unicode位点进行排序。

返回排序后的数组。原数组已经被排序后的数组代替。

如果没有指明 compareFunction ，那么元素会按照转换为的字符串的诸个字符的Unicode位点进行排序。例如 "Banana" 会被排列到 "cherry" 之前。当数字按由小到大排序时，9 出现在 80 之前，但因为（没有指明 compareFunction），比较的数字会先被转换为字符串，所以在Unicode顺序上 "80" 要比 "9" 要靠前。

如果指明了 compareFunction ，那么数组会按照调用该函数的返回值排序。即 a 和 b 是两个将要被比较的元素：

- 如果 compareFunction(a, b) 小于 0 ，那么 a 会被排列到 b 之前；
- 如果 compareFunction(a, b) 等于 0 ， a 和 b 的相对位置不变。备注： ECMAScript 标准并不保证这一行为，而且也不是所有浏览器都会遵守（例如 Mozilla 在 2003 年之前的版本）；
- 如果 compareFunction(a, b) 大于 0 ， b 会被排列到 a 之前。

  compareFunction(a, b) 必须总是对相同的输入返回相同的比较结果，否则排序的结果将是不确定的。
