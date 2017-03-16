.. _caching-purge:

8장. Caching 무효화
******************

이 장에서는 Caching된 콘텐츠를 사용자가 임의대로 갱신하는 방법에 대해 설명한다.
보통 Purge로 지칭되지만 STON 미디어 서버는 여러상황에 맞춰 활용할 수 있는 다양한 API를 제공한다.

원본서버로부터 캐싱된 콘텐츠는 :ref:`caching-policy-ttl` 에 기반한 갱신주기를 가진다.
하지만 명백히 콘텐츠가 변경되었고 관리자가 이를 즉시 반영하고 싶을 경우 :ref:`caching-policy-ttl` 이 만료될 때까지 기다릴 필요는 없다.
`Purge`_ / `Expire`_ / `HardPurge`_ 등을 사용하면 즉시 콘텐츠를 무효화시킬 수 있다.

.. note:

   API로 입력되는 파라미터는 원본서버 URL을 기준으로 한다. 
   각 프로토콜 URL표현을 구분할 경우 자칫 프로토콜마다 콘텐츠를 별도로 캐싱하는 것처럼 오해할 수 있기 때문이다. ::
   
      http://127.0.0.1:20040/command/purge?url=www.example.com/bar/mp4:trip.mp4

   예를 들어 위 URL처럼 HLS형식을 입력해도 동작하지만 자칫 HLS에서만 `Purge`_ 되는 것처럼 인식될 수 있다. 
   따라서 아래와 같은 표현이 올바르다. ::
   
      http://127.0.0.1:20040/command/purge?url=www.example.com/bar/trip.mp4



.. toctree::
   :maxdepth: 2



.. _api-monitoring-fileinfo:
   
캐싱조회
====================================

캐싱하고 있는 파일상태를 조회한다.
일반적으로 파일은 URL로 구분되지만 같은 URL에 다른 옵션(i.e. Accept-Encoding등)이 존재하는 경우 여러 개의 파일이 존재할 수 있다. ::

    http://127.0.0.1:20040/monitoring/fileinfo?url=example.com/bar/trip.mp4
    
결과는 JSON형식으로 제공된다.
다음은 /trip.mp4파일의 정보를 열람한 결과이다. ::

    {
        "version": "1.0.0",
        "method": "fileinfo",
        "status": "OK",
        "result":
        [ 
            {
                "URI": "/trip.mp4",
                "Accept-Encoding": "N",
                "RefCount": 0,
                "Disk-Index": 0,
                "Size": 2100267,
                "FID": 24267,
                "LocalPath": "/cache1/example.com/000i/q3.bin",
                "File-Opened ": "N",
                "File-Updating": "-",
                "Downloader-Count": "0",
                "LastAccess": "[ 2016.09.03 14:29:50, -2 ]",
                "UpdateTime": "[ 2016.09.03 13:53:43, -2169 ]",
                "TTL-Left": "[ 2012.10.03 13:53:43, 2589831 ]",
                "ResponseCode": 200,
                "ContentType": "text/plain",
                "LastModifiedTime": "[ 2010.11.22 20:31:47, -56224685 ]",
                "ExpireTime": "[ 0, 0 ]",
                "CacheControl": "not-specified",
                "ETag": "502dd614:200c2b",
                "CustomTTL": 0,
                "NoMoreExist": "N",
                "LocalFileExist": "Y",
                "SmallFile": "N",
                "State": "Cached",
                "Deleted": "N",
                "AddedSize": "Y",
                "TransferEncoding": "N",
                "Compression": "-",
                "Purge": "N",
                "Ignore-IMS ": "N",
                "Redirect-Location ": "-",
                "Content-Disposition ": "-",
                "NoCache": "N"
            }
        ]
    }
    
