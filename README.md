# sas
Sas 是一个可以递归的(同/异)步库。
S代表sync/同步 AS代表async/异步。
可以用在[Node.js](http://nodejs.org) 

`npm install sas`

也可用用在前端,支持IE8以上的浏览器.
#如何使用
## 很简单,只要记住:
#对象是异步、数组是同步、函数是任务
##就行。

#快速示例
nodejs全部用异步方法`mkdir`创建目录树。

先创建一个根目录，再在根目录下创建三子目录，再在三子目录下各创建三子目录。
```javascript
var sas = require('sas');
var fs = require('fs');

var mktree = function(path) {
  return function(cb, ext) {
    if (ext.Sparent) {
      path = ext.Sparent[0] + path;
    }
    fs.mkdir(path, 777, function(err, result) {
      if (err) {
        return cb('$STOP');
      }
      cb(path);
    });
  }
}
var plan = [
	mktree(__dirname + '/root' + Date.now()), {
		'1': [mktree('/1'), {
			'1-1': mktree('/1-1'),
			'1-2': mktree('/1-2'),
			'1-3': mktree('/1-3')
		}],
		'2': [mktree('/2'), {
			'2-1': mktree('/2-1'),
			'2-2': mktree('/2-2'),
			'2-3': mktree('/2-3')
		}],
		'3': [mktree('/3'), {
			'3-1': mktree('/3-1'),
			'3-2': mktree('/3-2'),
			'3-3': mktree('/3-3')
		}]
	}
];
sas(plan);
```
#详细说明

## sas只有一个function: 
#sas(arr,opt);
##arr

第一个参数是一个数组,包含三种元素:
- 数组Array:代表同步sync(因为有序)
- 对象Object:代表异步async(因为有key,顺序乱了也没事)
- 函数Function:基本元素。结构为：
```javascript
function(cb,ext){

}
```
###cb 
回调，必须。如果arguments.length<=1的话，当前元素会被替换为result。例：
```javascript
var line;

line = [
  function(cb) {
    setTimeout(function() {
      cb('first');
    }, 200);
  },
  function(cb) {
    setTimeout(function() {
      cb('last');
      console.log(line);
      // line =  ['first', 'last']
    }, 200);
  }
]
sas(line);
```
魔法字$：
`$STOP`: 中止程序
`$END`: 中止this

如果arguments.length>1，当前元素会被替换为一个数组。
例：
```javascript
var sas = require('sas');
var http = require('http');

function httpCreate(cb) {
	http.createServer(cb).listen(1337, '127.0.0.1');
	console.log('Server running at http://127.0.0.1:1337/');
}
function OK(cb) {
	var res= this[0][1];//注意：this指向的不是全局而是当前
	res.end('OK');
}
var line = [
	httpCreate,
	OK
];
sas(line);
```
###ext是一个对象，可选。提供当前元素导航。目前有：

	`index` 当前元素index;
	`path` 是一个数组，包含所有当前元素到root的key和index值。
	`parent` 父元素
	`pIndex` 父元素 index
	`Sparent` 是当前元素第一个同步的父级。如第一个例子：
```javascript
    if (ext.Sparent) {
      path = ext.Sparent[0] + path;
    }
```

---------------------------------------

基本元素若为其它类型而##opt属性iterator不为true的话，会抛出一个错误。

##opt

第二个参数是一个对象,可选：

`debug:bool` 强大的追踪。不管是异步还是同步都能追踪到，默认为false。
第一个的例子:
```javascript
sas(plan,{debug:true});
```
console将会显示log：

![image](https://github.com/hezedu/SomethingBoring/blob/master/sas/saslog.png?raw=true)

其中灰色代表异步，白色代表同步。

`iterator:Function` 返回一个function.结构为：
```javascript
function test(opt){
  return function(cb,ext){
  
  }
}
```
如果 ##arr 里的基本元素type不为function，iterator将会被调用。
第一个例子:
```javascript
var plan = [
	__dirname + '/root' + Date.now(), {
		'1': ['/1', {
			'1-1': '/1-1',
			'1-2': '/1-2',
			'1-3': '/1-3'
		}],
		'2': ['/2', {
			'2-1': '/2-1',
			'2-2': '/2-2',
			'2-3': '/2-3'
		}],
		'3': ['/3', {
			'3-1': '/3-1',
			'3-2': '/3-2',
			'3-3': '/3-3'
		}]
	}
];
sas(plan,{iterator:mktree});
```
将会得到相同的结果。
#其它
如果`sas.debug＝true`,则debug默认为true。
