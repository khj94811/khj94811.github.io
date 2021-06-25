---
layout: post
title: Android IPC - Messenger vs AIDL
date: 2021-06-25 01:26:23 +0900
category: Android
---
# Messenger vs AIDL

### Messenger

* 비동기 통신
* Intent를 통해서 Remote Process로 전송을 하는 Handler에 대한 참조 역할 수행
* 구현 복잡도는 중간정도 된다
* Messenger를 통해서 메세지는 Remote Process에서 Local Handler로 전송된다.
* Messnger를 사용하면 모든 클라이언트에 대해 요청을 받는 큐를 생성한다.
    * Service는 한번에 하나씩 메세지를 수신한다.
    * Single-Thread에서 수행된다.
* 때문에, 다수의 요청을 동시에 처리하고 싶은 서비스를 개발하고자 한다면, AIDL을 사용하여 Multi-Threading으로 처리해야한다.
    * thread-safety 해야한다.

### AIDL

* Synchronous / Asynchronous IPC
* 기본적으로는 Synchronous지만, "oneway" keyworkd를 사용하여 Asynchronous 통신이 가능하다.
* 1대1 통신을 한다
* thread-safe code가 요구된다.
* Binder transaction buffer는 fixed size이며, 모든 process에 대한 progress에서의 transaction에서 공유된다.
* AIDL은 개발자에게 interface를 다른 Application에 노출시키도록 허용한다.
    * Client와 Service 모두 통신을 위해 필요