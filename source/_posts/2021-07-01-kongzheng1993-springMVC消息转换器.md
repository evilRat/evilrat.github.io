---
title: SpringMVC消息转换器
excerpt: 'spring'
tags: [spring]
categories: [spring]
comments: true
date: 2021-07-01 13:30:10
---

今天是建党百年的日子，也是公司六周年。我们发了理想one的车模还有一件polo衫。车模我选的baby blue。

<img src="1.jpg"/>

<img src="2.jpg"/>

<img src="3.jpg"/>

圆规正转！

前几天我们上线了一个给公司app提供接口的api服务，当时ios的开发希望我们能统一处理一下返回报文json中的null，比如字符串为空修改为""，对象为空修改为{}，List为空修改为[]，原因是他不好判空。。然后我们直接实现WebMvcConfigurer，重写了configureMessageConverters，增加了一个处理json的FastJsonHttpMessageConverter，实现了ios开发要求的效果。

```java
@Configuration
public class ApiMvcConfig implements WebMvcConfigurer {
    /**
     * 增加对Json的处理
     * @param converters
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        //针对字段的处理
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteNullListAsEmpty,// List字段如果为null,输出为[],而非null
                SerializerFeature.WriteMapNullValue,//加上后，字段为null的也会输出
                SerializerFeature.WriteNullStringAsEmpty,//字符类型字段如果为null,输出为”“,而非null
                SerializerFeature.WriteNullBooleanAsFalse,//Boolean字段如果为null,输出为false,而非null
                //SerializerFeature.WriteNullNumberAsZero, // Number 包装类如果为null，输出为0
                SerializerFeature.DisableCircularReferenceDetect,
                SerializerFeature.PrettyFormat  //结果是否格式化,默认为false
        );
        //日期格式化
        fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
        converter.setFastJsonConfig(fastJsonConfig);
        converters.add(0, converter);//返回是string的话，默认把这个放在最前，否则ResponseAdvisor 处理字符串返回时会报类型不一致的问题
    }
}
```



上线后却发现系统没能接入prometheus。在和公司平台云服务部的同学沟通之后，发现我们给prometheus返回的报文全都加上了转译字符，导致prometheus解析不了。。。

下面是有问题的报文：

