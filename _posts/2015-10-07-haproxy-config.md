---
layout: post
title: Introducing haproxy config
comments: true
---


### 전역 옵션 섹션
**global**    

* daemon: 백그라운드 모드(background mode)로 실행   
* log: syslog 설정
* log-send-hostname: hostname 설정
* uid: 프로세스의 userid를 number로 변경
* user: 프로세스의 userid를 name으로 변경
* node: 두 개 이상의 프로세스나 서버가 같은 IP 주소를 공유할 때 name 설정(HA 설정)
* maxconn: 프로세스당 최대 연결 개수

### 기본 옵션 섹션
**Defaults** 

* log: syslog 설정
* maxconn: 프로세스당 최대 연결 개수

**listen webfarm 10.101.22.76:80 : haproxy name ip:port**

* mode http: 연결 프로토콜
* option httpchk: health check
* option log-health-checks: health 로그 남김 여부
* option forwardfor: 클라이언트 정보 전달
* option httpclose: keep-alive 문제 발생 시 off 옵션
* cookie SERVERID rewrite: 쿠키로 서버 구별 시 사용 여부
* cookie JSESSIONID prefix: HA 구성 시 prefix 이후에 서버 정보 주입 여부
* balance roundrobin: 순환 분배 방식
* stats enable: 서버 상태 보기 가능 여부
* stats uri /admin: 서버 상태 보기 uri
* server xvadm01.ncli 10.101.22.18:80 cookie admin_portal_1 check inter 1000 rise 2 fall 5: real server 정보(server [host명] [ip]:[port] cookie [서버쿠키명] check inter [주기(m/s)] rise [서버구동여부점검횟수], fall [서비스중단여부점검횟수])


### balance 옵션

로드 밸런싱의 경우 round robin 방식을 일반적으로 사용하지만 다른 여러 방식이 있다. 옵션에 적용할 수 있는 로드 밸런싱 알고리즘은 다음과 같다.

* roundrobin: 순차적으로 분배(최대 연결 가능 서버 4128개)
* static-rr: 서버에 부여된 가중치에 따라서 분배
* leastconn: 접속 수가 가장 적은 서버로 분배
* source: 운영 중인 서버의 가중치를 나눠서 접속자 IP를 해싱(hashing)해서 분배
* uri: 접속하는 URI를 해싱해서 운영 중인 서버의 가중치를 나눠서 분배(URI의 길이 또는 depth로 해싱)
* url_param: HTTP GET 요청에 대해서 특정 패턴이 있는지 여부 확인 후 조건에 맞는 서버로 분배(조건 없는 경우 round robin으로 처리)
* hdr: HTTP 헤더 에서 hdr(<name>)으로 지정된 조건이 있는 경우에 대해서만 분배(조건 없는 경우 round robin으로 처리)
* rdp-cookie: TCP 요청에 대한 RDP 쿠키에 따른 분배


```
#서버 정보
#LB ip       10.101.22.33
#server-1 10.101.27.49
#server-2 10.101.26.50

global
        log 127.0.0.1   local0
        log 127.0.0.1   local1 notice
        maxconn 4096
        uid 99
        gid 99
        daemon
        log-send-hostname
        #debug
        #quiet

defaults
        log     global

listen  webfarm 10.101.22.33:80
        mode http
        option httpchk GET /l7check.html HTTP/1.0
        option log-health-checks
        option forwardfor
        option httpclose
        cookie SERVERID rewrite
        cookie JSESSIONID prefix
        balance roundrobin
        stats enable
        stats uri /admin
        server  xvadm01.ncli 10.101.27.49:80 cookie admin_portal_1 check inter 1000 rise 2 fall 5
        server  xvadm02.ncli 10.101.26.50:80 cookie admin_portal_2 check inter 1000 rise 2 fall 5
```
