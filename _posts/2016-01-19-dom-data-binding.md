---
layout: post
title: "DOM 双向绑定"
description: ""
category: 
tags: [JavaScript]
---
{% include JB/setup %}

因为觉得AngularJS太重了，一般都会想尝试重新造轮子，先做一些基础调查吧。

<http://www.html-js.com/article/A-day-to-learn-JavaScript-using-the-native-JavaScript-data-binding>

ps: 上面链接的讨论圆点功能不错考虑后期加上^_^!

<http://www.jianshu.com/p/ee014d86cd3f>

<https://github.com/xufei/Make-Your-Own-AngularJS/blob/master/01.md>

AngularJS是脏值检测，想着用原生办法解决，试试吧，如果不用jQuery的话。

<http://www.zhuowenli.com/frontend/easy-two-way-data-binding-in-javascript.html>

这篇文章讲的比较细致

但是对于操作上需要使用set函数，太麻烦了。

其实想想jquery 80，90 kb，angularjs1.x 123kb，加起来就是200kb。
按照手机10kb/s就是20秒，但是大多都是牛逼的手机吧，应该两秒内可以下完，但是，哎哎哎，为什么要考虑手机呢。。。我也就是自己想重新造个轮子。。。不过200kb确实似乎有点耗时间啊>_<


再贴一篇：<http://www.ituring.com.cn/article/48463>

囫囵吞枣的了解了大概，估计过两天就忘了。。。

大致原理就是通过html的自定义属性绑定DOM标签，加上$watch （突然想到了硬件上的watchdog，一直觉得当时老师说喂狗，让狗叫很萌啊。。。）

找到了`前端乱炖`的那个圆点了，叫tips.js，不知道LICENSE是如何，觉得可能需要自己重写一个，那个真的好好玩啊

代码丢上来~ 如侵权联系我删

    var Tip = function(){
        this.ele = null;
        this.url = null;
        this.commentForm =null;
        this.lastTip = null;
        this.commentsTpl = null;
        this.commentList = null;
        this.is_hide = true;
    }
    var getRandomColor = function() {
        return  '#' +
            (function (color) {
                return (color += '0123456789abcdef'[Math.floor(Math.random() * 16)])
                    && (color.length == 6) ? color : arguments.callee(color);
            })('');
    }
    Tip.prototype = {
        init:function(_config){$("<div class='tip-comments'></div>")
            this.ele = $(_config.selector);
            this.url = _config.url;
            this.uuid = _config.uuid
            this.commentsTpl = $("#tip_tpl").html()
            this.container = $("<div class='tip-comments'><ul class='comment-list'></ul></div>")
            this.commentList = $(".comment-list",this.container);
            this.container.appendTo(this.ele)
            this._createView();
            this._bind();
            this._loadTips();
            this._hideComments();

        },
        _load:function(){

        },
        _createView:function(){
            this.commentForm = $('<form class="tip-comment-form">'+
                '<textarea placeholder="添加讨论…" class="tip-comment-textarea" ></textarea>'+
                '<div class="form-control" >'+
                '<button class="tip-submit" type="submit">提交</button>'+
                '</div>'+
                '</form>')
            this.commentForm.appendTo(this.container)
        },
        _bind:function(){
            var self = this;
            this.ele.on("dblclick",function(e){
                if(self.is_hide){
                    self._createTip(e.pageX-self.ele.offset().left, e.pageY-self.ele.offset().top);
                    e.stopPropagation()
                }else{

                }
            })
            $(".cancel",this.commentForm).on("click",function(){

            })
            this.commentForm.on("click",function(e){
                e.stopPropagation();
            }).on("submit",function(e){
                e.preventDefault();

                var submitData = {
                    target_id:self.uuid,
                    content:$("textarea",self.commentForm).val(),
                    page_x:self.lastTip?self.lastTip.x:0,
                    page_y:self.lastTip?self.lastTip.y:0
                }
                if(!submitData.content ){
                    messageTip.show("请填写内容~~")
                    return;
                }
                if(self.commentForm.tipId){
                    submitData.parent_id = self.commentForm.tipId
                }
                HtmlJS.util.ajax(self.url,submitData,"post",function(data){
                    if(data.code){
                        alert("在一个页面最多只能添加5枚评注")
                    }else{
                        self._loadTip(self.commentForm.tipId?self.commentForm.tipId:data.id)
                        $("textarea",self.commentForm).val("")
                        self.lastTip.uuid = data.id
                    }

                },function(){

                },null,function(){
                    HtmlJS.util.ajax(self.url,submitData,"post",function(data){
                        if(data.code){
                            alert("在一个页面最多只能添加5枚评注")
                        }else{
                            self._loadTip(self.commentForm.tipId?self.commentForm.tipId:data.id)
                            $("textarea",self.commentForm).val("")
                            self.lastTip.uuid = data.id
                        }
                    },function(){

                    },null,function(){

                    })
                })
            })
            $(document.body).on("click",function(){
                self._hideComments()
            })
            $("#hide_tip").on("click",function(e){
                e.stopPropagation();
                if(this.checked){
                    $(".tip-point").hide();
                }else{
                    $(".tip-point").show();
                }
            })
        },
        _hideComments:function(){
            this.is_hide = true;
            this.container.hide();
        },
        _showComments:function(){
            this.is_hide = false;
            this.container.show();
        },
        _loadTip:function(tipId){
            var self = this;
            $.ajax({
                url:"/tip/"+tipId,
                success:function(tips){
                    if(tips.length){
                        var html = Mustache.render(self.commentsTpl,{
                            tips:tips
                        })
                        self.commentList.html(html);
                        self.commentForm.tipId = tips[0].id
                        self.container.css({
                            left:tips[0].page_x,
                            top:tips[0].page_y
                        })
                        self._showComments();

                    }
                }
            })
        },
        _loadTips:function(){
            var self = this;
            $.ajax({
                url:"/tip/p/"+this.uuid,
                success:function(tips){
                    if(tips.length){
                        tips.forEach(function(t){
                            var tip = $("<a class='tip-point' data-id='"+ t.id+"'><div class='pulse-inner1'></div><div class='pulse-inner2'></div></a>")
                            tip.css({
                                left: t.page_x-9,
                                top: t.page_y-9,
                                background:getRandomColor()
                            })
                            tip.x = t.page_x;
                            tip.y = t.page_y;
                            tip.id = t.id
                            self.ele.append(tip);
                            tip.on("mouseenter",function(){
                                $(".tip-point").removeClass("active");
                                $(this).addClass("active")
                                self._loadTip(tip.id)
                            }).on("mouseleave",function(){

                            })

                        })

                    }
                }
            })
        },
        _createTip:function(x, y){
            if(this.lastTip&&!this.lastTip.uuid){
                this.lastTip.remove();
            }
            var tip = $("<a class='tip-point'><div class='pulse-inner1'></div><div class='pulse-inner2'></div></a>")
            tip.css({
                left:x-9,
                top:y-9,
                background:getRandomColor()
            })
            tip.x = x;
            tip.y = y;
            this.commentList.html("")
            this._showComments();
            this.container.css({
                left:x,
                top:y
            })
            tip.addClass("active");
            this.commentForm.tipId = null;
            this.ele.append(tip);
            this.lastTip = tip;
        }
    }

