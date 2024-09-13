[合集 \- Redis 入门(6\)](https://github.com)[1\.Redis 入门 \- 简介09\-04](https://github.com/hugogoos/p/18395520)[2\.Redis 入门 \- 安装最全讲解（Windows、Linux、Docker）09\-07](https://github.com/hugogoos/p/18401273)[3\.Redis 入门 \- 图形化管理工具如何选择，最全分类09\-08](https://github.com/hugogoos/p/18403519):[westworld加速](https://tianchuang88.com)[4\.Redis 入门 \- 五大基础类型及其指令学习09\-11](https://github.com/hugogoos/p/18407559)[5\.Redis 入门 \- C\#\|.NET Core客户端库六种选择09\-12](https://github.com/hugogoos/p/18409367)6\.Redis入门 \- C\#\|.NET Core封装Nuget包09\-13收起
经过前面章节的学习，可以说大家已经算Redis开发入门了。已经可以去到项目上磨砺了。


但是今天我还想和大家分享一章：封装自己的Redis C\#库，然后打包成Nuget包。


![](https://img2024.cnblogs.com/blog/386841/202409/386841-20240913002741924-1478649221.jpg)


首先要说明的是：不是要自己开发一个Redis客户端库，而是基于上篇文章中介绍的6大库，做一个简单封装，真的很简单的那种，就是包个壳子。


再来说说为什么要做这个事情。


# ***01***、原因


## 1\.可测试


我希望代码是以服务的方式注入到程序中，而不是静态方法的方式去调用。使用依赖注入来提供服务使得程序可测试性增强，如果做单元测试，依赖注入的服务很容易通过mock来测试，而静态方法往往很难被模拟，测试起来很不灵活；


## 2\.解耦


对于一个系统来说会使用到各种技术来达到某种能力，实现一个功能可能会有很多种方法，我们在意的是实现这个功能，而不是你用了什么方法。回到主题，我们需要的是使用Redis来实现业务功能，而具体用那个客户端库并不重要。


再举个例子比如今天我们选择了ServiceStack.Redis库接过遇到问题我们解决不了怎么办？难道业务不做了？不可能吧！而恰巧这个时候CSRedisCore库可以解决，你会怎么选？


这时候可能会想换的成本有多大？两种库方法名不统一，功能也不一样，如果系统中到处散落Redis方法调用，这可怎么换啊。


试想如果我们封装了一层，提供了一组接口，打包成Nuget包，这个时候大家用的就是这个Nuget包，而我们只需要把Nuget包里用ServiceStack.Redis实现的方法换成用CSRedisCore实现一下，大家直接更新一下Nuget包，可能一行代码都不用改就完成了替换。


因为我们依赖的是我们自己封装的接口，而不是具体Redis客户端库，因此可以解耦轻松替换。


![](https://img2024.cnblogs.com/blog/386841/202409/386841-20240913002733368-315592443.png)


## 3\.扩展


Redis的原生功能可以理解为基础功能，表明Redis有这种能力。但是我们怎么使用，怎么更好的发挥它的价值，这就是我们自己能力的体现了。


大家相过想过没有，为什么有的库商业化做的特别好，特别简单容易上手。商业化好说明提供的服务好，也就是说明它能帮你做很多事情，它在原生功能上加了很多自己的创作。


项目做多了，我们自己也会遇到一些相似的功能，如果这个时候我们想把这些相似功能封装一下，要放哪里呢？怎么给别人用？如果我们遇到一个功能现有库都没有相应能力，需要我们基于原生自己开发实现又应该在哪做呢？


这些功能积累的多了总要有个地方放吧，而这时如果我们自己封装了一层，放哪的问题是不是一下子就解决了，说不定做着做着就成了一个产品了呢。


不知道到这里，大家有没有感觉思路一下子被打开了。可能我们没写几行代码，但是这个格局一下子就打开了，而这几行代码也可能产生意想不到的收获，可能是无限可能，也可能就是你实现自己Redis客户端库的开端。


![](https://img2024.cnblogs.com/blog/386841/202409/386841-20240913002725213-634957087.jpg)


当然这也是我自己对编码，对封装的个人拙见。说这么多也是希望可以给大家一些帮助。


# ***02***、实现


下面闲话少说进入正题，如果来封装呢？


我们先梳理一下大致思路：


1\.我们需要一个接口，里面包含：原库原生能力Client、其他我们自定义功能。


2\.一个入口，别人要用，总要有个入口吧。


其实要求就这么简单。


下面我们就以封装CSRedisClient为例，首先定义IRedisService接口，里面包含Client字段以及两个演示的自定义方法



```
using CSRedis;
namespace Redis.RedisExtension
{
    public interface IRedisService
    {
        /// 
        /// RedisClient
        /// 
        /// 
        CSRedisClient Client { get; }
        #region 自定义方法
        /// 
        /// 获取指定 key 的值
        /// 
        /// 
        /// 
        /// 
        T Get<T>(string key);
        /// 
        /// 获取指定 key 的值
        /// 
        /// 
        /// 
        /// 
        Task<T> GetAsync<T>(string key);
        #endregion
    }
}

```

然后需要实现IRedisService，同时以CSRedisClient为构造函数入参，具体代码如下：



```
using CSRedis;
namespace Redis.RedisExtension
{
    public class RedisService : IRedisService
    {
        /// 
        /// 构造函数
        /// 
        /// 
        public RedisService(CSRedisClient redisClient)
        {
            Client = redisClient;
        }
        /// 
        /// CSRedis
        /// 
        /// 
        public CSRedisClient Client { get; }
        #region 自定义方法
        /// 
        /// 获取指定 key 的值
        /// 
        /// 
        public T Get<T>(string key)
        {
            return Client.Get(key);
        }
        /// 
        /// 获取指定 key 的值
        /// 
        /// 
        public Task<T> GetAsync<T>(string key)
        {
            return Client.GetAsync(key);
        }
        #endregion
    }
}

```

到这里第一个问题就解决了，对于第二个问题我们，我们可以对IServiceCollection进行扩展添加启动扩展方法AddRedisClientSetup，具体代码如下：



```
using CSRedis;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
namespace Redis.RedisExtension
{
    public static class RedisSetupExtensions
    {
        /// 
        /// Redis客户端启动项
        /// 
        /// 
        /// 
        public static IServiceCollection AddRedisClientSetup(this IServiceCollection services)
        {
            services.AddSingleton(serviceProvider =>
            {
                var configuration = serviceProvider.GetRequiredService();
                var setting = configuration["RedisConnectionString"];
                return new CSRedisClient(setting);
            });
            services.AddSingleton();
            return services;
        }
        /// 
        /// Redis客户端启动项
        /// 
        /// 
        /// 
        /// 
        public static IServiceCollection AddRedisClientSetup(this IServiceCollection services, string connectionString)
        {
            services.AddSingleton(serviceProvider =>
            {
                return new CSRedisClient(connectionString);
            });
            services.AddSingleton();
            return services;
        }
    }
}

```

为什么要提供两个重载方法，因为如果用户基于我们的约定，在配置文件中以"RedisConnectionString"命名Redis连接字符串，用户直接调用AddRedisClientSetup()方法即可完成Redis启动，但是可能因为各种原因用户没法遵守约定，因此我们也要提供一个用户可以指定Redis连接字符串方法的入口。


下面我们就用基于约定的方式，在配置文件中加入以下配置：



```
{
  "RedisConnectionString": "127.0.0.1:6379"
}

```

然后使用Client.Set方法设置key1，再用自定义方法Get方法读取，代码如下：



```
public static void Run()
{
    var configuration = new ConfigurationBuilder()
        .SetBasePath(AppContext.BaseDirectory)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .Build();
    var services = new ServiceCollection();
    services.AddSingleton(configuration);
    services.AddRedisClientSetup();
    var redisService = services.BuildServiceProvider().GetService();
    var setResult = redisService.Client.Set("key1", "value1");
    Console.WriteLine($"redisService.Client.Set(\"key1\",\"value1\")执行结果：{setResult}");
    var value = redisService.Get<string>("key1");
    Console.WriteLine($"redisService.Get(\"key1\")执行结果：{value}");
    redisService.Client.Del("key1");
}

```

执行结果如下：


![](https://img2024.cnblogs.com/blog/386841/202409/386841-20240913002709005-1933560411.jpg)


是不是很简单，然后我们只需要把上面三个文件IRedisService、RedisService、RedisSetupExtensions放到单独的类库中，然后打包发布成Nuget包，就可以给大家一起用啦，今天我这边就不发布Nuget包了，后面会有相关的计划，到时候再细聊。


***注***：测试方法代码以及示例源码都已经上传至代码库，有兴趣的可以看看。[https://gitee.com/hugogoos/Planner](https://github.com)


