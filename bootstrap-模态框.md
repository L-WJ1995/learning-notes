#理解Bootstrap模态框
今日在学习VUE时解除到了模态框这个词,了解一番后感觉模态框在实际开发中应用到的地方比较多,所以研究了一段时间,针对模态框写一些东西,方便自己日后查看。
-----
* 模态框bootstrap官方描述如下：
    1. Bootstrap 模态框是一个轻量级的多用途JavaScript弹出窗口，可自定义和响应式。可以使用它在网站中显示警告窗口、视频和图片。使用Bootstrap构建的网站可以使用模态框来显示条款和条件（作为注册过程的一部分），视频，甚至社交媒体小部件。  
  
    2. Bootstrap 模态框主要分为三个部分：头部（header），正文（body）和页脚（footer）。每个部分都有自己的意义，因此我们应该正确的使用它们。我们稍后将讨论这些。Bootstrap模态框最令人兴奋的是什么？你不需要写任何JavaScript代码就可以使用它！所有的代码和样式都是由Bootstrap预定义的。你所需要做的只是使用正确的标记和属性来触发它。   
      
* 默认的模态框：
    默认的模态框
    ![默认的模态框](http://www.runoob.com/wp-content/uploads/2014/07/modalplugin_demo.jpg)  
      
* 模态框的触发和所需的代码：
    在使用模态框之前,需要导入bootstrap的css文件和js文件,如果使用的功能比较多则可能还需要引入jQuery文件。
    1. 要触发模态框,需要添加链接或者按钮,如下所示：
    ```
    <a href="#" class="btn btn-lg btn-success" data-toggle="modal" data-target="#basicModal">Click to open Modal</a>
    ```
    模态框触发元素有两个自定义数据属性：data-toggle和data-target。toggle告诉Bootstrap要做什么，target告诉Bootstrap要打开哪个元素。所以每当点击这样的链接时，都会出现一个id为“basicModal”的模态框。  
      
    2. 接下来则是模态框需要的代码实例：
    ```
    <div class="modal fade" id="basicModal" tabindex="-1" role="dialog" aria-labelledby="basicModal" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
            <h4 class="modal-title" id="myModalLabel">Modal title</h4>
            </div>
            <div class="modal-body">
                <h3>Modal Body</h3>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                <button type="button" class="btn btn-primary">Save changes</button>
            </div>
        </div>
    </div>
    </div>
    ```

    此例中,模态框的父div应具有与上述触发元素中使用的相同的ID。在我们的例子中是id="basicModal"。

    在模态框的HTML代码中，我们可以看到一个封装div嵌套在父模态框div内。这个div的类modal-content告诉bootstrap.js在哪里查找模态框的内容。在这个div内，我们需要放置前面提到的三个部分：头部，正文和页脚。

    模态框头部，顾名思义，用于给模态添加一个标题或者如“x”关闭按钮等其他元素。这些元素还应该有一个data-dismiss属性告诉Bootstrap哪个元素要隐藏。

    然后我们看一下模态框的正文。可以把它看做一个打开的画布，你可以在其中添加任何类型的数据，包括嵌入YouTube视频，图像或者任何其他内容。

    最后，我们看一下模态框的页脚。该区域默认为右对齐。在这个区域，你可以放置“保存”，“关闭”，“接受”等操作按钮，这些按钮与“模态框”需要表现的行为相关联。  
      
* 改变模态框大小
    bootstrap是一个灵活的响应式框架,所以它的模态框插件继承了这个特性,改变模态框大小的方式就是在元素增加一个名`modal-dialogdiv`的属性,有两个值可选`modal-lg`和`modal-sm`,即可设置大小。

* 模态框还有对应的jQuery激活以及事件绑定。