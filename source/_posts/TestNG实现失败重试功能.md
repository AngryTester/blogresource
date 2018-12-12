---
title: TestNG实现失败重试功能
date: 2017-11-07 15:21:49
tags: [TestNg]
---

## 使用场景

前端自动化测试的稳定性是个很大的问题，在一些不追求速度追求稳定性的场景下，我们可以通过失败重试的功能来达到测试结果的稳定性．

## 为什么需要记录下来

之前为了兼容JDK7的环境，TestNG的版本一直用的是6.9.6的版本，但是注意了，这个版本是有bug的，对于不带DataProvider的情况，重试功能可以正常使用，但是一旦加上数据驱动，运行结果就各种混乱了．因为这个版本问题，困住近一周．

自己也是一根筋，没有想到去TestNG的官方issue里找答案，结果在开发者的解答中说6.9.7版本已经解决这个bug，抱着试一试的心态更新了一下TestNG的版本，果然，结果正常了，瞬间想骂脏字．

## 需要记录的几个点

- 1.TestNG的版本问题

<!-- more -->
> 这个前面已经说了，不再赘述．同时运行环境必须升级成JDK8.

- 2.关键是自定义监听实现`IRetryAnalyzer`接口:

```java
    private static Map<Long, Integer> retryMap = new HashMap<Long, Integer>();
	private static int maxRetryCount = 3


	private boolean isRetryAvailable(ITestResult result) {
		return retryMap.get(Thread.currentThread().getId()) < maxRetryCount;
	}

    @override
	public boolean retry(ITestResult result) {
		if (isRetryAvailable(result)) {
			int count = retryMap.remove(Thread.currentThread().getId());
			count++;
			System.out.println(result.getName() + "运行失败，下面进入" + "第" + count + "次" + "重运行");
			Reporter.log(result.getName() + "运行失败，下面进入" + "第" + count + "次" + "重运行");
			retryMap.put(Thread.currentThread().getId(), count);
			return true;
		}
		// retryCount = 0;
		return false;
	}
```

- 3.多线程运行

为了运行速度，我们还得把多线程加上，从上面的代码也可以看出来，我们会根据线程id来取对应的`count`来做判断．`retryMap`的初始化应该在每个方法执行开始时做．

```java
    @override
    public void beforeInvocation(IInvokedMethod method, ITestResult testResult) {
            // 按照线程id将重试次数初始化
            if (!retryMap.containsKey(Thread.currentThread().getId())) {
                retryMap.put(Thread.currentThread().getId(), 0);
            }
            ...
    }
```

## 其他

> 最大重试次数可以写在配置文件里，如果没有重试的属性或者属性值为0，则代表不重试．
