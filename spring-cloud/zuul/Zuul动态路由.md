下面的内容来自于《重新定义Spring Cloud一书》

# 解决动态路由的两种方式

```text
    1.结合Spring Cloud Config + Bus,动态刷新配置文件。这种方式的好处是不用Zuul维护映射规则,可以随时修改,
    随时生效;唯一不好的地方是需要单独集成一些使用并不频繁的组件, Config没有可视化界面,维护起规则来也相对麻烦。
    2.重写Zul的配置读取方式,采用事件刷新机制,从数据库读取路由映射规则。此种方式因为基于数据库,可轻松实现管理界面
    ,灵活度较高。
```

## 动态原理实现解析

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/31.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;locateRoutes()方法继承自SimpleRouteLocator类并重写了规则,该方法主要的功能就是将配置文件中的映射规则信息包装成LinkedHashMap<String, ZulRoute>,键String是路径path,值ZuulRoute是配置文件的封装类,以往所见的配置映射读取进来就是使用ZuulRoute来封装。refresh()实现自RefreshableRouteLocator接口,添加刷新功能必须要实现此方法,doRefresh)方法来自SimpleRouteLocator类。我们需要自定义路由配置加载器,以仿照它的实现。其中locateRoutes的结果如下所示

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/32.jpg?raw=true)



>&nbsp;&nbsp;&nbsp;&nbsp;SimpleRouteLocator是DiscoveryClientRouteLocator的父类,此类基本实现了RouteLocator接口,对读取的配置文件信息做一些基本处理,提供方法doRefresh()与locateRoutes()供子类实现刷新策略与映射规则加载策略,两个方法源码及签名如下:

```java

	/**
	 * Calculate all the routes and set up a cache for the values. Subclasses can call
	 * this method if they need to implement {@link RefreshableRouteLocator}.
	 */
	protected void doRefresh() {
		this.routes.set(locateRoutes());
	}

	/**
	 * Compute a map of path pattern to route. The default is just a static map from the
	 * {@link ZuulProperties}, but subclasses can add dynamic calculations.
	 * @return map of Zuul routes
	 */
	protected Map<String, ZuulRoute> locateRoutes() {
		LinkedHashMap<String, ZuulRoute> routesMap = new LinkedHashMap<>();
		for (ZuulRoute route : this.properties.getRoutes().values()) {
			routesMap.put(route.getPath(), route);
		}
		return routesMap;
	}
```

>&nbsp;&nbsp;&nbsp;&nbsp;如果要调用doRefresh就需要自己去实现RefreshableRouteLocator接口。

>&nbsp;&nbsp;&nbsp;&nbsp;ZuulServerAutoConfiguration注册过滤器和监听器等其他功能。Zuul在注册中心新增服务后刷新监听器也是在此注册的。

```java
private static class ZuulRefreshListener
			implements ApplicationListener<ApplicationEvent> {

		@Autowired
		private ZuulHandlerMapping zuulHandlerMapping;

		private HeartbeatMonitor heartbeatMonitor = new HeartbeatMonitor();

		@Override
		public void onApplicationEvent(ApplicationEvent event) {
			if (event instanceof ContextRefreshedEvent
					|| event instanceof RefreshScopeRefreshedEvent
					|| event instanceof RoutesRefreshedEvent
					|| event instanceof InstanceRegisteredEvent) {
				reset();
			}
			else if (event instanceof ParentHeartbeatEvent) {
				ParentHeartbeatEvent e = (ParentHeartbeatEvent) event;
				resetIfNeeded(e.getValue());
			}
			else if (event instanceof HeartbeatEvent) {
				HeartbeatEvent e = (HeartbeatEvent) event;
				resetIfNeeded(e.getValue());
			}
		}

		private void resetIfNeeded(Object value) {
			if (this.heartbeatMonitor.update(value)) {
				reset();
			}
		}

		private void reset() {
			this.zuulHandlerMapping.setDirty(true);
		}

	}
```

>&nbsp;&nbsp;&nbsp;&nbsp;由方法onApplicationEvent(ApplicationEvent event)可知, Zul会接收3种事件ContextRefreshedEvent,RefreshScopeRefreshedEvent, RoutesRefreshedEvent通知去刷新路由映射配置信息,此外心跳续约监视器HeartbeatMonitor也会触发这个动作。


>&nbsp;&nbsp;&nbsp;&nbsp;在ZuulHandlerMapping中有一个很重的dirty属性。

```java
    public void setDirty(boolean dirty) {
		this.dirty = dirty;
		if (this.routeLocator instanceof RefreshableRouteLocator) {
			((RefreshableRouteLocator) this.routeLocator).refresh();
		}
	}

```

>&nbsp;&nbsp;&nbsp;&nbsp;	这个dirty属性很重要,它是用来控制当前是否需要重新加载映射配置信息的标记,在Zuul每次进行路由操作的时候都会检查这个值,如果为true,就会触发配置信息的重新加载,同时再将其回设为false。也由setDirty(boolean dirty)可知,启动刷新动作必须要实现RefreshableRouteLocator接口。
&nbsp;&nbsp;&nbsp;&nbsp;本节讲解了路由映射规则的加载原理以及Zuul的事件刷新方式。我们在构建动态路由的时候,只需要重写SimpleRouteLocator类的locateRoutes()方法,并且实现RefreshableRouteLocator接口的refresh)方法,再在内部调用SimpleRouteLocator类的doRefresh)方法,就可以构建起一个由Zul内部事件触发的自定义动态路由加载器。如果不想使用内部事件触发配置更新操作,改为手动触发,可以重写onApplicationEvent(ApplicationEvent event)方法内部实现方式,事实上手动触发的控制性更好。