这个是css的

    .tip-point{
        background: #c6bf28;
        border:2px solid rgba(255,255,255,1);
        width:12px;
        height:12px;
        display: block;
        opacity: 0.6;
        box-shadow: 0px 0px 4px #333;
        position: absolute;
        border-radius: 20px;
        box-sizing: border-box;
        z-index:1001;
    }
    .tip-point.active{
        z-index:1003;
    }
    .tip-comment-form{
        background: #fff;
        width:300px;
        height:111px;
        position: absolute;
        border:1px solid #ddd;
        box-sizing: border-box;
    }
    .tip-comments {
        position: absolute;
        text-align: left;
        z-index:1002;
    }
    .tip-comments{
        background: #fff;
        width:300px;
        border:1px solid #ddd;
        box-sizing: border-box;
    }
    .tip-comment-form textarea{
        width:100%;
        border:none;
        box-sizing: border-box;
        box-shadow: none;
        height:70px;
        overflow: auto; word-wrap: break-word; resize: none;
    }
    .tip-comment-form button{
        width:100%;
        box-sizing: border-box;
        background: #ccc;
        color:#fff;
        height:30px;
        border:none;
    }
    .tip-comment-form button:hover{
        background: #ddd;
    }
    .tip-comment-form .tip-submit{
        background: #3498DB;
    }
    .tip-comment-form .tip-submit:hover{
        background: #3cb1ff;
    }
    .pulse-inner1,.pulse-inner2 {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        border: 4px solid #eee;
        background-color: transparent;
        box-shadow: 0 0 5px #000;
        position: absolute;
        top: -50%;
        left: -50%;
        -webkit-animation: pulse 2s infinite;
        -ms-animation: pulse 2s infinite;
        animation: pulse 2s infinite;
        transform-origin:center center;
    }

    .pulse-inner2 {
        -webkit-animation-delay: -1s;
        -ms-animation-delay: -1s;
        animation-delay: -1s
    }

    @-webkit-keyframes pulse {
        0% {
            -webkit-transform: scale(0)
        }

        100% {
            -webkit-transform: scale(1);
            opacity: 0
        }
    }

    @-ms-keyframes pulse {
        0% {
            -ms-transform: scale(0)
        }

        100% {
            -webkit-transform: scale(1);
            opacity: 0
        }
    }

    @keyframes pulse {
        0% {
            -webkit-transform: scale(0);
            transform: scale(0)
        }

        100% {
            -webkit-transform: scale(1);
            opacity: 0
        }
    }


    .tip-comments ul.comment-list {
        list-style: none;
        padding: 0;
        margin: 0;
        max-height: 400px;
        overflow: auto;
    }

    .tip-comments ul.comment-list li {
        padding: 10px 20px;
    }

    .tip-comments ul.comment-list li.single-comment {
        border-bottom: 1px solid #eee
    }

    .tip-comments ul.comment-list li.single-comment:last-child {
        border-bottom: 0
    }

    .tip-comments ul.comment-list li.single-comment .comment-body {
        line-height: 1.5;
        word-break: break-all
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta {
        color: #BDC3C7;
        font-size: 11px;
        margin-top: 5px;
        position: relative
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .author-name {
        font-weight: 700;
        display: inline-block;
        vertical-align: top;
        max-width: 100px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .archive {
        opacity: 0
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .archive a {
        cursor: pointer;
        background: #fff
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .archive .delete {
        color: #C0392B
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .archive .delete:hover {
        color: #E74C3C
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .icon-trash {
        width: 11px;
        height: 11px;
        position: absolute;
        right: 0;
        bottom: 0;
        background: #fff;
        -ms-box-shadow: inset 0 0 0 0;
        -o-box-shadow: inset 0 0 0 0;
        box-shadow: 0 0;
        padding: 0;
        cursor: pointer;
        color: #C0392B;
        opacity: 0
    }

    .tip-comments ul.comment-list li.single-comment .comment-meta .icon-trash:hover {
        color: #E74C3C
    }

    .tip-comments ul.comment-list li.single-comment:hover .comment-meta .archive,.tip-comments ul.comment-list li.single-comment:hover .comment-meta .icon-trash {
        opacity: 1
    }




