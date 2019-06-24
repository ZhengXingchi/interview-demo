## 方图链
1. websocket心跳机制以及异常捕获
[初探和实现websocket心跳重连(npm: websocket-heartbeat-js)](https://www.cnblogs.com/1wen/p/5808276.html)
[zimv/websocket-heartbeat-js](https://github.com/zimv/websocket-heartbeat-js)
```
/**
 * `WebsocketHeartbeatJs` constructor.
 *
 * @param {Object} opts
 * {
 *  url                  websocket链接地址
 *  pingTimeout 未收到消息多少秒之后发送ping请求，默认15000毫秒
    pongTimeout  发送ping之后，未收到消息超时时间，默认10000毫秒
    reconnectTimeout
    pingMsg
 * }
 * @api public
 */

function WebsocketHeartbeatJs({
    url, 
    pingTimeout = 15000,
    pongTimeout = 10000,
    reconnectTimeout = 2000,
    pingMsg = 'heartbeat',
    repeatLimit = null
}){
    this.opts ={
        url: url,
        pingTimeout: pingTimeout,
        pongTimeout: pongTimeout,
        reconnectTimeout: reconnectTimeout,
        pingMsg: pingMsg,
        repeatLimit: repeatLimit
    };
    this.ws = null;//websocket实例
    this.repeat = 0;

    //override hook function
    this.onclose = () => {};
    this.onerror = () => {};
    this.onopen = () => {};
    this.onmessage = () => {};
    this.onreconnect = () => {};

    this.createWebSocket();
}
WebsocketHeartbeatJs.prototype.createWebSocket = function(){
    try {
        this.ws = new WebSocket(this.opts.url);
        this.initEventHandle();
    } catch (e) {
        this.reconnect();
        throw e;
    }     
};

WebsocketHeartbeatJs.prototype.initEventHandle = function(){
    this.ws.onclose = () => {
        this.onclose();
        this.reconnect();
    };
    this.ws.onerror = () => {
        this.onerror();
        this.reconnect();
    };
    this.ws.onopen = () => {
        this.repeat = 0;
        this.onopen();
        //心跳检测重置
        this.heartCheck();
    };
    this.ws.onmessage = (event) => {
        this.onmessage(event);
        //如果获取到消息，心跳检测重置
        //拿到任何消息都说明当前连接是正常的
        this.heartCheck();
    };
};

WebsocketHeartbeatJs.prototype.reconnect = function(){
    if(this.opts.repeatLimit>0 && this.opts.repeatLimit <= this.repeat) return;//limit repeat the number
    if(this.lockReconnect || this.forbidReconnect) return;
    this.lockReconnect = true;
    this.repeat++;//必须在lockReconnect之后，避免进行无效计数
    this.onreconnect();
    //没连接上会一直重连，设置延迟避免请求过多
    setTimeout(() => {
        this.createWebSocket();
        this.lockReconnect = false;
    }, this.opts.reconnectTimeout);
};
WebsocketHeartbeatJs.prototype.send = function(msg){
    this.ws.send(msg);
};
//心跳检测
WebsocketHeartbeatJs.prototype.heartCheck = function(){
    this.heartReset();
    this.heartStart();
};
WebsocketHeartbeatJs.prototype.heartStart = function(){
    if(this.forbidReconnect) return;//不再重连就不再执行心跳
    this.pingTimeoutId = setTimeout(() => {
        //这里发送一个心跳，后端收到后，返回一个心跳消息，
        //onmessage拿到返回的心跳就说明连接正常
        this.ws.send(this.opts.pingMsg);
        //如果超过一定时间还没重置，说明后端主动断开了
        this.pongTimeoutId = setTimeout(() => {
            //如果onclose会执行reconnect，我们执行ws.close()就行了.如果直接执行reconnect 会触发onclose导致重连两次
            this.ws.close();
        }, this.opts.pongTimeout);
    }, this.opts.pingTimeout);
};
WebsocketHeartbeatJs.prototype.heartReset = function(){
    clearTimeout(this.pingTimeoutId);
    clearTimeout(this.pongTimeoutId);
};
WebsocketHeartbeatJs.prototype.close = function(){
    //如果手动关闭连接，不再重连
    this.forbidReconnect = true;
    this.heartReset();
    this.ws.close();
};
if(window) window.WebsocketHeartbeatJs = WebsocketHeartbeatJs;
export default WebsocketHeartbeatJs;

```








2. react实现动画效果

3. echarts的使用


## 依图医疗
1. 防抖实现
```
function debounce(fn,delay){
  let timer=null;
  return function(){
  	let args=Array.prototype.slice.call(arguments)
  	let context=this
  	clearTimeout(timer)
  	timer=setTimeout(()=>{
      fn.apply(context,args)
  	},delay)
  }
}

```


2. 节流实现

setTimeout的方式
```
function throttle(func,delay){
	let timer=null;
	return function(){
		let args=Array.prototype.slice.call(arguments)
  	    let context=this
  	    if(!timer){
  	      setTimeout(()=>{
			func.apply(context,args)
			timer=null
		  },delay)
  	    }
		
	}

}

```

时间戳
```
function throttle(func,delay){
	let prev=Date.now();
	return function(){
		let args=Array.prototype.slice.call(arguments)
  	    let context=this
  	    let now=Date.now()
  	    if(now-prev>=delay){       
			func.apply(context,args)
			prev=Date.now();		   
  	    }		
	}
}

```


3. 综合使用时间戳与定时器，完成一个事件触发时立即执行，触发完毕还能执行一次的节流函数
```
function throttle(func, delay){
  let timer = null;
  let startTime = Date.now();

  return function(){
    let curTime = Date.now();
    let remaining = delay - (curTime - startTime);
    const context = this;
    const args = arguments;

    clearTimeout(timer);
    if(remaining <= 0){
      func.apply(context,args);
      startTime = Date.now();
    }else{
      timer = setTimeout(func, remaining);
    }
  }
}
 

```



4. 为什么diff是n3时间复杂度，react的diff采用的是广度优先还是深度优先检索


5. 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。
参考[剑指Offer系列刷题笔记汇总](https://cuijiahua.com/blog/2018/02/basis_67.html)
```
class list{
	constructor(){
		this.stack1=new Stack()
        this.stack2=new Stack()
	}
  push(item){
    this.stack1.push(item)
  }
  pop(){
  	if(this.stack2.empty()){
      while(this.stack1.size()>0){
      	this.stack2.push(stack1.pop())
      }
      return this.stack2.pop()
  	}
  }
}
```


6. EventLoop
参考[从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://segmentfault.com/a/1190000012925872?utm_source=tag-newest)
[js|setTimeOut,Promise面试题集合，详解js事件机制EventLoop](https://blog.csdn.net/weixin_28900307/article/details/88603745)

```
setTimeout(() => {
   console.log(1);

   Promise.resolve().then(() => {
     console.log(4);
   });
 });
    
 Promise.resolve().then(() => {
   console.log(2);

   setTimeout(() => {
     console.log(3);
   });
 });

```
请输出上面的执行结果：...
浏览器的输出结果：2 1 4 3，
node的输出结果为： 2 1 3 4
[关于Promise和setTimeout的执行顺序，感觉自己理解的eventloop都是错的](https://segmentfault.com/q/1010000017555410)


7. 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

```
//异或符合交换律
function getnum(arr){
  let result=0;
  for(let i=0;i<arr.length;i++){
  	result=result^arr[i]
  }
  return result
}

```

```
function getNum(arr){
	let map=new Map()
	for(let i=0;i<arr.length;i++){
		if(map.has(arr[i])){
			let temp=map.get(arr[i])
			map.set(arr[i],temp++)
		}else{
			map.set(arr[i],1)
		}
	}
	for(let key of map.keys()){
       if(map.get(key)==1){
         return map.get(key)
       }
	}

}

```

复杂版本
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字
参考[ECMAScript 位运算符](http://www.w3school.com.cn/js/pro_js_operators_bitwise.asp)
[一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字](https://www.cnblogs.com/hezhiyao/p/7539024.html)
```
function getNum(arr){
	if(arr.length<2){
		return arr
	}
	let result=[0,0]
	let temp=0
    for(let i=0;i<arr.length;i++){
      temp=temp^arr[0]
    }
    let index=0
    for(let j=0;j<32;j++){
    	if(temp>>i & 1==1){
    		index=j
    		break;
    	}
    }
    for(let k=0;k<arr.length;k++){
    	if(arr[k]>>index & 1==0){
    		result[0]=result[0]^arr[k])
    	}else[
    		result[1]=result[1]^arr[k]
    	]
    }
}
```

8. 检索一段话中出现次数最多的英文单词
参考[Java——检索一段话中出现次数最多的英文单词](https://blog.csdn.net/u012325167/article/details/50884373)

```
function answer(str){
    str = str.replace('\'', ' ');//将'号用空格替换
    str = str.replace(',', ' ');//将逗号用空格替换
    str = str.replace('.', ' ');//将句号用空格替换
    let arr=str.split("\\s+");
    let map=new Map()
    for(let i=0;i<arr.length;i++){
       if(map.has(arr[i])){
         let temp=map.get(arr[i])
         temp++
         map.set(arr[i],temp)
       }else{
       	  map.set(arr[i],1)
       }
    }
    let max=0
    let maxStr=null
    for(let key of map.keys()){
      if(map.get(key)>max){
      	max=map.get(key)
      	maxStr=key
      }
    }
}



```




## 如程
```
function test1(){
	var a=10
  {
   var a=20
   console.log(a)
  }
  console.log(a)
}

function test2(){
  let a=10
  {
   let a=20
   console.log(a)
  }
  console.log(a)
}
test1()
test2()

输出
20
20
20
10
```





## 数澜科技
1. vue与react区别





2. webpack与gulp区别，webpack打包实践



## 企朋
1. es6中const定义的属性是否可以改变_为什么有人说const并非一定为常量
const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。
对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。


## 其他
1. http的长连接与短连接
[HTTP长连接、短连接究竟是什么？](https://www.cnblogs.com/gotodsp/p/6366163.html)
[WebSocket 和 HTTP 的区别及原理](https://www.v2ex.com/amp/t/473376)
HTTP的长连接指的是传输层TCP不断开,但是还是需要发起request/response





2. 从输入url到界面显示经历了哪些过程



3. 三栏布局
display:inline-block并且在父节点设置font-size：0；然后在子节点重置font-size
也可以将子节点间的间隙注释比如
```html
<div class="wrapper">
     <p></p><!--
  --><p></p><!--
  --><p></p>
</div>

```


4. 对象存储OSS





## 小牛哥整装家装
1. react的mixin  HOC HOOK

[React中使用swiper的具体方法](https://www.jianshu.com/p/341b48019ad7)
[useState源码浅析](https://juejin.im/post/5c8f4b54f265da61103b3cae)
[你要的 React 面试知识点，都在这了](https://juejin.im/post/5cf0733de51d4510803ce34e)

[react 16.7 hook概述](https://www.jianshu.com/p/e61faf452565)
[TIL/front-end/react/hooks/intro.md](https://github.com/xiaohesong/TIL/blob/master/front-end/react/hooks/intro.md)


2. 受控表单和非受控表单
参考[Controlled and uncontrolled form inputs in React don't have to be complicated](https://goshakkk.name/controlled-vs-uncontrolled-inputs-react/)
[表单](http://caibaojian.com/react/forms.html)
非受控表单
```
class Form extends Component {
  handleSubmitClick = () => {
    const name = this._name.value;
    // do something with `name`
  }

  render() {
    return (
      <div>
        <input type="text" ref={input => this._name = input} />
        <button onClick={this.handleSubmitClick}>Sign up</button>
      </div>
    );
  }
}
```

受控表单
```
class Form extends Component {
  constructor() {
    super();
    this.state = {
      name: '',
    };
  }

  handleNameChange = (event) => {
    this.setState({ name: event.target.value });
  };

  render() {
    return (
      <div>
        <input
          type="text"
          value={this.state.name}
          onChange={this.handleNameChange}
        />
      </div>
    );
  }
}
```

