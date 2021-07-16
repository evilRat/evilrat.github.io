---
title: feign上传文件
excerpt: 'feign'
tags: [SpringCloud]
categories: [SpringCloud]
comments: true
date: 2021-07-15 13:30:10
---

今天要给app提供一个上传文件的接口，这块我们再pc端的服务上已经有了这样一个接口，所以提供给app的接口，要在给app提供能力的服务上通过feign调用pc端服务的上传接口来实现，这样可以保证pc端和移动端一致，后期也好维护。

说着就在app服务增加了一个feignClient
```java
    /**
     * 图片上传 feignClient
     *
     * @param
     * @return
     */
    @ApiOperation("文件流方式上传")
    @PostMapping(value = "/file-upload")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "prefix",value = "文件前缀（例如：工单号）",required = true),
            @ApiImplicitParam(name = "filename",value = "文件名称（上传文件名称,包含后缀名，如123456.jpg）",required = true)
    })
    @ResponseBody
    Response<UploadResult> uploadFile(
            @RequestParam("file") MultipartFile upfile,
            @RequestParam(value = "prefix") String prefix,
            @RequestParam(value = "filename") String filename
    ) ;
```

然后一个api接口：

```java
    /**
     * 图片上传 api
     *
     * @param
     * @return
     */
    @ApiOperation("文件流方式上传")
    @PostMapping(value = "/file-upload")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "prefix",value = "文件前缀（例如：工单号）",required = true),
            @ApiImplicitParam(name = "filename",value = "文件名称（上传文件名称,包含后缀名，如123456.jpg）",required = true)
    })
    @ResponseBody
    Response<UploadResult> uploadFile(
            @RequestParam("file") MultipartFile upfile,
            @RequestParam(value = "prefix") String prefix,
            @RequestParam(value = "filename") String filename
    ) {
        return datumFeign.uploadFile(upfile, prefix, filename);
    }
```

然后就开心的自测，感觉应该没啥问题。

没想到收到PC端服务返回的`org.springframework.web.multipart.MultipartException: Current request is not a multipart request`，很奇怪，我的feignClient传了MultipartFile了，而且打断点看了的。

后来查了一些资料，发现FeignClient是不支持MultipartFile的，需要使用openfeign的feign-form来实现。

增加依赖：

```xml
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.0.1</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.0.1</version>
</dependency>
```

feignClient的PostMapping注解增加属性：`consumes = MediaType.MULTIPART_FORM_DATA_VALUE`

```java
/**
     * 图片上传
     *
     * @param
     * @return
     */
    @ApiOperation("文件流方式上传")
    @PostMapping(value = "/file-upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ApiImplicitParams({
            @ApiImplicitParam(name = "prefix",value = "文件前缀（例如：工单号）",required = true),
            @ApiImplicitParam(name = "filename",value = "文件名称（上传文件名称,包含后缀名，如123456.jpg）",required = true)
    })
    @ResponseBody
    Response<UploadResult> uploadFile(
            @RequestParam("file") MultipartFile upfile,
            @RequestParam(value = "prefix") String prefix,
            @RequestParam(value = "filename") String filename
    ) ;
```

这时候在测试一下接口，返回`org.springframework.web.multipart.support.MissingServletRequestPartException: Required request part 'file' is not present`

也就是说，第一次都不是个multipart request，现在是少是个multipart request了，只是没有file这个request part。

翻看了《Spring实战》的“处理multipart形式的数据”章节，`multipart/form-data`不是普通的表单，而是每个输入域都是一个part，接收这写part需要使用`@RequestPart`注解。

于是将feignClient中的`@RequestParam`修改为`@RequestPart`

```java
/**
     * 图片上传 api
     *
     * @param
     * @return
     */
    @ApiOperation("文件流方式上传")
    @PostMapping(value = "/file-upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ApiImplicitParams({
            @ApiImplicitParam(name = "prefix",value = "文件前缀（例如：工单号）",required = true),
            @ApiImplicitParam(name = "filename",value = "文件名称（上传文件名称,包含后缀名，如123456.jpg）",required = true)
    })
    @ResponseBody
    Response<UploadResult> uploadFile(
            @RequestPart("file") MultipartFile upfile,
            @RequestParam(value = "prefix") String prefix,
            @RequestParam(value = "filename") String filename
    ) ;
```

至此，上传文件成功！

