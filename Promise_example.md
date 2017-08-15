title: promise 案例
speaker: 毕文清
url: https://merfais.github.io/nodeppt/promise.htmnl
transition: cards
files: /js/demo.js,/css/demo.css

[slide]
## 回调金字塔
``` javascript
step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // Do something with value4
            });
        });
    });
});
```
``` javascript
Promise.resolve(step1(data))
.then(data => {
  return step2(data);
}).then(data => {
  return step3(data);
}).then(data =>{
  return step4(data);
})
```


[slide]
# 回调迷宫 [1]
``` javascript
function getURLCallback(URL, callback) {
    var req = new XMLHttpRequest();
    req.open('GET', URL, true);
    req.onload = function () {
        if (req.status === 200) {
            callback(null, req.responseText);
        } else {
            callback(new Error(req.statusText), req.response);
        }
    };
    req.onerror = function () {
        callback(new Error(req.statusText));
    };
    req.send();
}
// <1> 对JSON数据进行安全的解析
function jsonParse(callback, error, value) {
    if (error) {
        callback(error, value);
    } else {
        try {
            var result = JSON.parse(value);
            callback(null, result);
        } catch (e) {
            callback(e, value);
        }
    }
}
// <2> 发送XHR请求
var request = {
        comment: function getComment(callback) {
            return getURLCallback('http://yunshan.net.cn', jsonParse.bind(null, callback));
        },
        people: function getPeople(callback) {
            return getURLCallback('http://yunshan.net.cn', jsonParse.bind(null, callback));
        }
    };
// <3> 启动多个XHR请求，当所有请求返回时调用callback
function allRequest(requests, callback, results) {
    if (requests.length === 0) {
        return callback(null, results);
    }
    var req = requests.shift();
    req(function (error, value) {
        if (error) {
            callback(error, value);
        } else {
            results.push(value);
            // 这里使用了 递归
            allRequest(requests, callback, results);
        }
    });
}
function main(callback) {
    allRequest([request.comment, request.people], callback, []);
}
// 运行的例子
main(function(error, results){
    if(error){
        return console.error(error);
    }
    console.log(results);
});
```

[slide]
# then 流式控制 [0]
``` javascript
function getURL(URL) {
    return new Promise(function (resolve, reject) {
        var req = new XMLHttpRequest();
        req.open('GET', URL, true);
        req.onload = function () {
            if (req.status === 200) {
                resolve(req.responseText);
            } else {
                reject(new Error(req.statusText));
            }
        };
        req.onerror = function () {
            reject(new Error(req.statusText));
        };
        req.send();
    });
}
var request = {
        comment: function getComment() {
            return getURL('http://yunshan.net.cn').then(JSON.parse);
        },
        people: function getPeople() {
            return getURL('http://yunshan.net.cn').then(JSON.parse);
        }
    };
function main() {
    return Promise.all([request.comment(), request.people()]);
}
// 运行示例
main().then(function (value) {
    console.log(value);
}).catch(function(error){
    console.log(error);
});
```

[slide]
# 高质量的回调
```javascript
const drawImg = function(draw, imgType) {
    return function(options) {
        const id = options.id;
        let chart;
        return {
            loadData: function() {
                return function(data, newOptions) {
                    if(newOptions) {
                        $.extend(true, options, newOptions);
                    }
                    if(!chart) {
                        options.width = parseInt($('#' + id).width());
                        options.height = parseInt($('#' + id).height());
                        const plotCfg = options.plotCfg ?
                            _.defaults(options.plotCfg, global.plotCfg) :
                            _.clone(global.plotCfg);
                        const forceFit = options.forceFit ? options.forceFit : global.forceFit;
                        if(imgType === 'pie') {
                            // 对于pie图的plotCfg,legend需要单独设置
                            plotCfg.margin = [5, 0, 80, 0];
                            const tmpLegend = {
                                dy: 10,
                                spacingY: 1,
                                title: null,
                                itemWrap: true,
                                position: 'buttom',
                                word: {
                                    // 'font-size': 1,
                                    fill: '#999'
                                }
                            };
                            options.legend = options.legend ?
                                _.defaults(tmpLegend, global.legend, options.legend) :
                                _.defaults(tmpLegend, global.legend);
                        }
                        chart = new G2.Chart({
                            id: id,
                            width: options.width,
                            height: options.height,
                            plotCfg: plotCfg,
                            forceFit: forceFit
                        });
                    }
                    preStep(chart, data, options);
                    draw(chart, data, options);
                    chart.render();
                };
            },
            remove: function() {
                if(chart) {
                    chart.destroy();
                }
            }
        };
    };
};
module.exports = {
    pie: drawImg(function(chart, data, options) {
        data = _.sortBy(data, 'TIMESTAMP');
        chart.source(data, formatDefs(data, options.defs, options.position, false));
        chart.coord('theta', {
            radius: options.radius || 1
        });
        chart.intervalStack().position(G2.Stat.summary.percent(options.position)).color(options.color);
    }, 'pie'),
};
```