-  ``URI`` 파일 URI
-  ``Accept-Encoding`` ("Y" or "N") Accept-Encoding을 지원한다면 "Y"
-  ``RefCount`` 파일참조 카운트
-  ``Size`` (Bytes) 파일크기
-  ``Disk-Index`` (0부터 시작) 저장된 디스크 인덱스
-  ``FID`` 파일 ID
-  ``LocalPath`` 로컬 경로
-  ``File-Opened`` ("Y" or "N") 로컬파일을 열고 있다면 "Y"
-  ``File-Updating`` 파일을 갱신 중이라면 갱신하는 객체의 포인터가 명시
-  ``Downloader-Count`` 원본서버에서 이 파일을 다운로드 받는 현재 세션의 개수
-  ``LastAccess`` (마지막 접근시간, 마지막 접근시간-현재시간) [ 2012.09.03 14:29:50, -2 ]의 의미는 2012.09.03 14:29:50에 접근됐으며 현재로부터 2초 전에 접근됐다는 의미이다.
-  ``UpdateTime`` (갱신시간, 갱신시간-현재시간) 파일이 마지막으로 갱신된 시간. 304 Not Modified에도 시간은 갱신된다.
-  ``TTL-Left`` (만료시간, 만료시간-현재시간) 컨텐츠 만료 예정시간. TTL이 남았다면 양수로, 만료됐다면 음수로 표기된다.
-  ``ResponseCode`` 원본서버 응답코드
-  ``ContentType`` MIME Type
-  ``LastModifiedTime`` (Last Modified Time, Last Modified Time`` 현재시간) 원본서버가 보낸 Last Modified Time. 원본서버가 이 값을 보내지 않았다면 0으로 표시된다.
-  ``ExpireTime`` (Expire Time, Expire Time`` 현재시간) 원본서버가 보낸 Expire Time. 원본서버가 이 값을 보내지 않았다면 0으로 표시된다.
-  ``CacheControl`` ("no-cache" or "not-specified" or (정수)) 원본서버가 보낸 Cache-Contorl 값
-  ``ETag`` STON이 생성한 ETag
-  ``CustomTTL`` 커스텀 TTL. 설정되어 있지 않다면 0이다.
-  ``NoMoreExist`` ("Y" or "N") 파일을 파기예약되어 있다면 "Y"
-  ``LocalFileExist`` ("Y" or "N") 로컬에 파일이 존재하면 "Y" (200 OK가 아닌 파일들은 항상 "Y")
-  ``SmallFile`` ("Y" or "N") 파일을 작은파일로 판단한다면 "Y" (개발적인 이유)
-  ``State`` ("Not Init" or "Cached" or "Error") 파일 상태
-  ``Deleted`` ("Y" or "N") 삭제되었다면 "Y" (개발적인 이유)
-  ``AddedSize`` ("Y" or "N") 크기가 통계에 반영되었다면 "Y" (개발적인 이유)
-  ``TransferEncoding`` ("Y" or "N") Transfer-Encoding을 지원한다면 "Y"
-  ``Compression`` 압축방식
-  ``Purge`` ("Y" or "N") Purge됐다면 "Y"
-  ``Ignore-IMS`` ("Y" or "N") 갱신할 때 If-Modified-Since헤더를 보내지 않도록 설정되었다면 "Y"
-  ``Redirect-Location`` Location 헤더 값
-  ``Content-Disposition`` Content-Disposition 헤더 값
-  ``NoCache`` ("Y" or "N") 원본서버에서 no-cache응답을 줬다면 "Y"



.. _api-cmd-purge:

Purge
====================================

타겟 컨텐츠를 무효화시켜 원본서버로부터 컨텐츠를 다시 다운로드 받도록 한다.
Purge후 최초 접근 시점에 원본서버로부터 컨텐츠를 다시 캐싱한다.
만약 원본서버에 장애가 발생하여 컨텐츠를 가져올 수 없다면 무효화된 컨텐츠를 다시 복원시켜
서비스에 장애가 없도록 처리한다.
이렇게 복원된 컨텐츠는 해당 시점으로부터 ConnectTimeout설정만큼 뒤에 갱신한다. ::

    http://127.0.0.1:20040/command/purge?url=...

타겟 컨텐츠는 URL, 패턴으로 지정할 수 있을 뿐만 아니라 "|"(Vertical Bar)를
구분자를 사용하여 복수의 도메인에 복수의 타겟을 지정할 수 있다.
만약 도메인 이름이 생략되었다면 최근 사용된 도메인을 사용한다. ::

    http://127.0.0.1:20040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:20040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:20040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:20040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:20040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:20040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html

