.. _snmp:

11장. SNMP
******************

이 장에서는 SNMP(Simple Network Management Protocol)에 대해 다룬다.
:ref:`monitoring_stats` 의 모든 수치는 SNMP로도 제공된다.
뿐만 아니라 더욱 세분화된 시간단위와 시스템 상태정보까지 제공한다.
가상호스트별로 실시간 통계와 최대 60분까지 "분" 단위의 평균 통계를 제공한다. 

- 별도의 패키지가 필요없다.
- snmpd를 별도로 실행하지 않는다.
- SNMP v1과 v2c를 지원한다.

.. toctree::
   :maxdepth: 2


.. _snmp-var:

변수
====================================

설정이나 사용자의 의도에 의하여 변경될 수 있는 값을 [변수명]으로 명시한다. 
예를 들어 디스크는 여러개가 존재할 수 있다. 
이 경우 각 디스크를 가리키는 고유 번호가 필요하며 입력된 순서대로 1부터 할당된다. 
이런 변수를 ``[diskIndex]`` 로 명시한다.

-  ``[diskIndex]``

   Storage에 설정된 디스크를 의미한다. ::
   
      # server.xml - <Server><Cache>
   
      <Storage>
         <Disk>/cache1</Disk>
         <Disk>/cache2</Disk>
         <Disk>/cache3</Disk>
      </Storage>
      
   위와 같이 3개의 디스크가 설정된 환경에서 /cache1의 
   ``[diskIndex]`` 는 1, /cache3의 ``[diskIndex]`` 는 3을 가진다. 
   예를 들어 /cache1의 전체용량에 해당하는 OID는 
   system.diskInfo.diskInfoTotalSize.1 
   (1.3.6.1.4.1.40002.1.2.18.1.3.1이 된다. 
   마지막 .1은 첫번째 디스크를 의미한다.
   
-  ``[vhostIndex]`` 

   가상호스트가 로딩될 때 자동으로 부여된다. ::
   
      # vhosts.xml
   
      <Vhosts>
         <Vhost Status="Active" Name="kim.com"> ... </Vhost>
         <Vhost Status="Active" Name="lee.com"> ... </Vhost>
         <Vhost Status="Active" Name="park.com" StaticIndex="10300"> ... </Vhost>
      </Vhosts>
   
   최초 위와 같이 3개의 가상호스트가 로딩되면 1부터 순차적으로  ``[vhostIndex]`` 가 부여된다. 
   이후 가상호스트는  ``[vhostIndex]`` 를 기억하며, 가상호스트가 삭제되더라도  ``[vhostIndex]`` 는 변하지 않는다. 
   가상호스트의 삭제와 추가가 동시에 발생할 경우 삭제가 먼저 동작하며, 
   신규 추가된 가상호스트는 비어있는  ``[vhostIndex]`` 를 부여 받는다.
   
   .. figure:: img/snmp_vhostindex.png
      :align: center
      
      ``[vhostIndex]`` 의 동작방식

-  ``[diskMin]`` , ``[vhostMin]`` 

   시간(분)을 의미한다. 
   5는 5분의 평균을 의미하며 60은 60분의 평균을 의미한다. 
   이 값은 1(분)부터 60(분)까지 범위를 가지며 0은 실시간(1초) 데이터를 의미한다.
   
SNMP에서는 동적으로 값이 바뀔 수 있는 항목에 대하여 Table구조를 사용한다. 
예를 들어 "디스크 전체크기"는 디스크의 개수에 따라 제공하는 데이터 개수가 
달라지기 때문에 Table구조를 사용하여 표현해야 한다. 
STON은 모든 가상호스트에 대하여 "분"단위 통계를 제공한다. 
그러므로 ``[vhostMin]`` . ``[vhostIndex]`` 라는 다소 난해한 표현을 제공한다. 

이 표현은 가상호스트별로 원하는 "분" 단위의 통계를 볼 수 있다는 장점을 가지고 있지만 
변수가 2개이므로 Table구조로 표현하기 어렵다는 단점이 있다. 
이런 문제를 극복하기 위하여  ``[vhostMin]`` 의 기본값을 설정하여 
SNMPWalk가 동작할 수 있도록 한다.


.. _snmp-conf:

활성화
====================================

전역설정(server.xml)을 통해 SNMP동작방식과 ACL을 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP Port="161" Status="Inactive">
      <Allow>192.168.5.1</Allow>
      <Allow>192.168.6.0/24</Allow>    
   </SNMP>   

-  ``<SNMP>`` 속성을 통해 SNMP의 동작방식을 설정한다.

   - ``Port (기본: 161)`` SNMP 서비스 포트
   
   - ``Status (기본: Inactive)`` SNMP를 활성화 하려면 이 값을 ``Active`` 로 설정한다.
   
-  ``<Allow>`` SNMP접근을 허가할 IP주소를 설정한다. 
    IP지정, IP범위지정, 비트마스크, 서브넷 이상 네 가지 형식을 지원한다. 
    접속한 소켓이 허가된 IP가 아니면 응답을 주지 않는다.
    
    

가상호스트/View 변수
====================================

SNMP를 통해 제공되는 가상호스트/View 개수와 기본시간(분)을 설정한다. ::

   # server.xml - <Server><Host>
   
   <SNMP VHostCount=0, VHostMin=5 ViewCount=0, ViewMin=5 />

-  ``VHostCount (기본: 0)`` 0일 경우 존재하는 가상호스트까지만 응답을 한다. 
   0보다 큰 값일 경우 가상호스트 존재 유무에 상관없이 설정된 가상호스트까지 응답한다. 
   
-  ``ViewCount (기본: 0)`` View에 적용. ( ``VHostCount`` 와 동일)
   
-  ``VHostMin (기본: 5분, 최대: 60분)``  ``[vhostMin]``  값을 설정한다. 
   0~60까지의 값을 가진다. 
   0일 경우 실시간 데이터를 제공하하며 1~60사이인 경우 해당 분만큼의 평균값을 제공한다.
   
-  ``ViewMin (기본: 0)`` View에 적용. ( ``VHostMin`` 와 동일)

예를 들어 3개의 가상호스트가 설정되어 있는 환경에서 SNMPWalk의 동작방식이 달라진다.

- VHostCount=0인 경우 ::

    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    
- VHostCount=5 경우 ::

    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.4 = ""
    SNMPv2-SMI::enterprises.40002.1.4.2.1.2.5 = ""



기타 변수
---------------------

기타 변수를 설정한다. ::

   # server.xml - <Server><Host>
   
   <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />
    
-  ``GlobalMin (기본: 5분, 최대: 60분)``  ``[globalMin]``  값을 설정한다.

-  ``DiskMin (기본: 5분, 최대: 60분)``  ``[diskMin]``  값을 설정한다.

-  ``ConfCount (기본: 10)`` 설정목록을 n개까지 열람한다. 
   1~100사이에서 지정 가능하다. 
   1은 현재 반영된 설정을 의미하며 2는 이전 설정을 의미한다. 
   100은 현재를 기준으로 99번 이전의 설정을 의미한다.
   


Community
====================================

Community를 설정하여 허가된 OID에만 접근/차단되도록 설정한다. ::

   # server.xml - <Server><Host>

   <SNMP UnregisteredCommunity="Allow">
      <Community Name="example1" OID="Allow">
         <OID>1.3.6.1.4.1.40002.1.4.1</OID>
         <OID>1.3.6.1.4.1.40002.1.4.2</OID>
         <OID>1.3.6.1.4.1.40002.1.4.4</OID>
      </Community>
      <Community Name="example2" OID="Deny">
         <OID>1.3.6.1.4.1.40002.1.4.3.1.11.11.10.1-61</OID>
      </Community>
   </SNMP>
    
``<SNMP>`` 의 ``UnregisteredCommunity`` 를 "Deny"로 설정하면 등록되지 않은 Community 요청은 차단한다.

-  ``<Community>`` Community를 설정한다.

   - ``Name`` Community 이름.
   
   - ``OID (기본: Allow)`` 하위 ``<OID>`` 태그의 값을 설정한다.
     속성 값이 ``Allow`` 라면 하위 ``<OID>`` 목록만 접근 가능하다. 
     반대로 속성 값이 ``Deny`` 라면 하위 <OID>목록에는 접근이 불가능하다.

명시적인 OID(1.3.6.1.4.1.40002.1.4.4)와 범위OID(1.3.6.1.4.1.40002.1.4.3.1.11.11.10.1-61) 표현이 가능하다. 
OID를 허용/차단할 경우 하위 모든 OID에 대해 같은 규칙이 적용된다.



.. _snmp-meta:

meta
====================================

::

   OID = 1.3.6.1.4.1.40002.1.1

메타정보를 제공한다. 

===== ============= ========= ===========================================
OID   Name          Type      Description
===== ============= ========= ===========================================
.1    manufacture   String    "WineSOFT Inc."
.2    software      String    "STON"
.3    version       String    버전
.4    hostname      String    호스트 이름
.5    state         String    "Healthy" 또는 "Inactive" 또는 "Emergency"
.6    uptime        Integer   실행시간 (초)
.7    admin         String    <Admin> ... </Admin>
.10   Conf          OID       Conf 확장
===== ============= ========= ===========================================



.. _snmp-meta-conf:

meta.conf
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.1.10

``[confIndex]`` 는 ``<SNMP>`` 의 ``ConfCount`` 속성에서 설정한다.
``[confIndex]`` 가 1인 경우는 항상 현재 적용된 설정 값을, 
2인 경우는 이전 설정 값을 의미한다. 
10 이라면 현재(1)로부터 9번째 이전의 설정을 의미한다.

==================== ======= ======= =============================================================================================
OID                  Name    Type    Description
==================== ======= ======= =============================================================================================
.1. ``[confIndex]``  ID      Integer 설정 ID
.2. ``[confIndex]``  Time    Integer 설정시간 (Unix 시간)
.3. ``[confIndex]``  Type    Integer 설정형태 (0 = Unknown, 1 = STON 시작, 2 = /conf/reload, 3 = /conf/upload, 4 = /conf/restore)
.4. ``[confIndex]``  Size    Integer 설정파일 크기
.5. ``[confIndex]``  Hash    String  설정파일 Hash문자열
.6. ``[confIndex]``  Path    String  설정파일 저장경로
.7. ``[confIndex]``  Ver     String  설정할 때의 STON 버전
==================== ======= ======= =============================================================================================



.. _snmp-meta-system:

system
====================================

::

   OID = 1.3.6.1.4.1.40002.1.2

STON이 동작하는 시스템 정보를 제공한다.
``[sysMin]`` 변수는 0~60분까지의 값을 가지며 실시간 또는 원하는 시간만큼의 평균 값을 제공한다. 
SNMPWalk에서  ``[sysMin]`` 은 0으로 설정되며 현재 정보를 제공한다.

=================== ========================================= ======= ===============================================
OID                 Name                                      Type    Description
=================== ========================================= ======= ===============================================
.1. ``[sysMin]``    cpuTotal                                  Integer 전체 CPU 사용률 (100%)
.2. ``[sysMin]``                                                      전체 CPU 사용률 (10000%)
.3. ``[sysMin]``    cpuKernel                                 Integer	CPU(Kernel) 사용률 (100%)
.4. ``[sysMin]``                                                      CPU(Kernel) 사용률 (10000%)
.5. ``[sysMin]``    cpuUser                                   Integer CPU(User) 사용률 (100%)
.6. ``[sysMin]``                                                      CPU(User) 사용률 (10000%)
.7. ``[sysMin]``    cpuIdle                                   Integer CPU(Idle) 사용률 (100%)
.8. ``[sysMin]``                                                      CPU(Idle) 사용률 (10000%)
.9                  memTotal                                  Integer 시스템 전체 메모리 (KB)
.10. ``[sysMin]``   memUse                                    Integer 시스템 사용 메모리 (KB)
.11. ``[sysMin]``   memFree                                   Integer 시스템 여유 메모리 (KB)
.12. ``[sysMin]``   memSTON                                   Integer STON 사용 메모리 (KB)
.13. ``[sysMin]``   memUseRatio                               Integer 시스템 메모리 사용률 (100%)
.14. ``[sysMin]``                                                     시스템 메모리 사용률 (10000%)
.15. ``[sysMin]``   memSTONRatio                              Integer STON 메모리 사용률 (100%)
.16. ``[sysMin]``                                                     STON 메모리 사용률 (10000%)
.17                 diskCount                                 Integer disk개수
.18.1               diskInfo                                  OID     diskInfo확장
.19.1               diskPerf                                  OID     diskPerf확장
.20. ``[sysMin]``   cpuProcKernel                             Integer STON이 사용하는 CPU(Kernel) 사용률 (100%)
.21. ``[sysMin]``                                                     STON이 사용하는 CPU(Kernel) 사용률 (10000%)
.22. ``[sysMin]``   cpuProcUser                               Integer STON이 사용하는 CPU(User) 사용률 (100%)
.23. ``[sysMin]``                                                     STON이 사용하는 CPU(User) 사용률 (10000%)
.24. ``[sysMin]``   sysLoadAverage                            Integer Load Average 1분 평균 (0.01)
.25. ``[sysMin]``                                                     Load Average 5분 평균 (0.01)
.26. ``[sysMin]``                                                     Load Average 15분 평균 (0.01)
.27. ``[sysMin]``   cpuNice                                   Integer CPU(Nice) (100%)
.28. ``[sysMin]``                                                     CPU(Nice) (10000%)
.29. ``[sysMin]``   cpuIOWait                                 Integer CPU(IOWait) (100%)
.30. ``[sysMin]``                                                     CPU(IOWait) (10000%)
.31. ``[sysMin]``   cpuIRQ                                    Integer CPU(IRQ) (100%)
.32. ``[sysMin]``                                                     CPU(IRQ) (10000%)
.33. ``[sysMin]``   cpuSoftIRQ                                Integer CPU(SoftIRQ) (100%)
.34. ``[sysMin]``                                                     CPU(SoftIRQ) (10000%)
.35. ``[sysMin]``   cpuSteal                                  Integer CPU(Steal) (100%)
.36. ``[sysMin]``   CPU(Steal)                                Integer (10000%)
.40. ``[sysMin]``   TCPSocket.Established. ``[globalMin]``    Integer Established상태의 TCP 연결개수
.41. ``[sysMin]``   TCPSocket.Timewait. ``[globalMin]``       Integer TIME_WAIT 상태의 TCP 연결개수
.42. ``[sysMin]``   TCPSocket.Orphan. ``[globalMin]``         Integer 아직 file handle에 attach되지 않은 TCP 연결
.43. ``[sysMin]``   TCPSocket.Alloc. ``[globalMin]``          Integer 할당된 TCP 연결
.44. ``[sysMin]``   TCPSocket.Mem. ``[globalMin]``            Integer undocumented
=================== ========================================= ======= ===============================================



.. _snmp-meta-system-diskinfo:
                                    
system.diskInfo
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.2.18.1

디스크 정보를 제공한다.

======================= ================== =========== =========================================
OID                     Name               Type        Description
======================= ================== =========== =========================================
.2. ``[diskIndex]``     diskInfoPath       String      디스크 경로                                 
.3. ``[diskIndex]``     diskInfoTotalSize  Integer     디스크 전체용량 (MB)                    
.4. ``[diskIndex]``     diskInfoUseSize    Integer     디스크 사용량 (MB)                          
.5. ``[diskIndex]``     diskInfoFreeSize   Integer     디스크 사용 가능량 (MB)                 
.6. ``[diskIndex]``     diskInfoUseRatio   Integer     디스크 사용률 (100%)                    
.7. ``[diskIndex]``                                    디스크 사용률 (10000%)                                              
.8. ``[diskIndex]``     diskInfoStatus     String      "Normal" 또는 "Invalid" 또는 "Unmounted"
======================= ================== =========== =========================================



.. _snmp-meta-system-diskperf:
                                    
system.diskPerf
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.2.19.1

디스크 성능상태를 제공한다.

======================================== =========================== ========== ===============================
OID                                      Name                        Type       Description
======================================== =========================== ========== ===============================
.2. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadCount           Integer    읽기 성공 횟수
.3. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadMergedCount     Integer    읽기가 병합된 횟수
.4. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadSectorsCount    Integer    읽은 섹터 수
.5. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadTime            Integer    읽기 소요시간(ms)
.6. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteCount          Integer    쓰기 성공 횟수
.7. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteMergedCount    Integer    쓰기가 병합된 횟수
.8. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteSectorsCount   Integer    써진 섹터 수
.9. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteTime           Integer    쓰기 소요시간(ms)
.10. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOProgressCount     Integer    진행 중인 IO개수
.11. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTime              Integer    IO 소요시간(ms)
.12. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTimeWeighted      Integer    IO 소요시간(ms, 가중치 적용)
======================================== =========================== ========== ===============================



.. _snmp-global:

global
====================================

::

   OID = 1.3.6.1.4.1.40002.1.3

STON의 모든 모듈이 공통적으로 사용하는 자원정보(소켓, 이벤트 등)를 제공한다. 

-  **ServerSocket**
   
   클라이언트 ~ STON구간. STON이 클라이언트의 요청을 처리할 용도로 사용하는 소켓
   
-  **ClientSocket**

   STON ~ 원본서버구간. STON이 원본서버로 요청을 보내는 용도로 사용하는 소켓

===== =========================================== ========== ==================================================
OID   Name                                        Type       Description
===== =========================================== ========== ==================================================
.5    EQ. ``[globalMin]``                         Integer    STON Framework에서 아직 처리되지 않은 Event개수
.6    RQ. ``[globalMin]``                         Integer    최근 서비스된 컨텐츠 참조 큐에 저장된 Event 개수
.7    waitingFiles2Write. ``[globalMin]``         Integer    쓰기대기 중인 파일개수
.10   ServerSocket.Total. ``[globalMin]``         Integer    전체 서버소켓 수
.11   ServerSocket.Established. ``[globalMin]``   Integer    연결된 상태의 서버소켓 수
.12   ServerSocket.Accepted. ``[globalMin]``      Integer    새롭게 연결된 서버소켓 수
.13   ServerSocket.Closed. ``[globalMin]``        Integer    연결이 종료된 서버소켓 수
.20   ClientSocket.Total. ``[globalMin]``         Integer    전체 클라이언트소켓 수
.21   ClientSocket.Established. ``[globalMin]``   Integer    연결된 상태의 클라이언트소켓 수
.22   ClientSocket.Accepted. ``[globalMin]``      Integer    새롭게 연결된 클라이언트소켓 수
.23   ClientSocket.Closed. ``[globalMin]``        Integer    연결이 종료된 클라이언트소켓 수
.30   ServiceAccess.Allow. ``[globalMin]``        Integer    ServiceAccess에 의해 허가(Allow)된 소켓 수
.31   ServiceAccess.Deny. ``[globalMin]``         Integer    ServiceAccess에 의해 거부(Deny)된 소켓 수
===== =========================================== ========== ==================================================



.. _snmp-cache:

cache
====================================

::
   
    OID = 1.3.6.1.4.1.40002.1.4

캐시 서비스의 통계는 가상호스트별로 상세하게 수집/제공된다.

====== ============== ========= ============================================================
OID    Name           Type      Description
====== ============== ========= ============================================================
.1     host           OID       호스트 (확장)
.2     vhostCount     Integer   가상호스트 개수
.3.1   vhost          OID       가상호스트별 통계
.4     vhostIndexMax  Integer    ``[vhostIndex]``  최대 값. SNMPWalk는 이 수치를 기준으로 동작한다.
.10    viewCount      Integer   View 개수
.11.1  view           OID       View별 통계
.12    viewIndexMax   Integer   [viewIndex] 최대 값. SNMPWalk는 이 수치를 기준으로 동작한다.
====== ============== ========= ============================================================



.. _snmp-cache-host:

cache.host
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.1

호스트(=모든 가상호스트)의 정보를 제공한다.

===== ========= =========== =========================
OID   Name      Type        Description
===== ========= =========== =========================
.2    name      String      호스트 이름
.3    status    String      "Healthy" 또는 "Inactive"
.4    uptime    Integer     STON 실행시간 (초)
.10   contents  OID         컨텐츠 정보 (확장)
.11   traffic   OID         통계 (확장)
===== ========= =========== =========================



.. _snmp-cache-host-contents:

cache.host.contents
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.1.10

호스트(=모든 가상호스트)가 서비스하는 컨텐츠 통계를 제공한다.

====== ================ ========== ============================
OID    Name             Type       Description
====== ================ ========== ============================
.1     memory           Integer    메모리 캐싱 크기(KB)
.2     filesTotalCount  Integer    서비스 중인 파일개수
.3     filesTotalSize   Integer    서비스 중인 전체 파일량(MB)
.10    filesCountU1KB   Integer    1KB미만 파일개수
.11    filesCountU2KB   Integer    2KB미만 파일개수
.12    filesCountU4KB   Integer    4KB미만 파일개수
.13    filesCountU8KB   Integer    8KB미만 파일개수
.14    filesCountU16KB  Integer    16KB미만 파일개수
.15    filesCountU32KB  Integer    32KB미만 파일개수
.16    filesCountU64KB  Integer    64KB미만 파일개수
.17    filesCountU128KB Integer    128KB미만 파일개수
.18    filesCountU256KB Integer    256KB미만 파일개수
.19    filesCountU512KB Integer    512KB미만 파일개수
.20    filesCountU1MB   Integer    1MB미만 파일개수
.21    filesCountU2MB   Integer    2MB미만 파일개수
.22    filesCountU4MB   Integer    4MB미만 파일개수
.23    filesCountU8MB   Integer    8MB미만 파일개수
.24    filesCountU16MB  Integer    16MB미만 파일개수
.25    filesCountU32MB  Integer    32MB미만 파일개수
.26    filesCountU64MB  Integer    64MB미만 파일개수
.27    filesCountU128MB Integer    128MB미만 파일개수
.28    filesCountU256MB Integer    256MB미만 파일개수
.29    filesCountU512MB Integer    512MB미만 파일개수
.30    filesCountU1GB   Integer    1GB미만 파일개수
.31    filesCountU2GB   Integer    2GB미만 파일개수
.32    filesCountU4GB   Integer    4GB미만 파일개수
.33    filesCountU8GB   Integer    8GB미만 파일개수
.34    filesCountU16GB  Integer    16GB미만 파일개수
.35    filesCountO16GB  Integer    16GB이상 파일개수
====== ================ ========== ============================



.. _snmp-cache-host-traffic:

cache.host.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.1.11

호스트(=모든 가상호스트)의 캐시 서비스와 트래픽 통계를 제공한다. 
traffic의 모든 통계는 최대 60분까지의 평균으로 제공한다. 
min은 '분'을 의미하며 최대 60까지의 값을 가진다. 
min이 생략되거나 0이라면 실시간정보를 제공한다.

===================== =============== ======= ==============================
OID                   Name            Type    Description
===================== =============== ======= ==============================
.1. ``[vhostMin]``    requestHitRatio Integer Request Hit Ratio(100%)
.2. ``[vhostMin]``                            Request Hit Ratio(10000%)
.3. ``[vhostMin]``    bytesHitRatio   Integer Bytes Hit Ratio(100%)
.4. ``[vhostMin]``                            Bytes Hit Ratio(10000%)
.10                   origin          OID     원본 트래픽 정보 (확장)
.11                   client          OID     클라이언트 트래픽 정보 (확장)
===================== =============== ======= ==============================



.. _snmp-cache-host-traffic-origin:

cache.host.traffic.origin
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.1.11.10

종합적인 원본서버 트래픽 통계를 제공한다. 

========================== =================================== ========== ===================================================================
OID                        Name                                Type       Description
========================== =================================== ========== ===================================================================
.1. ``[vhostMin]``         inbound                             Integer    원본서버로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                            Integer    원본서버로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                   Integer    전체 원본서버 평균 세션수
.4. ``[vhostMin]``         activeSessionAverage                Integer    전체 원본서버 세션수 중 전송 중인 평균 세션수
.10                        http                                OID        HTTP 트래픽 (확장)
.11                        hls                                 OID        HLS 트래픽 (확장)
.12                        rtmp                                OID        RTMP 트래픽 (확장)
========================== =================================== ========== ===================================================================




.. _snmp-cache-host-traffic-origin-http:

cache.host.traffic.origin.http
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.1.11.10.10

HTTP 원본서버 트래픽 통계를 제공한다. 

========================== =================================== ========== ===================================================================
OID                        Name                                Type       Description
========================== =================================== ========== ===================================================================
.1. ``[vhostMin]``         inbound                             Integer    원본서버로부터 받는 평균 HTTP 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                            Integer    원본서버로 보내는 평균 HTTP 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                   Integer    원본서버 평균 HTTP세션 수
.4. ``[vhostMin]``         activeSessionAverage                Integer    원본서버 평균 HTTP세션 중 전송 중인 평균 세션수
.10. ``[vhostMin]``        reqHeaderSize                       Integer    원본서버로 보내는 평균 HTTP Header 트래픽(Bytes)
.11. ``[vhostMin]``        reqBodySize                         Integer    원본서버로 보내는 평균 HTTP Body 트래픽(Bytes)
.12. ``[vhostMin]``        resHeaderSize                       Integer    원본서버로부터 받는 평균 HTTP Header트래픽(Bytes)
.13. ``[vhostMin]``        resBodySize                         Integer    원본서버로부터 받는 평균 HTTP Body트래픽(Bytes)
.14. ``[vhostMin]``        reqAverage                          Integer    원본서버로 보낸 평균 HTTP요청 개수
.15. ``[vhostMin]``        reqCount                            Integer    원본서버로 보낸 HTTP요청 개수
.20. ``[vhostMin]``        res2xxAverage                       Integer    원본서버가 보낸 평균 2xx응답 개수
.21. ``[vhostMin]``        res2xxCompleteAverage               Integer    원본서버로부터 성공한 평균 2xx 트랜잭션 개수
.22. ``[vhostMin]``        res2xxTimeRes                       Integer    원본서버로부터 2xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.23. ``[vhostMin]``        res2xxTimeComplete                  Integer    원본서버로부터 2xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]``        res2xxCount                         Integer    원본서버가 보낸 2xx응답 개수
.25. ``[vhostMin]``        res2xxCompleteCount                 Integer    원본서버로부터 성공한 2xx 트랜잭션 개수
.30. ``[vhostMin]``        res3xxAverage                       Integer    원본서버가 보낸 평균 3xx응답 개수
.31. ``[vhostMin]``        res3xxCompleteAverage               Integer    원본서버로부터 성공한 평균 3xx 트랜잭션 개수
.32. ``[vhostMin]``        res3xxTimeRes                       Integer    원본서버로부터 3xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.33. ``[vhostMin]``        res3xxTimeComplete                  Integer    원본서버로부터 3xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]``        res3xxCount                         Integer    원본서버가 보낸 3xx응답 개수
.35. ``[vhostMin]``        res3xxCompleteCount                 Integer    원본서버로부터 성공한 3xx 트랜잭션 개수
.40. ``[vhostMin]``        res4xxAverage                       Integer    원본서버가 보낸 평균 4xx응답 개수
.41. ``[vhostMin]``        res4xxCompleteAverage               Integer    원본서버로부터 성공한 평균 4xx 트랜잭션 개수
.42. ``[vhostMin]``        res4xxTimeRes                       Integer    원본서버로부터 4xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.43. ``[vhostMin]``        res4xxTimeComplete                  Integer    원본서버로부터 4xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]``        res4xxCount                         Integer    원본서버가 보낸 4xx응답 개수
.45. ``[vhostMin]``        res4xxCompleteCount                 Integer    원본서버로부터 성공한 4xx 트랜잭션 개수
.50. ``[vhostMin]``        res5xxAverage                       Integer    원본서버가 보낸 평균 5xx응답 개수
.51. ``[vhostMin]``        res5xxCompleteAverage               Integer    원본서버로부터 성공한 평균 5xx 트랜잭션 개수
.52. ``[vhostMin]``        res5xxTimeRes                       Integer    원본서버로부터 5xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.53. ``[vhostMin]``        res5xxTimeComplete                  Integer    원본서버로부터 5xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]``        res5xxCount                         Integer    원본서버가 보낸 5xx응답 개수
.55. ``[vhostMin]``        res5xxCompleteCount                 Integer    원본서버로부터 성공한 5xx 트랜잭션 개수
.80. ``[vhostMin]``        resTotalAverage                     Integer    원본서버가 보낸 전체 평균 HTTP응답 개수
.81. ``[vhostMin]``        resTotalCompleteAverage             Integer    원본서버로부터 성공한 평균 HTTP트랜잭션 개수
.82. ``[vhostMin]``        resTotalTimeRes                     Integer    원본서버로부터 응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.83. ``[vhostMin]``        resTotalTimeComplete                Integer    원본서버로부터 응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]``        resTotalCount                       Integer    원본서버가 보낸 전체 HTTP응답 개수
.85. ``[vhostMin]``        resTotalCompleteCount               Integer    원본서버로부터 성공한 HTTP트랜잭션 개수
.90. ``[vhostMin]``        connectTimeoutAverage               Integer    평균 원본서버 접속실패 횟수
.91. ``[vhostMin]``        receiveTimeoutAverage               Integer    평균 원본서버 전송실패 횟수
.92. ``[vhostMin]``        connectAverage                      Integer    평균 원본서버 접속성공 횟수
.93. ``[vhostMin]``        dnsQueryTime                        Integer    원본서버 접속 시 평균 DNS쿼리 소요시간
.94. ``[vhostMin]``        connectTime                         Integer    원본서버 평균 접속 소요시간(0.01ms)
.95. ``[vhostMin]``        connectTimeoutCount                 Integer    원본서버 접속실패 횟수
.96. ``[vhostMin]``        receiveTimeoutCount                 Integer    원본서버 전송실패 횟수
.97. ``[vhostMin]``        connectCount                        Integer    원본서버 접속성공 횟수
.98. ``[vhostMin]``        closeAverage                        Integer    전송 중 원본서버에서 먼저 소켓을 종료한 평균 횟수
.99. ``[vhostMin]``        closeCount                          Integer    전송 중 원본서버에서 먼저 소켓을 종료한 횟수
========================== =================================== ========== ===================================================================
                           
                           
                           
