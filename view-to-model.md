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

    function defineReactive(vm, key, val) {
        Object.defineProperty(vm, key, {
            get: function () {
                return val
            },
            set: function (newVal) {
                if (newVal === val) return
                val = newVal
                console.log(val)
            }
        })
    }

    function observe(obj, vm) {
        Object.keys(obj).forEach((key) => {
            //将obj.data属性代理到this上
            // defineReactive(vm, key, obj[key])
            Object.defineProperty(vm, key, {
                get: function () {
                    return obj[key]
                },
                set: function (newVal) {
                    if (newVal === obj[key]) return
                    obj[key] = newVal
                    console.log(obj[key])
                }
            })
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