결과는 JSON형식으로 제공된다.
타겟 컨텐츠 개수/용량 및 처리시간(단위: ms)이 명시된다.
이미 Purge 된 컨텐츠는 다시 Purge되지 않는다. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }

``<Purge2Expire>`` 를 통해 특정조건의 Purge를 Expire로 동작하도록 설정할 수 있다.
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다.


.. note::

   원본서버가 장애로 인해 모두 배제되었다면 컨텐츠를 갱신할 수 없기 때문에 Purge가 동작하지 않는다.


.. _api-cmd-expire:

Expire
====================================

타겟 컨텐츠의 TTL을 즉시 만료시킨다.
Expire후 최초 접근 시점에 원본서버로부터 변경여부를 확인한다.
변경되지 않았다면 TTL연장만 있을 뿐 컨텐츠 다운로드는 발생하지 않는다. ::

    http://127.0.0.1:20040/command/expire?url=...

그 외의 모든 동작은 `Purge`_ 와 동일하다.


.. _api-cmd-expireafter:

ExpireAfter
====================================

타겟 컨텐츠의 TTL만료 시간을 현재(API호출시점)로부터 입력된 시간(초)만큼 뒤에 설정한다.
ExpireAfter로 만료시간을 앞당겨 컨텐츠를 더 빨리 갱신하거나,
반대로 만료시간을 늘려 원본서버 부하를 줄일 수 있다. ::

   http://127.0.0.1:20040/command/expireafter?sec=86400&url=...

함수 호출규격은 `Purge`_ / `Expire`_ 와 유사하지만 sec파라미터(단위: 초)를 통해
TTL만료 시간을 지정할 수 있다.
sec가 생략된다면 기본 값은 1일(86400초)로 설정되며 0을 입력할 경우 실패한다.
결과는 `Purge`_ / `Expire`_ 와 동일하지만 원본서버 장애여부와 상관없이 동작한다.
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다.

.. note::
   ExpireAfter는 캐싱되어있는 컨텐츠의 현재 만료시간만을 설정할 뿐 커스텀TTL이나
   설정된 기본 TTL을 변경시키는 API가 아니다.
   ExpireAfter 호출뒤에 캐싱된 컨텐츠들은 영향을 받지 않는다.


   url파라미터를 먼저 입력하는 경우 sec파라미터가 url파라미터의 QueryString으로 인식될 수 있다.
   그러므로 sec파라미터가 먼저 입력되는 것이 안전하다.



.. _api-cmd-hardpurge:

HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 이상의 API는 원본서버 장애상황에서도 컨텐츠가
사라지지 않고 정상적으로 동작한다.
하지만 HardPurge는 컨텐츠의 완전한 삭제를 의미한다.
HardPurge는 가장 강력한 삭제방법이지만 삭제한 컨텐츠는 원본서버에 장애가 발생해도 되살릴 수 없다.
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다. ::

    http://127.0.0.1:20040/command/hardpurge?url=...



.. _api-monitoring-fileinfo:
   
정책설정
====================================


관리자는 다음과 같이 몇가지 동작방식에 대해 설정할 수 있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>

