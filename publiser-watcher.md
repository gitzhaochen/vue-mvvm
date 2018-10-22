``` 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    <input type="text" v-model="text">
    {{text}}
</div>
<script>
    function Watcher(vm,node,name) {
        Dep.target = this;
        this.name = name;
        this.node = node;
        this.vm = vm;
        this.update();
        Dep.target = null;
    }

    Watcher.prototype = {
        update: function(){
            this.get();
            this.node.nodeValue = this.value
        },
        get: function(){
            this.value = this.vm[this.name]//触发相应属性的get，把自己放到dep.subs订阅者数组里
        }
    }

    function compile(node, vm) {
        let reg = /\{\{(.*)\}\}/;
        //节点是元素
        if (node.nodeType === 1) {
            let attrs = [...node.attributes];//类数组转真数组
            attrs.forEach((attr, i, attrs) => {
                let name = attr.nodeValue;
                node.addEventListener('input', function (e) {
                    //出发set方法
                    vm[name] = e.target.value;
                })
                node.value = vm[name];
                node.removeAttribute('v-model');
            })
        }
        //节点是文本
        if (node.nodeType === 3) {
            if (reg.test(node.nodeValue)) {
                let name = RegExp.$1;
                name = name.trim();
                node.nodeValue = vm[name]
                //在编译 HTML 的过程中，会为每个与数据绑定相关的节点生成一个订阅者 watcher，
                new Watcher(vm, node, name)
            }
        }
    }

    function nodeToFragment(node, vm) {
        let flag = document.createDocumentFragment();//创建文档碎片
        let child;
        while (child = node.firstChild) {
            //appendChild 成功后，会把节点从原来的节点位置移除；
            //参考 https://developer.mozilla.org/zh-CN/docs/Web/API/Node/appendChild
            compile(child, vm)
            flag.appendChild(child)
        }
        return flag
    }

    function Dep() {
        this.subs = [];

    }

    Dep.prototype = {
        addSub: function(sub){
            this.subs.push(sub)
        },
        notify: function(){
            this.subs.forEach((sub) => {
                sub.update()
            })
        }
    }

    function defineReactive(vm, key, val) {
        //在监听数据的过程中，会为 data 中的每一个属性生成一个发布者对象 dep
        let dep = new Dep();
        Object.defineProperty(vm, key, {
            get: function () {
                if (Dep.target) {
                    //添加订阅者watcher
                    dep.addSub(Dep.target)
                }
                return val
            },
            set: function (newVal) {
                if (newVal === val) return
                val = newVal
                //作为发布者发出通知
                dep.notify()
            }
        })
    }

    function observe(obj, vm) {
        Object.keys(obj).forEach((key) => {
            //将obj.data属性代理到this上
            defineReactive(vm, key, obj[key])
        })
    }

    function Vue(options) {
        this.data = options.data;
        let data = this.data;
        observe(data, this);
        let id = options.el;
        let dom = nodeToFragment(document.getElementById(id), this);
        document.getElementById(id).appendChild(dom)
    }

    let VM = new Vue({
        el: 'app',
        data: {
            text: 'hello world'
        }
    })
</script>
</body>
</html>
```