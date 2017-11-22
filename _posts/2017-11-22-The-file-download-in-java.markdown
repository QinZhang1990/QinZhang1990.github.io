---
layout: post
title: "java中文件下载"
date: 2017-11-22 21:42:00.000000000 +00:00
tags: 他山之石
---

### 1 单个文件下载

（1）通过数据库查询或是现有参数集合中获取待下载文件的路径filePath

（2）构建文件对象

```swift
File file = new File(filePath);
```
（3）获取待下载文件内容的字节数据组
```swift
FileInputStream stream = new FileInputStream(file);
	ByteArrayOutputStream out = new ByteArrayOutputStream(1000);
	byte[] b = new byte[1000];
	int n;
	while((n=stream.read(b))!=-1){
		out.write(b, 0, n);
		stream.close();
		out.close();
	}
	bytes[] fileBytes = out.toByteArray();
```

（4）设置请求响应Response的头部，并将字节内容写入到Response的输出内容中，其中最关键的两个设置，即如下代码中的`[1]`
和`[2]`，[1]告诉浏览器以附件attachment形式下载，[2]告诉浏览器下载文件的类型，除了[2]中的形式，也可以设为二进制的形式。
```swift
public static String outputOneFile(HttpServletResponse resp, byte[] fileBytes, String fileName){
		resp.setHeader(“Content-Transfer-Encoding”, “binary”);
		resp.setHeader(“Cache-Control”, “must-revalidate,post-check=0, pre-check=0”);
		resp.setHeader(“Pragma”, “public”);
		resp.setHeader(“Content-Disposition”, (new StringBuilder(“attachment; filename=”))
		.append(new String(filename.getBytes(“GBK”), “ISO-8895-1”)).toString());**[1]**
		// resp.setContentType(“application/octet-stream”);
		resp.setContentType(“application/x-msdownload”);**[2]**
		resp.setContentLength(fileBytes.length);
		//将字节内容写入输入流中
		OutputStream outs = null;
		try{
		   outs = resp.getOutputStream();
		   outs.write(fileBytes);
		} catch(Exception e){
		}finally{
		   try{
		      if(outs != null){
		            outs.close();
		      }
		   }catch(Exception e){
		    e.printStachTrace();
		   }
		}
		return null;
	}
```

### 2 多个文件打包压缩ZIP下载
java中文件的打包压缩下载通过ZipOutputStream和ZipEntry类来实现，具体的代码实现如下：
```swift
public void downloadMultiFiles(HttpServletResponse response,List<FileInfoDTO> fileList) throws IOException {
    		String zipName = "压缩包的名称.zip";
           //设置头部
            response.setContentType("APPLICATION/OCTET-STREAM");
            try {
                response.setHeader(
                  "Content-Disposition",
                   (new StringBuilder("attachment;filename=")).append(
                         new String(zipName.getBytes("GBK"), "ISO-8859-1")).toString());
            } catch (UnsupportedEncodingException e1) {
                e1.printStackTrace();
            }
            ZipOutputStream out = new ZipOutputStream(response.getOutputStream());
            try {
            	for(FileInfoDTO fi:fileList){ //逐个文件遍历，写入到压缩包中
            		String filePath = fi.getFilePath();
String fileName = fi.getEntryName();
            		doZip(filePath, out, fileName);
            		response.flushBuffer();
            	}
            } catch (Exception e) {
        		e.printStackTrace();

            }finally{
                out.close();
            }
    	}
}


public void doZip(String filePath, ZipOutputStream out, String fileName) throws IOException{
    	File inFile = new File(filePath);
    	//对应压缩文件的名称
    	String entryName = fileName
     //设置压缩包中子文件的名称
ZipEntry entry = new ZipEntry("压缩包的名称"+File.separator + entryName);
     out.putNextEntry(entry);

     int len = 0 ;
     byte[] buffer = new byte[1024];
     FileInputStream fis = new FileInputStream(inFile);
     while ((len = fis.read(buffer)) > 0) {
         out.write(buffer, 0, len);
         out.flush();
     }
     out.closeEntry();
     fis.close();
}
```
在实现该功能模块的过程中，也遇到如下问题：
（1）采用J2EE自带的压缩类，即java.util.zip.ZipOutputStream，java.util.zip.ZipEntry实现文件压缩下载，可能会出现
文件名中文乱码的情况，此处可以采用Apache提供的同名类，处理该问题，即`org.apache.tools.zip.ZipOutputStream`，
`org.apache.tools.zip.ZipEntry`。
（2）曾尝试通过Ajax发送异步请求来打包压缩下载文件，发现当请求响应回来之后，并不能触发浏览器打开下载框，进行下载，
百度了一下，贴上两个较为合理的原因，如下：
```swift
$.ajax({
        type: "GET",
        url: "/rc.data.btDownload.do",
        data: {
        	calculateDate:calculateDate,
        	companyNo:companyNo,
        	templateType:templateType
       	 	},
        dataType: "json",
        success: function(data){
        	console.log(data);
        },
         error: function(data){
       	  alert("操作保存失败!");
       	  console.log(data.responseText);
         }
    });
```

1）因为response原因，一般请求浏览器是会处理服务器输出的response，例如生成png、文件下载等，然而ajax请求只是个
“字符型”的请求，即请求的内容是以文本类型存放的。文件的下载是以二进制形式进行的，虽然可以读取到返回的response，
但只是读取而已，是无法执行的，说白点就是js无法调用到浏览器的下载处理机制和程序。[查看原文](http://blog.csdn.net/fan510988896/article/details/71520390)

2）实验：ajax方式下载文件时无法触发浏览器打开保存文件对话框，也就无法将下载的文件保存到硬盘上！
原因：ajax方式请求的数据只能存放在javascipt内存空间，可以通过javascript访问，但是无法保存到硬盘，因为javascript
不能直接和硬盘交互，否则将是一个安全问题。[查看原文](http://www.cnblogs.com/nuccch/p/7151228.html)