```shell
➜  ~ curl -i http://127.0.0.1:11187/dayu/prometheus
HTTP/1.1 200
Content-Type: text/plain; version=0.0.4;charset=utf-8
Content-Length: 19296
Date: Fri, 02 Jul 2021 08:35:22 GMT

"# HELP jvm_threads_daemon The current number of live daemon threads\n# TYPE jvm_threads_daemon gauge\njvm_threads_daemon{application=\"saos-pdi-api\",env=\"test\",namespace=\"\",pod=\"\",} 69.0\n# HELP tomcat_sessions_active_max  \n# TYPE tomcat_sessions_active_max gauge\ntomcat_sessions_active_max{application=\"saos-pdi-api\",env=\"test\",namespace=\"\",pod=\"\",} 5.0\n# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for the Java virtual machine to use\n# TYPE jvm_memory_committed_bytes gauge\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Code Cache\",namespace=\"\",pod=\"\",} 2.2740992E7\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Metaspace\",namespace=\"\",pod=\"\",} 9.728E7\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Compressed Class Space\",namespace=\"\",pod=\"\",} 1.2894208E7\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"heap\",env=\"test\",id=\"PS Eden Space\",namespace=\"\",pod=\"\",} 7.99014912E8\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"heap\",env=\"test\",id=\"PS Survivor Space\",namespace=\"\",pod=\"\",} 2.2020096E7\njvm_memory_committed_bytes{application=\"saos-pdi-api\",area=\"heap\",env=\"test\",id=\"PS Old Gen\",namespace=\"\",pod=\"\",} 1.9136512E8\n# HELP tomcat_sessions_expired_total  \n# TYPE tomcat_sessions_expired_total counter\ntomcat_sessions_expired_total{application=\"saos-pdi-api\",env=\"test\",namespace=\"\",pod=\"\",} 0.0\n# HELP logback_events_total Number of error level events that made it to the logs\n# TYPE logback_events_total counter\nlogback_events_total{application=\"saos-pdi-api\",env=\"test\",level=\"error\",namespace=\"\",pod=\"\",} 0.0\nlogback_events_total{application=\"saos-pdi-api\",env=\"test\",level=\"warn\",namespace=\"\",pod=\"\",} 5.0\nlogback_events_total{application=\"saos-pdi-api\",env=\"test\",level=\"info\",namespace=\"\",pod=\"\",} 401.0\nlogback_events_total{application=\"saos-pdi-api\",env=\"test\",level=\"debug\",namespace=\"\",pod=\"\",} 60.0\nlogback_events_total{application=\"saos-pdi-api\",env=\"test\",level=\"trace\",namespace=\"\",pod=\"\",} 0.0\n# HELP jvm_buffer_total_capacity_bytes An estimate of the total capacity of the buffers in this pool\n# TYPE jvm_buffer_total_capacity_bytes gauge\njvm_buffer_total_capacity_bytes{application=\"saos-pdi-api\",env=\"test\",id=\"direct\",namespace=\"\",pod=\"\",} 170147.0\njvm_buffer_total_capacity_bytes{application=\"saos-pdi-api\",env=\"test\",id=\"mapped\",namespace=\"\",pod=\"\",} 0.0\n# HELP system_cpu_usage The \"recent cpu usage\" for the whole system\n# TYPE system_cpu_usage gauge\nsystem_cpu_usage{application=\"saos-pdi-api\",env=\"test\",namespace=\"\",pod=\"\",} 0.03545880166806259\n# HELP tomcat_threads_config_max  \n# TYPE tomcat_threads_config_max gauge\ntomcat_threads_config_max{application=\"saos-pdi-api\",env=\"test\",name=\"http-nio-11187\",namespace=\"\",pod=\"\",} 200.0\n# HELP tomcat_threads_current  \n# TYPE tomcat_threads_current gauge\ntomcat_threads_current{application=\"saos-pdi-api\",env=\"test\",name=\"http-nio-11187\",namespace=\"\",pod=\"\",} 10.0\n# HELP tomcat_sessions_rejected_total  \n# TYPE tomcat_sessions_rejected_total counter\ntomcat_sessions_rejected_total{application=\"saos-pdi-api\",env=\"test\",namespace=\"\",pod=\"\",} 0.0\n# HELP tomcat_global_request_seconds  \n# TYPE tomcat_global_request_seconds summary\ntomcat_global_request_seconds_count{application=\"saos-pdi-api\",env=\"test\",name=\"http-nio-11187\",namespace=\"\",pod=\"\",} 21.0\ntomcat_global_request_seconds_sum{application=\"saos-pdi-api\",env=\"test\",name=\"http-nio-11187\",namespace=\"\",pod=\"\",} 14.12\n# HELP jvm_memory_max_bytes The maximum amount of memory in bytes that can be used for memory management\n# TYPE jvm_memory_max_bytes gauge\njvm_memory_max_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Code Cache\",namespace=\"\",pod=\"\",} 2.5165824E8\njvm_memory_max_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Metaspace\",namespace=\"\",pod=\"\",} -1.0\njvm_memory_max_bytes{application=\"saos-pdi-api\",area=\"nonheap\",env=\"test\",id=\"Compressed Class Space\",namespace=\"\",pod=\"\",} 1.073741824E9\njvm_memory_max_bytes{application=\"saos-pdi-api\",area=\"heap\",env=\"test\",id=\"PS Eden Space\",namespace=\"\",pod=\"\",} 1.311244288E9\njvm_memory_max_bytes{application=\"saos-pdi-api\",area=\"heap\",env=\"test\",id=\"PS Survivor Space\
...
```

下面是正常的报文：

