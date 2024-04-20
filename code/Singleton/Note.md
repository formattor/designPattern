# Singleton Mode

## 定义

一个类存在只存在一个实例，并提供一个访问它的全局访问点

## 实现单例模式

------

### 以构造函数属性的方式创建标志位

1. 以function的方式创建单例构造函数（类）
2. 在构造函数上设置标识符表示有没有创建过实例
3. 在构造函数上创建方法，生成实例（单例逻辑，标志位通过this调用）

<details>
  <summary>相关代码</summary>
    <blockcode> 
        var Singleton = function (name) {
            this.name = name;
        }

        Singleton.instance = null;
        Singleton.createSingleton = function (name) {
            if (!this.instance) {
                this.instance = new Singleton(name)
            }
            return this.instance;
        }
    
        var s1 = Singleton.createSingleton('张与时1')
        var s2 = Singleton.createSingleton('张与时2')
    
        console.log(s1 === s2); // true
</details>


### 以自执行函数+闭包的方式创建标志位

1. 以function的方式创建单例构造函数（类）
2. 在构造函数上创建方法
    + 通过自执行函数在函数声明时将标志位产生闭包保存
    + 返回实例函数

<details>
  <summary>相关代码</summary>
    <blockcode> 

        var closureSingleton = function (name) {
            this.name = name;
        }
    
        closureSingleton.createSingleton = (function () {
            var instance = null;
            return function (name) {
                if (!instance) {
                    instance = new closureSingleton(name)
                }
                return instance;
            }
        })()
    
        var s1 = closureSingleton.createSingleton('张与时')
        var s3 = closureSingleton.createSingleton('张与时3')
        console.log(s1 === s3);
</details>

### 缺陷

不透明：必须以`Class.property`的方式调取

## 通过new来创建单例（透明）

------

### 创建

1. 将构造函数变为自执行函数，直接生成闭包标志位
2. 在构造函数的方法中创建创建实例的方法
    + 包括单例逻辑和初始化逻辑（违反开闭原则）
    + 通过return instance = this的方法生成实例
3. 在构造函数的原型上构造初始化方法
4. 返回方法（方法中返回实例）

<details>
  <summary>相关代码</summary>
    <blockcode> 

            var createDiv = (function () {
            var instance = null;
            var createDDD = function (html) {
                if (instance) {
                    return instance
                }
                this.html = html;
                this.init();
                return instance = this;
            }
    
            createDDD.prototype.init = function () {
                var div = document.createElement('div');
                div.innerHTML = this.html;
                document.body.appendChild(div);
            }
    
            return createDDD;
        })()
    
        var s1 = new createDiv('sss1')
        var s2 = new createDiv('sss2')
    
        console.log(s1 === s2, s1, s2);
</details>

### 缺点
1. 构造函数同时包含初始化和闭包，违反`单一职责原则`
2. 如果不需要使之成为单例类，要修改控制单例的代码

## 通过代理实现单例模式

------

### 实现

1. 构造一个普通的方法类，在其中写入属性如初始化方法
2. 在原型上定义初始化方法
3. 构造代理类，闭包+自执行函数实现单例逻辑（缓存代理）

<details>
  <summary>相关代码</summary>
    <blockcode> 

        var CreateDDD = function (html) {
            this.html = html;
            this.init();
        }
    
        CreateDDD.prototype.init = function () {
            var div = document.createElement('div');
            div.innerHTML = this.html;
            document.body.appendChild(div);
        }
    
        var proxyCreateDDD = (function () {
            var instance = null;
            return function (html) {
                if (!instance) {
                    instance = new CreateDDD(html)
                }
                return instance;
            }
        })()
    
        let s1 = new proxyCreateDDD('zzz1')
        let s2 = new proxyCreateDDD('zzz2')
        console.log(s1, s2, s1 === s2);
</details>

### 缺点
代码运行的时候直接生成单例，并非随调随用

## 惰性单例

------

### 实现

1. 需要标志位（闭包）
2. 不需要自执行生成实例对象（仅自执行生成标志位，返回实例对象方法）
3. 根据返回的方法新建实例对象，随调随用

<details>
  <summary>相关代码</summary>
    <blockcode> 

    <button id="login">登录</button>
    <script>
        // var loginLayer = (function () {
        //     var div = document.createElement('div')
        //     div.innerHTML = '登录浮窗'
        //     div.style.display = 'none'
        //     document.body.appendChild(div)
        //     return div
        // })();
    
        // document.getElementById('login').onclick = function () {
        //     loginLayer.style.display = 'block';
        // }
    
        var CreateLoginLayer = (function () {
            var div;
            return function () {
                if (!div) {
                    div = document.createElement('div')
                    div.innerHTML = '登录浮窗'
                    div.style.display = 'none'
                    document.body.appendChild(div)
                }
                return div
            }
        })()
    
        document.getElementById('login').onclick = function () {
            var loginLayer = CreateLoginLayer();
            loginLayer.style.display = 'block';
        };
    </script>
</details>

### 缺点
违反开闭原则

## 通用惰性单例

---

### 实现

1. 单例逻辑 - 返回单例方法
2. 构造函数 - 返回实例
3. 单例逻辑函数（构造函数） - 返回单例方法
4. 调用单例方法，赋值给实例

<details>
  <summary>相关代码</summary>
    <blockcode> 

    <button id="login">登录</button>
    <script>
        var getSingle = function (initFn) {
            var instance;
            return function () {
                return instance || (instance = initFn.apply(this, arguments))
            }
        }
    
        var createLoginLayer = function () {
            var div = document.createElement('div');
            div.innerHTML = '登录页面'
            div.style.display = 'none';
            document.body.appendChild(div)
            return div
        }
    
        var loginInstance = getSingle(createLoginLayer);
        document.getElementById('login').onclick = function () {
            var login = loginInstance();
            login.style.display = 'block'
        }
    </script>
</details>