-  ``<Purge2Expire> (기본: NONE)``

   `Purge`_ 요청을 설정에 따라 `Expire`_ 로 처리한다.
   예를 들어 특정 패턴(*.jpg)를 `Purge`_ 하는 경우 의도하지 않게 많은 컨텐츠가
   삭제되어 원본에 과도한 부하를 발생시킬 수 있다.
   이런 경우 `Expire`_ 로 처리하도록 설정하면 과도한 원본부하를 방지할 수 있다.

   - ``NONE`` `Expire`_ 로 처리하지 않는다.
   - ``ROOT`` 도메인 전체(/*)에 대한 `Purge`_ 를 `Expire`_ 로 처리한다.
   - ``PATTERN`` 모든 패턴 `Purge`_ 를 `Expire`_ 로 처리한다.
   - ``ALL`` 모든 `Purge`_ 를 `Expire`_ 로 처리한다.

-  ``<RootPurgeExpire> (기본: ON)``

   전체 콘텐츠에 대한 의도하지 않은 `Purge`_ / `Expire`_ 는 과도한 원본서버 부하를 발생시킬 수 있다.
   이 설정을 통하여 전체 콘텐츠에 대한 `Purge`_ / `Expire`_ 를 차단할 수 있다.
   이 설정은 ``<Purge2Expire>`` 보다 우선한다.

   - ``ON`` `Purge`_ / `Expire`_ 를 허용한다.
   - ``PURGE`` `Purge`_ 만 허용한다.
   - ``EXPIRE`` `Expire`_ 만 허용한다.
   - ``OFF`` 모든 `Purge`_ / `Expire`_ 를 금지한다.

-  ``<ResCodeNoCtrlTarget> (기본: 200)``

   `Purge`_ , `Expire`_ , `HardPurge`_ , `ExpireAfter`_ 의 대상객체가 없을 때의
   HTTP 응답코드를 설정한다.


대상 지정은 URL, 패턴 2가지로 표현한다. ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // 패턴
   example.com/img/*         // 패턴

명확한 URL 외에 패턴(*.jpg)으로 무효화가 가능하다.
하지만 작업을 수행하기 전까지 대상개수를 명확히 알 수 없다.
이는 자칫 관리자의 의도와 다르게 너무 많은 대상을 지정할 수 있다.
이는 실제로 CPU자원을 너무 많이 소모하게 되어 시스템 전체에 부담을 줄 수 있다.

그러므로 실 서비스 중에는 명확한 URL만을 사용할 것을 강력히 권장한다.
패턴표현은 서비스에서 배제된 상태에서 관리용도로 사용하기 위함이다.


.. note::

   보안적인 이유로 example.com/files/ 같은 특정 디렉토리에 대한 접근은 403 FORBIDDEN등으로 차단된다.
   하지만 루트 디렉토리는 예외를 가진다.
   예를 들어 사용자가 example.com에 접근하면 브라우저는 루트 디렉토리(/)를 요청한다. ::

      GET / HTTP/1.1
      Host: example.com

   이에 대해 웹서버는 관리자가 설정한 기본 페이지(아마도 index.html 또는 index.htm)로 응답한다.
   분명 웹 서비스 구성에서 루트 디렉토리(/)는 디렉토리가 아닌 페이지로 동작한다.

   하지만 Cache서버는 루트 디렉토리(/)에 접근했더니 200 OK 페이지가 왔다고 이해한다.
   심지어 원본서버가 어떤 페이지를 응답했는지 알지 못한다.
   간단히 정리하면 Cache서버의 관점에서는 디렉토리 표현도 URL의 한 종류일 뿐이다. ::

      example.com/img/          // example.com 가상호스트의 /img/ 에 접근한 결과 페이지
      example.com/              // example.com 가상호스트의 기본 페이지(/)
      example.com/img/*         // example.com 가상호스트의 /img 디렉토리와 그 하위 페이지
      example.com/*             // example.com 가상호스트의 모든 콘텐츠




Purge 기본동작
====================================

Purge API가 호출될 때 컨텐츠 복구 여부를 선택한다. ::

   # server.xml - <Server><Cache>

   <Purge>Normal</Purge>

-  ``<Purge>``

   - ``Normal (기본)`` `Purge`_ 로 동작한다. (원본장애 시 복구 함)

   - ``Hard`` `HardPurge`_ 로 동작한다. (원본장애 시 복구하지 않음)


HTTP Method
====================================

무효화 API를 확장 HTTP Method로 호출할 수 있다. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr

HTTP Method는 기본적으로 Manager포트와 서비스(80)포트에서 동작한다.
서비스포트로 요청되는 HTTP Method의 :ref:`env-host` 에서 설정한다.


.. _api-etc-post:

POST 규격
====================================

무효화 API를 다음과 같이 POST로 호출할 수 있다. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37

   url=http://ston.winesoft.co.kr/sample.dat
