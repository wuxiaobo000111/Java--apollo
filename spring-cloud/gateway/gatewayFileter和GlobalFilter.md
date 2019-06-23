# Gateway Filter和Global Filter 概述

&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud Gateway中的Filter从接口实现上分为两种:一种是Gateway Filter,另外一种是Global Filter。那要怎么理解这两种Filter的设计呢?通过和Gateway的核心设计者进行设计交流,我对这两者有了更深刻的了解。下面我们介绍一下Gateway Filter和Global Filter之间的区别和联系。
## Gateway Filter概述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Gateway Filter是从Web Filter中复制过来的,相当于一个Filter过滤器,可以对访问的URL过滤，进行横切处理(切面处理),应用场景包括超时、安全等。
## Global Filter概述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud Gateway定义了Global Filter的接口,让我们可以自定义实现自己的Globa)Filtero Global Filter是一个全局的Filter,作用于所有路由。
## Gateway Filter和Global Filter的区别
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从路由的作用范围来看, Global filter会被应用到所有的路由上,而Gateway filter则应用到单个路由或者一个分组的路由上。从源码设计来看, Gateway Filter和Global Filter两个接口中定义的方法一样都是Mono filter),唯一的区别就是GatewayFilter继承了ShortcutConfigurable,而GlobalFilter没有任何继承。
