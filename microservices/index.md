# 微服务介绍

> 微服务架构（MicroServices Architecture，MSA）：微服务架构可以看做是**面向服务架构和分布式服务架构**的拓展，使用更细粒度的服务（所以叫微服务）和一组设计准则来考虑大规模的复杂系统架构设计。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

## 常见的微服务组件及概念
* 服务注册：服务提供方将自己调用地址注册到服务注册中心，让服务调用方能够方便地找到自己。
* 服务发现：服务调用方从服务注册中心找到自己需要调用的服务的地址。
* 负载均衡：服务提供方一般以多实例的形式提供服务，负载均衡功能能够让服务调用方连接到合适的服务节点。并且，节点选择的工作对服务调用方来说是透明的。
* 服务网关：服务网关是服务调用的唯一入口，可以在这个组件是实现用户鉴权、动态路由、灰度发布、A/B 测试、负载限流等功能。
* 配置中心：将本地化的配置信息（properties, xml, yaml 等）注册到配置中心，实现程序包在开发、测试、生产环境的无差别性，方便程序包的迁移。
* API 管理：以方便的形式编写及更新 API 文档，并以方便的形式供调用者查看和测试。
* 集成框架：微服务组件都以职责单一的程序包对外提供服务，集成框架以配置的形式将所有微服务组件（特别是管理端组件）集成到统一的界面框架下，让用户能够在统一的界面中使用系统。
* 分布式事务：对于重要的业务，需要通过分布式事务技术（TCC、高可用消息服务、最大努力通知）保证数据的一致性。
* 调用链：记录完成一个业务逻辑时调用到的微服务，并将这种串行或并行的调用关系展示出来。在系统出错时，可以方便地找到出错点。
* 支撑平台：系统微服务化后，系统变得更加碎片化，系统的部署、运维、监控等都比单体架构更加复杂，那么，就需要将大部分的工作自动化。现在，可以通过 Docker 等工具来中和这些微服务架构带来的弊端。 例如持续集成、蓝绿发布、健康检查、性能健康等等。严重点，以我们两年的实践经验，可以这么说，如果没有合适的支撑平台或工具，就不要使用微服务架构。

## 微服务架构的优点
* 降低系统复杂度：每个服务都比较简单，只关注于一个业务功能。
* 松耦合：微服务架构方式是松耦合的，每个微服务可由不同团队独立开发，互不影响。
* 跨语言：只要符合服务 API 契约，开发人员可以自由选择开发技术。这就意味着开发人员可以采用新技术编写或重构服务，由于服务相对较小，所以这并不会对整体应用造成太大影响。
* 独立部署：微服务架构可以使每个微服务独立部署。开发人员无需协调对服务升级或更改的部署。这些更改可以在测试通过后立即部署。所以微服务架构也使得 CI／CD 成为可能。
* Docker 容器：和 Docker 容器结合的更好。
* DDD 领域驱动设计：和 DDD 的概念契合，结合开发会更好。

## 微服务架构的缺点
* 微服务强调了服务大小，但实际上这并没有一个统一的标准：业务逻辑应该按照什么规则划分为微服务，这本身就是一个经验工程。有些开发者主张 10-100 行代码就应该建立一个微服务。微服务的目标是充分分解应用程序，以促进敏捷开发和持续集成部署。
* 微服务的分布式特点带来的复杂性：开发人员需要基于 RPC 或者消息实现微服务之间的调用和通信，而这就使得服务之间的发现、服务调用链的跟踪和质量问题变得的相当棘手。
* 分区的数据库体系和分布式事务：更新多个业务实体的业务交易相当普遍，不同服务可能拥有不同的数据库。CAP 原理的约束，使得我们不得不放弃传统的强一致性，而转而追求最终一致性，这个对开发人员来说是一个挑战。
* 测试挑战：传统的单体WEB应用只需测试单一的 REST API 即可，而对微服务进行测试，需要启动它依赖的所有其他服务。这种复杂性不可低估。
* 跨多个服务的更改：比如在传统单体应用中，若有 A、B、C 三个服务需要更改，A 依赖 B，B 依赖 C。我们只需更改相应的模块，然后一次性部署即可。但是在微服务架构中，我们需要仔细规划和* 调每个服务的变更部署。我们需要先更新 C，然后更新 B，最后更新 A。
* 部署复杂：微服务由不同的大量服务构成。每种服务可能拥有自己的配置、应用实例数量以及基础服务地址。这里就需要不同的配置、部署、扩展和监控组件。此外，我们还需要服务发现机制，以便服* 可以发现与其通信的其他服务的地址。因此，成功部署微服务应用需要开发人员有更好地部署策略和高度自动化的水平。
* 总结（问题和挑战）：API Gateway、服务间调用、服务发现、服务容错、服务部署、数据调用。

不过，现在很多微服务的框架（比如 Spring Cloud、Dubbo）已经很好的解决了上面的问题。

## 微服务常见问题
### 服务雪崩
服务雪崩效应产生的原因：因为Tomcat默认情况下只有一个线程池来维护客户端发送的所有的请求，这时候某一接口在某一时刻被大量访问就会占据tomcat线程池中的所有线程，其他请求处于等待状态，无法连接到服务接口。

#### 服务雪崩解决方案-1.服务降级
当客户端请求服务器端的时候，防止客户端一直等待，不会处理业务逻辑代码，直接返回一个友好的提示给客户端。

熔断和降级的区别，其实应该要这么理解:
* 服务降级有很多种降级方式！如开关降级、限流降级、熔断降级!
* 服务熔断属于降级方式的一种！

#### 服务雪崩解决方案-2.服务熔断 
> 服务熔断：当下游的服务因为某种原因突然变得不可用或响应过慢，上游服务为了保证自己整体服务的可用性，不再继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。

是在服务降级的基础上更直接的一种保护方式，需要说明的是熔断其实是一个框架级的处理，那么这套熔断机制的设计，基本上业内用的是断路器模式

#### 服务雪崩解决方案-3.服务隔离
是Hystrix为隔离的服务开启一个独立的线程池，这样在高并发的情况下不会影响其他服务。服务隔离有线程池和信号量两种实现方式，一般使用线程池方式。

## 微服务的三个阶段
1. 仅使用注册发现，基于SpringCloud或者Dubbo进行开发。
2. 使用了限流、降级、熔断等服务治理策略，并配备完整服务工具和平台。
3. Service Mesh将服务治理作为通用组件，下沉到平台层实现，应用层仅仅关注业务逻辑，平台层可以根据业务监控自动调度和参数调整，实现AIOps和智能调度。
   * 微服务更像是一个服务之间的生态，专注于服务治理等方面，而服务网格更专注于服务之间的通信，以及和 DevOps 更好的结合。

## 参考文章
* https://www.cnblogs.com/xishuai/p/microservices-and-service-mesh.html
* https://www.infoq.cn/article/xveohtcortxrspcf2ldd
* https://segmentfault.com/a/1190000039672263
* https://mp.weixin.qq.com/s/9w2W6mojhBWGwR9QmGUvvg
* [RPC基本原理](https://juejin.im/post/5c6d7640f265da2de80f5e9c)
