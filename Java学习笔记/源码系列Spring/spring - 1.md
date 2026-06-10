
第一步： 断点打启动入口

### 要回答的问题： 

## 1）`run()` 进去之后，Spring 先干了什么？

> `run()` 先做的是**启动前准备**，包括：
> 
> - 计时
> - 获取并处理启动参数
> - 准备 environment
> - 通知 listeners
> - 打印 banner

## 2）`ApplicationContext` 是什么时候创建的？

> 在 environment 准备好、banner 打印完成后，通过 `createApplicationContext()` 创建 `ApplicationContext`。

## 3）`refresh()` 是什么时候调用的？

> `prepareContext()` 把 context 基本准备好以后，`refreshContext(context)` 会调用 `context.refresh()`，真正启动 Spring 容器。



接下来重点关注Spring的四个核心能力： 

## 1. `obtainFreshBeanFactory`

> IOC 容器的基础工厂准备好了

## 2. `invokeBeanFactoryPostProcessors`

> 开始处理配置类、扫描、BeanDefinition

## 3. `registerBeanPostProcessors`

> 为后续 Bean 生命周期增强做准备

## 4. `finishBeanFactoryInitialization`

> 真正开始创建单例 Bean


# 你这一轮要回答的新问题

下一轮 Debug 完以后，你回来回答我这 3 个问题：

### 1.`finishBeanFactoryInitialization()` 里，最终是谁触发了单例 Bean 的批量实例化？



### 2.`BeanFactoryPostProcessor` 和 `BeanPostProcessor` 的区别是什么？



### 3.你自己的 `TestService` 是不是在 `refresh()` 阶段就被创建了？



# 我再给你一个非常实用的理解口诀

你先背这个，特别有用：

先有 BeanDefinition，  
再改 BeanDefinition，  
再注册 BeanPostProcessor，  
最后实例化单例 Bean。

英文版你也可以顺便记一下：

First define beans,  
then process bean definitions,  
then register bean post-processors,  
and finally instantiate singleton beans.