---
layout: post
title: ZookKeeper Overview
date: 2021-01-26 01:26:23 +0900
category: Zookeeper
---
# ZookKeeper: A Distributed Coordination Service for Distributed Applications

### ZooKeeper란 무엇인가?

- Distributed Application 들을 위한 오픈 소스 분산 관리 서비스
- 분산 시스템 설계의 문제점
    - 분산 시스템들 간 정보를 어떻게 공유할 것인가?
    - 클러스터에 있는 서버들의 상태를 어떻게 체크할 것인가?
    - 분산 서버들 간의 동기화를 위해서 어떻게 lock 처리를 할 것인가?
- 이러한 문제를 해결하는 시스템을 Coordination Service라고 한다
    - Apache Zookeeper는 이러한 Coordination Service의 대표적인 것
- Coordination Service는 분산 시스템 내의 중요한 상태 정보, 설정 정보등을 유지해야 하기 때문에 장애가 발생하면 전체 시스템에 장애로 연결된다.
    - 이중화 등을 통하여 고 가용성을 제공한다
- Zookeeper는 디렉토리 구조 기반으로 znode라는 key-value식 데이터 저장 객체를 제공하고, 이 객체에 데이터를 넣고 빼는 기능만 제공

### Data Model

- 아래와 같이 Directory 구조의 각 노드에 데이터를 저장한다
- ![](https://zookeeper.apache.org/doc/r3.1.2/images/zknamespace.jpg)
- Node의 종류
    - Persistent Node
        - Node에 데이터를 저장하면 일부러 삭제하지 않는 이상 영구 저장
    - Ephemeral Node
        - Node를 생성한 Client의 세션이 연결되어 있을 때만 유효한 노드
        - Client와 연결이 끊어지는 순간 삭제
        - 이를 통해서 Client와 연결이 되어있는지 판단 가능
    - Sequence Node
        - Node를 생성할 때, 자동으로 Sequence 번호가 붙는 노드
        - Distributed Lock 구현시에 이용됨
- Watcher
    - Client가 특정 znode에 watch를 걸어놓으면, 해당 znode가 변경되었을 때 Client로 Callback 호출을 날려서 이를 Client에 알리고 Watcher는 삭제된다.

### Zookeeper의 활용

- Queue
    - Watcher와 Sequence Node를 이용하여 Queue 구현 가능
        - Queue라는 Node를 만든 후, 해당 Node의 Child를 Sequence Node로 구성
        - 새롭게 생성되는 메세지들은 Sequence Node로 순차적으로 생성됨
        - Queue를 읽는 Client가 Queue Node를 Watch하도록 하면, Client는 새로운 메세지가 들어올 때 마다 Callback을 받아서 MQ의 pub/sub와 같은 형태의 효과를 낼 수 있다.
- 서버 설정 정보
    - 가장 일반적으로 클러스터 내의 각 서버들의 설정정보를 저장하는 저장소로 쓸 수 있다.
    - 정보가 안정적으로 저장되며, Watcher를 이용하여 설정 정보가 저장되는 경우 각 서버로 알려서 바로 반영
- 클러스터 정보
    - 현재 클러스터에서 기동 중인 서버 목록을 유지할 수 있다
        - Ephemeral Node는 Zookeeper Client가 살아있을 경우에만 유효하기 때문에, 클러스터 내에 살아있는 Node 리스트만 유지
        - 클러스터가 Master/Slave 구조인 경우,
            - Master 서버가 죽었을 때, 다른 Master 서버를 선출하는 Election Logic 구현 가능
- Global Lock
    - 여러 개의 서버로 구성된 분산 서버가 Shared Resource를 접근하려고 할 때, 그 작업에 Lock을 거는 기능의 구현이 가능