```shell
➜  ~ curl -i http://127.0.0.1:11187/dayu/prometheus
HTTP/1.1 200
Content-Type: text/plain; version=0.0.4;charset=utf-8
Content-Length: 12594
Date: Fri, 02 Jul 2021 08:39:14 GMT

# HELP tomcat_global_request_max_seconds
# TYPE tomcat_global_request_max_seconds gauge
tomcat_global_request_max_seconds{application="saos-pdi-api",env="test",name="http-nio-11187",namespace="",pod="",} 0.0
# HELP tomcat_global_received_bytes_total
# TYPE tomcat_global_received_bytes_total counter
tomcat_global_received_bytes_total{application="saos-pdi-api",env="test",name="http-nio-11187",namespace="",pod="",} 0.0
# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{application="saos-pdi-api",area="nonheap",env="test",id="Code Cache",namespace="",pod="",} 1.9857408E7
jvm_memory_committed_bytes{application="saos-pdi-api",area="nonheap",env="test",id="Metaspace",namespace="",pod="",} 8.7318528E7
jvm_memory_committed_bytes{application="saos-pdi-api",area="nonheap",env="test",id="Compressed Class Space",namespace="",pod="",} 1.1583488E7
jvm_memory_committed_bytes{application="saos-pdi-api",area="heap",env="test",id="PS Eden Space",namespace="",pod="",} 1.120927744E9
jvm_memory_committed_bytes{application="saos-pdi-api",area="heap",env="test",id="PS Survivor Space",namespace="",pod="",} 2.359296E7
jvm_memory_committed_bytes{application="saos-pdi-api",area="heap",env="test",id="PS Old Gen",namespace="",pod="",} 1.86646528E8
# HELP tomcat_servlet_request_seconds
# TYPE tomcat_servlet_request_seconds summary
tomcat_servlet_request_seconds_count{application="saos-pdi-api",env="test",name="dispatcherServlet",namespace="",pod="",} 1.0
tomcat_servlet_request_seconds_sum{application="saos-pdi-api",env="test",name="dispatcherServlet",namespace="",pod="",} 0.0
# HELP health
# TYPE health gauge
health{application="saos-pdi-api",env="test",namespace="",pod="",} 3.0
# HELP jvm_buffer_memory_used_bytes An estimate of the memory that the Java virtual machine is using for this buffer pool
# TYPE jvm_buffer_memory_used_bytes gauge
jvm_buffer_memory_used_bytes{application="saos-pdi-api",env="test",id="direct",namespace="",pod="",} 91015.0
jvm_buffer_memory_used_bytes{application="saos-pdi-api",env="test",id="mapped",namespace="",pod="",} 0.0
# HELP tomcat_threads_current
# TYPE tomcat_threads_current gauge
tomcat_threads_current{application="saos-pdi-api",env="test",name="http-nio-11187",namespace="",pod="",} 10.0
# HELP process_start_time_seconds Start time of the process since unix epoch.
# TYPE process_start_time_seconds gauge
process_start_time_seconds{application="saos-pdi-api",env="test",namespace="",pod="",} 1.625215112786E9
# HELP tomcat_threads_config_max
# TYPE tomcat_threads_config_max gauge
tomcat_threads_config_max{application="saos-pdi-api",env="test",name="http-nio-11187",namespace="",pod="",} 200.0
# HELP jvm_gc_memory_promoted_bytes_total Count of positive increases in the size of the old generation memory pool before GC to after GC
# TYPE jvm_gc_memory_promoted_bytes_total counter
jvm_gc_memory_promoted_bytes_total{application="saos-pdi-api",env="test",namespace="",pod="",} 4.5149664E7
# HELP jvm_gc_memory_allocated_bytes_total Incremented for an increase in the size of the young generation memory pool after one GC to before the next
# TYPE jvm_gc_memory_allocated_bytes_total counter
jvm_gc_memory_allocated_bytes_total{application="saos-pdi-api",env="test",namespace="",pod="",} 3.43775952E9
# HELP tomcat_global_sent_bytes_total
# TYPE tomcat_global_sent_bytes_total counter
tomcat_global_sent_bytes_total{application="saos-pdi-api",env="test",name="http-nio-11187",namespace="",pod="",} 0.0
# HELP jvm_classes_unloaded_total The total number of classes unloaded since the Java virtual machine has started execution
```

