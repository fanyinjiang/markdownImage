![Swoft](https://raw.githubusercontent.com/sakuraovq/markdownImage/master/v2-695ab4abd8e3171d51f0331584a28559_1200x500.jpg)
# How to use PHP to implement microservices?

### Why do you say service governance?

   With the increasing popularity of Internet browsing, the traditional MVC single architecture has become more and more bloated and more difficult to maintain as the scale of applications continues to expand.
   
   We must take measures to split the application, which is to split the original application into multiple applications according to business characteristics. For example, a large e-commerce system may include user systems, commodity systems, order systems, evaluation systems, etc., and we can separate them into separate applications. The multi-application architecture is characterized by independent applications and not calling each other.
  
  Although multiple applications solve the bloated application problem, the applications are independent of each other, and some common services or codes cannot be reused.
  
### Single application solution

   For a large Internet system, there are usually multiple applications, and there are often common services between applications, and there are call relationships between applications. In addition, there are other challenges for large-scale Internet systems, such as how to deal with rapidly growing users, how to manage R&D teams to quickly iterate product development, how to keep product upgrades more stable, and so on.
 
   Therefore, in order to make the service be well reused, the module is easier to expand and maintain. We want the service to be separated from the application. A service is no longer an application, but is maintained as a separate service. The application itself is no longer a bloated module stack, but a modular service component.


## Service

### Features

So what is the feature of using "services"?
- Application split into services by business
- Individual services can be deployed independently
- Service can be shared by multiple apps
- Communication between services
- The system is more clear on the architecture
- Core modules are stable and upgraded in units of service components, avoiding the risk of frequent releases
- Easy development and management
- Individual team maintenance, clear work, clear responsibilities
- Service reuse, code reuse
- Very easy to expand

### The challenge of service

After the system is serviced, the dependency is complicated, and the number of interactions between the service and the service is increased. In the development mode of `fpm`, because the resident memory cannot be brought to us, every request must be from zero. Starting to load to the exit process, adding a lot of useless overhead, the database connection can not be reused and can not be protected, because `fpm` is the process-based `fpm` process number also determines the number of concurrent, this is also Fpm` development is simple to bring us problems. So why is the Internet platform `Java` more popular now, `.NET` and `PHP` will not work in this regard. Needless to say `PHP non-memory resident`. In addition, there are many other issues that need to be addressed.
- More and more services, complex configuration management
- Complex service dependencies
- Load balancing between services
- Service expansion
- Service monitoring
- Service downgrade
- Service authentication
- Service online and offline
- Service documentation
......

You can imagine the benefits that resident memory brings to us.

- **only start frame initialization** If resident memory we just initialize the memory in the memory at startup, concentrate on processing the request

- **Connection multiplexing**, some engineers can not specifically understand, if you do not need to connect to the pool, how about making a connection to a request? This will result in too many backend resource connections. For some basic services, such as Redis, databases, connections are an expensive drain.

So is there a good plan? The answer is yes, and many people are using this framework, it is - `Swoft`. `Swoft` is a [RPC](https://en.swoft.org/docs/2.x/zh-CN/rpc-server/index.html) framework with the `Service Governance` feature. `Swoft` is the first PHP resident memory coroutine full stack framework, based on the core concept of the `Spring Boot` convention is greater than the configuration

`Swoft` provides a more elegant way to use `RPB` services like `Dubbo`, `Swoft` performance is great with similar `Golang` performance, here is my 'PC` vs. `Swoft` performance. Happening.
 
![Swoft](https://raw.githubusercontent.com/sakuraovq/markdownImage/master/test.png)
`ab` stress test processing speed is very amazing, in the `i78 generation `CPU, `16GB` memory ``100000` million requests only use `5s` time in the `fpm` development mode is basically impossible to achieve. Sufficient to demonstrate the high performance and stability of `Swoft`,

## Elegant service governance

### [Service Registration and Discovery](https://en.swoft.org/docs/2.x/en/ms/govern/register-discovery.html)

In the microservices governance process, registration of services initiated to third-party clusters, such as consul / etcd, is often involved. This chapter uses the swoft-consul component in the Swoft framework to implement service registration and discovery.
![Swoft](https://raw.githubusercontent.com/sakuraovq/markdownImage/master/20170103144218850.png)
Implementation logic
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
### [Service Blow](https://en.swoft.org/docs/2.x/en/ms/govern/breaker.html)

In a distributed environment, especially a distributed system of microservice architectures, it is very common for one software system to call another remote system. The callee of such a remote call may be another process, or another host across the network. The biggest difference between this remote call and the internal call of the process is that the remote call may fail or hang. No response until timeout. Worse, if there are multiple callers calling the same suspended service, it is very likely that a service's timeout waits quickly spread to the entire distributed system, causing a chain reaction that consumes the entire A large amount of resources in distributed systems. Eventually it can lead to system paralysis.

The Circuit Breaker mode is designed to prevent disasters caused by such waterfall-like chain reactions in distributed systems.

![](https://raw.githubusercontent.com/swoft-cloud/swoft-doc/2.x/zh-CN/ms/govern/../../image/ms/breaker_ext.png)

In the basic circuit breaker mode, the protection supplier is not called when the circuit breaker is in the open state, but we also need additional measures to reset the circuit breaker after the supplier resumes service. One possible solution is that the circuit breaker periodically detects whether the service of the supplier is restored. Once restored, the status is set to close. The state when the circuit breaker is retried is a half-open state.

The use of the fuse is simple and powerful. It can be annotated with a `@Breaker`. The fuse of the `Swoft` can be used in any scenario, such as when the service is called. It can be blown when requesting a third party. Downgrade
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
    }
}
```
### [Service Restriction](https://en.swoft.org/docs/2.x/en/ms/govern/limiter.html)
**Restriction, Fuse, Degradation** This emphasis cannot be overstated because it is really important. When the service is not working, it must be blown. Current limiting is a tool to protect itself. If there is no self-protection mechanism, no matter how many connections are received, if the back-end processing is not enough, the front-end traffic will definitely hang when it is very large.

The current limit is to limit the number of concurrent and requested when accessing scarce resources, such as spikes and snapped goods, so as to effectively cut the peak and smooth the flow curve. The purpose of current limiting is to limit the rate of concurrent access and concurrent requests, or to request a speed limit within a time window to protect the system. Once the rate limit is reached or exceeded, the service can be denied, or queued.

The bottom layer of the `Swoft` current limiter uses the token bucket algorithm, and the underlying layer relies on `Redis` to implement distributed current limiting.

The Swoft speed limiter not only limits the flow controller, it also limits the methods in any bean and controls the access rate of the method. Here is a detailed explanation of the following examples.
```php
<?php declare(strict_types=1);

namespace App\Model\Logic;

use Swoft\Bean\Annotation\Mapping\Bean;
use Swoft\Limiter\Annotation\Mapping\RateLimiter;

/**
 * Class LimiterLogic
 *
 * @since 2.0
 *
 * @Bean()
 */
class LimiterLogic
{
    /**
     * @RequestMapping()
     * @RateLimiter(rate=20, fallback="limiterFallback")
     *
     * @param Request $request
     *
     * @return array
     */
    public function requestLimiter2(Request $request): array
    {
        $uri = $request->getUriPath();
        return ['requestLimiter2', $uri];
    }
    
    /**
     * @param Request $request
     *
     * @return array
     */
    public function limiterFallback(Request $request): array
    {
        $uri = $request->getUriPath();
        return ['limiterFallback', $uri];
    }
}
```
Key This supports the `symfony/expression-language` expression. If the speed limit is called, the `limiterFallback` method defined in `fallback` will be called.

### [Configuration Center](https://en.swoft.org/docs/2.x/en/ms/govern/config.html)
Before we talk about the configuration center, let's talk about the configuration file. We are no stranger to it. It provides us with the ability to dynamically modify the program. A quote from someone else is:

> Dynamic adjustment of the flight attitude of the system runtime!

I can call our work to repair parts on fast-flying airplanes. We humans are always unable to control and predict everything. For our system, we always need to reserve some control lines to make adjustments when we need them, to control the system direction (such as gray control, current limit adjustment), which is especially important for the Internet industry that embraces change.

For the stand-alone version, we call it the configuration (file); for the distributed cluster system, we call it the configuration center (system);

#### What exactly is a distributed configuration center?

With the development of the business and the upgrade of the micro-service architecture, the number of services and the configuration of programs are increasing (various micro-services, various server addresses, various parameters), and the traditional configuration file method and database method cannot meet the development. Personnel requirements for configuration management:

- Security: The configuration follows the source code stored in the code base, which is easy to cause configuration leaks;
- Timeliness: Modify the configuration and restart the service to take effect.
- Limitations: Dynamic adjustments cannot be supported: for example, log switches, function switches;

Therefore, we need to configure the center to manage the configuration! Freeing business developers from complex and cumbersome configurations, they only need to focus on the business code itself, which can significantly improve development and operational efficiency. At the same time, the configuration and release of the package will further enhance the success rate of the release, and provide strong support for the fine control and emergency handling of operation and maintenance.

 About distributed configuration centers, there are many open source solutions on the web, such as:

Apollo is a distributed configuration center developed by Ctrip's framework department. It can centrally manage the configuration of different environments and different clusters of applications. It can be pushed to the application end in real time after configuration modification. It has the features of standardized authority and process management, and is suitable for microservices. Configure management scenarios.

This chapter uses `Apollo` as an example to pull configuration and secure restart services from the remote configuration center. If you are unfamiliar with `Apollo`, you can first look at the `Swoft` extension [`Apollo`](https://en.swoft.org/docs/2.x/en/extra/apollo.html) component and read `Apollo ` Official documentation.

This chapter uses `Apollo` in `Swoft` as an example. When the `Apollo` configuration changes, restart the service (http-server / rpc-server/ ws-server). The following is an example of an agent:
```php
<?php declare(strict_types=1);


namespace App\Model\Logic;

use Swoft\Apollo\Config;
use Swoft\Apollo\Exception\ApolloException;
use Swoft\Bean\Annotation\Mapping\Bean;
use Swoft\Bean\Annotation\Mapping\Inject;

/**
 * Class ApolloLogic
 *
 * @since 2.0
 *
 * @Bean()
 */
class ApolloLogic
{
    /**
     * @Inject()
     *
     * @var Config
     */
    private $config;

    /**
     * @throws ApolloException
     */
    public function pull(): void
    {
        $data = $this->config->pull('application');
        
        // Print data
        var_dump($data);
    }
}
```
The above is a simple Apollo configuration pull, in addition to this method, swowt-apollo provides more ways to use.

## Official link

- [Github](https://github.com/swoft-cloud/swoft)
- [Doc](https://en.swoft.org/docs)
- [Swoft-cloud/community](https://gitter.im/swoft-cloud/community)