[slide] 
# then 的不同格式
```javascript
var promise = Promise.resolve('success message');
promise.then(function (data) {
    console.log(data);
});
```
```javascript
// 一般用于调试或测试时，抛出错误
var promise = Promise.reject(new Error("error message"));
// 相当于 promise.catch(error => {})
promise.then(undefined, function (error) {
    console.error(error);
});
```
```javascript
var promise = Promise.resolve('blablabla');
promise.then(data => {
    console.log(data);
}, error => {
    console.error(error);
});
```
[slide]
# 永远不要忘了return [1]
```javascript
// 同步操作，怎么都好说
var promise = Promise.resolve(1);
promise.then(data => {
    console.log('need 1 =>', data);
    return 2;
}).then(data => {
    console.log('need 2 =>', data);
}).then(data => {
    console.log('is undefined =>', data);
})
```
[slide]
# 永远不要忘了return [0]
```javascript
// 异步操作
function test02(count) {
  // 要return promise 实例
  return new Promise((resolve, reject) => {
    setTimeout(()=>{
      resolve(count)
    },1000);
  });
}
test02(1).then(data => {
    console.log('need 1 =>', data);
    // 如果这里面还是个异步的
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
          resolve(2)
        },1000);
    });
}).then(data => {
    console.log('need 2 =>', data);
    // 如果这里面还是个异步的，封装成函数
    // 这个return 非常重要
    return test02(3);
}).then(data => {
    console.log('need 3 =>', data);
})
```
[slide]
# 每次`.then`都会返回全新的promise  [1]
```javascript
// 1: 对同一个promise对象同时调用 `then` 方法
var aPromise = new Promise(function (resolve) {
    resolve(100);
});
aPromise.then(function (value) {
    return value * 2;
});
aPromise.then(function (value) {
    return value * 2;
});
aPromise.then(function (value) {
    console.log("1: " + value); // => 100
})
// 包括但不限于使用`forEach`循环调用，都是错误的
```
[slide]
# 每次`.then`都会返回全新的promise [0]
```javascript
function badAsyncCall() {
    var promise = Promise.resolve();
    promise.then(function() {
        // 任意处理
        return newVar;
    });
    // 不要觉得 return 了就OK，这是错误的
    return promise;
}
```
```javascript
// 这是正确的
function anAsyncCall() {
    var promise = Promise.resolve();
    return promise.then(function() {
        // 任意处理
        return newVar;
    });
}
```

[slide]
# Promise Chain 中的异步操作
```javascript
Promise.resolve(1).then(data => {
    console.log('need 1 =>', data);
    // 异步事件，程序并不会等待return，所以在异步后return使无法实现的
    // 控制异步事件，只能通过return 一个 pending状态的promise
    // 在异步逻辑后 resolve 或 reject 使promise进入 Settled 状态
    // 使后面的chain继续下去
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
          resolve(2)
          // return 2; 这是无法实现流程控制的
        },1000);
    });
}).then(data => {
    console.log('need 2 =>', data);
    // 或者封装成函数，函数需要返回也是 pending的promise
    return request(3);
}).then(data => {
    console.log('need 3 =>', data);
});

// 异步操作
function request(options) {
  // 要return promise 实例
  return new Promise((resolve, reject) => {
    // 先请求一次数据
    Vue.http(options).then(data1 => {
        // 处理数据，再请求
        var opt = doSth(data1, options);
        Vue.http(opt).then(data2 => {
            // 需要计算两次请求的数据
            resolve(data1 + data2)
        })
    })
  });
}
```

