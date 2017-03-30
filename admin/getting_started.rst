.. _getting-started:

2장. 서버구성과 설치
******************

이 장에서는 시스템 구성/설치 그리고 예제 가상호스트까지 구성해본다
텍스트 편집기만 있으면 누구나 할 수 있다.

STON 미디어 서버는 표준 Linux 서버에서 동작하도록 개발되었다.
개발 단계부터 HW뿐만 아니라 OS, 파일시스템 등 종속성을 가질 수 있는 요소는 최대한 배제하였다.
고객이 합리적인 장비를 선택할 수 있도록 돕는 것은 매우 중요하다.
왜냐하면 서비스의 특성과 규모에 따라 적절한 서버를 구성하는 것이 서비스의 시작이기 때문이다.


.. toctree::
   :maxdepth: 2


.. _getting-started-serverconf:

서버 구성
====================================

CPU, 메모리, 디스크 자원에 대해 고려해야 한다.

-  **CPU**

   코어가 많을수록 더 많은 미디어에 대한 동시처리가 가능하다.
   8코어 이상을 추천한다.


-  **메모리**

   메모리-인덱싱 방식을 사용하므로 8GB이상을 권장한다.
   자주 요청되는 파일은 항상 메모리에 상주하지만 그렇지 않은 파일은 디스크에서 로딩한다.
   따라서 파일이 많고 집중도가 낮다면(Long-tail) 디스크 부하 증가로 성능이 저하될 수 있다.
   파일 크기가 커서 디스크 I/O 부하가 높다면 메모리를 증설하거나 SSD를 선택하는 것이 좋은 옵션이 될 수 있다.


-  **디스크**

   OS를 포함하여 최소 3개 이상을 권장한다.
   디스크 역시 많으면 많을수록 많은 파일을 캐싱할 수 있을뿐만 아니라 I/O부하도 분산된다.

   .. figure:: img/02_disk.png
      :align: center

      항상 OS와 STON은 콘텐츠와 별도의 디스크로 구성한다.

   일반적으로 OS가 설치된 디스크에 STON을 설치한다.
   로그 역시 같은 디스크에 구성하는 것이 일반적이다.
   로그는 서비스 상황을 실시간으로 기록하기 때문에 항상 Write부하가 발생한다.

   STON 미디어 서버는 디스크를 RAID 0처럼 사용한다.
   성능과 RAID의 상관여부는 고객 서비스 특성에 따라 달라진다.
   하지만 파일 변경이 빈번하지 않고 콘텐츠의 크기가 물리적 메모리 크기보다
   훨씬 큰 경우 RAID를 통한 Read속도 향상이 효과적일 수 있다.


.. _getting-started-os:

OS 구성
====================================

가장 기본적인 형태로 설치한다.
표준 64bit Linux 배포판(Cent 6.2이상, Ubuntu 10.04이상) 이라면 정상동작한다.
패키지 의존성을 가지지 않는다.


.. _getting-started-install:

설치
====================================

1. 최신버전의 STON 미디어 서버를 다운로드 받는다. ::

      [root@localhost ~]# wget  http://foobar.com/sms/STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz
      --2017-02-24 13:29:14--  http://foobar.com/sms/STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz
      Resolving foobar.com... 192.168.0.14
      Connecting to foobar.com|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz”

      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s

      2017-02-24 13:29:15 (42.9 MB/s) - “STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 압축을 해지한다. ::

		[root@localhost ~]# tar -zxf STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz

3. 설치 스크립트를 실행한다. ::

		[root@localhost ~]# ./STONMediaServer.1.0.0.rhel.2.6.32.x64.sh

4. 설치과정은 install.log에 기록된다. 로그를 통해 설치 중 발생하는 문제를 알 수 있다. ::

      #DownloadURL: http://foobar.com/sms/STONMediaServer.1.0.0.rhel.2.6.32.x64.tar.gz
      #DownloadTime: 13 sec
      #Target: STONMediaServer 1.0.0
      #Date: 2017.03.03 16:48:35
      Prepare for STONMediaServer 1.0.0 install process
          Stopping STONMediaServer...
              STONMediaServer stopped
      [Copying files]
          `./fuse.conf' -> `/etc/fuse.conf'
          `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
          `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
          `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
          `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
          `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
          `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
          `./libtbb.so' -> `/usr/local/ston/libtbb.so'
          `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
          `./stond' -> `/usr/local/ston/stond'
          `./stonx' -> `/usr/local/ston/stonx'
          `./stonr' -> `/usr/local/ston/stonr'
          `./stonu' -> `/usr/local/ston/stonu'
          `./stonmapi' -> `/usr/local/ston/stonmapi'
          `./server.xml.default' -> `/usr/local/ston/server.xml.default'
          `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
          `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
          `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
          `./wm.sh' -> `/usr/local/ston/wm.sh'
      [Exporting config files]
          #Export so directory
          /usr/local/ston/ to ld.so.conf
          #Export sysctl to /etc/sysctl.conf
          vm.swappiness=0
          vm.min_free_kbytes=524288
          #Export sudoers for WM
          Defaults    !requiretty
          winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
      [Configuring STON daemon script]
          STON deamon activate in run-level 2345.
      [Installing sub-packages]
          curl installed.
          libjpeg installed.
          libgomp installed.
          rrdtool installed.
      [Installing WM]
          Stopping WM...
          WM stopped
          `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
          `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
          WM configuration found. Current WM port : 8500
          PHP module for Legacy(CentOS 5.5) installed
          `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
          WM installation almost complete. Changing WM privileges.
      Installation successfully complete


.. _getting-started-license:

라이선스 발급
====================================

신규 고객의 경우 다음 절차를 통해 라이선스를 발급한다.

* `신청양식 <http://ston.winesoft.co.kr/EULR.doc>`_ 작성
* license@winesoft.co.kr 로 전송
* 확인절차 후 발급