.. _snmp-cache-host-traffic-origin-hls:

cache.host.traffic.origin.hls
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.1.11.10.11

(지원예정) HLS 원본서버 트래픽 통계를 제공한다.



.. _snmp-cache-host-traffic-origin-http:

cache.host.traffic.origin.http
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.1.11.10.12

(지원예정) RTMP 원본서버 트래픽 통계를 제공한다.
                           
                           
                           
.. _snmp-cache-host-traffic-client:

cache.host.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.1.11.11

종합적인 클라이언트 트래픽 통계를 제공한다.

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                          Integer    전체 클라이언트 평균 세션수
.4. ``[vhostMin]``         activeSessionAverage                       Integer    전체 클라이언트 중 전송 중인 평균 세션수
.5                         requestHitAverage                          OID        전체 평균 캐시 HIT결과
.5.1. ``[vhostMin]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.5.2. ``[vhostMin]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.5.3. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.5.4. ``[vhostMin]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.5.5. ``[vhostMin]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.5.6. ``[vhostMin]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.5.7. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.5.8. ``[vhostMin]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.5.9. ``[vhostMin]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.5.10. ``[vhostMin]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.5.11. ``[vhostMin]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.6                         requestHitCount                            OID        캐시 HIT결과 개수
.6.1. ``[vhostMin]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.6.2. ``[vhostMin]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.6.3. ``[vhostMin]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.6.4. ``[vhostMin]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.6.5. ``[vhostMin]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.6.6. ``[vhostMin]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.6.7. ``[vhostMin]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.6.8. ``[vhostMin]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.6.9. ``[vhostMin]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.6.10. ``[vhostMin]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.6.11. ``[vhostMin]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10                        http                                       OID        HTTP 트래픽 (확장)
.11                        hls                                        OID        HLS 트래픽 (확장)
.12                        rtmp                                       OID        RTMP 트래픽 (확장)
========================== ========================================== ========== =============================================================


