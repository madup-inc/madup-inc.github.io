---
title: 서버 퍼포먼스 테스트 툴 사용후기
excerpt: 퍼포먼스 테스트 툴 5개의 설치, 튜토리얼, 간단한 사용후기입니다.
image: performance_test_tool/thumbnail.jpg
categories: [tech]
author: willy
---

## 개요

5가지 서버 퍼포먼스 툴의 사용 예시와 용도, 장단점을 따지는 글입니다.
예시는 거창한 내용은 아니고 설치 방법과 튜토리얼을 요약한 내용입니다.

가장 유명한 HP의 `loadrunner`는 상용 툴이기에 제외했습니다.

1. `Wrk`
1. `Vegeta`
1. `Gatling`
1. `JMeter`
1. `nGrinder`


## 서버 환경

akka-http를 사용해서 요청을 받으면 로컬에 로깅하고 로그파일을 S3에 올리는 간단한 scala 프로그램을 만들었습니다. 

서버에서 sbt를 이용해 jar 파일을 실행시킵니다.

프로그램이 올라간 서버는 AWS EC2의 t2.medium (CPU 2개, 메모리 4GIB)입니다.

---

## Wrk

`Wrk`는 멀티 스레드로 사용자를 만들어 테스트하는 오픈소스 툴 입니다.

- 장점 
  - 터미널에서 가볍게 사용하기 좋음
  - lua 스크립트를 쓸수 있음

- 단점
  - 보낸 요청 개수가 정확하지 않음
  - 자세한 리포트나 그래프를 만들어주는 기능이 없음

### 사용 예시

    // wrk -t[쓰레드] -c[유저수] -d[시간] -s [lua script]  "http://url/postback" --latency
    
    wrk -t8 -c256 -d10s -s pipeline.lua  "http://url/postback" --latency

pipeline.lua

    request = function()
      wrk.headers["Connection"] = "Keep-Alive"
      param_value1 = math.random(1, 5)
      param_value2 = math.random(1, 5)
      type_rand = math.random(1, 30)
      path1 = "?name=" .. type_rand
      path2 = "&p1=" .. param_value1
      path3 = "&p2=" .. param_value2
      return wrk.format(nil, "http://url/postback" .. path1 .. path2 .. path3)
    end
    
결과
    
    Running 10s test @ http://ec2-ip.ap-northeast-2.compute.amazonaws.com:18080/postback
      8 threads and 256 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   464.61ms  253.14ms   1.99s    87.46%
        Req/Sec    48.67     34.53   217.00     70.43%
      Latency Distribution
         50%  409.18ms
         75%  549.74ms
         90%  664.65ms
         99%    1.63s
      3279 requests in 10.07s, 658.96KB read
      Socket errors: connect 0, read 0, write 0, timeout 130
    Requests/sec:    325.46
    Transfer/sec:     65.41KB


## 후기

light java 라는 라이브러리 Github 페이지에서 다른 프레임워크와 밴치마킹을 하는 것을 보고 사용했는데 
간단하고 CLI 환경에서 사용하기는 괜찮았습니다.

한 가지 이상했던 점은 결과 값으로 나온 3279 + 120 요청보다 실제 로그에 기록된 횟수는 더 많았습니다.
실제 몇 개가 날라갔는지 알 방법은 없어 보였습니다.

---

## Vegeta

`Vegeta`는 go 기반으로 만들어졌고 Github 리드미에도 드래곤볼 베지터가 그려진 나름 센스있는 툴입니다.

- 장점
  - 명령형 cli와 go util 둘 다 지원
  - 초당 보낼 요청 개수를 정할 수 있음
  - 리포트와 그래프를 뽑아 볼 수 있음

- 단점
  - 검색시 드래곤볼의 베지터가 나와서 검색이 불편함
  - 어떤 파라미터가 있는지 확인이 불편함 (그냥 Readme 문서)

### 사용예시

    //echo [method url] | vageta attack -[시간] -[시간당 보내는 개수] -[시간제한] | tee [에러셋 리포트 이름] | vegeta report
    
    echo "GET http://url/postback?name=user1&ak=123&ck=456&t=test" | vegeta attack -duration=5s -rate=200 -timeout=5s | tee results.bin | vegeta report
    
    Requests      [total, rate]            1000, 200.17
    Duration      [total, attack, wait]    11.790169924s, 4.995631893s, 6.794538031s
    Latencies     [mean, 50, 95, 99, max]  3.497947328s, 3.541281665s, 6.435849969s, 6.760585856s, 6.804998234s
    Bytes In      [total, mean]            78000, 78.00
    Bytes Out     [total, mean]            0, 0.00
    Success       [ratio]                  100.00%
    Status Codes  [code:count]             200:1000
    Error Set:


### 후기

