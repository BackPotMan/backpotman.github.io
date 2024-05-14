---
id: 43
title: 'python递归实现Easyui combotree树'
date: '2015-11-01T17:49:59+08:00'
author: huangdc
layout: post
guid: 'http://www.huangdc.com/?p=43'
permalink: /43
views:
    - '8357'
bigfa_ding:
    - '6'
duoshuo_thread_id:
    - '6214583362514846466'
categories:
    - Python
    - 技术杂谈
tags:
    - python
---

自动化发布系统在选择文件时，使用jQuery EasyUI 创建页面 树形菜单(Tree) 及 后端python 递归实现Easyui combotree 树。这里主要分享2点：

 1、linux 中 python 递归实现Easyui combotree 树

 2、jQuery EasyUI就不用介绍了，一款轻量级UI框架，集成了各种用户界面插件。

关于jQuery EasyUI 可以参考： http://www.w3cschool.cc/jeasyui/jqueryeasyui-tutorial.html

关于自动化发布系统，可以查看：<http://www.huangdc.com/13>

<span style="padding: 0px; margin: 0px;">本文环境基础：<http://www.huangdc.com/21></span>

<span style="padding: 0px; margin: 0px;">目的：用户在页面选择对应项目，前端展示对应项目svn目录列表</span>

## <span style="font-size: 18px;">**废话不多说，先上图看看UI**</span>

选择项目时，展示项目 目录的 tree

![](/assets/wp-content/uploads/2015/11/104854_yPRF_588586.jpg "104854_yPRF_588586.jpg")

## <span style="font-size: 18px;">jQuery EasyUI tree 代码 html </span>

 ##需要调用easyui.css 和 jquery.easyui.min.js

 index.html

```
<!DOCTYPE html>
<!-- saved from url=(0050)http://getbootstrap.com/examples/starter-template/ -->
<html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">
    <link rel="icon" href="http://getbootstrap.com/favicon.ico">
 
    <title>Python EasyUI tree @DcHuang</title>
    <!-- 需要加上easyui.css 和 bootstrap.min.css -->
    <link rel="stylesheet" type="text/css" href="http://www.jeasyui.com/easyui/themes/default/easyui.css">
    <!-- Bootstrap core CSS -->
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    </head>
 
    <body>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
      <div>
        <div>
          <a href="http://getbootstrap.com/examples/starter-template/#">Project name</a>
        </div>
        <div id="navbar" class="collapse navbar-collapse">
          <ul class="nav navbar-nav">
            <li><a href="http://getbootstrap.com/examples/starter-template/#">Home</a></li>
            <li><a href="http://getbootstrap.com/examples/starter-template/#about">About</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav>
 
    <div class="container">
      <div class="starter-template">
        <p>@DcHuang</p>
        </br>
        </br>
      </div>
      <!--重点关注一下-->
      <div class="well">
        <form class="form-horizontal" role="form" action="/deployserver/deploy/" method="post" id="Form-DeployProject">
                                <div class="form-group">
                                        <label class="col-sm-2 control-label">标题<span style="color:red">*</span></label>
                                        <div class="col-sm-8">
                                            <input type="text" class="form-control" name="DeployTitle" id="DeployTitle">
                                        </div>
                                </div>
 
 
                                <div class="form-group">
                                        <label class="col-sm-2 control-label">发布项目<span style="color:red">*</span></label>
                                                <div class="col-sm-4">
                                                        <select class="form-control" name="DeployProject" id="DeployProject" >
                                                                <option value="0" selected >--项目0--</option>
                                                                <option value="1" selected >--项目1--</option>
                                                        </select>
                                                </div>
 
                                                <div class="col-sm-4">
                                                        <select class="form-control" name="DeployTransitip" id="DeployTransitip">
                                                                <option value="0" selected >--可选中转IP--</option>
                                                        </select>
                                                                                                        </div>
 
                                                <div class="col-sm-4 col-sm-offset-2" style="margin-top:10px">
                                                        <div style="border: 1px solid #ccc;border-radius: 6px 6px 6px 6px;">
                                                                <div class="control-group" style="height:25px;text-align:center;">
                                                                --选择目标IP--
                                                                </div>
                                                                <div class="control-group" style="margin-bottom:6px;margin-left:20px;width:100%" id="div_server">
                                                                </div>
                                                        </div>
                                                </div>
 
                                                <div class="col-sm-4" style="margin-top:10px">
                                                        <div style="border: 1px solid #ccc;border-radius: 6px 6px 6px 6px;">
                                                                <div class="control-group" style="height:25px;text-align:center;">
                                                                --中转IP--
                                                                </div>
                                                                <div class="control-group" style="margin-bottom:6px;margin-left:20px;width:100%" id="div_transitip">
                                                                </div>
                                                        </div>
                                                </div>
                                </div>
 
                                <!-- 重点：easyui-tree -->
                                <div class="form-group">
                                        <label class="col-sm-2 control-label">更新文件<span style="color:red">*</span></label>
                                        <div class="col-sm-9">
                                                <button type="button" class="btn btn-primary btn-xs" name="reloadAll" onclick="reloadall();">刷新列表</button>
                                                <button type="button" class="btn btn-primary btn-xs" name="expandAll" onclick="expandall();">展开</button>
                                                <button type="button" class="btn btn-primary btn-xs" name="collapseAll" onclick="collapseall();">收缩</button>
                                                <input type="button"  class="btn btn-success btn-xs" id="svnversion" value="SVN版本" disabled="disabled"/>
 
                                        </div>
                                        <div class="col-sm-9 col-sm-offset-2" style="margin-top:10px">
                                                <div class="easyui-panel" style="padding:5px;" id="div_filelist">
                                                        <ul id="file_list" class="easyui-tree"></ul>
                                                </div>
                                        </div>
                                </div>
 
                                <div class="form-group">
                                        <label class="col-sm-2 control-label">发布说明</label>
                                        <div class="col-sm-8">
                                            <input type="text" class="form-control" name="DeployNotes" id="DeployNotes">
                                        </div>
                                </div>
 
                                <div class="form-group">
                                        <div class="col-sm-offset-2 col-sm-10">
                                                <button type="button" class="btn btn-default" name="b_deploy_reset" onclick="form_deploy_reset();">重置</button>
                                                <button type="button" class="btn btn-primary" name="b_deploy_upload" onclick="form_deploy();">提交</button>
                                        </div>
                                </div>
        </form>
    </div>
 
    </div><!-- /.container -->
        <!-- jquery JavaScript -->
    <script type="text/javascript" src="http://code.jquery.com/jquery-1.6.min.js"></script>
    <script type="text/javascript" src="http://www.jeasyui.com/easyui/jquery.easyui.min.js"></script>
    <!-- Bootstrap core JavaScript -->
    <script src="/static/js/bootstrap.min.js"></script>
    </body>
    </html>
```


