﻿.. _vhost:

4장. 가상호스트
******************

이 장에서는 가상호스트와 URL표현에 대해 설명한다.
가상호스트는 서비스의 기본단위로 vhosts.xml을 통해 설정하며 개수제한은 없다.

가상호스트가 생성되면 멀티프로토콜이 사용하는 포트(HTTP/HLS=80, RTMP=1935)가 오픈된다.
가상호스트 설정에 따라 제공되는 멀티프로토콜 URL(Uniform Resource Locator)표현이 조금씩 차이난다.
가상호스트는 클라이언트가 요청하는 URL을 통해 어떤 프로토콜을 사용해야 하는지 알 수 있다.

.. note:

   STON 미디어 서버의 URL 표현은 Adobe 미디어 서버(= Flash 미디어 서버)와 완벽히 호환된다.


프로토콜에 따라 지원되는 미디어 포맷은 아래와 같다.

========================== =============== =============== ===============
프로토콜                     확장자            Video Codec     Audio Codec
========================== =============== =============== ===============
RTMP                       .mp4            H.264           AAC
Apple HLS                  .mp4            H.264           AAC
HTTP Pseudo-Streaming      .mp4            H.264           AAC
========================== =============== =============== ===============


.. toctree::
   :maxdepth: 2



.. _vhost-create-destroy:

생성/파괴
====================================

``<Vhosts>`` 하위에 ``<Vhost>`` 로 가상호스트를 설정한다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Name="www.example.com"> ... </Vhost>
        <Vhost Name="/foo" Status="Active"> ... </Vhost>
        <Vhost Name="www.example.com/bar" Status="Inactive"> ... </Vhost>
        <Vhost Name="/foobar" Prefix="http/"> ... </Vhost>
    </Vhosts>

-  ``<Vhost>`` 가상호스트를 설정한다.

   - ``Name`` 가상호스트 이름. 중복될 수 없으며 3가지 형태로 구성이 가능하다.

      - 도메인 (www.example.com)
      - 1 depth 디렉토리 (/foo)
      - 도메인 (www.example.com) + 1depth 디렉토리(/bar)

   - ``Status (기본: Active)`` Inactive인 경우 해당 가상호스트는 임시적으로 서비스되지 않는다.

   - ``Prefix`` 기존주소와 호환성을 맞추기 위한 문자열 설정

가상호스트가 생성되면 멀티프로토콜(HTTP, HLS, RTMP) 서비스가 기본으로 활성화된다.

.. note::

   1 depth 디렉토리 ``Name`` 표현은 Adobe 미디어 서버의 Application과 같은 개념이다.


``<Vhost>`` 를 삭제하면 해당 가상호스트가 삭제된다.
삭제된 가상호스트의 모든 콘텐츠는 삭제된다.
다시 추가해도 콘텐츠는 되살아나지 않는다.




.. _vhost-alias:

Alias
====================================

하나의 가상호스트를 여러 이름으로 서비스하고 싶다면 ``<Alias>`` 를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Alias>sample.com</Alias>
        <Alias>*.sub.example.com</Alias>
        <Alias>/myvideo</Alias>
    </Vhost>

-  ``<Alias>``

   가상호스트의 별명(Alias)을 설정하며 개수에 제한은 없다.
   명확한 표현(sample.com)과 패턴표현(*.sub.example.com)을 지원한다.
   패턴은 복잡한 정규표현식이 아닌 prefix에 * 표현을 하나만 붙일 수 있는 간단한 형식만을 지원한다.


가상호스트 검색 순서는 다음과 같다.

1. ``<Vhost>`` 의 ``Name`` 과 일치하는가?
2. 명시적인 ``<Alias>`` 와 일치하는가?
3. 패턴 ``<Alias>`` 를 만족하는가?




.. _vhost-defaultvhost:

Default 가상호스트
====================================

클라이언트 요청을 처리할 가상호스트를 찾지못한 경우 서비스를 제공할 기본 가상호스트를 지정한다.
클라이언트 요청을 처리하고 싶지 않다면 설정하지 않아도 된다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Name="www.example.com"> ... </Vhost>
        <Vhost Name="/foo"> ... </Vhost>
        <Vhost Name="www.example.com/bar"> ... </Vhost>
        <Default>/foo</Default>
    </Vhosts>

-  ``<Default>``

   기본 가상호스트 이름을 설정한다.
   반드시 ``<Vhost>`` 의 ``Name`` 속성과 같아야 한다.



.. _vhost-serviceport:

서비스 포트
====================================

프로토콜별로 서비스 포트를 설정한다. 기본 포트로 HTTP는 80, RTMP는 1935를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>
          <Http>*:80</Http>
          <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

포트설정에 제한은 없지만, 이미 특정 프로토콜에 바인딩된 포트를 다른 프로토콜에서 열 수 없다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="/foo">
        <Listen>
            <Http>*:80</Http>
            <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

    <Vhost Name="www.example.com/bar">
        <Listen>
            <Http>*:8080</Http>   // 가능
            <Rtmp>*:80</Rtmp>     // 불가능 - 이미 HTTP에서 사용
        </Listen>
    </Vhost>



.. _vhost-api_vhostslist:

목록조회
====================================

가상호스트 목록을 조회한다. ::

   http://127.0.0.1:20040/monitoring/vhostslist

결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.0.0",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","/foo", "www.example.com/bar" ]
   }