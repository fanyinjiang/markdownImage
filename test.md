# PHP 服务治理

   为什么要说`服务治理`, 随着互联网浏览越来越大. 传统的 MVC 单一架构随着应用规模的不断扩大，应用模块不断增加，整个应用也显得越来越臃肿，维护起来也更加困难.
   
   我们必须采取措施,按应用拆分，就是把原来的应用按照业务特点拆分成多个应用。比如一个大型电商系统可能包含用户系统、商品系统、订单系统、评价系统等等，我们可以把他们独立出来形成一个个单独的应用。多应用架构的特点是应用之间各自独立 ，不相互调用。
  
  多应用虽然解决了应用臃肿问题，但应用之间相互独立，有些共同的业务或代码无法复用。
  
   对于一个大型的互联网系统，一般会包含多个应用，而且应用之间往往还存在共同的业务，并且应用之间还存在调用关系。除此之外 ，对于大型的互联网系统还有一些其它的挑战，比如如何应对急剧增长的用户，如何管理好研发团队快速迭代产品研发，如何保持产品升级更加稳定等等 。
 
   因此，为了使业务得到很好的复用，模块更加容易拓展和维护，我们希望业务与应用分离，某个业务不再属于一个应用，而是作为一个独立的服务单独进行维护。应用本身不再是一个臃肿的模块堆积，而是由一个个模块化的服务组件组合而成。


## 服务化

### 特点

那么采用`服务化`给有那些亮点的特色呢 ?
- 应用按业务拆分成服务
- 各个服务均可独立部署
- 服务可被多个应用共享
- 服务之间可以通信
- 架构上系统更加清晰
- 核心模块稳定，以服务组件为单位进行升级，避免了频繁发布带来的风险
- 开发管理方便
- 单独团队维护、工作分明，职责清晰
- 业务复用、代码复用
- 非常容易拓展

### 服务化面临的挑战

系统服务化之后, 增加了依赖关系复杂, 也会增加服务与服务之间交互的次数. 在 `fpm` 的开发模式下.  因为无法常驻内存给我们带来了, 每一次请求都要从零开始加载到退出进程, 增加了很多无用的开销, 数据库连接无法复用也得不到保护, 由于`fpm`是以进程为单位的`fpm`的进程数也决定了并发数, 这也是是`fpm`开发简单给我们带来的问题. 除此之外，还有很多其他问题需要解决。
- 服务越来越多，配置管理复杂
- 服务间依赖关系复杂
- 服务之间的负载均衡
- 服务的拓展
- 服务监控
- 服务降级
- 服务鉴权
- 服务上线与下线
- 服务文档
......