## <span style="font-size: 18px;">js代码 </span>

可以放在index.html文件后面，根据所选择项目id，后端生成tree文件

```
    <script>
        $("#DeployProject").change(function(){
                var project_id=$("#DeployProject").val();
                //后端刷新tree json文件
                $.ajax({
                    type:"POST",
                    url:"/",
                    data:{'type':"select_project",'project_id':project_id},
                    success:function(response){
                        //alert(response);
                        // tree 
                        $("#file_list").tree({
                                // 记得加上new Date() ，原因是cache问题 ；有兴趣你试试不加，更改目录文件看看
                                url:'/static/data/tree_data_'+project_id+'.json?tm=' + new Date(),
                                method:'get',
                                animate:true,
                                checkbox:true
                        });
                        $('#file_list').tree('reload');
                    },
                    error:function(xhr,textStatus){
                        alert(textStatus);
                    }
                });
        });
    </script>
```


## <span style="font-size: 18px;">python 递归实现Easyui combotree 树，后端pyhton代码</span>

 views.py 文件

```
from django.shortcuts import render,render_to_response
from django.http import HttpResponse,HttpResponseRedirect
import time,json,os,commands,urllib,sys,subprocess,datetime,signal
#coding=utf-8
#import time,simplejson,json,os,commands
##
default_encoding = 'utf-8'
if sys.getdefaultencoding() != default_encoding:
        reload(sys)
        sys.setdefaultencoding(default_encoding)
 
# Create your views here.
def index(req):
        ##
        if req.method == 'POST':
                if req.POST.has_key('type') and req.POST['type']=="select_project":
                        project_id=req.POST['project_id']
                        ##调用CreateDirTree函数
                        CreateDirTree('/data/myproject/static/','/data/myproject/static/data/tree_data_'+str(project_id)+'.json')
                        return HttpResponse("ok")
        #return HttpResponse("aa")
        return render_to_response('index.html',)
 
## 创建文件树SVN
## python递归实现Easyui combotree树
def CreateDirTree(path,file_json):
        global id_num
        id_num=0
        def createDict(path):
                global id_num
                tree_list=[]
                pathList = os.listdir(path)
                if '.svn' in pathList:
                        pathList.remove('.svn')
                for i,item in enumerate(pathList):
                        id_num+=1
                        children_map={}
                        children_map['id']=id_num
                        children_map['text']=item
                        #children_map['checked']='true'
                        children_map['attributes']={'url':path}
                        if isDir(getJoinPath(path,item)):
                                path = getJoinPath(path,item)
                                tmp_listdir=os.listdir(path)
                                if '.svn' in tmp_listdir:
                                        tmp_listdir.remove('.svn')
                                if  len(tmp_listdir) != 0:
                                        children_map['state']='closed'
                                        """开始递归子节点"""
                                        children_map['children'] = createDict(path)
                                tree_list.append(children_map)
                                path = '/'.join(path.split('/')[:-1])
                        else:
                                tree_list.append(children_map)
                return tree_list
 
        '''合并路径和目录，返回完整路径'''
        def getJoinPath(path,item):
                return os.path.join(path,item)
 
        '''判断是否为目录'''
        def isDir(path):
                if os.path.isdir(path):
                        return True
                return False
 
        '''返回json格式数据'''
        def getJson(treestr):
                return json.dumps(treestr)
 
        '''json 文件'''
        tree_list=createDict(path)
        fjson = getJson(tree_list)
        with open(file_json,'w') as lf:
                lf.write(fjson)
        ##end
```