[slide]
# 使用 try{}catch(e){reject(e)} [3]
```javascript
function test01(options) {
  return new Promise((resolve, reject) => {
     // 使用未声明的变量
     if (nononon) {}
     // 使用 throw new Error
     // throw new Error('oops !!!! ')
     setTimeout(()=>{
      resolve('1')
    },3000);
  });
}

test01().then(data => {
    console.log(data);
}).catch(err => {
    console.log(err);
})
```

[slide]
# 使用 try{}catch(e){reject(e)} [2]
```javascript
function test02(options) {
  return new Promise((resolve, reject) => {
    setTimeout(()=>{
      resolve('2')
    },3000);
  });
}

test02().then(data => {
    //if(nonono) {}
    throw new Error('oops !!!! ')
    console.log(data);
}).catch(err => {
    console.log(err);
})
```

[slide]
# 使用 `try{}catch(e){reject(e)}` [1]
> 因为 new Promise() 创建的Promise处于`pending`状态
> 
> 只用 `settled` 状态 才会进入 promise chain 的流程

[slide]
# 使用 try{}catch(e){reject(e)} [0]
```javascript
// 正确做法
function test01(options) {
  return new Promise((resolve, reject) => {
    try {
        if (nononon) {}
        setTimeout(()=>{
          resolve('1')
        },3000);
    }catch(err){
        reject(err);
    }
  });
}

test01().then(data => {
    console.log(data);
}).catch(err => {
    console.log(err);
})
```

[slide]
即使已经`resolve` or `reject`，但后面的代码依旧我行我素
```javascript
function test02() {
  let pms = new Promise((resolve, reject) => {
    console.log('测试二');
    console.log('测试二延时开始,1秒钟');
    setTimeout(()=>{
      console.log('测试二延时结束，1秒钟');
      resolve('测试二 resolve 时刻')
      console.log('测试二即使resolve了后面的依旧会执行')
    },1000);
    console.log('测试二执行了setTimeout 顺后面的输出')
  });
  return pms;
}
test02().then(data => {
    console.log('测试到第一个then data = ', data)
})
```

[slide]
# Promise Chain也不会停下前进的脚步
```javascript
function test03() {
  let pms = new Promise((resolve, reject) => {
    setTimeout(()=>{
      resolve(3)
    },1000);
  });
  return pms;
}
test03().then(data =>{
  console.log('抛出异常')
  return Promise.reject('我是异常');
}).catch(err => {
  console.log('接住异常 =》', err)
  console.log('我想后面不处理里，就不return了')
}).then(data => {
  console.log('但是我还执行到了这里');
  console.log('由于没有return 所以 data =>', data );
  console.log('第二次抛出异常')
  return Promise.reject('还是我，异常！！！')
})catch(err => {
  console.log('抓到异常 =》', err)
  console.log('这次我return异常')
  return err;
}).then(data=>{
  console.log('我也收到了异常的数据',data)
})
```
+  因为 {:&.bounceIn}
    > 本质上 resolve reject 只是回调函数
    > 
    > 不同控制流程的 中断，取消 和 跳转