.. _snmp-cache-host-traffic-client-http:                                                                                                           
                                                                                                                                              
cache.host.traffic.client.http                                                                                                                 
---------------------                                                                                                                         
                                                                                                                                              
::                                                                                                                                            
                                                                                                                                              
   OID = 1.3.6.1.4.1.40002.1.4.1.11.11.10
                                                                                                                                              
HTTP 클라이언트 트래픽 통계를 제공한다.                                                                                                                      
                                                                                                                                              
========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description                                                  
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]``                                                               Request Hit Ratio(10000%)
.7                         requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                         requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]``        res2xxAverage                              Integer    클라이언트로 보낸 평균 2xx응답 개수
.21. ``[vhostMin]``        res2xxCompleteAverage                      Integer    클라이언트가 완료한 평균 2xx트랜잭션 개수
.22. ``[vhostMin]``        res2xxTimeRes                              Integer    클라이언트 2xx응답 평균 소요시간(0.01ms)
.23. ``[vhostMin]``        res2xxTimeComplete                         Integer    클라이언트 2xx응답 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]``        res2xxCount                                Integer    클라이언트로 보낸 2xx응답 개수
.25. ``[vhostMin]``        res2xxCompleteCount                        Integer    클라이언트가 완료한 2xx트랜잭션 개수
.30. ``[vhostMin]``        res3xxAverage                              Integer    클라이언트로 보낸 평균 3xx응답 개수
.31. ``[vhostMin]``        res3xxCompleteAverage                      Integer    클라이언트가 완료한 평균 3xx트랜잭션 개수
.32. ``[vhostMin]``        res3xxTimeRes                              Integer    클라이언트 3xx응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]``        res3xxTimeComplete                         Integer    클라이언트 3xx응답 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]``        res3xxCount                                Integer    클라이언트로 보낸 3xx응답 개수
.35. ``[vhostMin]``        res3xxCompleteCount                        Integer    클라이언트가 완료한 3xx트랜잭션 개수
.40. ``[vhostMin]``        res4xxAverage                              Integer    클라이언트로 보낸 평균 4xx응답 개수
.41. ``[vhostMin]``        res4xxCompleteAverage                      Integer    클라이언트가 완료한 평균 4xx트랜잭션 개수
.42. ``[vhostMin]``        res4xxTimeRes                              Integer    클라이언트 4xx응답 평균 소요시간(0.01ms)
.43. ``[vhostMin]``        res4xxTimeComplete                         Integer    클라이언트 4xx응답 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]``        res4xxCount                                Integer    클라이언트로 보낸 4xx응답 개수
.45. ``[vhostMin]``        res4xxCompleteCount                        Integer    클라이언트가 완료한 4xx트랜잭션 개수
.50. ``[vhostMin]``        res5xxAverage                              Integer    클라이언트로 보낸 평균 5xx응답 개수
.51. ``[vhostMin]``        res5xxCompleteAverage                      Integer    클라이언트가 완료한 평균 5xx트랜잭션 개수
.52. ``[vhostMin]``        res5xxTimeRes                              Integer    클라이언트 5xx응답 평균 소요시간(0.01ms)
.53. ``[vhostMin]``        res5xxTimeComplete                         Integer    클라이언트 5xx응답 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]``        res5xxCount                                Integer    클라이언트로 보낸 5xx응답 개수
.55. ``[vhostMin]``        res5xxCompleteCount                        Integer    클라이언트가 완료한 5xx트랜잭션 개수
.80. ``[vhostMin]``        resTotalAverage                            Integer    클라이언트로 보낸 평균 전체응답 개수
.81. ``[vhostMin]``        resTotalCompleteAverage                    Integer    클라이언트가 완료한 평균 트랜잭션 개수
.82. ``[vhostMin]``        resTotalTimeRes                            Integer    클라이언트 응답 평균 소요시간(0.01ms)
.83. ``[vhostMin]``        resTotalTimeComplete                       Integer    클라이언트 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]``        resTotalCount                              Integer    클라이언트로 보낸 전체응답 개수
.85. ``[vhostMin]``        resTotalCompleteCount                      Integer    클라이언트가 완료한 트랜잭션 개수
========================== ========================================== ========== =============================================================



