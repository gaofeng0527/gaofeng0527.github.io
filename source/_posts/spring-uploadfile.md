---
title: Spring结合uploadFile插件上传文件
date: 2019-05-24 13:49:12
tags: springmvc
---

# 简介

今天整理了一个上传文件的基于Jquery开发的插件。[参考](http://www.jq22.com/jquery-info13450)

提供单，多图片、文件上传；进度条支持；同步，异步上传等。

# 使用

下载插件包，把文件复制到工程WEB-INF目录下

<!-- more -->

应用实例

```html
<!DOCTYPE html>
<html lang="ch">
<head>
    <meta charset="UTF-8">
    <title>文件上传案例</title>
    <link href="css/iconfont.css" rel="stylesheet" type="text/css"/>
    <link href="css/fileUpload.css" rel="stylesheet" type="text/css">
</head>
<body>
    <!--创建一个文件上传的容器-->
    <div id="fileUploadContent" class="fileUploadContent"></div>
    <button onclick="success()">模拟上传成功</button><button onclick="fail()">模拟上传失败</button>
    <a href="doc.html">文档说明</a>
</body>
</html>
<script src="http://www.jq22.com/jquery/jquery-2.1.1.js"></script>
<script type="text/javascript" src="js/fileUpload.js"></script>
<script type="text/javascript">
    let wfu = new WpFileUpload("fileUploadContent");
    wfu.initUpload({
        "uploadUrl":"#",//上传文件信息地址
        "progressUrl":"#",//获取进度信息地址，可选，注意需要返回的data格式如下（{bytesRead: 102516060, contentLength: 102516060, items: 1, percent: 100, startTime: 1489223136317, useTime: 2767}）
        "scheduleStandard":true,
    });

    // //显示文件，设置删除事件
    wfu.showFileResult("img/a2.png","1",null,true,true,deleteFileByMySelf,downloadByMySelf);
    // //如果不需要删除
    wfu.showFileResult("img/a3.png","2",null,false,false,null,null);
    </script>
```

## 重要参数介绍

```js
 let wfu = new WpFileUpload("fileUploadContent");
    wfu.initUpload({
        "uploadUrl":"#",//上传文件信息地址
        "progressUrl":"#",//获取进度信息地址，可选，注意需要返回的data格式如下（{bytesRead: 102516060, contentLength: 102516060, items: 1, percent: 100, startTime: 1489223136317, useTime: 2767}）
        "scheduleStandard":true,
      	"selfUploadBtId":"fileUploadId",//自定义文件上传按钮ID
        "selfUploadBtId":false,//	是否记住已经上传过的文件，如果是记住，那么下次在上传的时候将不会允许，注意：当浏览器刷新的时候，这个设置将无效d
      	"autoCommit":false,//是否自动上传，如果为true的话，会根据配置的uploadUrl属性直接把文件上传到服务器
      	"isHiddenUploadBt":false,//是否隐藏上传按钮，这个上传按钮是选着文件旁边的那个按钮（如果设置为自动上传的话，该属性意义不大）
      	"isHiddenCleanBt":false,//是否隐藏清楚按钮
      	"isAutoClean":false,//上传完成后是否自动清除
      	"fileType":['png','docx'],//文件类型限制，默认不限制，注意写的是文件后缀，如：['png','jpg','docx','doc']
      	"size":1024,//文件大小限制，单位kb
      	"totalSize":51200,//总上传文件的大小
      	"maxFileNumber":12,//每次上传文件的总个数限制
      	"showSummerProgress":true,//是否显示总进度条，默认显示
      	"showFileItemProgress":true,//是否显示单文件进度条，默认显示
      	"showProgressNum":false,//是否显示进度条的值
      	"uploadFileParamIteration":true,//要上传的文件参数是否迭代，如果true：files0，files1，files2...... ， 如果是false：所有的文件通过一个参数uploadFileParam所设定的值（默认：files）来上传文件
      	"beforeUpload":function(){
          console.log("文件上传之前");
          console.log(wfu.uploadFileParam);//uploadFileParam上传文件数据的参数，可以通过实例调用
      	},
      	"onUpload":function(){
          console.log("文件上传完成");
          console.log(wfu.resultData);//resultData上传成功后返回的信息将会存在该参数下，可以通过实例调用
      	}
    });
//WpFileUpload的几个重要函数
//文件上传失败
function fialy(){
  wfu.uploadError();
}
//文件上传成功
function fileUploadSuccess(){
  wfu.uploadSuccess();
}

function deleteFileByMySelf(fileId){
    alert("要删除文件了："+fileId);
	wfu.removeShowFileItem(fileId);
} 

/*
1、上传状态显示:
（1）上传失败：
wfu.uploadError();
（2）上传成功：
wfu.uploadSuccess();
2、删除回显的文件:
wfu.removeShowFileItem(fileId); // fileId指文件的Id
3、上传成功后回显文件:
wfu.showFileResult(fileUrl,fileId,defineFileName,deleteFile,downloadFile,deleteEvent,downLoadEvent)
fileUrl:文件地址 
fileId:文件Id 
defineFileName:自定义文件名，注意带后缀 
deleteFile:是否可删除文件 
downloadFile:是否可下载文件 
deleteEvent:删除文件事件 ，注意，返回只有一个数据：文件的ID（fileId）
downLoadEvent:下载文件事件，注意，返回有两个数据：文件的ID（fileId）和文件的地址（fileUrl）

关于上传注意事项：
1、可以同时关闭主进度条和文件进度条

2、如果没有进度信息，设置scheduleStandard:false,progressUrl:#,默认会上传到60%，然后根据后台返回的结果进行设置成功还是失败

3、如果是PHP开发，请使用上文文件参数迭代：files0，files1，files2，.........
*/
```

# 后台部分

后台使用springmvc，参考[springmvc文件上传](fileUpload.md)