那么有没有好的方案呢？答案是有的，而且很多人都在用这个框架，它就是-`Swoft`。`Swoft`就是一个带有`服务治理`功能的[RPC](https://en.swoft.org/docs/2.x/zh-CN/rpc-server/index.html)框架。`Swoft`是首个 PHP常驻内存协程全栈框架, 基于 `Spring Boot`提出的约定大于配置的核心理念


`Swoft` 提供了类似 `Dubbo` 更为优雅的方式使用 `RPC` 服务, `Swoft` 性能是非常棒的有着类似`Golang`性能, 下面是我的`PC`对`Swoft` 性能的压测情况. 
![Swoft](https://raw.githubusercontent.com/sakuraovq/markdownImage/master/test.png)
`ab`压力测试处理速度十分惊人, 在 `i78代``16GB 内存`下 `100000` 万个请求只用了 `5s` 时间在`fpm`开发模式下基本不可能达到. 这也足以证明`Swoft` 的高性能和稳定性,

## 优雅的服务治理

### 服务注册与发现

微服务治理过程中，经常会涉及注册启动的服务到第三方集群，比如 consul / etcd 等等，本章以 Swoft 框架中使用 swoft-consul 组件，实现服务注册与发现为例。
![Swoft](https://raw.githubusercontent.com/sakuraovq/markdownImage/master/20170103144218850.png)
实现逻辑
```php
<?php declare(strict_types=1);


namespace App\Common;


use ReflectionException;
use Swoft\Bean\Annotation\Mapping\Bean;
use Swoft\Bean\Annotation\Mapping\Inject;
use Swoft\Bean\Exception\ContainerException;
use Swoft\Consul\Agent;
use Swoft\Consul\Exception\ClientException;
use Swoft\Consul\Exception\ServerException;
use Swoft\Rpc\Client\Client;
use Swoft\Rpc\Client\Contract\ProviderInterface;

/**
 * Class RpcProvider
 *
 * @since 2.0
 *        
 * @Bean()
 */
class RpcProvider implements ProviderInterface
{
    /**
     * @Inject()
     *
     * @var Agent
     */
    private $agent;

    /**
     * @param Client $client
     *
     * @return array
     * @throws ReflectionException
     * @throws ContainerException
     * @throws ClientException
     * @throws ServerException
     * @example
     * [
     *     'host:port',
     *     'host:port',
     *     'host:port',
     * ]
     */
    public function getList(Client $client): array
    {
        // Get health service from consul
        $services = $this->agent->services();

        $services = [
        
        ];

        return $services;
    }
}
```
### 服务熔断

在分布式环境下，特别是微服务结构的分布式系统中， 一个软件系统调用另外一个远程系统是非常普遍的。这种远程调用的被调用方可能是另外一个进程，或者是跨网路的另外一台主机, 这种远程的调用和进程的内部调用最大的区别是，远程调用可能会失败，或者挂起而没有任何回应，直到超时。更坏的情况是， 如果有多个调用者对同一个挂起的服务进行调用，那么就很有可能的是一个服务的超时等待迅速蔓延到整个分布式系统，引起连锁反应， 从而消耗掉整个分布式系统大量资源。最终可能导致系统瘫痪。

断路器（Circuit Breaker）模式就是为了防止在分布式系统中出现这种瀑布似的连锁反应导致的灾难。

![](https://raw.githubusercontent.com/swoft-cloud/swoft-doc/2.x/zh-CN/ms/govern/../../image/ms/breaker_ext.png)

基本的断路器模式下，保证了断路器在open状态时，保护supplier不会被调用， 但我们还需要额外的措施可以在supplier恢复服务后，可以重置断路器。一种可行的办法是断路器定期探测supplier的服务是否恢复， 一但恢复， 就将状态设置成close。断路器进行重试时的状态为半开（half-open）状态。

熔断器的使用想到简单且功能强大，使用一个 `@Breaker` 注解即可，`Swoft` 的熔断器可以用于任何场景, 例如 服务调用的时候使用, 请求第三方的时候都可以对它进行熔断降级

```php
<?php declare(strict_types=1);


namespace App\Model\Logic;

use Exception;
use Swoft\Bean\Annotation\Mapping\Bean;
use Swoft\Breaker\Annotation\Mapping\Breaker;

/**
 * Class BreakerLogic
 *
 * @since 2.0
 *
 * @Bean()
 */
class BreakerLogic
{
    /**
     * @Breaker(fallback="loopFallback")
     *
     * @return string
     * @throws Exception
     */
    public function loop(): string
    {
        // Do something
        throw new Exception('Breaker exception');
    }

    /**
     * @return string
     * @throws Exception
     */
    public function loopFallback(): string
    {
        // Do something
        throw new Exception('Breaker exception');
    }
}
```
### 服务限流

### 配置中心





## 参考资料

- [服务注册与发现](https://en.swoft.org/docs/2.x/en/ms/govern/register-discovery.html)
- [服务熔断](https://en.swoft.org/docs/2.x/en/ms/govern/breaker.html)
- [服务限流](https://en.swoft.org/docs/2.x/en/ms/govern/limiter.html)
- [配置中心](https://en.swoft.org/docs/2.x/en/ms/govern/config.html)

如果你喜欢 `Swoft` 的话, 欢迎给我们 [Start](https://github.com/swoft-cloud/swoft)
