# 概述

> &nbsp;&nbsp;&nbsp;&nbsp;restTemplate对象会使用ribbon的自动化配置，同时通过配置@LoadBalance还能后开启客户端的负载均衡。该对象有常用的以下几种请求方式(GET,POST,PUT,DELETE)，其实可以用来构建restful服务。

# GET请求

> &nbsp;&nbsp;&nbsp;&nbsp;GET请求有两种请求方式，一个是getForEntity函数，一个是getForObject函数。


## getForEntity函数

> &nbsp;&nbsp;&nbsp;&nbsp;这个方式中有三种重载方式： 

```java
 1.getForEntity(String url, Class responseType, Object ... urlVariables）
 2.getForEntity(String url, Class responseType, Map  urlVariables）
 3.getForEntity(URI url, class responseType）
```

### getForEntity(String url, Class responseType, Object ... urlVariables）

> &nbsp;&nbsp;&nbsp;&nbsp;该方法提供了三个参数,其中url为请求的地址, responseType为请求响应体body的包装类型, urlVariables为url中的参数绑定。GET请求的参数绑定通常使用url中拼接的方式,比如http: //USER-SERVICE/user?name=didi,我们可以像这样自己将参数拼接到ur1中,但更好的方法是在url中使用占位符并配合urlVariables参数实现GET请求的参数绑定,比如url定义为http: //USER-SERVICE/user?name={1},然后可以这样来调用: getForEntity("http://USER-SERVICE/user?name={1)", String.class, "didi"),其中第三个参数didi会替换url中的(1}占位符。这里需要注意的是,由于urlVariables参数是一个数组,所以它的顺序会对应ur1中占位符定义的数字顺序。

### getForEntity(String url, Class responseType, Map  urlVariables）

> &nbsp;&nbsp;&nbsp;&nbsp;getForEntity(String url, Class responseType, Map urlVariables):该方法提供的参数中,只有urlVariables的参数类型与上面的方法不同。这里使用了Map类型,所以使用该方法进行参数绑定时需要在占位符中指定Map中参数的key值,比如url定义为http: //USER-SERVICE/user?name={name},在Map类型的urlVariables中,我们就需要put一个key为name的参数来绑定url中{name)占位符的值,比如

```java
	RestTemplate restTemplate = new RestTemplate ();
	Map<String, String> params = new HashMap<> ();params.put ("name", "dada");
	ResponseEntity<String> responseEntity = restTemplate.getForEntity ("http://USER-SERVICE/user?name= (name)", String.class, params);

```

### getForEntity(URI url, class responseType）

> &nbsp;&nbsp;&nbsp;&nbsp;getForEntity (URI url, Class responseType):该方法使用URI对象来替代之前的url和urlVariables参数来指定访问地址和参数绑定。URI是JDKjava.net包下的一个类,它表示一个统一资源标识符(Uniform Resource Identifier)引用。比如下面的例子:

```java
RestTemplate restTemplate = new RestTemplate ();UriComponents uriComponents = UriComponentsBuilder.fromUristring(
	"http://USER-SERVICE/user?name= (name)")
	.build ()
	.expand ("dodo")
	.encode ();
URI uri = uriComponents.toUri();
ResponseEntity<String> responseEntity = restTemplate.getForEntity (uri,String.class).getBody ();
```

## getForObject

> &nbsp;&nbsp;&nbsp;&nbsp;该方法可以理解为对getForEntity的进一步封装,它通过HttpMessageConverterExtractor对HTTP的请求响应体body内容进行对象转换,实现请求直接返回包装好的对象内容。比如:

```java
RestTemplate restTemplate = new RestTemplate ();
String result = restTemplate.getForObject (uri, String.class);
```

>&nbsp;&nbsp;&nbsp;&nbsp;通过该方法可以直接返回服务的返回值，而不用关心http协议中其他的部分，比如说请求头和请求体等等。  

```text
重载方法：
getForObject (String url, Class responseType, Object... urlVariables):与getForEntity的方法类似, url参数指定访问的地址, responseType参数定义该方法的返回类型, urlVariables参数为url中占位符对应的参数。
getForObject (String url, Class responseType, Map urlVariables):在该函数中,使用Map类型的urlVariables替代上面数组形式的urlVariables,因此使用时在ur1中需要将占位符的名称与Map类型中的key一一对应设置。
getForObject (URI url, Class responseType):该方法使用URI对象来,替代之前的url和urlVariables参数使用。
```

