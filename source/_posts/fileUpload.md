---
title: springmvc文件上传、下载
date: 2017-11-30 15:28:33
tags: [springmvc]
---

现在基本web应用都需要用到文件上传的功能，不废话，直接上

# 文件上传

这里使用的是CommonsMultipartResolver。

<!-- more -->

## 导入jar

```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
	<version>1.3.1</version>
</dependency>
```

## 添加CommonsMultipartResolver配置

```xml
<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
   <!-- 设置上传文件的最大值 -->
	<property name="maxUploadSize" value="1048576000"></property>
  	<!-- 多大的文件会被保存在磁盘上 -->
	<property name="maxInMemorySize" value="4096" />
	<property name="defaultEncoding" value="UTF-8"></property>
</bean>
```

## HTML

这里注意一下设置form的enctype属性为multipart/form-data

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>

<form action="fileUpload" method="post" enctype="multipart/form-data">
	file :<input type="file" name="file">
	<input type="submit">
</form>

</body>
</html>
```

这里如果想上传多个文件的话，只需要多放几个`<input type="file" name="file">`

## controller

```java
package springmvc.controller;

import java.io.File;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Date;
import java.util.List;


import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

@Controller
@RequestMapping(value = "/person")
public class PersonController {

	public PersonController() {
		// TODO Auto-generated constructor stub
	}
	
	@RequestMapping(value="/fileUpload" ,method=RequestMethod.GET)
	public String fileUpload() {
		return "person/fileUpload";
	}
	
	
	@RequestMapping(value="/fileUpload" ,method=RequestMethod.POST)
	public String fileUpload(@RequestParam("file") CommonsMultipartFile file) throws IllegalStateException, IOException {
		long starttimes = System.currentTimeMillis();
		System.out.println(file.getOriginalFilename());
		System.out.println(file.getContentType());
		System.out.println(file.getName());
		System.out.println(file.getSize());
		String path="E:/"+(new Date()).getTime()+file.getOriginalFilename();
		File fi = new File(path);
		file.transferTo(fi);
		long endTime =System.currentTimeMillis();
		System.out.println("方法的运行时间："+String.valueOf(endTime-starttimes)+"ms");
		return "person/fileUpload";
	}

}

```

启动运行，没问题

如果是多个文件同时上传，CommonsMultipartFile参数修改为数组即可

>    Javahxjs-j2-gjtx9(jb51.net).rar
>    application/octet-stream    
>    file
>    121780292
>    方法的运行时间：83ms

# 文件上传出现异常的时候

## 自定义一个异常处理类

```java
package springmvc;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.multipart.MaxUploadSizeExceededException;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

/**
 * 自定义异常处理类
 * @author Administrator
 *
 */
public class ExceptionHandler implements HandlerExceptionResolver {

	public ExceptionHandler() {
	}

	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object obj,
			Exception ep) {

		ModelAndView mview = new ModelAndView();
		if(ep instanceof MaxUploadSizeExceededException) {
			mview.addObject("errormessage", "上传文件大小超出限制");
			mview.setViewName("error");
			return mview;
		}
		return null;
	}

}

```

## 在spring上下文中注册该Handler

```xml
<bean class="springmvc.ExceptionHandler"></bean>
```

## 编写错误页面

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<label th:text="${errormessage}">系统出现异常</label>
</body>
</html>
```

运行，搞定

>    上述的代码虽然可以成功将文件上传到服务器上面的指定目录当中，但是文件上传功能有许多需要注意的小细节问题，以下列出的几点需要特别注意的：
>
>    　　（1）、为保证服务器安全，上传文件应该放在外界无法直接访问的目录下，比如放于WEB-INF目录下。
>
>    　　（2）、为防止文件覆盖的现象发生，要为上传文件产生一个唯一的文件名。
>
>    　　（3）、为防止一个目录下面出现太多文件，要使用hash算法打散存储。
>
>    　　（4）、要限制上传文件的最大值。
>
>    　　（5）、要限制上传文件的类型，在收到上传文件名时，判断后缀名是否合法。

# 文件下载

## ResponseEntity`<byte[]>`方式

```java
	@RequestMapping("/download-file")
	public ResponseEntity<byte[]> downLoadFile(@RequestParam("filePath") String filePath, @RequestParam("oldName") String oldName) {
		byte[] body = null;
		File file = new File(filePath);
		// 获取文件
		InputStream is;
		try {
			is = new FileInputStream(file);
			body = new byte[is.available()];
			is.read(body);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		HttpHeaders headers = new HttpHeaders();
		// 设置文件类型
		headers.add("Content-Disposition", "attchement;filename=" + oldName);
		// 设置Http状态码
		HttpStatus statusCode = HttpStatus.OK;
		// 返回数据
		ResponseEntity<byte[]> entity = new ResponseEntity<byte[]>(body, headers, statusCode);
		return entity;
	}
```

## 传统模式

```java
	@RequestMapping("/download")
	@ResponseBody
	public void downloadCert(HttpServletRequest request, HttpServletResponse response)
			throws IOException {
		//得到要下载的文件名
         String fileName = request.getParameter("filename");
         fileName = new String(fileName.getBytes("iso8859-1"),"UTF-8");
         //假设上传的文件都是保存在/WEB-INF/upload目录下的子目录当中
         String fileSaveRootPath=this.getServletContext().getRealPath("/WEB-INF/upload");
         //处理文件名
         String realname = fileName.substring(fileName.indexOf("_")+1);
         //通过文件名找出文件的所在目录
         String path = findFileSavePathByFileName(fileName,fileSaveRootPath);
         //得到要下载的文件
         File file = new File(path+File.separator+fileName);
         //如果文件不存在
         if(!file.exists()){
             request.setAttribute("message", "您要下载的资源已被删除！！");
             request.getRequestDispatcher("/message.jsp").forward(request, response);
             return;
         }
         
          //设置响应头，控制浏览器下载该文件
          response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(realname, "UTF-8"));
          //读取要下载的文件，保存到文件输入流
          FileInputStream fis = new FileInputStream(path + File.separator + fileName);
          //创建输出流
          OutputStream fos = response.getOutputStream();
          //设置缓存区
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          //输入通道
          FileChannel readChannel = fis.getChannel();
          //输出通道
          FileChannel writeChannel = ((FileOutputStream)fos).getChannel();
          while(true){
              buffer.clear();
              int len = readChannel.read(buffer);//读入数据
              if(len < 0){
                  break;//传输结束
              }
              buffer.flip();
              writeChannel.write(buffer);//写入数据
          }
          //关闭输入流
          fis.close();
          //关闭输出流
          fos.close();
	}
```

[参考](https://www.cnblogs.com/lcngu/p/5471610.html)