.. _snmp-cache-host-traffic-client-hls:                                                                                                           
                                                                                                                                              
cache.host.traffic.client.hls                                                                                                     
---------------------                                                                                                                         
                                                                                                                                              
::                                                                                                                                            
                                                                                                                                              
   OID = 1.3.6.1.4.1.40002.1.4.1.11.11.11
                                                                                                                                              
HLS 클라이언트 트래픽 통계를 제공한다. 

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description                                                  
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]``                                                               Request Hit Ratio(10000%)
.7                         requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                         requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]``        res2xxAverage                              Integer    클라이언트로 보낸 평균 2xx응답 개수
.21. ``[vhostMin]``        res2xxCompleteAverage                      Integer    클라이언트가 완료한 평균 2xx트랜잭션 개수
.22. ``[vhostMin]``        res2xxTimeRes                              Integer    클라이언트 2xx응답 평균 소요시간(0.01ms)
.23. ``[vhostMin]``        res2xxTimeComplete                         Integer    클라이언트 2xx응답 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]``        res2xxCount                                Integer    클라이언트로 보낸 2xx응답 개수
.25. ``[vhostMin]``        res2xxCompleteCount                        Integer    클라이언트가 완료한 2xx트랜잭션 개수
.30. ``[vhostMin]``        res3xxAverage                              Integer    클라이언트로 보낸 평균 3xx응답 개수
.31. ``[vhostMin]``        res3xxCompleteAverage                      Integer    클라이언트가 완료한 평균 3xx트랜잭션 개수
.32. ``[vhostMin]``        res3xxTimeRes                              Integer    클라이언트 3xx응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]``        res3xxTimeComplete                         Integer    클라이언트 3xx응답 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]``        res3xxCount                                Integer    클라이언트로 보낸 3xx응답 개수
.35. ``[vhostMin]``        res3xxCompleteCount                        Integer    클라이언트가 완료한 3xx트랜잭션 개수
.40. ``[vhostMin]``        res4xxAverage                              Integer    클라이언트로 보낸 평균 4xx응답 개수
.41. ``[vhostMin]``        res4xxCompleteAverage                      Integer    클라이언트가 완료한 평균 4xx트랜잭션 개수
.42. ``[vhostMin]``        res4xxTimeRes                              Integer    클라이언트 4xx응답 평균 소요시간(0.01ms)
.43. ``[vhostMin]``        res4xxTimeComplete                         Integer    클라이언트 4xx응답 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]``        res4xxCount                                Integer    클라이언트로 보낸 4xx응답 개수
.45. ``[vhostMin]``        res4xxCompleteCount                        Integer    클라이언트가 완료한 4xx트랜잭션 개수
.50. ``[vhostMin]``        res5xxAverage                              Integer    클라이언트로 보낸 평균 5xx응답 개수
.51. ``[vhostMin]``        res5xxCompleteAverage                      Integer    클라이언트가 완료한 평균 5xx트랜잭션 개수
.52. ``[vhostMin]``        res5xxTimeRes                              Integer    클라이언트 5xx응답 평균 소요시간(0.01ms)
.53. ``[vhostMin]``        res5xxTimeComplete                         Integer    클라이언트 5xx응답 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]``        res5xxCount                                Integer    클라이언트로 보낸 5xx응답 개수
.55. ``[vhostMin]``        res5xxCompleteCount                        Integer    클라이언트가 완료한 5xx트랜잭션 개수
.80. ``[vhostMin]``        resTotalAverage                            Integer    클라이언트로 보낸 평균 전체응답 개수
.81. ``[vhostMin]``        resTotalCompleteAverage                    Integer    클라이언트가 완료한 평균 트랜잭션 개수
.82. ``[vhostMin]``        resTotalTimeRes                            Integer    클라이언트 응답 평균 소요시간(0.01ms)
.83. ``[vhostMin]``        resTotalTimeComplete                       Integer    클라이언트 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]``        resTotalCount                              Integer    클라이언트로 보낸 전체응답 개수
.85. ``[vhostMin]``        resTotalCompleteCount                      Integer    클라이언트가 완료한 트랜잭션 개수
========================== ========================================== ========== =============================================================