`Wrk`와 다르게 `Vegeta`는 보낸 요청의 timeout이 안 지나면 요청에 응답이 돌아올 때까지 기다리고 리포트를 띄우는 걸로 보입니다.
duration 인자에 시간과 별개로 시간이 걸리고 Error Set을 출력해줍니다.

Go와 명령행에서 둘 다 쓸수 있는 점은 매우 좋아보입니다.

인자들을 확인하려면 다음 링크에서 확인하세요

https://github.com/tsenart/vegeta

---

## Gatling

`Gatling`은 scala 기반으로 netty, akka를 사용한 테스트 툴 입니다.

- 장점
    - 응답시간 분포를 리포트로 제공
    - 시나리오를 작성가능
    - 스레드 기반이 아닌 비동기 높은 성능이 기대됨
    
- 단점
    - 스크립트를 스칼라로 작성해야함 (중요)

### 사용예시

gatling 사이트에서 들어가면 다운로드 파일을 받습니다.

gatling 파일에 테스트 스크립트를 작성합니다.


    import io.gatling.core.Predef._
    import io.gatling.http.Predef._
    import scala.concurrent.duration._
    
    class BasicSimulation extends Simulation {
    
    // http의 헤더 설정
      val httpConf = http
        .baseURL("http://url")
        .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
        .doNotTrackHeader("1")
        .acceptLanguageHeader("en-US,en;q=0.5")
        .acceptEncodingHeader("gzip, deflate")
        .userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:16.0) Gecko/20100101 Firefox/16.0")
    
    
      def randNum = (Math.random() * 10 + 1).toInt
    
        // 임의에 query param
      def randParam = Map(
        "name" -> s"name${randNum}",
        "p1" -> randNum,
        "p2" -> randNum
      )
    
    // 실제 시나리오
      val scn = scenario("BasicSimulation") // 시나리오 이름
        .exec(http("request-1").get("/hello")) // 시나리오 1
        .pause(2) // 대기
        .exec(http("request-2").get("").queryParamMap(randParam)) // 시나리오 2
        .pause(2)
    
      setUp(
        scn.inject(atOnceUsers(1))
      ).protocols(httpConf)
    }



환경변수를 넣어줍니다.


    export GATLING_HOME=/Users/Downloads/gatling-charts-highcharts-bundle-2.3.1
 
 
gatling.sh를 실행합니다. 윈도우에서는 gatling.bat 파일을 실행해줍니다.


    sudo $GATLING_HOME/bin/gatling.sh

    
