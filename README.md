# Web-Front-end-Performance-Optimization2

## js缓存sdk（仅供参考，不一定对的）
``` javascript
window.Xhrfactory = function() {
    this.init.apply(this, arguments);
};

window.Xhrfactory.prototype = {
    init: function() {
        this.xhr = this.create();
    },
    create: function() {
        var xhr = null;
        if (window.XMLHttpRequest) {
            xhr = new XMLHttpRequest();
        } else if (window.ActiveXobject) {
            xhr = new ActiveXobject('Msml2.Xmlhttp');
        } else {
            xhr = new ActiveXobject('Microsoft.Xmlhttp');
        }
        return xhr;
    },
    readystate: function(callback) {
        this.xhr.onreadystatechange = function() {
            if (this.readyState === 4 && this.status === 200) {
                callback(this.responseText);
                console.log(this);
            }
        }
    },
    para: function(data) {
        var datastr = '';
        if (data && Object.prototype.toString.call(data) === "[object object]") {
            for (var i in data) {
                for (var i = 0; i < lenght; i++) {
                    datastr += i + '='
                    data[i] + '&';
                }
            }
        }
    },

    get: function(url, data, callback) {

        this.readystate(callback);
        var newurl = url;
        var datastr = this.para(data);
        newurl = url + '?' + datastr;
        this.xhr.open('get', newurl, true);
        this.xhr.send(null);

    }
};

// 后台程序的模板变量
var localStorageSign = 'on';
// 版本控制
var resourceVersion = '12312443243202';

// 本地的Sdk主方法
window.mLocalSdk = {
    resourceJavascriptList: [{
        id: '1232131241',
        url: '/dest/js/lib/core.js',
        type: 'javascript'
    }, {
        id: '1232131242',
        url: '/dest/js/lib/log.js',
        type: 'javascript'
    }, {
        id: '1232131243',
        url: '/dest/js/lib/report.js',
        type: 'javascript'
    }],
    needUpdate: (function() {
        // 判断后台传过来的版本跟本地存储的版本是否一致
        return localStorage.getItem('resourceVersion') === resourceVersion;
    })(),
    isIE: (function() {
            var v = 3;
            var div = document.createElement('div');
            var all = div.getElementsByTagName('i');
            while (
                div.innerHTML = '<!-- [if gt IE' + (++v) + ']><i></i><![endif] -->', !all[0])
                 { 
                    if(v > 11){return false}
                }
            return v > 3 ? v : false;
        })(),
    checkHedge: function() { // 计算localStorage数量
        var localStorageLength = localStorage.length;  // 
        var localStorageSize = 0;
        for (var i = 0; i < localStorageLength; i++) {
            var key = localStorage.key(i);
            localStorageSize += localStorage.getItem(key).length;
        }
        return localStorageSize;
    },
    saveSdk: function() { // 请求js并保存
        try {
            localStorage.setItem('resourceVersion', resourceVersion);
        } catch (oException) {
            if (oException.name == 'QuotaExceededError') {
                localStorage.clear();
                localStorage.setItem('resourceVersion', resourceVersion);
            }
        }

        for (var i = 0; i < this.resourceJavascriptList.length; i++) {
            _self = this;
            (function(i){
                var scriptId = _self.resourceJavascriptList[i]['id'];
                var xhr = new Xhrfactory();
                xhr.get(_self.resourceJavascriptList[i]['url'], null, function(data) {
                    try {
                        // 如果localStorage满了会报错
                        localStorage.setItem(scriptId, data);
                    } catch (oException) {
                        if (oException.name == 'QuotaExceededError') {  // 如果是存满了溢出错误
                            localStorage.clear();   // 先清除localStorage
                            localStorage.setItem(scriptId, data);  // 再次执行localStorage
                        }
                    }
                });
            })(i);
        }
    },
    startup: function() {
        // 满足一下条件
        var _self = this;
        // 后台通知需要开启缓存localStorageSign === 'on'，不是IE，
        if (localStorageSign === 'on' && !this.isIE && window.localStorage) {

            if (this.needUpdate === true) { // 已经有缓存，并且缓存仍有效
                //不需要更新
                return (function() {
                    
                    for (var i = 0; i < _self.resourceJavascriptList.length; i++) {
                        // 获取本地缓存列表 输入到html上
                        console.log('在循环使用我们自己搞的本地js缓存');
                        var scriptId = _self.resourceJavascriptList[i]['id'];
                        // 把我们的列表中的js文件 渲染到页面

                        // 去读取本地文件
                        window.mDomUtils.addJavascriptByInline(scriptId);
                    }
                })();
                
            } else { // 没有缓存或者缓存已经失效了
                //***
                // 把通过xhr取到的javascript 输入到html上；
                // save localstroage
                // 保存我们请求到的js文件
                return (function() {
                    _self.saveSdk();
                    for (var i = 0; i < _self.resourceJavascriptList.length; i++) {
                        // 获取本地缓存列表 输入到html上
                        var scriptId = _self.resourceJavascriptList[i]['id'];
                        // 把我们的 列表中的js文件 渲染到页面

                        // 去读取本地文件
                        window.mDomUtils.addJavascriptByInline(scriptId);
                    }
                })();

            }
        } else {  // 后台通知不需要进行js缓存时、在IE中、不支持localStorage
            //***
            // 把从网络获取到的javascript 输入到html上；
            // 原始方法(script直接)加载javascriopt
            return function() {
                for (var i = 0; i < resourceJavascriptList.length; i++) {
                    // 获取本地缓存列表 输入到html上
                    var scriptId = resourceJavascriptList[i]['scriptId'];
                    // 把我们的列表中的js文件 渲染到页面

                    // 读取网络上得到的资源
                    window.mDomUtils.addJavascriptByLink(scriptId, resourceJavascriptList[i]['url']);
                }
            }

        }

    }
};

window.mDomUtils = {
    // 内联方式添加javascript
    addJavascriptByInline: function(scriptId) {
        var script = document.createElement('script');
        script.setAttribute('type', 'text/javascript');
        script.id = scriptId;
        var heads = document.getElementsByTagName('head');
        if (heads.lenght) {
            heads[0].appendChild(script);
        } else {
            document.documentElement.appendChild(script);
        }
        script.innerHTML = localStorage.getItem(scriptId);
    },


    // 外链方式添加javascript
    addJavascriptByLink: function(scriptId, url) {
        var script = document.createElemet('script');
        script.setAttribute('type', 'text/javascript');
        script.setAttribute('src', url);
        script.id = scriptId;
        var heads = document.getElementsByTagName('head');
        if (heads.length) {
            heads[0].appendChild(script);
        } else {
            document.documentElement.appendChild(script);
        }
    },


    // 外链方式添加css

    addCssByLink: function(url) {
        var doc = document;
        var link = doc.createElemet('link');
        link.setAttribute('type', 'text/css');
        link.setAttribute('rel', 'stylesheet');
        link.setAttribute('href', url);
        var heads = doc.getElementsByTagName('head');
        if (heads.length) {
            heads[0].appendChild(link);
        } else {
            doc.documentElement.appendChild(link);
        }
    },

    // 外链方式添加css

        addCssByLink: function(cssString) {
        var doc = document;
        var link = doc.createElemet('link');
        link.setAttribute('type', 'text/css');
        link.setAttribute('rel', 'stylesheet');

        if (link.stylesheet) {
            // IE支持
            link.stylesheet.cssText = cssString;
        } else {
            // w3c
            var cssText = doc.createTextNode(cssString);
            link.appendChild(cssText);
        }

        var heads = doc.getElementsByTagName('head');
        if (heads.length) {
            heads[0].appendChild(link);
        } else {
            doc.documentElement.appendChild(link);
        }
    }
};
```
## LS缓存方案的应用场景
> 1、非首屏渲染需要的css文件，可以做LS缓存。
>> 首屏渲染需要的css，需要按常规方式输出，因为SEO需要，不然爬虫爬取页面的时候，页面效果会很不好。而非首屏的css，则可以用LS缓存，减少资源下载时间。
> 
> 2、展示类、动画类等非业务主要逻辑的代码，可以做LS缓存。
>> 这样，可以一定程度上避免业务层的安全漏洞。当然，前端再怎么做防护都是一层薄纸。重要的，还是后台接口要做好安全保护。
>
> 3、 移动端可以做LS缓存。PC端做LS缓存，起到的优化作用不大。


## LS缓存方案的缺点
* xss攻击问题
* 首页的css缓存会影响SEO
* localStorage以页面的域名划分，而常见的静态资源都以资源本身的域名来缓存，意味着如果你的应用有多个等价域名，它们之间的localStorage不互通，会造成缓存多份浪费。
* 兼容性需要处理（不支持、隐私模式、写满、http/https、写的正确性等等）
* localStorage的key、value有字符数字的一个限制。大概在9800左右。


## localStorage的容量
* 根据目前的市场上浏览器对loaclStorage的兼容性来看，最佳的大小是2.5M左右，可以通过[此网站](http://dev-test.nemikor.com/web-storage/support-test/)进行计算查看详情的兼容性问题。
* localStorage 中存储的是字符串，根据这一条件，我们可以通过取出所有的localStorage的内容，而其长度就是大小。
* [点击此网站](https://arty.name/localstorage.html)可以计算出最大的容量，原理：通过先清空localStorage，然后不停的往里面写数据，直至报错。


## 腾讯的字符级别的缓存
> 可以到这个[页面](http://code.csdn.net/news/2820563)体会下腾讯的字符级别的更新方案