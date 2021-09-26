## 写在前面

JavaScript 专题系列是我写的第二个系列，第一个系列是 [JavaScript 深入系列](https://github.com/mqyqingfeng/Blog/issues/17)。

JavaScript 专题系列共计 20 篇，主要研究日常开发中一些功能点的实现，比如防抖、节流、去重、类型判断、拷贝、最值、扁平、柯里、递归、乱序、排序等，特点是研(chao)究(xi) underscore 和 jQuery 的实现方式。

JavaScript 专题系列自 6 月 2 日发布第一篇文章，到 10 月 20 日发布最后一篇，感谢各位朋友的收藏、点赞，鼓励、指正。

20 篇下来，我们已经跟着 underscore 写了 debounce、throttle、unique、isElement、flatten、findIndex、findLastIndex、sortedIndex、indexOf、lastIndexOf、eq、partial、compose、memorize 共 14 个功能函数，跟着 jQuery 写了 type、isArray、isFunction、isPlainObject、isWindow、isArrayLike、extend、each 共 8 个功能函数，自己实现了 shallowCopy、deepCopy、curry、shuffle 共 4 个功能函数，加起来共有 26 个功能函数，除此之外，最后一篇还研究了 V8 的排序源码，真心希望读者能从这个系列中收获颇丰。

顺便宣传一下该博客的 Github 仓库：[https://github.com/mqyqingfeng/Blog](https://github.com/mqyqingfeng/Blog)，欢迎 star，鼓励一下作者。

而此篇，作为专题系列的总结篇，除了汇总各篇文章，作为目录篇之外，还希望跟大家聊聊，我为什么要写这个系列？

## 我为什么写专题系列？

如果说深入系列是为了了解 JavaScript 这门语言本身，专题系列就是为了用 JavaScript 具体实现一些功能，我希望它抽离于实践，无关乎 DOM、BOM，能对大家有所帮助，一想到这些，映入脑海的竟是那些年做过的前端面试题……

是的，回顾整个系列，你会发现，防抖、节流、去重、深浅拷贝、数组扁平化、乱序、柯里化等等不都是面试的经典吗？我还记得曾经为了准备面试，死记硬背了一个去重的函数，却从来没有研究过其他去重的方法，也从来没有想过它们之前的区别，防抖和节流更是傻傻分不清楚，深浅拷贝反正有 jQuery 的 extend 呢，数组扁平化，我也就有一个递归的思路，具体怎么实现我还真是不清楚，乱序我就没有思路了……哎，都是一知半解或是只是有所耳闻。

想着想着，便不知不觉写下了很多待研究的课题，研究的方法也随之浮现，那就是研究 underscore 以及 jQuery 的实现方式，曾经它们看起来很是神秘，也知道阅读起来并非难事，可还是想一探究竟。

然而研究的过程确实是十分的艰难，因为要做到看懂源码，理解实现的原理，然而，一段源码的实现往往会牵涉到多个地方，结果为了看懂某一个函数的具体实现，还要一连串的看多个函数，在理解源码的过程中，也会有很多的疑惑，我会告诉自己去理解每一个产生疑惑的地方，这句话说起来简单，做起来很难，我来举个例子吧，在数组乱序中，有一个方法是：

```js
arr.sort(function(){
    return Math.random() - 0.5;
});
```

然而，这个方法的实现是有问题的，它并不能做到真正的乱序。很多文章中，只是用 demo 验证了这种方法有问题，却从来没有说过这个方法究竟哪里有问题，然而我就是对此感到非常疑惑，因为我觉得这个方法很不错呀，思路巧妙，初见时，还有点小惊艳呢……可是为什么会有问题呢？我百思不得其解，搜了很多文章，也无果，最终，为了解决这个困惑，去看了 v8 的 sort 源码，然而这段源码也并不是很容易看的，资料少之又少，先要理解插入排序，快速排序，再去理解 v8 做的诸多优化，结果为了解决这个疑惑，看完了 v8 的 sort 源码，理解了 sort 的原理后，以数组 [1, 2, 3] 为例，细细分析这种乱序方法在 v8 下具体的排序过程，最后算出来 [1, 2, 3] 乱序后的 6 种结果的概率分别是多少，结果 3 还在原位置的概率有 50%! 到此，才算是心满意足的解决了这个困惑。

关于这个困惑的具体内容，可以查看该系列的第 19 篇文章。

除此之外，所有的函数我都会自己实现一遍，然而即便看懂了原理，实现也并非能一蹴而就，毕竟如果是你写，怎么能一开始就想得如此完善呢？所以我都是从一个简单的写法开始，向着 underscore 和 jquery 的最终实现方式，一个功能一个功能的迭代实现，你看这个系列很多的文章，都会跟大家讲解如何从零实现，一版一版的代码其实就是迭代实现时的记录。

感叹一下，写文章不容易呀~

## 全目录

1. [JavaScript专题之跟着underscore学防抖](https://github.com/mqyqingfeng/Blog/issues/22)
2. [JavaScript专题之跟着underscore学节流](https://github.com/mqyqingfeng/Blog/issues/26)
3. [JavaScript专题之数组去重](https://github.com/mqyqingfeng/Blog/issues/27)
4. [JavaScript专题之类型判断(上)](https://github.com/mqyqingfeng/Blog/issues/28)
5. [JavaScript专题之类型判断(下)](https://github.com/mqyqingfeng/Blog/issues/30)
6. [JavaScript专题之深浅拷贝](https://github.com/mqyqingfeng/Blog/issues/32)
7. [JavaScript专题之从零实现jQuery的extend](https://github.com/mqyqingfeng/Blog/issues/33)
8. [JavaScript专题之如何求数组的最大值和最小值](https://github.com/mqyqingfeng/Blog/issues/35)
9. [JavaScript专题之数组扁平化](https://github.com/mqyqingfeng/Blog/issues/36)
10. [JavaScript专题之学underscore在数组中查找指定元素](https://github.com/mqyqingfeng/Blog/issues/37)
11. [JavaScript专题之jQuery通用遍历方法each的实现](https://github.com/mqyqingfeng/Blog/issues/40)
12. [JavaScript专题之如何判断两个对象相等](https://github.com/mqyqingfeng/Blog/issues/41)
13. [JavaScript专题之函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42)
14. [JavaScript专题之偏函数](https://github.com/mqyqingfeng/Blog/issues/43)
15. [JavaScript专题之惰性函数](https://github.com/mqyqingfeng/Blog/issues/44)
16. [JavaScript专题之函数组合](https://github.com/mqyqingfeng/Blog/issues/45)
17. [JavaScript专题之函数记忆](https://github.com/mqyqingfeng/Blog/issues/46)
18. [JavaScript专题之递归](https://github.com/mqyqingfeng/Blog/issues/49)
19. [JavaScript专题之乱序](https://github.com/mqyqingfeng/Blog/issues/51)
20. [JavaScript专题之解读v8排序源码](https://github.com/mqyqingfeng/Blog/issues/52)

## 致歉

有些读者给我的文章留言，或感谢，或讨论，或指正，因为各种各样的原因，没能回复或及时回复，对此致以歉意。

## 下期预告

在我 Github 博客仓库的描述中，说到我预计写四个系列：JavaScript深入系列、JavaScript专题系列、ES6系列、React系列。专题系列完结，本来应该是写 ES6 系列，可是有一个朋友跟我说，写了这么多函数，可是该如何组织这些函数，形成自己的工具函数库呢？

对呀，既然都写了这么多工具函数，为什么不再进一步，将它们以某种方式组织起来呢？

我首先想到的便是借鉴 underscore，underscore 是如何组织代码的？又是如何实现链式调用的？又是如何实现拓展的？有很多值得研究的地方，所以我决定，在 ES6 系列之前，再进一步，写一个 underscore 系列，旨在帮助大家写出一个自己的 “underscore”。

感谢大家的阅读和支持，我是冴羽，underscore 系列再见啦！[]\~(￣▽￣)\~**