.. _snmp-cache-host-traffic-client-rtmp:
                                                                                                                                              
cache.host.traffic.client.rtmp
---------------------                                                                                                                         
                                                                                                                                              
::                                                                                                                                            
                                                                                                                                              
   OID = 1.3.6.1.4.1.40002.1.4.1.11.11.12
                                                                                                                                              
RTMP 클라이언트 트래픽 통계를 제공한다. 

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description                                                  
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]``                                                               Request Hit Ratio(10000%)
.7                         requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                         requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]``        NetConnection.Connect                      Integer    NetConnection.Connect 응답통계
.20.1. ``[vhostMin]``      NetConnection.Connect.SuccessAverage       Integer    성공응답 평균개수 (평균)
.20.2. ``[vhostMin]``      NetConnection.Connect.SuccessCount         Integer    성공응답 평균개수
.20.3. ``[vhostMin]``      NetConnection.Connect.FailAverage          Integer    실패응답 평균개수 (평균)
.20.4. ``[vhostMin]``      NetConnection.Connect.FailCount            Integer    실패응답 평균개수
.20.5. ``[vhostMin]``      NetConnection.Connect.TimeRes              Integer    응답 평균 소요시간(0.01ms)
.21. ``[vhostMin]``        NetConnection.CreateStream                 Integer    NetConnection.CreateStream 응답통계
.21.1. ``[vhostMin]``      NetConnection.CreateStream.SuccessAverage  Integer    성공응답 평균개수 (평균)
.21.2. ``[vhostMin]``      NetConnection.CreateStream.SuccessCount    Integer    성공응답 평균개수
.21.3. ``[vhostMin]``      NetConnection.CreateStream.FailAverage     Integer    실패응답 평균개수 (평균)
.21.4. ``[vhostMin]``      NetConnection.CreateStream.FailCount       Integer    실패응답 평균개수
.21.5. ``[vhostMin]``      NetConnection.CreateStream.TimeRes         Integer    응답 평균 소요시간(0.01ms)
.30. ``[vhostMin]``        NetStream.Play                             Integer    NetStream.Play 응답통계
.30.1. ``[vhostMin]``      NetStream.Play.SuccessAverage              Integer    성공응답 평균개수 (평균)
.30.2. ``[vhostMin]``      NetStream.Play.SuccessCount                Integer    성공응답 평균개수
.30.3. ``[vhostMin]``      NetStream.Play.FailAverage                 Integer    실패응답 평균개수 (평균)
.30.4. ``[vhostMin]``      NetStream.Play.FailCount                   Integer    실패응답 평균개수
.30.5. ``[vhostMin]``      NetStream.Play.TimeRes                     Integer    응답 평균 소요시간(0.01ms)
.31. ``[vhostMin]``        NetStream.Close                            Integer    NetStream.Close 응답통계
.31.1. ``[vhostMin]``      NetStream.Close.SuccessAverage             Integer    성공응답 평균개수 (평균)
.31.2. ``[vhostMin]``      NetStream.Close.SuccessCount               Integer    성공응답 평균개수
.31.3. ``[vhostMin]``      NetStream.Close.FailAverage                Integer    실패응답 평균개수 (평균)
.31.4. ``[vhostMin]``      NetStream.Close.FailCount                  Integer    실패응답 평균개수
.31.5. ``[vhostMin]``      NetStream.Close.TimeRes                    Integer    응답 평균 소요시간(0.01ms)
.32. ``[vhostMin]``        NetStream.Delete                           Integer    NetStream.Delete 응답통계
.32.1. ``[vhostMin]``      NetStream.Delete.SuccessAverage            Integer    성공응답 평균개수 (평균)
.32.2. ``[vhostMin]``      NetStream.Delete.SuccessCount              Integer    성공응답 평균개수
.32.3. ``[vhostMin]``      NetStream.Delete.FailAverage               Integer    실패응답 평균개수 (평균)
.32.4. ``[vhostMin]``      NetStream.Delete.FailCount                 Integer    실패응답 평균개수
.32.5. ``[vhostMin]``      NetStream.Delete.TimeRes                   Integer    응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]``        NetStream.Seek                             Integer    NetStream.Seek 응답통계
.33.1. ``[vhostMin]``      NetStream.Seek.SuccessAverage              Integer    성공응답 평균개수 (평균)
.33.2. ``[vhostMin]``      NetStream.Seek.SuccessCount                Integer    성공응답 평균개수
.33.3. ``[vhostMin]``      NetStream.Seek.FailAverage                 Integer    실패응답 평균개수 (평균)
.33.4. ``[vhostMin]``      NetStream.Seek.FailCount                   Integer    실패응답 평균개수
.33.5. ``[vhostMin]``      NetStream.Seek.TimeRes                     Integer    응답 평균 소요시간(0.01ms)
.34. ``[vhostMin]``        NetStream.Pause                            Integer    NetStream.Pause 응답통계
.34.1. ``[vhostMin]``      NetStream.Pause.SuccessAverage             Integer    성공응답 평균개수 (평균)
.34.2. ``[vhostMin]``      NetStream.Pause.SuccessCount               Integer    성공응답 평균개수
.34.3. ``[vhostMin]``      NetStream.Pause.FailAverage                Integer    실패응답 평균개수 (평균)
.34.4. ``[vhostMin]``      NetStream.Pause.FailCount                  Integer    실패응답 평균개수
.34.5. ``[vhostMin]``      NetStream.Pause.TimeRes                    Integer    응답 평균 소요시간(0.01ms)
========================== ========================================== ========== =============================================================



