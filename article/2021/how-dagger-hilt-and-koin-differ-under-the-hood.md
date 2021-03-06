> - 原文地址：[How Dagger, Hilt and Koin differ under the hood?](https://proandroiddev.com/how-dagger-hilt-and-koin-differ-under-the-hood-c3be1a2959d7)
> - 原文作者：[Paulina Sadowska](https://medium.com/@PaulinaSadowska)
> - 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> - 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/article/2021/how-dagger-hilt-and-koin-differ-under-the-hood.md](https://github.com/xitu/gold-miner/blob/master/article/2021/how-dagger-hilt-and-koin-differ-under-the-hood.md)
> * 译者：[keepmovingljzy](https://github.com/keepmovingljzy)
> * 校对者：[Kimhooo](https://github.com/Kimhooo)、[Mingron](https://github.com/1autodidact)

# Dagger，Hilt 以及 Koin 的本质区别是什么？

![](https://cdn-images-1.medium.com/max/7186/1*inIehvxU_kZ5kEAR1ln0tA.png)

Dagger 和 Koin 无疑是 Android 中最流行的两个依赖注入框架。这两个库具有相同的用途，而且看起来非常相似，但它们在底层的工作方式却非常不同。

**那么 Hilt 是什么呢**？Hilt 是一个内部使用 Dagger 的库，只是简化了它的用法，因此我在这里所说的有关 Dagger 的内容也适用于 Hilt。

在本文中，我不会告诉您应该选择哪个库。 相反，我想向您展示它们的本质区别以及**这些差异对您的应用造成的影响**。

## Dagger

如果我们希望 Dagger 提供某个类的实例，我们要做的就是在构造函数中添加 `@Inject` 注解。

![](https://cdn-images-1.medium.com/max/2420/1*i4L9ygcw9OR9t_vM--dHzQ.png)

添加这个注解后，Dagger 会在构建时为这个类生成一个 Factory。在该用例下，由于它的类名是 `CompositeAdapter`, 它会生成一个名为 `CompositeAdapter**_**Factory`的类。

此类包含创建 `CompositeAdapter` 类的实例所需的所有信息。

![code generated by Dagger (fragment)](https://cdn-images-1.medium.com/max/3240/1*efDF_mkL0ErVXeg83BCghg.png)

如你所看到该工厂类实现了 `get()` 并返回了一个新的 `CompositeAdapter` 实例。这实际上是此类实现的 `Provider \ <T>` 接口中指定的方法。其他类可以使用 `Provider \ <T>` 接口来获取一个类的实例。

![](https://cdn-images-1.medium.com/max/2664/1*zA4mSvWmvCd7jt-AfoMbXw.png)

### 如果我们用 Hilt 代替 Dagger 呢？

在这个例子中，没有任何区别。Hilt 是一个内部使用 Dagger 的库，我向你展示的类是由 Dagger 生成的。如果您使用 Hilt，它确实为我们生成了一些额外的类，这些类简化了 Dagger 的使用，并减少了我们需要编写的样板代码的数量。但核心部分保持不变。

![](https://cdn-images-1.medium.com/max/3340/1*zXxqXzl7dZjAeN6CFz9zgw.png)

## Koin

Koin 与 Dagger 以及 Hilt 相比，管理依赖项的方法完全不同。要在 Koin 中注册依赖项，我们不会使用任何注解，因为**Koin不会生成任何代码**。相反，我们必须为模块提供工厂，这些模块将用于创建项目中所需的每个类的实例。

Koin 将这些工厂类的引用添加到 `InstancesRegistry` 类中，该类包含对我们编写的所有工厂的引用。

![](https://cdn-images-1.medium.com/max/3336/1*XyDFpT26VnVQ4pbfShc1hQ.png)

该 map 中的 key 是类的全名或使用命名参数时提供的名称。对应的值是我们编写的工厂，将用于创建类的实例。

要获得依赖关系，我们需要调用 `get()` (比如在一个工厂类中) 或者通过在 activities 或 fragments 中调用 inject() 委托属性 ，从而懒加载 `get()`。`get()` 方法将查找为给定类型的类注册工厂，并将其注入其中。

![](https://cdn-images-1.medium.com/max/3140/1*H7AAyPRwZFTXQqX44UuhIA.png)

## 有什么影响？

Dagger 生成代码来提供依赖，而 Koin 不生成代码，这实际上带来了一些影响。

### 1. 错误处理

因为 **Dagger 是一个编译时依赖注入框架，**如果我们忘记提供某些依赖，我们几乎会立即知道我们的错误，因为我们的项目将**构建失败**。

例如，如果我们忘记向构造函数的 `CompositeAdapter` 中添加 `@Inject` 注解，并尝试将其注入 fragment 中，则构建将失败，并显示适当的错误，确切地告诉我们出了什么问题。

![Dagger build output where there is @ Inject annotation missing](https://cdn-images-1.medium.com/max/3628/1*VLDmTJ1ZRpQPg_AHGffapw.png)

在 Koin 中的情况有所不同，因为它**不会生成任何代码**。如果我们忘记为 `CompositeAdapter` 类添加工厂，**应用将会成功构建，但是会抛出 `RuntimeException` 一旦我们请求获取这个类的实例**。它可能会在应用启动时发生，因此我们可能会立即注意到它，但也可能稍后在其他屏幕上或当用户执行某些特定操作时发生。

![Koin throws an exception when a factory for CompositeAdapter is missing](https://cdn-images-1.medium.com/max/3560/1*VObvkpv2KSdB6vbX-xIIxQ.png)

### 2. 对构建时间的影响

Koin 不生成任何代码的优点是：**它对我们的构建时间的影响要小得多**。Dagger 需要使用注解处理器来扫描代码并生成适当的类。这可能需要一些时间，可能会减慢我们的构建。

### 3. 对运行时性能的影响

从另一方面来说，因为 Koin **在运行时解析依赖项**，所以它的**运行时性能**稍差一些。

![](https://cdn-images-1.medium.com/max/3016/1*eZc3sHc0KXNjTe9cXVMkCA.png)

**到底相差多少呢**？为了估算性能差异我们可以使用[该库](https://github.com/Sloy/android-dependency-injection-performance)，其中 Rafa Vázquez 基于不同的设备上测量并比较了这两个库。测试数据的编写方式可以模拟多个级别的传递依赖关系，因此它不仅仅是具有 4 个类的虚拟应用程序。

![source: [https://github.com/Sloy/android-dependency-injection-performance](https://github.com/Sloy/android-dependency-injection-performance)](https://cdn-images-1.medium.com/max/2332/1*Krd-dXtSa2sD-sweFsa3Uw.png)

如您所见，Dagger 对启动性能几乎没有影响。另一方面，在 Koin 中，我们可以看到它花费了很多时间。在 Dagger 中注入依赖也比在 Koin 中快一些。

## 总结

正如我在本文开始时所说的，我这里的目标不是告诉您要使用哪个库。我在两个不同的大项目中都使用了 Koin 和 Dagger。老实说，我认为选择 Dagger 还是 Koin 并不重要，**重要的是能够让你编写干净、简单且易于单元测试的代码**。我认为所有的库：Koin，Dagger 和 Hilt 都达到了这个目的。

所有这些库都有自己的优势，我希望了解它们在底层是如何工作的，能够帮助您自己决定哪种库最适合您的应用。

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

------

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
