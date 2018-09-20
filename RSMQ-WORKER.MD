![RSMQ-Worker](https://trello-attachments.s3.amazonaws.com/5481963992d9ba3848568a1b/600x194/61b3b6117eeb0881c919c6996adb2620/rsmq_worker_small.png)

[![Build Status](https://secure.travis-ci.org/mpneuried/rsmq-worker.png?branch=master)](http://travis-ci.org/mpneuried/rsmq-worker)
[![Windows Tests](https://img.shields.io/appveyor/ci/mpneuried/rsmq-worker.svg?label=WindowsTest)](https://ci.appveyor.com/project/mpneuried/rsmq-worker)
[![Coveralls Coverage](https://img.shields.io/coveralls/mpneuried/rsmq-worker.svg)](https://coveralls.io/github/mpneuried/rsmq-worker)

[![Deps Status](https://david-dm.org/mpneuried/rsmq-worker.png)](https://david-dm.org/mpneuried/rsmq-worker)
[![npm version](https://badge.fury.io/js/rsmq-worker.png)](http://badge.fury.io/js/rsmq-worker)
[![npm downloads](https://img.shields.io/npm/dt/rsmq-worker.svg?maxAge=2592000)](https://nodei.co/npm/rsmq-worker/)

[![Join the chat at https://gitter.im/mpneuried/rsmq-worker](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/mpneuried/rsmq-worker?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Helper to simply implement a worker [RSMQ ( Redis Simple Message Queue )](https://github.com/smrchy/rsmq).

[![NPM](https://nodei.co/npm/rsmq-worker.png?downloads=true&stars=true)](https://nodei.co/npm/rsmq-worker/)

>  :warning: **참고 :** RSMQ는 Redis `EVAL`명령 (LUA 스크립트)을 사용하므로 최소 Redis 버전이 `2.6+`됩니다.

## Install

```sh
  npm install rsmq-worker
```

## Initialize

```js
  new RSMQWorker( queuename, options );
```


**Example:**

```js
  var RSMQWorker = require( "rsmq-worker" );
  var worker = new RSMQWorker( "myqueue" );

  worker.on( "message", function( msg, next, id ){
  	// process your message
  	console.log("Message id : " + id);
  	console.log(msg);
  	next()
  });

  // optional error listeners
  worker.on('error', function( err, msg ){
      console.log( "ERROR", err, msg.id );
  });
  worker.on('exceeded', function( msg ){
      console.log( "EXCEEDED", msg.id );
  });
  worker.on('timeout', function( msg ){
      console.log( "TIMEOUT", msg.id, msg.rc );
  });

  worker.start();
```

**Config** 

- **queuename**: *( `String` required )* The queuename to pull the messages
- **options** *( `Object` optional )* The configuration object
	- **options.interval**: *( `Number[]` optional; default = `[ 0, 1, 5, 10 ]` )* 증가하는 대기 시간 (초 *)* 의 배열입니다. _[More details](#options-interval)_
	- **options.maxReceiveCount**: *( `Number` optional; default = `10` )* 메시지가 초과 될 때까지 수신
	- **options.invisibletime**: *( `Number` optional; default = `30` )* 메시지를받은 후 숨길 시간 (초 *)* .
	- **options.defaultDelay**: *( `Number` optional; default = `1` )* 새 메시지를 대기열로 보내는 데 필요한 기본 지연 (초)입니다.
	- **options.autostart**: *( `Boolean` optional; default = `false` )* init시에 자동 시작
	- **options.timeout**: *( `Number` optional; default = `3000` )* 메시지 처리 시간 초과 `ms`. 그래서 최소한 3000ms 후에 `next()`메서드 를 호출해야합니다 `message`. `0`그것으로 설정되면 무한대까지 기다릴 것입니다.
	- **options.customExceedCheck**: *( `Function` optional; )* 사용자 지정 초과 검사를 작성하기위한 원시 메시지 *(메시지 형식 참조)* 를 인수로 갖는 사용자 지정 함수입니다. 당신이 돌려 보내는 경우 `true`에 메시지는 초과하지 않을 것이다. 돌아 오면 `false`정기 점검 `maxReceiveCount`이 사용됩니다.
	- **options.alwaysLogErrors**: *( `Boolean` optional; default = `false` )* 오류 수신기가 연결되어 있어도 오류가 콘솔에 기록됩니다.
	- **options.rsmq**: *( `RedisSMQ` optional; default = `null` )* 새 클라이언트를 만드는 대신 사용할 기존의 rsmq 인스턴스
	- **options.redis**: *( `RedisClient` optional; default = `null` )* `rsmq` 인스턴스가 정의 되지 않은 경우 사용할 기존의 redis 클라이언트 인스턴스
	- **options.redisPrefix**: *( `String` optional; default = `""` )* `rsmq` 인스턴스가 정의 되지 않은 경우 rsmq의 접두사 / 네임 스페이스 redis . 이것은 `ns`RSMQ 옵션과 일치해야합니다 .
	- **options.host**: *( `String` optional; default = `"localhost"` )* `rsmq`또는 `redis`인스턴스가 정의 된 경우 redis에 연결할 호스트
	- **options.port**: *( `Number` optional; default = `6379` )* `rsmq`또는 `redis`인스턴스가 정의 된 경우 redis에 연결할 포트
	- **options.options**: *( `Object` optional; default = `{}` )* `rsmq`또는 `redis`인스턴스가 정의 된 경우 redis에 연결하는 옵션


## Raw message format

A message ( e.g. received by the event `data` or `customExceedCheck` ) 에는 다음 키가 포함됩니다:

- **msg.message** : *( `String` )* 큐 메시지 내용입니다. 문자열 화 된 JSON을 사용하여 복잡한 내용을 사용할 수 있습니다.
- **msg.id** : *( `String` )* The rsmq internal message id
- **msg.sent** : *( `Number` )* Timestamp of when this message was sent / created.
- **msg.fr** : *( `Number` )* Timestamp of when this message was first received.
- **msg.rc** : *( `Number` )* Number of times this message was received.

## Methods

### `.start()`

구성 `autostart`을 정의하지 않은 경우 메소드 `true`를 호출해야 `.start()`합니다.

**Return**

*( Self )*: 체인을위한 인스턴스 자체.

### `.stop()`

수신 간격을 멈추기 만하면됩니다. 그러면 rsmq / redis와의 연결이 끊어지지 않습니다. 스크립트를 끝내고 싶다면`.quit()`

**Return**

*( Self )*: 체인을위한 인스턴스 자체.

### `.send( msg [, delay ][, cb ] )`

도우미 기능으로 구성된 대기열에서 메시지를 보내기 만하면됩니다.

**Arguments**

* `msg` : *( `String` required )*: rsmq 메시지. 모범 사례로 추가 데이터가 포함 된 JSON 형식의 문자열입니다.
* `delay` : *( `Number` optional; default = `0` )*:  다음 `x`초 동안 이 메시지를 숨기는 메시지 지연 .
* `cb` : *( `Function` optional )*: 성공적인 전송을 위해 보안 응답을 얻기위한 선택적 콜백.

**Return**

*( Self )*: 체인을위한 인스턴스 자체.

### `.del( id [, cb ] )`

메시지가 처리 된 후 메시지를 삭제하는 도우미 기능.

**Arguments**

* `id` : *( `String` required )*: rsmq 메시지 ID.
* `cb` : *( `Function` optional )*: 성공적인 삭제에 대한 보안 응답을 얻기위한 선택적 콜백.

**Return**

*( Self )*: 체인을위한 인스턴스 자체.

### `.changeInterval( interval )`

작동중인 간격 시간 초과를 변경하십시오.

**Arguments**

* `interval` : *( `Number|Array` required )*: The new interval.

**Return**

*( Self )*:  체인을위한 인스턴스 자체.

### `.quit()`

작업자를 중지하고 연결을 닫으십시오. 이 후에는 더 이상 worker-instance를 재사용 할 수 없습니다. 스크립트가 종료 될 수 있도록 모든 타이머와 연결을 종료하기위한 것입니다.

### `.info( cb )`

현재의 큐 속성을 가져옵니다. 이것은 바로 가기 [`rsmq.getQueueAttributes`](https://github.com/freeserver1191/rsmq-traslation#getqueueattributes)입니다.

**Arguments**

* `cb` : *( `Function` )*: Callback with `( err, attributes )`. 자세한 내용은 [rsmq-docs](https://github.com/freeserver1191/rsmq-traslation#getqueueattributes) 를 참조하십시오.

### `.size( [hidden=false], cb )`

현재 큐 크기를 가져옵니다.

**Arguments**

- `hidden`: *( Boolean선택, 기본값 = false)* : 현재 숨겨진 / "in flight"메시지를 포함한 메시지 수.
- `cb`: *( Function옵션)* : 콜백 `( err, size )`. 는 `size`인 `Number`큐에있는 메시지의 수를 나타낸다. 경우 `hidden=true`당신은 현재 숨겨진 메시지의 번호를 받게됩니다.

**Return**

*( Self )*: 체인을위한 인스턴스 자체.

## Events

### `message`

메시지를 잡아서 처리하는 메인 이벤트.
이 이벤트에 대한 핸들러를 설정하지 않으면 아무 일도 일어나지 않습니다.

**Example:**

```js
worker.on( "message", function( message, next, msgid ){
	// process message ... 
	next();
});
```

**Arguments** 

- **message** : *( `String` )* 처리 할 대기열 메시지 내용입니다. 문자열 화 된 JSON을 사용하여 복잡한 내용을 사용할 수 있습니다.
- **next** : *( `Function` )* 메시지가 처리되었을 때 호출해야하는 함수.
  **Arguments** 
  * `delete`: *( `Boolean|Error` optional; default = true )* `Error` : 오류를 반환하면 오류 이벤트로 방출됩니다. `Boolean`: 작업자가 끝날 때 자동으로 메시지를 삭제하지 못하게 할 수 있습니다. 이것은 메시지를 여러 번 표시하려는 경우에 유용합니다. 이를 구현하려면 설정을 확인하십시오.`options.customExceedCheck`
- **msgid** : *( `String` )* 메시지 ID입니다. 이 기능은 메시지를 수동으로 삭제하려는 경우에 유용합니다.

### `ready`

작업자가 rsmq / redis에 연결되고 주어진 대기열 이름으로 초기화 될 때까지 발생합니다.

### `data`

메시지가 수신 된 원시 이벤트입니다.

**Arguments** 

- **msg** : *( `String` )* The raw rsmq message. ( See section Raw message format )

### `deleted`

메세지가 삭제 된 후에 시작됩니다.

**Arguments** 

- **id** : *( `String` )* The rsmq message id

### `exceeded`

메시지가 초과 된 후 시작되어 즉시 삭제됩니다.

**Arguments** 

- **msg** : *( `String` )* The raw rsmq message. ( See section Raw message format )

### `timeout`

메시지 처리가 구성된 시간 초과를 초과하면 시작됩니다.

**Arguments** 

- **msg** : *( `String` )* The raw rsmq message. ( See section Raw message format )

### `error`

메세지 처리로 에러가 발생했을 경우에 발행됩니다.

**Arguments** 

- **err** : *( `Error|Any` )* The thrown error
- **msg** : *( `String` )* The raw rsmq message. ( See section Raw message format )

## Advanced example

이것은 작동중인 일부 기능을 보여주는 고급 예제입니다.

```js
	var fs = require( "fs" );
	var RSMQWorker = require( "rsmq-worker" );

	var fnCheck = function( msg ){
		// 내용이`createmessages` 인 경우 메시지를 초과하지 않는 함수를 확인하십시오.
		if( msg.message === "createmessages" ){
			return true
		}
		return false
	}

	
	var worker = new RSMQWorker( "myqueue", {
		interval: [ .1, 1 ], // 수신 할 때마다 100ms를 기다리고 빈 경우에는 1,3까지받습니다.
		invisibletime: 2,	 // 5 초 동안받은 메시지 숨기기
		maxReceiveCount: 2,	 // 삭제 될 때까지 2 번만 메시지를받습니다.
		autostart: true,	 // init에서 작업자를 시작합니다.
		customExceedCheck: fnCheck // 사용자 정의 초과 검사를 설정합니다.
	});

	// Listen to errors
	worker.on('error', function( err, msg ){
	    console.log( "ERROR", err, msg.id );
	});
	worker.on('timeout', function( msg ){
	    console.log( "TIMEOUT", msg.id, msg.rc );
	});
	
	// 초과 된 메시지를 처리합니다.
	// 내부 rsmq 인스턴스 가져 오기
	var rsmq = worker._getRsmq();
	worker.on('exceeded', function( msg ){
		console.log( "EXCEEDED", msg.id );
		// NOTE: 큐가 있는지 확인하십시오.
		rsmq.sendMessage( "YOUR_EXCEEDED_QUEUE", msq, function( err, resp ){
			if( err ){
				console.error( "write-to-exceeded-queue", err )
			}
		});
	});

	// listen to messages
	worker.on( "message", function( message, next, id ){
		
		console.log( "message", message );
		
		if( message === "createmessages" ){
			next( false )
			worker.send( JSON.stringify( { type: "writefile", filename: "./test.txt", txt: "Foo Bar" } ) );
			worker.send( JSON.stringify( { type: "deletefile", filename: "./test.txt" } ) );
			return	
		}

		var _data = JSON.parse( message )
		switch( _data.type ){
			case "writefile": 
				fs.writeFile( _data.filename, _data.txt, function( err ){
					if( err ){
						next( err );
					}else{
						next()
					}
				});
				break;
			case "deletefile": 
				fs.unlink( _data.filename, function( err ){
					if( err ){
						next( err );
					}else{
						next()
					}
				});
				break;
		}
		
	});

	worker.send( "createmessages" );
```

## Details

### Options `interval`

옵션 `interval`은 다음을 수행 할 수 있습니다:

**A.)**
는 숫자이므로 작업자는 매 `n`초 마다 대기열을 폴링합니다 (예 : `interval: .5`= 초 = 두 번째 초).

**B.)**
숫자 배열입니다. 시작 `interval[0]`시 대기열을 폴링하는 시간입니다. 매번 작업자는 빈 응답을 수신 *(큐가 비어)* 다음은 `interval`다음 폴링 대기하는 데 사용됩니다 *( interval[+1])* 마지막 정의까지 `interval[ n ]`도달했습니다. 수신 된 모든 메시지에서 대기 시간이로 재설정됩니다 `interval[0]`.

> E.g: `interval: [ .2, 1, 3 ]`
> - 1st poll -> no message -> wait `.2s` = `200ms`
> - 2nd poll -> no message -> wait `1s`
> - 3rd poll -> no message -> wait `3s`
> - 4th poll -> no message -> wait `3s`
> - 5th poll -> 1 message -> wait `.2s`
> - 6th poll -> no message -> wait `1s`
> - 7th poll -> 1 message -> wait `.2s`
> - 8th poll -> 1 message -> wait `.2s`
> - 9th poll -> no message -> wait `.2s`
> - 10th poll -> no message -> wait `1s`
>   ...

## Todos/Ideas

- 더 많은 테스트!
- 선택적 병렬 실행. 여러 수신을 병렬로 수행하려면.
- 초과 된 메시지를 구성된 대기열에 자동으로 기록합니다.

## Release History
|Version|Date|Description|
|:--:|:--:|:--|
|0.5.2|2016-10-24|Optimized README and updated dependencies|
|0.5.1|2016-08-22|Fixed reconnect error [Issue#20](https://github.com/mpneuried/rsmq-worker/issues/20). Thanks to [mstduff](https://github.com/mstduff ); updated deps; removed generated code docs from repo|
|0.5.0|2016-07-14|Added methods `.info(cb)` ([Issue#17](https://github.com/mpneuried/rsmq-worker/issues/17)) and `.size( [hidden,] cb )`|
|0.4.3|2016-06-20|Optimized event listeners [Issue#15](https://github.com/mpneuried/rsmq-worker/issues/15). Thanks to [Kevin Turner](https://github.com/kpturner )|
|0.4.2|2016-05-06|Added the `.quit()` function [Issue#11](https://github.com/mpneuried/rsmq-worker/issues/11). Thanks to [Sam Fung](https://github.com/5amfung )|
|0.4.1|2016-04-05|Fixed missing isNumber function|
|0.4.0|2016-03-30|Updated dependencies (especially lodash to 4.x). Fixed a config bug caused by the array merge from `extend` [Issue#7](https://github.com/mpneuried/rsmq-worker/issues/7). Thanks to [Peter Hanneman](https://github.com/timelessvirtues )|
|0.3.8|2015-11-04|Fixed stop behavior. [Pull#5](https://github.com/mpneuried/rsmq-worker/pull/5). Thanks to [Exinferis](https://github.com/exinferis)|
|0.3.7|2015-09-02|Added tests to check the behavior during errors within message processing; Added option `alwaysLogErrors` to prevent console logs if an error event handler was attached. [Issue #3](https://github.com/mpneuried/rsmq-worker/issues/3)|
|0.3.6|2015-09-02|Updated dependencies; optimized readme (thanks to [Tobias Lidskog](https://github.com/tobli) for the [pull #4](https://github.com/mpneuried/rsmq-worker/pull/4))|
|0.3.5|2015-04-27|again ... fixed argument dispatch for `.send()`|
|0.3.4|2015-04-27|fixed argument dispatch for `.send()` and added optional cb for `.del()`|
|0.3.3|2015-03-27|added `changeInterval` to modify the interval in operation|
|0.3.2|2015-02-23|changed default prefix/namespace;|
|0.3.0|2015-02-16|It's now possible to return an error as first argument of `next`. This will lead to an error emit + optimized readme|
|0.2.2|2015-01-27|added option `defaultDelay` and optimized arguments of the `send` method; fixed travis.yml|
|0.2.0|2015-01-27|Added timeout, better error handling and send callback|
|0.1.2|2015-01-20|Reorganized code, added code docs and optimized readme|
|0.1.1|2015-01-17|Added test scripts and optimized repository file list|
|0.1.0|2015-01-16|First working and documented version
|0.0.1|2015-01-14|Initial commit|

[![NPM](https://nodei.co/npm-dl/rsmq-worker.png?months=6)](https://nodei.co/npm/rsmq-worker/)

> Initially Generated with [generator-mpnodemodule](https://github.com/mpneuried/generator-mpnodemodule)

## Other projects

|Name|Description|
|:--|:--|
|[**rsmq**](https://github.com/smrchy/rsmq)|A really simple message queue based on Redis|
|[**rsmq-cli**](https://github.com/mpneuried/rsmq-cli)|a terminal client for rsmq|
|[**rest-rsmq**](https://github.com/smrchy/rest-rsmq)|REST interface for.|
|[**redis-notifications**](https://github.com/mpneuried/redis-notifications)|A redis based notification engine. It implements the rsmq-worker to safely create notifications and recurring reports.|
|[**node-cache**](https://github.com/tcs-de/nodecache)|Simple and fast NodeJS internal caching. Node internal in memory cache like memcached.|
|[**redis-sessions**](https://github.com/smrchy/redis-sessions)|An advanced session store for NodeJS and Redis|
|[**obj-schema**](https://github.com/mpneuried/obj-schema)|Simple module to validate an object by a predefined schema|
|[**connect-redis-sessions**](https://github.com/mpneuried/connect-redis-sessions)|A connect or express middleware to simply use the [redis sessions](https://github.com/smrchy/redis-sessions). With [redis sessions](https://github.com/smrchy/redis-sessions) you can handle multiple sessions per user_id.|
|[**systemhealth**](https://github.com/mpneuried/systemhealth)|Node module to run simple custom checks for your machine or it's connections. It will use [redis-heartbeat](https://github.com/mpneuried/redis-heartbeat) to send the current state to redis.|
|[**task-queue-worker**](https://github.com/smrchy/task-queue-worker)|A powerful tool for background processing of tasks that are run by making standard http requests.|
|[**soyer**](https://github.com/mpneuried/soyer)|Soyer is small lib for serverside use of Google Closure Templates with node.js.|
|[**grunt-soy-compile**](https://github.com/mpneuried/grunt-soy-compile)|Compile Goggle Closure Templates ( SOY ) templates inclding the handling of XLIFF language files.|
|[**backlunr**](https://github.com/mpneuried/backlunr)|A solution to bring Backbone Collections together with the browser fulltext search engine Lunr.js|

## The MIT License (MIT)

Copyright © 2015 Mathias Peter, http://www.tcs.de

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