.. _snmp-cache-vhost:

cache.vhost
====================================

::
  
   OID = 1.3.6.1.4.1.40002.1.4.3.1

가상호스트의 정보를 제공한다.  ``[vhostIndex]`` 는 1부터 가상호스트 개수의 범위를 가진다.

======================= ========= ========== ============================================
OID                     Name      Type       Description
======================= ========= ========== ============================================
.2. ``[vhostIndex]``    name      String     가상호스트 이름
.3. ``[vhostIndex]``    status    String     "Healthy" 또는 "Inactive" 또는 "Emergency"
.4. ``[vhostIndex]``    uptime    Integer    가상호스트 실행시간 (초)
.10                     contents  OID        컨텐츠 정보 (확장)
.11                     traffic   OID        통계 (확장)
======================= ========= ========== ============================================



.. _snmp-cache-vhost-contents:

cache.vhost.contents
---------------------

::
   
   OID = 1.3.6.1.4.1.40002.1.4.3.1.10

가상호스트가 서비스하는 컨텐츠 통계를 제공한다.

========================= =================== ========== =============================
OID                       Name                Type       Description
========================= =================== ========== =============================
.1. ``[vhostIndex]``      memory              Integer    메모리 캐싱 크기(KB)
.2. ``[vhostIndex]``      filesTotalCount     Integer    서비스 중인 파일개수
.3. ``[vhostIndex]``      filesTotalSize      Integer    서비스 중인 전체 파일량(MB)
.10. ``[vhostIndex]``     filesCountU1KB      Integer    1KB미만 파일개수
.11. ``[vhostIndex]``     filesCountU2KB      Integer    2KB미만 파일개수
.12. ``[vhostIndex]``     filesCountU4KB      Integer    4KB미만 파일개수
.13. ``[vhostIndex]``     filesCountU8KB      Integer    8KB미만 파일개수
.14. ``[vhostIndex]``     filesCountU16KB     Integer    16KB미만 파일개수
.15. ``[vhostIndex]``     filesCountU32KB     Integer    32KB미만 파일개수
.16. ``[vhostIndex]``     filesCountU64KB     Integer    64KB미만 파일개수
.17. ``[vhostIndex]``     filesCountU128KB    Integer    128KB미만 파일개수
.18. ``[vhostIndex]``     filesCountU256KB    Integer    256KB미만 파일개수
.19. ``[vhostIndex]``     filesCountU512KB    Integer    512KB미만 파일개수
.20. ``[vhostIndex]``     filesCountU1MB      Integer    1MB미만 파일개수
.21. ``[vhostIndex]``     filesCountU2MB      Integer    2MB미만 파일개수
.22. ``[vhostIndex]``     filesCountU4MB      Integer    4MB미만 파일개수
.23. ``[vhostIndex]``     filesCountU8MB      Integer    8MB미만 파일개수
.24. ``[vhostIndex]``     filesCountU16MB     Integer    16MB미만 파일개수
.25. ``[vhostIndex]``     filesCountU32MB     Integer    32MB미만 파일개수
.26. ``[vhostIndex]``     filesCountU64MB     Integer    64MB미만 파일개수
.27. ``[vhostIndex]``     filesCountU128MB    Integer    128MB미만 파일개수
.28. ``[vhostIndex]``     filesCountU256MB    Integer    256MB미만 파일개수
.29. ``[vhostIndex]``     filesCountU512MB    Integer    512MB미만 파일개수
.30. ``[vhostIndex]``     filesCountU1GB      Integer    1GB미만 파일개수
.31. ``[vhostIndex]``     filesCountU2GB      Integer    2GB미만 파일개수
.32. ``[vhostIndex]``     filesCountU4GB      Integer    4GB미만 파일개수
.33. ``[vhostIndex]``     filesCountU8GB      Integer    8GB미만 파일개수
.34. ``[vhostIndex]``     filesCountU16GB     Integer    16GB미만 파일개수
.35. ``[vhostIndex]``     filesCountO16GB     Integer    16GB이상 파일개수
========================= =================== ========== =============================



.. _snmp-cache-vhost-traffic:

cache.vhost.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11

가상호스트의 캐시 서비스와 트래픽 통계를 제공한다. 
traffic의 모든 통계는 최대 60분까지의 평균으로 제공된다. 
min은 '분'을 의미하며 최대 60까지의 값을 가진다. 
min이 생략되거나 0이라면 실시간정보를 제공한다.

