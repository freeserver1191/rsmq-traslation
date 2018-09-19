![RSMQ: Redis Simple Message Queue for Node.js](https://img.webmart.de/rsmq_wide.png)

# Redis Simple Message Queue

전용 큐 서버가 필요없는 Node.js 용 경량 메시지 대기열. 그냥 Redis 서버.

[![Build Status](https://secure.travis-ci.org/smrchy/rsmq.png?branch=master)](http://travis-ci.org/smrchy/rsmq)
[![Dependency Status](https://david-dm.org/smrchy/rsmq.svg)](https://david-dm.org/smrchy/rsmq)

**tl;dr:** Redis 서버를 실행하고 현재 Amazon SQS 또는 비슷한 메시지 대기열을 사용하는 경우이 빠른 대체품을 사용할 수도 있습니다. 공유 Redis 서버를 사용하면 여러 Node.js 프로세스가 메시지를 보내고받을 수 있습니다.

## Features

- 경량 : **그냥 Redis** 및 자바 스크립트 ~ 500 라인.
- 속도 : 일반 컴퓨터에서 초당 1000 개 이상의 메시지를 보내고받습니다. **그냥 레디스** 예요 .
- 메시지 가시성 제한 시간 내에 **정확히 한 명의 수신자에게 메시지 배달** 보장 .
- 삭제되지 않은 수신 된 메시지는 가시성 제한 시간 후에 다시 나타납니다.
- [테스트 커버리지](http://travis-ci.org/smrchy/rsmq)
- 메시지는 메시지 ID에 의해 삭제됩니다. 메시지 ID는 `sendMessage`and `receiveMessage`메소드에 의해 리턴됩니다 .
- 삭제되지 않는 한 메시지는 대기열에 남아 있습니다.
- [rest-rsmq](https://github.com/smrchy/rest-rsmq) 를 통한 선택적 RESTful 인터페이스
- 입력 된 타이핑 ❤
  
**참고 :** RSMQ는 Redis EVAL 명령 (LUA 스크립트)을 사용하므로 최소 Redis 버전은 2.6 이상입니다.

## Usage

- 대기열을 만든 후에는 해당 대기열로 메시지를 보낼 수 있습니다.
- 메시지는 지연없이 지정되지 않는 한 **FIFO** (first in first out) 방식으로 처리됩니다 .
- 모든 메시지에는 메시지 `id`를 삭제하는 데 사용할 수 있는 고유 한 메시지가 있습니다.
- `sendMessage`방법은 반환됩니다 `id`보낸 메시지에 대해.
- `receiveMessage`방법은 반환됩니다 `id`메시지와 일부 통계와 함께.
- 메시지를 삭제하지 않으면 visibility timeout에 도달 한 후 다시 수신 할 수 있습니다.
- **visibility timeout** 및 **delay** 과 같은 선택적 매개 변수에 대해서는 아래 설명 된 `createQueue`및 `receiveMessage`메소드를 살펴보십시오 .


## Installation

`npm install rsmq`


## Modules for RSMQ

*RSMQ* 의 핵심을 유지하기 위해 추가 기능을 모듈로 사용할 수 있습니다.

- [**rsmq-worker RSMQ로 작업자**](https://github.com/mpneuried/rsmq-worker) 를 구현하는 도우미.
- [**rest-rsmq RSMQ를**](https://github.com/smrchy/rest-rsmq) 위한 RESTful 인터페이스.
- [**rsmq-cli RSMQ 용**](https://github.com/mpneuried/rsmq-cli) 명령 줄 인터페이스 / 터미널 클라이언트.
- [**rsmq-promise RSMQ에 대한**](https://github.com/msfidelis/rsmq-promise) Promise 인터페이스

## RSMQ in other languages

RSMQ의 단순성은 다른 언어에서도 유용합니다. 다음은 다른 언어로 구현 된 목록입니다.

- [**Java**](https://github.com/igr/jrsmq) 용[ **Java**](https://github.com/igr/jrsmq) RSMQ. [# 48](https://github.com/smrchy/rsmq/issues/48) 참조
- [**PHP를**](https://github.com/michsindelar/PhpRSMQ) 위한[ **PHP**](https://github.com/michsindelar/PhpRSMQ) RSMQ (진행중인 작업)

참고 : RSQM을 다른 언어로 포팅하려는 경우 모든 RSMQ 클라이언트와의 호환성을 보장하는 테스트가 있는지 확인하십시오. 그리고 물론 : 알려 주시면 여기에 귀하의 항구를 언급 할 수 있습니다.

## Example

### Initialize

```javascript
RedisSMQ = require("rsmq");
rsmq = new RedisSMQ( {host: "127.0.0.1", port: 6379, ns: "rsmq"} );
```
*options* 객체 를 통한 RedisSMQ의 매개 변수 :

- `host`(String) : *optional (Default : "127.0.0.1")* Redis 서버
- `port`(Number) : *optional (Default : 6379)* Redis 포트
- `options`(Object) : *optional (Default : {})* [Redis options](https://github.com/NodeRedis/node_redis#options-object-properties) 객체.
- `client`(RedisClient) : *optional* 기존의 redis 클라이언트 인스턴스. `host`그리고 `server`무시됩니다.
- `ns`(String) : *optional (기본값 : "rsmq")* RSMQ가 만든 모든 키에 사용 된 이름 공간 접두사
- `realtime`(Boolean) : *optional (기본값 : false)* 새 메시지의 실시간 게시 활성화 ( [실시간 섹션 참조](https://github.com/smrchy/rsmq/blob/master/README.md#realtime) )


### Create a queue

큐를 만들 때 옵션 매개 변수 에 대해서는 메서드 섹션을 참조하십시오.

```javascript
rsmq.createQueue({qname:"myqueue"}, function (err, resp) {
		if (resp===1) {
			console.log("queue created")
		}
});

```


### Send a message


```javascript
rsmq.sendMessage({qname:"myqueue", message:"Hello World"}, function (err, resp) {
	if (resp) {
		console.log("Message sent. ID:", resp);
	}
});
```


### Receive a message


```javascript
rsmq.receiveMessage({qname:"myqueue"}, function (err, resp) {
	if (resp.id) {
		console.log("Message received.", resp)	
	}
	else {
		console.log("No messages for me...")
	}
});
```

### Delete a message


```javascript
rsmq.deleteMessage({qname:"myqueue", id:"dhoiwpiirm15ce77305a5c3a3b0f230c6e20f09b55"}, function (err, resp) {
	if (resp===1) {
		console.log("Message deleted.")	
	}
	else {
		console.log("Message not found.")
	}
});
```

### List queues


```javascript
rsmq.listQueues( function (err, queues) {
	if( err ){
		console.error( err )
		return
	}
	console.log("Active queues: " + queues.join( "," ) )
});
```

  
## Methods


### changeMessageVisibility

단일 메시지의 공개 타이머를 변경하십시오. 메시지를 다시 볼 수있는 시간은 현재 시간 (현재) + `vt` 에서 계산됩니다.

Parameters:

* `qname` (String): The Queue name.
* `id` (String): The message id.
* `vt` (Number): 이 메시지를 볼 수없는 시간 (초)입니다. Allowed values: 0-9999999 (around 115 days)

Returns: 

* `1` if successful, `0` if the message was not found.



### createQueue

Create a new queue.

Parameters:

* `qname` (String): 대기열 이름입니다. 최대 160 자. 영숫자, 하이픈 (-) 및 밑줄 (_)이 허용됩니다.
* `vt` (Number): *optional* *(Default: 30)* 대기열에서받은 메시지가 메시지 수신을 요청할 때 다른 수신 구성 요소에서 볼 수 없게되는 시간 (초). Allowed values: 0-9999999 (around 115 days)
* `delay` (Number): *optional* *(Default: 0)* 대기열에있는 모든 새 메시지의 배달이 지연되는 시간 (초 ). Allowed values: 0-9999999 (around 115 days)
* `maxsize` (Number): *optional* *(Default: 65536)* The maximum message size in bytes. Allowed values: 1024-65536 and -1 (for unlimited size)

Returns:

* `1`



### deleteMessage

Parameters:

* `qname` (String): The Queue name.
* `id` (String): message id to delete.

Returns:

* `1` if successful, `0` if the message was not found.



### deleteQueue

Deletes a queue and all messages.

Parameters:

* `qname` (String): The Queue name.

Returns:

* `1`



### getQueueAttributes

Get queue attributes, counter and stats

Parameters:

* `qname` (String): The Queue name.

Returns an object:

* `vt`: The visibility timeout for the queue in seconds
* `delay`: The delay for new messages in seconds
* `maxsize`: The maximum size of a message in bytes
* `totalrecv`: Total number of messages received from the queue
* `totalsent`: Total number of messages sent to the queue
* `created`: 대기열이 생성 된 시간 스탬프 (초 단위)
* `modified`: 큐가 마지막으로 수정 된 시간 스탬프 (초 단위) `setQueueAttributes`
* `msgs`: 대기열에있는 현재 메시지 수
* `hiddenmsgs`: Current number of hidden / not visible messages. A message can be hidden while "in flight" due to a `vt` parameter or when sent with a `delay`



### listQueues

List all queues

Returns an array:

* `["qname1", "qname2"]`



### popMessage

대기열에서 다음 메시지를 수신 **and delete it**.

**Important:** 이 방법은 즉시 수신 한 메시지를 삭제합니다. 메시지를 작업하는 중에 문제가 발생하면 메시지를 다시받을 방법이 없습니다.

Parameters:

* `qname` (String): The Queue name.

Returns an object:

  * `message`: The message's contents.
  * `id`: The internal message id.
  * `sent`: Timestamp of when this message was sent / created.
  * `fr`: Timestamp of when this message was first received.
  * `rc`: Number of times this message was received.

Note: Will return an empty object if no message is there  



### receiveMessage

대기열에서 다음 메시지를 수신합니다.

Parameters:

* `qname` (String): The Queue name.
* `vt` (Number): *optional* *(Default: queue settings)* 수신 된 메시지를 다른 사람이 볼 수 없게하는 시간 (초).  Allowed values: 0-9999999 (around 115 days)

Returns an object:

  * `message`: The message's contents.
  * `id`: The internal message id.
  * `sent`: Timestamp of when this message was sent / created.
  * `fr`: Timestamp of when this message was first received.
  * `rc`: Number of times this message was received.

Note: Will return an empty object if no message is there  



### sendMessage

새 메시지를 보냅니다.

Parameters:

* `qname` (String)
* `message` (String)
* `delay` (Number): *optional* *(Default: queue settings)* 메시지 배달이 지연되는 시간 (초). Allowed values: 0-9999999 (around 115 days)

Returns:

* `id`: The internal message id.


    
### setQueueAttributes

Sets queue parameters.

Parameters:

* `qname` (String): The Queue name.
* `vt` (Number): *optional* * 대기열에서 수신 한 메시지가 메시지 수신을 요청할 때 다른 수신 구성 요소에서 볼 수없는 시간 (초).  Allowed values: 0-9999999 (around 115 days)
* `delay` (Number): *optional* 큐에있는 모든 새 메시지의 배달이 지연되는 시간(초). Allowed values: 0-9999999 (around 115 days)
* `maxsize` (Number): *optional* The maximum message size in bytes. Allowed values: 1024-65536 and -1 (for unlimited size)

Note: 하나 이상의 속성 (vt, delay, maxsize)을 제공해야합니다. 제공되는 속성 만 수정됩니다.

Returns an object:

* `vt`: The visibility timeout for the queue in seconds
* `delay`: The delay for new messages in seconds
* `maxsize`: The maximum size of a message in bytes
* `totalrecv`: Total number of messages received from the queue
* `totalsent`: Total number of messages sent to the queue
* `created`: Timestamp (epoch in seconds) when the queue was created
* `modified`: Timestamp (epoch in seconds) when the queue was last modified with `setQueueAttributes`
* `msgs`: Current number of messages in the queue
* `hiddenmsgs`: Current number of hidden / not visible messages. A message can be hidden while "in flight" due to a `vt` parameter or when sent with a `delay`

    
### quit

redis 클라이언트를 연결 해제하십시오.
이는 스크립트 내에서 rsmq를 사용하고 노드를 종료 할 수있게하려는 경우에만 유용합니다.

## Realtime

RSMQ를 [초기화](https://github.com/smrchy/rsmq/blob/master/README.md#initialize) 할 때 새 메시지에 대해 실시간 PUBLISH를 활성화 할 수 있습니다. `sendMessage`Redis PUBLISH 를 통해 RSQM으로 전송되는 모든 새 메시지가 발행됩니다 `{rsmq.ns}:rt:{qname}`.

기본 설정이있는 RSMQ 예제 :

- 대기열에 `testQueue`이미 5 개의 메시지가 있습니다.
- 새 메시지가 대기열로 보내집니다 `testQueue`.
- 다음 Redis 명령이 실행됩니다. `PUBLISH rsmq:rt:testQueue 6`

### How to use the realtime option

새로운 메시지가 RSMQ로 전송 될 때 PUBLISH 외에 다른 것은 일어나지 않을 것이다. 앱은 Redis SUBSCRIBE 명령을 사용하여 새 메시지에 대한 알림을 받고 문제를 발행 할 수 `receiveMessage`있습니다. 그러나 여러 개의 동시 `receiveMessage`호출 을 방지하기 위해 SUBSCRIBE를 사용하여 새 메시지에 대해 여러 작업자의 말을 듣지 않도록하십시오.

## Changes

see the [CHANGELOG](https://github.com/smrchy/rsmq/blob/master/CHANGELOG.md)


## Other projects

|Name|Description|
|:--|:--|
|[**node-cache**](https://github.com/tcs-de/nodecache)|간단하고 빠른 Node.js 내부 캐싱. memcached와 같은 메모리 캐시 내부의 노드.|
|[**redis-tagging**](https://github.com/smrchy/redis-tagging)|모든 레거시 데이터베이스 (SQL 또는 NoSQL)의 항목에 쉽고 빠르게 태그를 지정할 수있는 Node.js 도우미 라이브러리|
|[**redis-sessions**](https://github.com/smrchy/redis-sessions)|Node.js 및 Redis의 고급 세션 저장소|
|[**rsmq-worker**](https://github.com/mpneuried/rsmq-worker)|도우미를 기반으로 작업자 구현하는 RSMQ (레디 스 간단한 메시지 큐) .|
|[**redis-notifications**](https://github.com/mpneuried/redis-notifications)|Redis 기반 알림 엔진. 알림 및 반복 보고서를 안전하게 작성하기 위해 rsmq-worker를 구현합니다.|
|[**connect-redis-sessions**](https://github.com/mpneuried/connect-redis-sessions)|user_id 당 여러 개의 세션을 처리 할 수있는 redis 세션 을 사용하는 미들웨어를 연결하거나 표현 합니다.|


## The MIT License

Please see the LICENSE.md file.