[slide]
# Pending 到 Settled 只有一次机会
```javascript
function test03() {
  const pms = new Promise((resolve, reject) => {
    console.log('测试三');
    console.log('测试三延时开始,2秒钟');
    setTimeout(()=>{
      console.log('测试三延时结束，2秒钟');
      // 我这里resolve了
      resolve('测试三 resolve 时刻')
      console.log('测试三即使resolve了后面的依旧会执行')
      // 这里又 reject
      reject('测试三，那么我又reject了')
      //setTimeout(()=>{
      //  reject('我延时了总可以吧')
      //},1000);
      console.log('测试三即使reject了后面的依旧会执行')
    },2000);
    console.log('测试三执行了setTimeout 顺序后面的输出')
  });
  return pms;
}
test03().then(data => {
    console.log('我收到的data是', data)
}).catch( err => {
    console.log('我收到的error是', err)
})

// 所以 resolve 与 reject 必须放到不同的分支中处理 
```
[slide]
# `.catch` 要比 `.then(null,onReject)` 好用一万倍
```javascript
Promise.resolve('1').then(data =>{
  console.log('1 通过 期望 1 =》',data)
  return Promise.reject('2');
}).catch(err => {
  console.log('2 拒绝 期望 2=》',err)
  return 3;
}).then(data => {
  console.log('3 通过 期望 3=》',data)
  return Promise.reject(4);
}, err => {
  console.log('4 拒绝 期望 4 =》', err);
  return 5;
}).then(data => {
  console.log('5 通过 期望 5 =》', data);
  return Promise.reject(6);
}, err => {
  console.log('6 拒绝 期望 6 =》', err)
  return 7;
}).then(data => {
  console.log('7 通过 期望 7 =》', data);
  return 8
}).then(data => {
  console.log('8 通过 期望 8 =》', data);
  return Promise.reject(9)
}).then(data => {
  console.log('这里到不了')
}, err => {
  console.log('onreject 响应了', err)
}).catch(err => {
  console.log('catch 响应了', err)
}).then(() => {
   // 如果在这里 error了
   if(l){}
}).then(data => {
  console.log('这里到不了')
}, err => {
  console.log('onreject 响应了', err)
}).catch(err => {
  console.log('catch 响应了', err)
})

```
[slide]
# 对Promise的hack [1]
```javascript
const STOP_SIGNAL = {};
Object.defineProperty(Promise, 'stop', {
  writable: false,
  enumerable: false,
  configurable: true,
  value: function(data) {
    return Promise.resolve({
      signal: STOP_SIGNAL,
      value: data,
    });
  }
})

Promise.prototype.next = function(onResolved, onRejected) {
  return this.then(data => {
    if (data && data.signal && data.signal === STOP_SIGNAL) {
      return data;
    } else {
      return onResolved(data);
    }
  }, onRejected);
}

Promise.prototype.whenStop = function(onResolved, onRejected) {
  return this.then(data => {
    if (data && data.signal && data.signal === STOP_SIGNAL) {
      return onResolved(data.value)
    } else {
      return data;
    }
  }, onRejected);
}
```
[slide]
# 对Promise的hack [0]
```javascript
function test02() {
  let pms = new Promise((resolve, reject) => {
    try {
      setTimeout(() => {
        resolve('1')
      }, 1000);
    } catch (err) {
      reject(err);
    }
  });
  return pms;
}


function test03() {
  return new Promise((resolve, reject) => {
    try {
      setTimeout(() => {
        resolve(3);
      }, 2000)
    } catch(err) {
      reject(err);
    }
    
  });
}

function test04() {
  return Promise.resolve().then(() => {
    return test02().next(data => {
      console.log('need 1', data)
      return 2;
    }).catch(err => {
      console.log('err 2 is', err)
      return Promise.stop('err 2');
    }).next(data => {
      console.log('need 2', data)
      //  throw new Error('oops no !!!')
      return test03().then(data => {
        console.log('need 3', data);
        return 4;
      }).catch(err => {
        console.log('err 4 is', err)
        return Promise.stop('err 4')
      }).next(data => {
        console.log('need 4', data);
        return 5;
      }).catch(err => {
        console.log('err 5 is ', err);
        return Promise.stop('err 5')
      });
    }).catch(data => {
      return Promise.stop('err 3');
    })
  })
}

function test05() {
  return test04().next(data => {
    console.log('in test05 data =>', data);
    return 6;
  }).catch(err => {
    console.log('err 6 is', err);
    return Promise.stop('err 6')
  }).whenStop(data => {
    console.log('oops catch the stop message is ', data);
    // throw new Error('oops no !!!')
    return 7;
  }).then(data => {
    console.log('if stop need 7, else need 6',data);
    return 8;
  }).catch(err => {
    console.log('err 7 is ', err);
    return Promise.stop('err 7');
  }).next(data => {
    console.log('need 8 ', data);
    return 9;
  }).catch(err => {
    console.log('err 8 is ', err);
    return Promise.stop('err 8')
  }).whenStop(data => {
    console.log('oops catch stop message is ', data);
    return 10;
  })
}

test05().then(data => {
  console.log('final data is ', data)
});

```