# POST请求

>  &nbsp;&nbsp;&nbsp;&nbsp;restTemplate对象中对于POST请求有三种实现方式：

```java
1. postForEntity 函数
2. postForObject 函数
3. postForLocation函数
```


## postForEntity 

>&nbsp;&nbsp;&nbsp;&nbsp;其实请求方式和getForEntity方式差不多。源代码如下所示:

```java
	@Override
	public <T> ResponseEntity<T> postForEntity(String url, Object request, Class<T> responseType, Object... uriVariables)
			throws RestClientException {

		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
	}

	@Override
	public <T> ResponseEntity<T> postForEntity(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)
			throws RestClientException {

		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
	}

	@Override
	public <T> ResponseEntity<T> postForEntity(URI url, Object request, Class<T> responseType) throws RestClientException {
		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor);
	}
```

>&nbsp;&nbsp;&nbsp;&nbsp;其中重点提一下request参数，request参数,该参数可以是一个普通对象,也可以是一个HttpEntity对象。如果是一个普通
对象,而非HttpEntity对象的时候, RestTemplate会将请求对象转换为一个HttpEntity对象来处理,其中Object就是request的类型, request内容会被视作完整的body来处理;而如果request是一个HttpEntity对象,那么就会被当作一个完成的HTTP请求对象来处理,这个request中不仅包含了body的内容,也包含了header的内容。

## postForOject

>&nbsp;&nbsp;&nbsp;&nbsp;用法和getForObject方式差不多，源代码如下所示:

 ```java
 	@Override
	public <T> T postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)
			throws RestClientException {

		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		HttpMessageConverterExtractor<T> responseExtractor =
				new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
	}

	@Override
	public <T> T postForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)
			throws RestClientException {

		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		HttpMessageConverterExtractor<T> responseExtractor =
				new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
	}

	@Override
	public <T> T postForObject(URI url, Object request, Class<T> responseType) throws RestClientException {
		RequestCallback requestCallback = httpEntityCallback(request, responseType);
		HttpMessageConverterExtractor<T> responseExtractor =
				new HttpMessageConverterExtractor<T>(responseType, getMessageConverters());
		return execute(url, HttpMethod.POST, requestCallback, responseExtractor);
	}
 ```

## postForLocation

>&nbsp;&nbsp;&nbsp;&nbsp;源代码如下所示:

```java

	@Override
	public URI postForLocation(String url, Object request, Object... uriVariables) throws RestClientException {
		RequestCallback requestCallback = httpEntityCallback(request);
		HttpHeaders headers = execute(url, HttpMethod.POST, requestCallback, headersExtractor(), uriVariables);
		return headers.getLocation();
	}

	@Override
	public URI postForLocation(String url, Object request, Map<String, ?> uriVariables) throws RestClientException {
		RequestCallback requestCallback = httpEntityCallback(request);
		HttpHeaders headers = execute(url, HttpMethod.POST, requestCallback, headersExtractor(), uriVariables);
		return headers.getLocation();
	}

	@Override
	public URI postForLocation(URI url, Object request) throws RestClientException {
		RequestCallback requestCallback = httpEntityCallback(request);
		HttpHeaders headers = execute(url, HttpMethod.POST, requestCallback, headersExtractor());
		return headers.getLocation();
	}

```
	 
>&nbsp;&nbsp;&nbsp;&nbsp;该方法实现了使用Post方式提交资源，并且还返回了Uri地址。

#  PUT请求

>&nbsp;&nbsp;&nbsp;&nbsp;put函数也实现了三种不同的重载方法:

```text
put (String url, Object request, Object... urlVariables) 
put (String url, Object request, Map urlVariables)
put (URI url, Object request)
put函数为void类型,所以没有返回内容,也就没有其他函数定义的,  responseType参数,除此之外的其他传入参数定义
与用法与postForObject基本一致。
```

# DELETE请求

>&nbsp;&nbsp;&nbsp;&nbsp;delete函数也实现了三种不同的重载方法

```text

delete (String url, Object... urlVariables)
delete (String url, Map urlVariables)
delete (URI url)
由于我们在进行REST请求时,通常都将DELETE请求的唯一标识拼接在url中,所以DELETE请求也不需要request的
body信息,就如上面的三个函数实现一样,非常简单。url指定DELETE请求的位置, urlVariables绑定url中的参数
即可。
```