<span style="font-size: 16px;">**生成的文件格式如下：**</span>

```
[{"text": "css", "state": "closed", "children": [{"text": "bootstrap.css", "id": 2, "attributes": {"url": "/data/myproject/static/css"}}, {"text": "bootstrap-theme.min.css", "id": 3, "attributes": {"url": "/data/myproject/static/css"}}, {"text": "bootstrap.min.css", "id": 4, "attributes": {"url": "/data/myproject/static/css"}}, {"text": "bootstrap.css.map", "id": 5, "attributes": {"url": "/data/myproject/static/css"}}, {"text": "bootstrap-theme.css", "id": 6, "attributes": {"url": "/data/myproject/static/css"}}, {"text": "bootstrap-theme.css.map", "id": 7, "attributes": {"url": "/data/myproject/static/css"}}], "id": 1, "attributes": {"url": "/data/myproject/static/"}}, {"text": "js", "state": "closed", "children": [{"text": "bootstrap.js", "id": 9, "attributes": {"url": "/data/myproject/static/js"}}, {"text": "bootstrap.min.js", "id": 10, "attributes": {"url": "/data/myproject/static/js"}}, {"text": "npm.js", "id": 11, "attributes": {"url": "/data/myproject/static/js"}}, {"text": "jquery-1.11.1.min.js", "id": 12, "attributes": {"url": "/data/myproject/static/js"}}], "id": 8, "attributes": {"url": "/data/myproject/static"}}, {"text": "data", "state": "closed", "children": [{"text": "tree_data_0.json", "id": 14, "attributes": {"url": "/data/myproject/static/data"}}, {"text": "tree_data_1.json", "id": 15, "attributes": {"url": "/data/myproject/static/data"}}], "id": 13, "attributes": {"url": "/data/myproject/static"}}, {"text": "fonts", "state": "closed", "children": [{"text": "glyphicons-halflings-regular.ttf", "id": 17, "attributes": {"url": "/data/myproject/static/fonts"}}, {"text": "glyphicons-halflings-regular.svg", "id": 18, "attributes": {"url": "/data/myproject/static/fonts"}}, {"text": "glyphicons-halflings-regular.woff", "id": 19, "attributes": {"url": "/data/myproject/static/fonts"}}, {"text": "glyphicons-halflings-regular.eot", "id": 20, "attributes": {"url": "/data/myproject/static/fonts"}}], "id": 16, "attributes": {"url": "/data/myproject/static"}}]
```

转载请注明：[Huangdc](https://www.huangdc.com) » [python递归实现Easyui combotree树](https://www.huangdc.com/43)

</body></html>