테스트 결과 다음과 같습니다.


    Select simulation id (default is 'basicsimulation'). Accepted characters are a-z, A-Z, 0-9, - and _
    test-1
    Select run description (optional)
    
    Simulation computerdatabase.BasicSimulation started...
    
    ================================================================================
    2018-07-10 18:07:04                                           5s elapsed
    ---- Requests ------------------------------------------------------------------
    > Global                                                   (OK=200    KO=0     )
    > request-1                                                (OK=100    KO=0     )
    > request-2                                                (OK=100    KO=0     )
    
    ---- BasicSimulation -----------------------------------------------------------
    [##########################################################################]100%
              waiting: 0      / active: 0      / done:100
    ================================================================================
    
    Simulation computerdatabase.BasicSimulation completed in 3 seconds
    Parsing log file(s)...
    Parsing log file(s) done
    Generating reports...
    
    ================================================================================
    ---- Global Information --------------------------------------------------------
    > request count                                        200 (OK=200    KO=0     )
    > min response time                                    442 (OK=442    KO=-     )
    > max response time                                   1311 (OK=1311   KO=-     )
    > mean response time                                   796 (OK=796    KO=-     )
    > std deviation                                        189 (OK=189    KO=-     )
    > response time 50th percentile                        748 (OK=748    KO=-     )
    > response time 75th percentile                        898 (OK=898    KO=-     )
    > response time 95th percentile                       1100 (OK=1100   KO=-     )
    > response time 99th percentile                       1102 (OK=1102   KO=-     )
    > mean requests/sec                                     50 (OK=50     KO=-     )
    ---- Response Time Distribution ------------------------------------------------
    > t < 800 ms                                           131 ( 66%)
    > 800 ms < t < 1200 ms                                  68 ( 34%)
    > t > 1200 ms                                            1 (  1%)
    > failed                                                 0 (  0%)
    ================================================================================




터미널에 로그도 띄어주고 응답시간이나 성공, 실패등을 그래프로 그려서 html 파일에 저장해줍니다.

{% include image.html img="performance_test_tool/1.png" alt="Gatling1" %}

{% include image.html img="performance_test_tool/2.png" alt="Gatling2" %}

html 리포트 파일을 보시면 응답시간 분포를 확인할 수 있습니다. 

### 후기

시간별 응답 분포를 볼 수 있고 테스트 중에서도 퍼포먼스만 테스트한다면 괜찮은 툴로 보였습니다. 하지만 단점으로는 스칼라로 스크립트를 해야하는 점이 별로였습니다.
스칼라 특유의 난해함을 스크립트 작성때도 느껴야 하는 것은 별로입니다.

---

## JMeter

`JMeter`는 아파치에서 만든 java 기반의 오픈소스로 오래된 테스트 툴중 하나입니다.

- 장점
  - 아파치에서 만든 오래된 툴 유명하고 자료가 많다
  - 다양한 프로토콜 지원
  - GUI, 이메일, DB, SSL 지원하는 기능과 플러그인이 많다

- 단점
  - 모든 기능이 다 필요한가?
  - 결과는 리스너로 만들어 보는데 모니터링이 불편함
  - 스레드 기반이라 성능제약이 있다고 함

### 사용예시

`JMeter` 기본적으로 자바가 설치되어 있어야합니다.

설치는 https://jmeter.apache.org/download_jmeter.cgi를 참고하거나
mac과 우분투에서는 apt, brew 같은 패키지 관리자로도 설치가 가능합니다.

    brew install jmeter

저는 brew로 설치해 응용프로그램에 jmeter가 안 보여서 콘솔에서 jmeter를 켜야했습니다. 

    sudo jmeter

먼저 시나리오를 만들어서 그 안에 스레드 그룹을 만들고 유저수(스레드)와 몇 번 반복할지를 입력합니다.

{% include image.html img="performance_test_tool/3.png" alt="jmeter1" %}


그 다음 원하는 add 버튼을 눌러서 프로토콜의 request를 추가하고 server ip와 포트를 입력하고
인자 값까지 추가 할수 있습니다.

{% include image.html img="performance_test_tool/4.png" alt="jmeter2" %}


리포트는 graph, summary, table 등 다양하게 있습니다.
보고 싶은 리포트가 있다면 add 버튼에서 listener를 추가할 수 있습니다.

{% include image.html img="performance_test_tool/5.png" alt="jmeter3" %}

start 버튼으로 실행시키면 테스트가 실행됩니다.

추가적으로 groovy, java, shell 스크립트를 추가할 수 있습니다.

제 서버에서는 S3에서 파일을 다운 받아 제대로 로깅 됬는지 확인하는 스크립트를 작성할수 있었습니다.

http://doloadtest.blogspot.com/2018/04/upload-download-from-aws-s3-via-jmeter.html

### 후기

위의 테스트 툴이 웹서버에만 한정해서 테스트가 가능했다면 `JMeter`는 서버 전체의 다양한 테스트가 가능했습니다.

구관이 명관이란 말이 있듯 좋은 툴입니다. 부하테스트만 할 생각인데 불필요할 정도로 기능이 많고
플러그인도 많고 부하 테스트 외 다른 테스트 목적으로도 쓸모 있어 보였습니다.

---

## nGrinder

`nGrinder`는 네이버에서 만든 오픈 소스입니다.

- 장점
  - 설치만 하면 사용하기 쉬움
  - 예약, 모니터링, ramp up, 스크립트 기능 지원
  - docker에서 사용 가능

- 단점
  - agent와 controller를 각자 실행해야됨
  - controller가 tomcat 필요함 tomcat이 싫다면 docker를 사용하는 것도 방법
  - Thread 기반으로 구현되어 있어 성능과 동시성에 대해 제한이 있다고 함 (grinder)

### 사용 예시

저는 tomcat을 설치하고 싶지 않아서 `nGrinder` controller는 docker에서 설치했습니다.

    docker pull ngrinder/controller:3.4
    
    docker run -d -v ~/ngrinder-controller:/opt/ngrinder-controller -p 80:80 -p 16001:16001 -p 12000-12009:12000-12009 ngrinder/controller:3.4


그 후 localhost에 들어가 로그인하고 agent를 설치했습니다.
초기 아이디 비밀번호는 admin으로 들어가시면 됩니다.

agent를 실행하면 controller에서 agent를 찾아냅니다.


    /ngrinder-agent/run_agent.sh
    


다른 툴들과 같이 입력할 사항 스크립트를 등록하고 테스트를 실행시킵니다.

{% include image.html img="performance_test_tool/6.png" alt="ngrinder1" %}


그러면 모니터링을 볼수 있고 결과가 남습니다.

{% include image.html img="performance_test_tool/7.png" alt="ngrinder2" %}

### 후기

테스트를 예약해서 주기적으로 할 수 있고 모니터링도 심플하고 좋았습니다. 개인적으로 위의 4가지 툴 보다 편리했습니다.

---

## 요약

`Wrk` : 몇 개 날리는게 상관 없이 간단하게 서버 부하 테스트를 하고 싶을 때 

`Vegeta` : 초당 몇 개의 부하를 버티는지 테스트 하고 싶을 때 

`Gatling` : 높은 성능, 시나리오 작성, html 보고서

`JMeter` : 성능 보단 다양한 기능이나 플러그인

`nGrinder` : 지속적인 테스트, 모니터링