========================================= ================= =========== ==============================
OID                                       Name              Type        Description
========================================= ================= =========== ==============================
.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitRatio   Integer     Request Hit Ratio(100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                   Request Hit Ratio(10000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``     bytesHitRatio     Integer     Bytes Hit Ratio(100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                   Bytes Hit Ratio(10000%)
.10                                       origin            OID         원본 트래픽 정보 (확장)
.11                                       client            OID         클라이언트 트래픽 정보 (확장)
========================================= ================= =========== ==============================



.. _snmp-cache-vhost-traffic-origin:

cache.vhost.traffic.origin
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.10

종합적인 원본서버 트래픽 통계를 제공한다.


============================================= ===================================== ========== =================================================================
OID                                           Name                                  Type       Description
============================================= ===================================== ========== =================================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                               Integer    원본서버로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                              Integer    원본서버로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                     Integer    전체 원본서버 평균 세션수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                  Integer    전체 원본서버 세션수 중 전송 중인 평균 세션수
.10                                           http                                  OID        HTTP 트래픽 (cache.vhost.traffic.origin.http)
.11                                           hls                                   OID        HLS 트래픽 (cache.vhost.traffic.origin.hls)
.12                                           rtmp                                  OID        RTMP 트래픽 (cache.vhost.traffic.origin.rtmp)
============================================= ===================================== ========== =================================================================


.. _snmp-cache-vhost-traffic-origin-http:

cache.vhost.traffic.origin.http
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.10.10

HTTP 원본서버 트래픽 통계를 제공한다.

============================================= ===================================== ========== =================================================================
OID                                           Name                                  Type       Description
============================================= ===================================== ========== =================================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                               Integer    원본서버로부터 받는 평균 HTTP 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                              Integer    원본서버로 보내는 평균 HTTP 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                     Integer    원본서버 평균 HTTP세션 수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                  Integer    원본서버 평균 HTTP세션 중 전송 중인 평균 세션수
.10. ``[vhostMin]`` . ``[vhostIndex]``        reqHeaderSize                         Integer    원본서버로 보내는 평균 HTTP Header 트래픽(Bytes)
.11. ``[vhostMin]`` . ``[vhostIndex]``        reqBodySize                           Integer    원본서버로 보내는 평균 HTTP Body 트래픽(Bytes)
.12. ``[vhostMin]`` . ``[vhostIndex]``        resHeaderSize                         Integer    원본서버로부터 받는 평균 HTTP Header트래픽(Bytes)
.13. ``[vhostMin]`` . ``[vhostIndex]``        resBodySize                           Integer    원본서버로부터 받는 평균 HTTP Body트래픽(Bytes)
.14. ``[vhostMin]`` . ``[vhostIndex]``        reqAverage                            Integer    원본서버로 보낸 평균 HTTP요청 개수
.15. ``[vhostMin]`` . ``[vhostIndex]``        reqCount                              Integer    원본서버로 보낸 HTTP요청 개수
.20. ``[vhostMin]`` . ``[vhostIndex]``        res2xxAverage                         Integer    원본서버가 보낸 평균 2xx응답 개수
.21. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteAverage                 Integer    원본서버로부터 성공한 평균 2xx 트랜잭션 개수
.22. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeRes                         Integer    원본서버로부터 2xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.23. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeComplete                    Integer    원본서버로부터 2xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCount                           Integer    원본서버가 보낸 2xx응답 개수
.25. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteCount                   Integer    원본서버로부터 성공한 2xx 트랜잭션 개수
.30. ``[vhostMin]`` . ``[vhostIndex]``        res3xxAverage                         Integer    원본서버가 보낸 평균 3xx응답 개수
.31. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteAverage                 Integer    원본서버로부터 성공한 평균 3xx 트랜잭션 개수
.32. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeRes                         Integer    원본서버로부터 3xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.33. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeComplete                    Integer    원본서버로부터 3xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCount                           Integer    원본서버가 보낸 3xx응답 개수
.35. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteCount                   Integer    원본서버로부터 성공한 3xx 트랜잭션 개수
.40. ``[vhostMin]`` . ``[vhostIndex]``        res4xxAverage                         Integer    원본서버가 보낸 평균 4xx응답 개수
.41. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteAverage                 Integer    원본서버로부터 성공한 평균 4xx 트랜잭션 개수
.42. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeRes                         Integer    원본서버로부터 4xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.43. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeComplete                    Integer    원본서버로부터 4xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCount                           Integer    원본서버가 보낸 4xx응답 개수
.45. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteCount                   Integer    원본서버로부터 성공한 4xx 트랜잭션 개수
.50. ``[vhostMin]`` . ``[vhostIndex]``        res5xxAverage                         Integer    원본서버가 보낸 평균 5xx응답 개수
.51. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteAverage                 Integer    원본서버로부터 성공한 평균 5xx 트랜잭션 개수
.52. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeRes                         Integer    원본서버로부터 5xx응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.53. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeComplete                    Integer    원본서버로부터 5xx응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCount                           Integer    원본서버가 보낸 5xx응답 개수
.55. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteCount                   Integer    원본서버로부터 성공한 5xx 트랜잭션 개수
.80. ``[vhostMin]`` . ``[vhostIndex]``        resTotalAverage                       Integer    원본서버가 보낸 전체 평균 HTTP응답 개수
.81. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteAverage               Integer    원본서버로부터 성공한 평균 HTTP트랜잭션 개수
.82. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeRes                       Integer    원본서버로부터 응답 헤더를 받을때까지 평균 소요시간(0.01ms)
.83. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeComplete                  Integer    원본서버로부터 응답 HTTP 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCount                         Integer    원본서버가 보낸 전체 HTTP응답 개수
.85. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteCount                 Integer    원본서버로부터 성공한 HTTP트랜잭션 개수
.90. ``[vhostMin]`` . ``[vhostIndex]``        connectTimeoutAverage                 Integer    평균 원본서버 접속실패 횟수
.91. ``[vhostMin]`` . ``[vhostIndex]``        receiveTimeoutAverage                 Integer    평균 원본서버 전송실패 횟수
.92. ``[vhostMin]`` . ``[vhostIndex]``        connectAverage                        Integer    평균 원본서버 접속성공 횟수
.93. ``[vhostMin]`` . ``[vhostIndex]``        dnsQueryTime                          Integer    원본서버 접속 시 평균 DNS쿼리 소요시간
.94. ``[vhostMin]`` . ``[vhostIndex]``        connectTime                           Integer    원본서버 평균 접속 소요시간(0.01ms)
.95. ``[vhostMin]`` . ``[vhostIndex]``        connectTimeoutCount                   Integer    원본서버 접속실패 횟수
.96. ``[vhostMin]`` . ``[vhostIndex]``        receiveTimeoutCount                   Integer    원본서버 전송실패 횟수
.97. ``[vhostMin]`` . ``[vhostIndex]``        connectCount                          Integer    원본서버 접속성공 횟수
.98. ``[vhostMin]`` . ``[vhostIndex]``        closeAverage                          Integer    전송 중 원본서버에서 먼저 소켓을 종료한 평균 횟수
.99. ``[vhostMin]`` . ``[vhostIndex]``        closeCount                            Integer    전송 중 원본서버에서 먼저 소켓을 종료한 횟수
============================================= ===================================== ========== =================================================================



                           
.. _snmp-cache-vhost-traffic-origin-hls:

cache.vhost.traffic.origin.hls
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.3.1.11.10.11

(지원예정) HLS 원본서버 트래픽 통계를 제공한다.



.. _snmp-cache-vhost-traffic-origin-rtmp:

cache.vhost.traffic.origin.rtmp
---------------------

::
   
    OID = 1.3.6.1.4.1.40002.1.4.3.1.11.10.12

(지원예정) RTMP 원본서버 트래픽 통계를 제공한다.
                           
                           


.. _snmp-cache-vhost-traffic-client:

cache.vhost.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.11

종합적인 클라이언트 트래픽 통계를 제공한다. 

============================================= ========================================== ========== ==============================================================
OID                                           Name                                       Type       Description
============================================= ========================================== ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                          Integer    전체 클라이언트 평균 세션수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                       Integer    전체 클라이언트 중 전송 중인 평균 세션수
.5                                            requestHitAverage                          OID        전체 평균 캐시 HIT결과
.5.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.5.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.5.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.5.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.5.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.5.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.5.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.5.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.5.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.5.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.5.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.6                                            requestHitCount                            OID        캐시 HIT결과 개수
.6.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.6.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.6.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.6.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.6.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.6.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.6.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.6.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.6.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.6.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.6.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10                                           http                                       OID        HTTP 트래픽 (cache.vhost.traffic.client.http)
.11                                           hls                                        OID        HLS 트래픽 (cache.vhost.traffic.client.hls)
.12                                           rtmp                                       OID        RTMP 트래픽 (cache.vhost.traffic.client.rtmp)
============================================= ========================================== ========== ==============================================================



.. _snmp-cache-vhost-traffic-client-http:

cache.vhost.traffic.client.http
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.11.10

HTTP 클라이언트 트래픽 통계를 제공한다.                                                                                                                      
                                                                                                                                              
============================================= ========================================== ========== ==============================================================
OID                                           Name                                       Type       Description                                                  
============================================= ========================================== ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]`` . ``[vhostIndex]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]`` . ``[vhostIndex]``                                                               Request Hit Ratio(10000%)
.7                                            requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                                            requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]`` . ``[vhostIndex]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]`` . ``[vhostIndex]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]`` . ``[vhostIndex]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]`` . ``[vhostIndex]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]`` . ``[vhostIndex]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]`` . ``[vhostIndex]``        res2xxAverage                              Integer    클라이언트로 보낸 평균 2xx응답 개수
.21. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteAverage                      Integer    클라이언트가 완료한 평균 2xx트랜잭션 개수
.22. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeRes                              Integer    클라이언트 2xx응답 평균 소요시간(0.01ms)
.23. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeComplete                         Integer    클라이언트 2xx응답 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCount                                Integer    클라이언트로 보낸 2xx응답 개수
.25. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteCount                        Integer    클라이언트가 완료한 2xx트랜잭션 개수
.30. ``[vhostMin]`` . ``[vhostIndex]``        res3xxAverage                              Integer    클라이언트로 보낸 평균 3xx응답 개수
.31. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteAverage                      Integer    클라이언트가 완료한 평균 3xx트랜잭션 개수
.32. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeRes                              Integer    클라이언트 3xx응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeComplete                         Integer    클라이언트 3xx응답 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCount                                Integer    클라이언트로 보낸 3xx응답 개수
.35. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteCount                        Integer    클라이언트가 완료한 3xx트랜잭션 개수
.40. ``[vhostMin]`` . ``[vhostIndex]``        res4xxAverage                              Integer    클라이언트로 보낸 평균 4xx응답 개수
.41. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteAverage                      Integer    클라이언트가 완료한 평균 4xx트랜잭션 개수
.42. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeRes                              Integer    클라이언트 4xx응답 평균 소요시간(0.01ms)
.43. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeComplete                         Integer    클라이언트 4xx응답 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCount                                Integer    클라이언트로 보낸 4xx응답 개수
.45. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteCount                        Integer    클라이언트가 완료한 4xx트랜잭션 개수
.50. ``[vhostMin]`` . ``[vhostIndex]``        res5xxAverage                              Integer    클라이언트로 보낸 평균 5xx응답 개수
.51. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteAverage                      Integer    클라이언트가 완료한 평균 5xx트랜잭션 개수
.52. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeRes                              Integer    클라이언트 5xx응답 평균 소요시간(0.01ms)
.53. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeComplete                         Integer    클라이언트 5xx응답 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCount                                Integer    클라이언트로 보낸 5xx응답 개수
.55. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteCount                        Integer    클라이언트가 완료한 5xx트랜잭션 개수
.80. ``[vhostMin]`` . ``[vhostIndex]``        resTotalAverage                            Integer    클라이언트로 보낸 평균 전체응답 개수
.81. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteAverage                    Integer    클라이언트가 완료한 평균 트랜잭션 개수
.82. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeRes                            Integer    클라이언트 응답 평균 소요시간(0.01ms)
.83. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeComplete                       Integer    클라이언트 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCount                              Integer    클라이언트로 보낸 전체응답 개수
.85. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteCount                      Integer    클라이언트가 완료한 트랜잭션 개수
============================================= ========================================== ========== ==============================================================




.. _snmp-cache-vhost-traffic-client-hls:

cache.vhost.traffic.client.hls
---------------------

::

   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.11.11

HLS 클라이언트 트래픽 통계를 제공한다.                                                                                                                      
                                                                                                                                              