라이선스 파일(license.xml)이 반드시 설치경로에 존재해야 STON 미디어 서버가 정상적으로 구동된다.


.. _getting-started-update:

업데이트
====================================
최신버전이 배포되면 stonu명령어로 업데이트할 수 있다. ::

	./stonu 1.1.0

또는 :ref:`wm` 의 :ref:`wm-update` 를 통해 간편하게 업데이트를 진행할 수 있다.

   .. figure:: img/sms_wm_update.png
      :align: center


.. _getting-started-run:

실행하기
====================================

STON 미디어 서버는 다음 경로에 설치된다. ::

    /usr/local/StonMediaServer

다음 파일 중 하나라도 존재하지 않거나 XML문법에 맞지 않을 경우 실행되지 않는다.

- license.xml
- server.xml
- vhosts.xml

최초 설치시 모든 XML파일이 존재하지 않는다.
배포받은 라이센스파일을 설치 경로에 복사한다.
그리고 설치경로의 server.xml.default와 vhosts.xml.default를 복사 또는 수정하여 설정하길 바란다.
*.default파일은 항상 최신패키지와 함께 배포된다.



.. _api-etc-help:

API 호출
====================================

HTTP 기반의 API를 제공한다.
API 호출권한은 :ref:`env-host` 의 영향을 받는다.
허가되지 않았다면 곧바로 연결을 종료한다.

STON 미디어 서버 버전을 확인한다. ::

   http://127.0.0.1:20040/version

같은 API를 Linux Shell에서 명령어로 수행한다. ::

   ./stonmapi version

.. note:

   HTTP API는 &를 QueryString의 구분자로 인식하지만 Linux 콘솔에서는 다른 의미를 가진다.
   &가 들어가는 명령어를 호출하는 경우 \&로 입려하거나 반드시 괄호(" /...&... ")로 호출하는 URL을 묶어야 한다.


하드웨어 정보조회
====================================

하드웨어 정보를 조회한다. ::

   http://127.0.0.1:20040/monitoring/hwinfo

결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.0.0",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(생략)...",
         "STON" : "1.0.0",
         "CPU" :
         {
            "ProcCount": "4",
            "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz",
            "MHz": "1200.000",
            "Cache": "8192 KB"
         },
         "Memory" : "8 GB",
         "NIC" :
         [
            {
               "Dev" : "eth1",
               "Model" : "Intel Corporation 82574L Gigabit Network Connection",
               "IP" : "192.168.0.13",
               "MAC" : "00:25:90:36:f4:cb"
            }
         ],
         "Disk" :
         [
            {
               "Dev" : "sda",
               "Model" : "HP DG0146FAMWL (scsi)",
               "Total" : "238787584",
               "Usage" : "40181760"
            },
            {
               "Dev" : "sdb",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2101075968"
            },
            {
               "Dev" : "sdc",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2012160000"
            }
         ]
      }
   }


재시작/종료
====================================

명령어를 통해 STON 미디어 서버를 재시작/종료할 수 있다.
의도하지 않은 결과를 피하기 위해 웹 페이지를 통한 확인작업이 반드시 필요하도록 개발되었다. ::

   http://127.0.0.1:20040/command/restart
   http://127.0.0.1:20040/command/restart?key=JUSTDOIT       // 즉시 실행
   http://127.0.0.1:20040/command/terminate
   http://127.0.0.1:20040/command/terminate?key=JUSTDOIT       // 즉시 실행


.. _getting-started-reset:

Caching 초기화
====================================

서비스를 중단하며 캐싱된 모든 컨텐츠를 삭제한다.
설정된 모든 디스크를 포맷하며 작업이 완료되면 다시 서비스를 재개한다. ::

   http://127.0.0.1:20040/command/cacheclear
   http://127.0.0.1:20040/command/cacheclear?key=JUSTDOIT       // 즉시 실행

콘솔에서는 다음 명령어를 통해 전체 또는 하나의 가상호스트를 초기화한다. ::

   ./stonmapi reset
   ./stonmapi reset/www.example.com