可以看到，Prometheus这个接口的响应报文，并不是一个单纯的json格式，而是text里有json格式的东西。

当时就怀疑是Fastjson是不是有点啥问题[doge]……

果不其然，让我在github上找到一个[issues](https://github.com/alibaba/fastjson/issues/1373)

之前老版本的fastjson是有一个DisableCheckSpecialChar的SerializerFeature的，可以禁止特殊字符的转译，现在标了过期，直接不能用了，默认就给转译。。

使用老版本的fastjson，并加上SerializerFeature.DisableCheckSpecialChar后，是可以的。

但是我们都知道fastjson动不动就爆出漏洞，最新的版本都有点怕，更别说那么老的版本了。

只能再想办法。

我们从头开始想：

http响应的报文，是怎么转换的呢？--> HttpMessageConverter

所以这块还得研究透spring的HttpMessageConverter是怎么工作的！

我们都知道请求和响应都是要经过DispatcherServlet类的doDispatch方法。然后获取HandlerAdapter，然后获取对应的Handler，并调用handle方法，中间有对应的请求内容的消息转换器进行请求报文的转换，返回的时候，也会用消息解析器进行报文的转换，最后写入到http的outputStream，完成一次请求的响应。

看我上面增加的自定义的FastJsonHttpMessageConverter，可以看到，我将自定义的HttpMessageConverter放到了第一位，当时考虑是spring有一些默认的消息转换器，对应的json的转换器使用jackson实现的。spring在选择消息转换器的时候，是遍历所有的转换器，找到第一个合适的就使用。所以我必须放到jackson前面，但是放到第一位，也就在string的转换器之前了，暴露给Prometheus的接口是text/plain的，讲道理fastjson不应该处理它，应该留给StringHttpMessageConterver来转换才对啊。

下面的代码是spring默认的消息处理器：

```java
protected final void addDefaultHttpMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
		StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter();
		stringHttpMessageConverter.setWriteAcceptCharset(false);  // see SPR-7316

		messageConverters.add(new ByteArrayHttpMessageConverter());
		messageConverters.add(stringHttpMessageConverter);
		messageConverters.add(new ResourceHttpMessageConverter());
		messageConverters.add(new ResourceRegionHttpMessageConverter());
		messageConverters.add(new SourceHttpMessageConverter<>());
		messageConverters.add(new AllEncompassingFormHttpMessageConverter());

		if (romePresent) {
			messageConverters.add(new AtomFeedHttpMessageConverter());
			messageConverters.add(new RssChannelHttpMessageConverter());
		}

		if (jackson2XmlPresent) {
			Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.xml();
			if (this.applicationContext != null) {
				builder.applicationContext(this.applicationContext);
			}
			messageConverters.add(new MappingJackson2XmlHttpMessageConverter(builder.build()));
		}
		else if (jaxb2Present) {
			messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
		}

		if (jackson2Present) {
			Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json();
			if (this.applicationContext != null) {
				builder.applicationContext(this.applicationContext);
			}
			messageConverters.add(new MappingJackson2HttpMessageConverter(builder.build()));
		}
		else if (gsonPresent) {
			messageConverters.add(new GsonHttpMessageConverter());
		}
		else if (jsonbPresent) {
			messageConverters.add(new JsonbHttpMessageConverter());
		}

		if (jackson2SmilePresent) {
			Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.smile();
			if (this.applicationContext != null) {
				builder.applicationContext(this.applicationContext);
			}
			messageConverters.add(new MappingJackson2SmileHttpMessageConverter(builder.build()));
		}
		if (jackson2CborPresent) {
			Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.cbor();
			if (this.applicationContext != null) {
				builder.applicationContext(this.applicationContext);
			}
			messageConverters.add(new MappingJackson2CborHttpMessageConverter(builder.build()));
		}
	}
```



可以看到第一位是byte，第二位是String。。。。

试着调整我们自定义消息处理器的位置到后面：

```java
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        //针对字段的处理
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteNullListAsEmpty,// List字段如果为null,输出为[],而非null
                SerializerFeature.WriteMapNullValue,//加上后，字段为null的也会输出
                SerializerFeature.WriteNullStringAsEmpty,//字符类型字段如果为null,输出为”“,而非null
                SerializerFeature.WriteNullBooleanAsFalse,//Boolean字段如果为null,输出为false,而非null
                //SerializerFeature.WriteNullNumberAsZero, // Number 包装类如果为null，输出为0
                SerializerFeature.DisableCircularReferenceDetect,
                SerializerFeature.PrettyFormat  //结果是否格式化,默认为false
        );
        //日期格式化
        fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
        converter.setFastJsonConfig(fastJsonConfig);
        converters.add(3,converter);//返回是string的话，默认把这个放在最前，否则ResponseAdvisor 处理字符串返回时会报类型不一致的问题
    }
```

这里调整到第四位，可以满足要求。最后也是这样上线解决问题的。

现在还有一个问题，为什么FastJsonHttpMessageConverter要处理text/plain的请求呢？

AbstractMessageConverterMethodProcessor#writeWithMessageConverters中会选择消息转换器，能不能使用是通过消息转换器的canWrite方法来判断的，返回true就会使用这个消息转换器。

```java
for (HttpMessageConverter<?> converter : this.messageConverters) {
				GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
						(GenericHttpMessageConverter<?>) converter : null);
				if (genericConverter != null ?
						((GenericHttpMessageConverter) converter).canWrite(declaredType, valueType, selectedMediaType) :
						converter.canWrite(valueType, selectedMediaType)) {
					outputValue = getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType,
							(Class<? extends HttpMessageConverter<?>>) converter.getClass(),
							inputMessage, outputMessage);
					if (outputValue != null) {
						addContentDispositionHeader(inputMessage, outputMessage);
						if (genericConverter != null) {
							genericConverter.write(outputValue, declaredType, selectedMediaType, outputMessage);
						}
						else {
							((HttpMessageConverter) converter).write(outputValue, selectedMediaType, outputMessage);
						}
						if (logger.isDebugEnabled()) {
							logger.debug("Written [" + outputValue + "] as \"" + selectedMediaType +
									"\" using [" + converter + "]");
						}
					}
					return;
				}
			}
```

FastJsonHttpMessageConverter的canWrite调用了父类（AbstractHttpMessageConterver）的canWrite方法，这个方法的入参是Class<?> clazz(java.lang.String)和MediaType mediaType(text/plain;version=0.0.4;charset=utf-8)

AbstractHttpMessageConterver的canWrite长这样：

```java
    public boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType) {
        return this.supports(clazz) && this.canWrite(mediaType);
    }
```

supports方法直接就是写死的返回true，而canWrite走到最后text/*都会直接返回true，所以就这么愉快的通过了。。。

而人家jackson就实在很多，人家在JacksonHttpMessageConverterConfiguration中创建MappingJackson2HttpMessageConverter的时候，调用AbstractJackson2HttpMessageConverter中的构造器，传入的supportedMediaTypes只有两个关于json的！

```json
	/**
	 * Construct a new {@link MappingJackson2HttpMessageConverter} with a custom {@link ObjectMapper}.
	 * You can use {@link Jackson2ObjectMapperBuilder} to build it easily.
	 * @see Jackson2ObjectMapperBuilder#json()
	 */
	public MappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
		super(objectMapper, MediaType.APPLICATION_JSON, new MediaType("application", "*+json"));
	}
```

这块确实是挺坑的。。你说fastjson，你好好管你的json就完事了，你搞什么string啊。。

### 总结

Spring根据请求的MediaType选择对应的消息转换器，而我们这里使用的FastJsonHttpMessageConverter，并没有像jackson一样规定supportedMediaTypes，而是AbstractHttpMessageConverter中默认的*/*，导致将text/plain的报文也给转换了，才导致了这次遇到的问题。