============================================= ========================================== ========== ==============================================================
OID                                           Name                                       Type       Description                                                  
============================================= ========================================== ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]`` . ``[vhostIndex]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]`` . ``[vhostIndex]``                                                               Request Hit Ratio(10000%)
.7                                            requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                                            requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]`` . ``[vhostIndex]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]`` . ``[vhostIndex]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]`` . ``[vhostIndex]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]`` . ``[vhostIndex]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]`` . ``[vhostIndex]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]`` . ``[vhostIndex]``        res2xxAverage                              Integer    클라이언트로 보낸 평균 2xx응답 개수
.21. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteAverage                      Integer    클라이언트가 완료한 평균 2xx트랜잭션 개수
.22. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeRes                              Integer    클라이언트 2xx응답 평균 소요시간(0.01ms)
.23. ``[vhostMin]`` . ``[vhostIndex]``        res2xxTimeComplete                         Integer    클라이언트 2xx응답 트랜잭션 평균 완료시간(0.01ms)
.24. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCount                                Integer    클라이언트로 보낸 2xx응답 개수
.25. ``[vhostMin]`` . ``[vhostIndex]``        res2xxCompleteCount                        Integer    클라이언트가 완료한 2xx트랜잭션 개수
.30. ``[vhostMin]`` . ``[vhostIndex]``        res3xxAverage                              Integer    클라이언트로 보낸 평균 3xx응답 개수
.31. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteAverage                      Integer    클라이언트가 완료한 평균 3xx트랜잭션 개수
.32. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeRes                              Integer    클라이언트 3xx응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]`` . ``[vhostIndex]``        res3xxTimeComplete                         Integer    클라이언트 3xx응답 트랜잭션 평균 완료시간(0.01ms)
.34. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCount                                Integer    클라이언트로 보낸 3xx응답 개수
.35. ``[vhostMin]`` . ``[vhostIndex]``        res3xxCompleteCount                        Integer    클라이언트가 완료한 3xx트랜잭션 개수
.40. ``[vhostMin]`` . ``[vhostIndex]``        res4xxAverage                              Integer    클라이언트로 보낸 평균 4xx응답 개수
.41. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteAverage                      Integer    클라이언트가 완료한 평균 4xx트랜잭션 개수
.42. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeRes                              Integer    클라이언트 4xx응답 평균 소요시간(0.01ms)
.43. ``[vhostMin]`` . ``[vhostIndex]``        res4xxTimeComplete                         Integer    클라이언트 4xx응답 트랜잭션 평균 완료시간(0.01ms)
.44. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCount                                Integer    클라이언트로 보낸 4xx응답 개수
.45. ``[vhostMin]`` . ``[vhostIndex]``        res4xxCompleteCount                        Integer    클라이언트가 완료한 4xx트랜잭션 개수
.50. ``[vhostMin]`` . ``[vhostIndex]``        res5xxAverage                              Integer    클라이언트로 보낸 평균 5xx응답 개수
.51. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteAverage                      Integer    클라이언트가 완료한 평균 5xx트랜잭션 개수
.52. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeRes                              Integer    클라이언트 5xx응답 평균 소요시간(0.01ms)
.53. ``[vhostMin]`` . ``[vhostIndex]``        res5xxTimeComplete                         Integer    클라이언트 5xx응답 트랜잭션 평균 완료시간(0.01ms)
.54. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCount                                Integer    클라이언트로 보낸 5xx응답 개수
.55. ``[vhostMin]`` . ``[vhostIndex]``        res5xxCompleteCount                        Integer    클라이언트가 완료한 5xx트랜잭션 개수
.80. ``[vhostMin]`` . ``[vhostIndex]``        resTotalAverage                            Integer    클라이언트로 보낸 평균 전체응답 개수
.81. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteAverage                    Integer    클라이언트가 완료한 평균 트랜잭션 개수
.82. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeRes                            Integer    클라이언트 응답 평균 소요시간(0.01ms)
.83. ``[vhostMin]`` . ``[vhostIndex]``        resTotalTimeComplete                       Integer    클라이언트 트랜잭션 평균 완료시간(0.01ms)
.84. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCount                              Integer    클라이언트로 보낸 전체응답 개수
.85. ``[vhostMin]`` . ``[vhostIndex]``        resTotalCompleteCount                      Integer    클라이언트가 완료한 트랜잭션 개수
============================================= ========================================== ========== ==============================================================




.. _snmp-cache-vhost-traffic-client-rtmp:
                                                                                                                                              
cache.vhost.traffic.client.rtmp
---------------------                                                                                                                         
                                                                                                                                              
::                                                                                                                                            
                                                                                                                                              
   OID = 1.3.6.1.4.1.40002.1.4.3.1.11.11.12
                                                                                                                                              
RTMP 클라이언트 트래픽 통계를 제공한다. 

============================================= ========================================== ========== =============================================================
OID                                           Name                                       Type       Description                                                  
============================================= ========================================== ========== =============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                    Integer    클라이언트로부터 받는 평균 트래픽(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                   Integer    클라이언트로 보내는 평균 트래픽(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         allSessionAverage                          Integer    클라이언트 평균 세션수
.4. ``[vhostMin]`` . ``[vhostIndex]``         activeSessionAverage                       Integer    클라이언트 중 전송 중인 평균 세션수
.5. ``[vhostMin]`` . ``[vhostIndex]``         requestHitRatio                            Integer    Request Hit Ratio(100%)
.6. ``[vhostMin]`` . ``[vhostIndex]``                                                               Request Hit Ratio(10000%)
.7                                            requestHitAverage                          OID        평균 캐시 HIT결과
.7.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.8                                            requestHitCount                            OID        캐시 HIT결과 개수
.8.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_HIT                    Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_MISS                   Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``       requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``        reqHeaderSize                              Integer    클라이언트로부터 받는 평균 Header 트래픽(Bytes)
.11. ``[vhostMin]`` . ``[vhostIndex]``        reqBodySize                                Integer    클라이언트로부터 받는 평균 Body 트래픽(Bytes)
.12. ``[vhostMin]`` . ``[vhostIndex]``        resHeaderSize                              Integer    클라이언트로 보내는 평균 Header트래픽(Bytes)
.13. ``[vhostMin]`` . ``[vhostIndex]``        resBodySize                                Integer    클라이언트로 보내는 평균 Body트래픽(Bytes)
.14. ``[vhostMin]`` . ``[vhostIndex]``        reqAverage                                 Integer    클라이언트로부터 받은 평균요청 개수
.15. ``[vhostMin]`` . ``[vhostIndex]``        reqCount                                   Integer    클라이언트로부터 받은 누적요청 개수
.20. ``[vhostMin]`` . ``[vhostIndex]``        NetConnection.Connect                      Integer    NetConnection.Connect 응답통계
.20.1. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.Connect.SuccessAverage       Integer    성공응답 평균개수 (평균)
.20.2. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.Connect.SuccessCount         Integer    성공응답 평균개수
.20.3. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.Connect.FailAverage          Integer    실패응답 평균개수 (평균)
.20.4. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.Connect.FailCount            Integer    실패응답 평균개수
.20.5. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.Connect.TimeRes              Integer    응답 평균 소요시간(0.01ms)
.21. ``[vhostMin]`` . ``[vhostIndex]``        NetConnection.CreateStream                 Integer    NetConnection.CreateStream 응답통계
.21.1. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.CreateStream.SuccessAverage  Integer    성공응답 평균개수 (평균)
.21.2. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.CreateStream.SuccessCount    Integer    성공응답 평균개수
.21.3. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.CreateStream.FailAverage     Integer    실패응답 평균개수 (평균)
.21.4. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.CreateStream.FailCount       Integer    실패응답 평균개수
.21.5. ``[vhostMin]`` . ``[vhostIndex]``      NetConnection.CreateStream.TimeRes         Integer    응답 평균 소요시간(0.01ms)
.30. ``[vhostMin]`` . ``[vhostIndex]``        NetStream.Play                             Integer    NetStream.Play 응답통계
.30.1. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Play.SuccessAverage              Integer    성공응답 평균개수 (평균)
.30.2. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Play.SuccessCount                Integer    성공응답 평균개수
.30.3. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Play.FailAverage                 Integer    실패응답 평균개수 (평균)
.30.4. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Play.FailCount                   Integer    실패응답 평균개수
.30.5. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Play.TimeRes                     Integer    응답 평균 소요시간(0.01ms)
.31. ``[vhostMin]`` . ``[vhostIndex]``        NetStream.Close                            Integer    NetStream.Close 응답통계
.31.1. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Close.SuccessAverage             Integer    성공응답 평균개수 (평균)
.31.2. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Close.SuccessCount               Integer    성공응답 평균개수
.31.3. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Close.FailAverage                Integer    실패응답 평균개수 (평균)
.31.4. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Close.FailCount                  Integer    실패응답 평균개수
.31.5. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Close.TimeRes                    Integer    응답 평균 소요시간(0.01ms)
.32. ``[vhostMin]`` . ``[vhostIndex]``        NetStream.Delete                           Integer    NetStream.Delete 응답통계
.32.1. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Delete.SuccessAverage            Integer    성공응답 평균개수 (평균)
.32.2. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Delete.SuccessCount              Integer    성공응답 평균개수
.32.3. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Delete.FailAverage               Integer    실패응답 평균개수 (평균)
.32.4. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Delete.FailCount                 Integer    실패응답 평균개수
.32.5. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Delete.TimeRes                   Integer    응답 평균 소요시간(0.01ms)
.33. ``[vhostMin]`` . ``[vhostIndex]``        NetStream.Seek                             Integer    NetStream.Seek 응답통계
.33.1. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Seek.SuccessAverage              Integer    성공응답 평균개수 (평균)
.33.2. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Seek.SuccessCount                Integer    성공응답 평균개수
.33.3. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Seek.FailAverage                 Integer    실패응답 평균개수 (평균)
.33.4. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Seek.FailCount                   Integer    실패응답 평균개수
.33.5. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Seek.TimeRes                     Integer    응답 평균 소요시간(0.01ms)
.34. ``[vhostMin]`` . ``[vhostIndex]``        NetStream.Pause                            Integer    NetStream.Pause 응답통계
.34.1. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Pause.SuccessAverage             Integer    성공응답 개수 (평균)
.34.2. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Pause.SuccessCount               Integer    성공응답 개수
.34.3. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Pause.FailAverage                Integer    실패응답 개수 (평균)
.34.4. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Pause.FailCount                  Integer    실패응답 개수
.34.5. ``[vhostMin]`` . ``[vhostIndex]``      NetStream.Pause.TimeRes                    Integer    응답 평균 소요시간(0.01ms)
============================================= ========================================== ========== =============================================================


