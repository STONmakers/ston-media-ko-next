.. _monitoring_stats:

9장. 모니터링 & 통계
******************

이 장에서는 모니터링과 통계에 대해 설명한다.
모니터링과 통계는 용도에 따라 서로 다르게 이해되는 
경우도 있지만 서비스는 숫자로 이야기한다는 관점에서 둘은 같다고 볼 수 있다.

서비스 인프라의 가장 중요한 요소는 실시간성이다.
5분도 너무 길다.
실시간으로 서비스 상태변화를 볼 수 있어야 한다.
수 많은 정책이 적용과 동시에 효과를 내는지 즉시 알 수 있어야 한다.
STON에서 모든 통계는 1초단위로 수집되며 최소 단위가 된다.

통계는 가상호스트별로 따로 수집될 뿐만 아니라 실시간(1초), 5분 평균으로 제공된다. 
고객이 통계를 보다 쉽게 분석, 가공할 수 있도록 JSON과 XML 포맷으로 제공한다. ::

    http://127.0.0.1:20040/monitoring/realtime?type=[JSON 또는 XML]
    http://127.0.0.1:20040/monitoring/average?type=[JSON 또는 XML]
    
-  ``realtime`` 
   1초 전 서비스 상태를 제공한다.

-  ``average`` 
   5분 단위 통계를 제공한다.


.. toctree::
   :maxdepth: 2



호스트 종합통계
====================================

호스트 통계는 가장 상위 개념의 통계로 서비스하는 모든 가상호스트의 통계를 종합한다. 
같은 통계를 JSON과 XML형식으로 제공한다. ::

   {                                            <Host                                    
     "Host":                                      Version="1.0.0"                       
     {                                            Name="localhost"                       
       "Version":"1.0.0",                         State="Healthy"                        
       "Name":"localhost",                        Uptime="155986"                        
       "State":"Healthy",                         AllOriginSession="0" 
       "Uptime":155996,                           AllOriginActiveSession="0" 
       "AllOriginSession":33,                     AllOriginInbound="0" 
       "AllOriginActiveSession":20,               AllOriginOutbound="0" 
       "AllOriginInbound":688177,                 HttpOriginSession="0" 
       "AllOriginOutbound":14184,                 HttpOriginActiveSession="0" 
       "HttpOriginSession":62,                    HttpOriginInbound="0" 
       "HttpOriginActiveSession":62,              HttpOriginOutbound="0" 
       "HttpOriginInbound":2375,                  AllClientSession="0" 
       "HttpOriginOutbound":2509,                 AllClientActiveSession="0" 
       "AllClientSession":54,                     AllClientInbound="0" 
       "AllClientActiveSession":2327,             AllClientOutbound="0" 
       "AllClientInbound":2481,                   HttpClientSession="0" 
       "AllClientOutbound":8,                     HttpClientActiveSession="0" 
       "HttpClientSession":54,                    HttpClientInbound="0" 
       "HttpClientActiveSession":2327,            HttpClientOutbound="0" 
       "HttpClientInbound":2481,                  HlsClientSession="0" 
       "HttpClientOutbound":8,                    HlsClientActiveSession="0" 
       "HlsClientSession":54,                     HlsClientInbound="0" 
       "HlsClientActiveSession":2327,             HlsClientOutbound="0" 
       "HlsClientInbound":2481,                   RtmpClientSession="0" 
       "HlsClientOutbound":8,                     RtmpClientActiveSession="0" 
       "RtmpClientSession":54,                    RtmpClientInbound="0" 
       "RtmpClientActiveSession":2327,            RtmpClientOutbound="0" 
       "RtmpClientInbound":2481,                  RequestHitRatio="0" 
       "RtmpClientOutbound":8,                    ByteHitRatio="0" >
       "RequestHitRatio":6387,                      <VirtualHost> ... </VirtualHost>
       "ByteHitRatio":2926,                         <VirtualHost> ... </VirtualHost>
       "System":{ ... },                            <VirtualHost> ... </VirtualHost>
       "VirtualHost": [ ... ]                       <View> ... </View>
       "View": [ ... ]                              <View> ... </View>.
     }                                            </Host>
   }
   
-  ``Version`` STON 미디어 서버 버전
-  ``Name`` 호스트이름. 설정하지 않았다면 시스템 이름을 보여준다.
-  ``State`` 서비스 상태. (Healthy=정상 서비스, Inactive=라이센스 비활성화, Emergency)
-  ``Uptime (단위: 초)`` 서비스 실행시간
-  ``AllOriginSession`` 연결된 전체 원본세션 수
-  ``AllOriginActiveSession`` 전송 중인 전체 원본세션 수
-  ``AllOriginInbound (단위: Bytes, 평균)`` 전체 원본서버부터 받은 양
-  ``AllOriginOutbound (단위: Bytes, 평균)`` 전체 원본서버로 보낸 양
-  ``HttpOriginSession`` 연결된 HTTP 원본세션 수
-  ``HttpOriginActiveSession`` 전송 중인 HTTP 원본세션 수
-  ``HttpOriginInbound (단위: Bytes, 평균)`` HTTP를 이용해 원본서버부터 받은 양
-  ``HttpOriginOutbound (단위: Bytes, 평균)`` HTTP를 이용해  원본서버로 보낸 양
-  ``AllClientSession`` 연결된 전체 클라이언트 세션 수
-  ``AllClientActiveSession`` 전송 중인 전체 클라이언트 세션 수
-  ``AllClientInbound (단위: Bytes, 평균)`` 전체 클라이언트로부터 받은 양
-  ``AllClientOutbound (단위: Bytes, 평균)`` 전체 클라이언트로에게 보낸 양
-  ``HttpClientSession`` 연결된 HTTP 클라이언트 세션 수
-  ``HttpClientActiveSession`` 전송 중인 HTTP 클라이언트 세션 수
-  ``HttpClientInbound (단위: Bytes, 평균)`` HTTP를 이용해 클라이언트로부터 받은 양
-  ``HttpClientOutbound (단위: Bytes, 평균)`` HTTP를 이용해 클라이언트로 보낸 양
-  ``HlsClientSession`` 연결된 HLS 클라이언트 세션 수
-  ``HlsClientActiveSession`` 전송 중인 HLS 클라이언트 세션 수
-  ``HlsClientInbound (단위: Bytes, 평균)`` HLS를 이용해 클라이언트로부터 받은 양
-  ``HlsClientOutbound (단위: Bytes, 평균)`` HLS를 이용해 클라이언트로 보낸 양
-  ``RtmpClientSession`` 연결된 RTMP 클라이언트 세션 수
-  ``RtmpClientActiveSession`` 전송 중인 RTMP 클라이언트 세션 수
-  ``RtmpClientInbound (단위: Bytes, 평균)`` RTMP를 이용해 클라이언트로부터 받은 양
-  ``RtmpClientOutbound (단위: Bytes, 평균)`` RTMP를 이용해 클라이언트로 보낸 양
-  ``RequestHitRatio (단위: 0.01%, 평균)`` HIT율. 
   캐싱객체가 생성되어 있고 해당 객체가 초기화되어 있다면 HIT이다. 
   반대로 캐싱객체가 없거나 해당 객체가 원본서버로부터 초기화되지 않았다면 MISS이다.
   
-  ``ByteHitRatio (단위: 0.01%, 평균)`` 원본서버 대비 클라이언트 전송률. ::

      (클라이언트 Outbound - 원본서버 Inbound) / 클라이언트 Outbound
      
   원본서버가 훨씬 빠른 속도를 가지고 있거나 클라이언트 세션이 금방 끊어진다면 음수가 될 수 있다.



System 통계
====================================

시스템 및 전역자원 통계를 JSON과 XML형식으로 제공한다. ::

    "System":                                   <System>                                          
    {                                             <CPU                                            
      "CPU":                                          Kernel="689"                                
      {                                               User="1316"                                 
        "Kernel":689,                                 Idle="7993"                                 
        "User":1316,                                  ProcKernel="570"                            
        "Idle":7993,                                  ProcUser="1216"                             
        "ProcKernel":570,                             Nice="0"                                    
        "ProcUser":1216,                              IOWait="52"                                 
        "Nice":0,                                     IRQ="10"                                    
        "IOWait":52,                                  SoftIRQ="12"                                
        "IRQ":10,                                     Steal="0" />                                
        "SoftIRQ":12,                             <Mem Free="5914644" STON="9785800"/>            
        "Steal":0                                 <Storage>                                       
      },                                            <Disk                                         
      "Mem":                                        	Path="/cache1"                            
      {                                             	Status="Normal"                           
        "Free":5914644,                             	Read="23"                                 
        "STON":9785800                              	ReadMerged="0"                            
      },                                            	ReadSectors="344"                         
      "Storage":                                    	ReadTime="117"                            
      {                                             	Write="24"                                
        "Disk":                                     	WriteMerged="93"                          
        [                                           	WriteSectors="936"                        
          {                                         	WriteTime="256"                           
            "Path":"/cache1",                       	IOProgress="0"                            
            "Status":"Normal",                      	IOTime="173"                              
            "Read":23,                              	IOWeightedTime="373"/>                    
            "ReadMerged":0,                         <Disk                                         
            "ReadSectors":344,                      	Path="/cache2"                            
            "ReadTime":117,                         	Status="Normal"                           
            "Write":24,                             	Read="27"                                 
            "WriteMerged":93,                       	ReadMerged="1"                            
            "WriteSectors":936,                     	ReadSectors="488"                         
            "WriteTime":256,                        	ReadTime="144"                            
            "IOProgress":0,                         	Write="24"                                
            "IOTime":173,                           	WriteMerged="86"                          
            "IOWeightedTime":373                    	WriteSectors="880"                        
          },                                        	WriteTime="254"                           
          {                                         	IOProgress="0"                            
            "Path":"/cache2",                       	IOTime="189"                              
            "Status":"Normal",                      	IOWeightedTime="380"/>                    
            "Read":27,                            </Storage>                                      
            "ReadMerged":1,                       <ServerSocket                                   
            "ReadSectors":488,                    	Total="42"                                    
            "ReadTime":144,                       	Established="2"                               
            "Write":24,                            	Accepted="1"                                  
            "WriteMerged":86,                      	Closed="0"/>                                  
            "WriteSectors":880,                   <ClientSocket                                   
            "WriteTime":254,                       	Total="1"                                     
            "IOProgress":0,                        	Established="0"                               
            "IOTime":189,                          	Connected="0"                                 
            "IOWeightedTime":380                   	Closed="0"/>                                  
          }                                       <TCPSocket                                      
        ]                                          	Established="30"                              
      },                                           	Timewait="2"                                  
      "ServerSocket":                              	Orphan="0"                                    
      {                                            	Alloc="0"                                     
        "Total":42,                                	Mem="20"/>                                    
        "Established":1,                          <EQ>0</EQ>                                      
        "Accepted":0,                             <RQ>1000000</RQ>                                
        "Closed":0                                <WaitingFiles2Write>0</WaitingFiles2Write>      
      },                                          <ServiceAccess Allow="60" Deny="2"/>            
      "ClientSocket":                             <SystemLoadAverage Min1="0" Min5="0" Min15="0"/>
      {                                           <URLRewrite>57</URLRewrite>                     
        "Total":1,                              </System>                                         
        "Established":0,
        "Connected":0,
        "Closed":0
      },
      "TCPSocket":
      {
        "Established":30,
        "Timewait":2,
        "Orphan":0,
        "Alloc":0,
        "Mem":20
      },
      "EQ":0,
      "RQ":1000000,
      "WaitingFiles2Write":0,
      "ServiceAccess":{"Allow":60, "Deny":2}
      "SystemLoadAverage":
      {
        "Min1":0,
        "Min5":0,
        "Min15":0
      },
      "URLRewrite":57
    }

-  ``CPU (단위: 0.01%)`` CPU사용량. 전체 CPU사용량은 Kernel + User로 계산해야 한다.
   
   - ``Kernel`` CPU(Kernel) 사용량
   - ``User`` CPU(User) 사용량
   - ``Idle`` 사용되지 않는 CPU량
   - ``ProcKernel`` STON이 사용하는 CPU(Kernel) 사용량
   - ``ProcUser`` STON이 사용하는 CPU(User) 사용량
   - ``Nice`` niced processes executing in user mode
   - ``IOWait`` waiting for I/O to complete
   - ``IRQ`` servicing interrupts
   - ``SoftIRQ`` servicing softirqs
   - ``Steal`` involuntary wait

-  ``Mem (단위: Bytes)`` 메모리 사용량.
   - ``Free`` 시스템 Free 메모리 크기
   - ``STON`` STON이 사용하는 메모리 크기
   
-  ``Disk`` 디스크 성능지표

   - ``Path`` 디스크 경로
   - ``Status`` 디스크 상태 (Normal: 정상동작, Invalid: 장애로 배제됨, Unmounted: 관리자에 의해 Unmount됨)
   - ``Read`` 읽기 성공 횟수
   - ``ReadMerged`` 읽기가 병합된 횟수
   - ``ReadMerged`` 읽은 섹터 수
   - ``ReadTime (단위: ms)`` 읽기 소요시간
   - ``Write`` 쓰기 성공 횟수
   - ``WriteMerged`` 쓰기가 병합된 횟수
   - ``WriteSectors`` 써진 섹터 수
   - ``WriteTime (단위: ms)`` 쓰기 소요시간
   - ``IOProgress`` 진행 중인 IO개수
   - ``IOTime (단위: ms)`` IO 소요시간
   - ``IOWeightedTime (단위: ms)`` IO 소요시간(가중치 적용)
      
-  ``ServerSocket`` 서버 소켓(클라이언트와 STON 구간) 정보

   - ``Total`` 전체 서버소켓 수
   - ``Established`` 연결된 상태의 서버소켓 수
   - ``Accepted`` 새롭게 연결된 서버소켓 수
   - ``Closed`` 연결이 종료된 서버소켓 수
   
-  ``ClientSocket`` 클라이언트 소켓(STON과 원본서버 구간) 정보

   - ``Total`` 전체 클라이언트소켓 수
   - ``Established`` 연결된 상태의 클라이언트소켓 수
   - ``Connected`` 새롭게 연결된 클라이언트소켓 수
   - ``Closed`` 연결이 종료된 클라이언트소켓 수
   
-  ``TCPSocket`` 시스템(OS)이 제공하는 TCP상태 정보

   - ``Established`` Established상태의 TCP 연결개수
   - ``Timewait`` TIME_WAIT 상태의 TCP 연결개수
   - ``Orphan`` 아직 file handle에 attach되지 않은 TCP 연결
   - ``Alloc`` 할당된 TCP 연결
   - ``Mem`` undocumented
   
-  ``EQ`` STON Framework에서 아직 처리되지 않은 Event개수
-  ``RQ`` 최근 서비스된 컨텐츠 참조 큐에 저장된 Event 개수
-  ``WaitingFiles2Write`` 디스크에 쓰기 대기중인 파일개수
-  ``ServiceAccess`` ServiceAccess에 의해 허가(Allow), 거부(Deny)된 소켓 수
-  ``SystemLoadAverage`` System Load Average의 1분/5분/15분 평균
-  ``URLRewrite`` URL전처리에 의해 변환이 성공한 횟수
    
    
가상호스트 통계
====================================

가상호스트별로 통계가 제공된다. 
가상호스트 통계는 프로토콜별로 구분된다. ::

   "VirtualHost":                              <VirtualHost                                 
   [                                               Name="www.example.com"  
     {                                             Uptime="155986"              
       "Name":"www.example.com",                   AllOriginSession="0"                                   
       "Uptime":155996,                            AllOriginActiveSession="0"   
       "AllOriginSession":33,                      AllOriginInbound="0"                                    
       "AllOriginActiveSession":20,                AllOriginOutbound="0"                                   
       "AllOriginInbound":688177,                  HttpOriginSession="0"                                   
       "AllOriginOutbound":14184,                  HttpOriginActiveSession="0"                             
       "HttpOriginSession":62,                     HttpOriginInbound="0"                                   
       "HttpOriginActiveSession":62,               HttpOriginOutbound="0"                                  
       "HttpOriginInbound":2375,                   AllClientSession="0"                                    
       "HttpOriginOutbound":2509,                  AllClientActiveSession="0"                              
       "AllClientSession":54,                      AllClientInbound="0"                                    
       "AllClientActiveSession":2327,              AllClientOutbound="0"                                   
       "AllClientInbound":2481,                    HttpClientSession="0"                                   
       "AllClientOutbound":8,                      HttpClientActiveSession="0"                             
       "HttpClientSession":54,                     HttpClientInbound="0"                                   
       "HttpClientActiveSession":2327,             HttpClientOutbound="0"                                  
       "HttpClientInbound":2481,                   HlsClientSession="0"                                    
       "HttpClientOutbound":8,                     HlsClientActiveSession="0"                              
       "HlsClientSession":54,                      HlsClientInbound="0"                                    
       "HlsClientActiveSession":2327,              HlsClientOutbound="0"                                   
       "HlsClientInbound":2481,                    RtmpClientSession="0"                                   
       "HlsClientOutbound":8,                      RtmpClientActiveSession="0"                             
       "RtmpClientSession":54,                     RtmpClientInbound="0"                                   
       "RtmpClientActiveSession":2327,             RtmpClientOutbound="0"                                  
       "RtmpClientInbound":2481,                   RequestHitRatio="0"                                     
       "RtmpClientOutbound":8,                     ByteHitRatio="0">
       "RequestHitRatio":6387,                   <Memory>784786700</Memory>
       "ByteHitRatio":2926                       <SecuredMemory>0</SecuredMemory>
       "Memory":785740769,                       <Disk> ... </Disk>
       "SecuredMemory":0,                        <CacheFileEvent> ... </CacheFileEvent>
       "Disk": { ... },                          <OriginTraffics> ... </OriginTraffics>
       "CacheFileEvent": { ... },                <ClientTraffic> ... </ClientTraffic>
       "OriginTraffics": { ... },              </VirtualHost>
       "ClientTraffics": { ... },
     },
     ...
   ]
   
.. note:

   ※ Name부터 ByteHitRatio까지 호스트 통계와 동일하다.
   
-  ``Memory (단위: Bytes)`` 메모리에 적재된 컨텐츠 양
-  ``SecuredMemory (단위: Bytes)`` 메모리에서 삭제한 컨텐츠 양
-  ``Disk`` 디스크 정보
-  ``CacheFileEvent`` 캐싱파일 이벤트
-  ``OriginTraffics`` 원본서버 트래픽 통계
-  ``ClientTraffics`` 클라이언트 트래픽 통계


디스크 통계
------------------------------

가상호스트가 사용하는 디스크통계를 제공한다. ::

   "Disk":                                      <Disk>                              
   {                                              <TotalSize>22003701435</TotalSize>
     "TotalSize":22004057982,                     <Create>1</Create>                
     "Create":0,                                  <Open>10</Open>                   
     "Open":1,                                    <Delete>0</Delete>                
     "Delete":0,                                  <ReadCount>9</ReadCount>          
     "ReadCount":1,                               <ReadSize>735726</ReadSize>       
     "ReadSize":104744,                           <WriteCount>1</WriteCount>        
     "WriteCount":0,                              <WriteSize>157145</WriteSize>     
     "WriteSize":0,                               <Distribution                     
     "Distribution":                                U1K="45725"                    
     {                                              U2K="192523"  
       "U1K="45725,                                 U4K="137055"                   
       "U2K="192523,                                U8K="39740"                   
       "U4K="137055,                                U16K="13408"                  
       "U8K="39740,                                 U32K="12303"                  
       "U16K="13408,                                U64K="11462"                  
       "U32K="12303,                                U128K="2560"                  
       "U64K="11462,                                U256K="22"                      
       "U128K="2560,                                U512K="0"                       
       "U256K="22,                                  U1M="45725"                    
       "U512K="0,                                   U2M="192523"                    
       "U1M="45725,                                 U4M="137055"                   
       "U2M="192523,                                U8M="39740"                   
       "U4M="137055,                                U16M="13408"                  
       "U8M="39740,                                 U32M="12303"                  
       "U16M="13408,                                U64M="11462"                  
       "U32M="12303,                                U128M="2560"                  
       "U64M="11462,                                U256M="22"                      
       "U128M="2560,                                U512M="0"                       
       "U256M="22,                                  U1G="0"                        
       "U512M="0,                                   U2G="0"                         
       "U1G="0,                                     U4G="0"                       
       "U2G="0,                                     U8G="0"                       
       "U4G="0,                                     U16G="0"                      
       "U8G="0,                                     O16G="0" />
       "U16G":0,                                  </Disk>
       "O16G":0                                 

     }
   }

-  ``TotalSize (단위: Bytes)`` 로컬파일 크기 합
-  ``Create`` 로컬파일 생성 횟수
-  ``Open`` 로컬파일 Open 횟수
-  ``Delete`` 로컬파일 삭제 횟수
-  ``ReadCount`` 로컬파일에서 Read한 횟수
-  ``ReadSize (단위: Bytes)`` 로컬파일에서 Read한 크기
-  ``WriteCount`` 로컬파일에서 Write한 횟수
-  ``WriteSize (단위: Bytes)`` 로컬파일에서 Write한 크기
-  ``Distribution`` 로컬파일 크기별 분포

   - ``U1K`` 1KB 미만 파일 개수
   - ``U2K`` 2KB 미만 파일 개수
   - ``U4K`` 4KB 미만 파일 개수
   - ``U8K`` 8KB 미만 파일 개수
   - ``U16K`` 16KB 미만 파일 개수
   - ``U32K`` 32KB 미만 파일 개수
   - ``U64K`` 64KB 미만 파일 개수
   - ``U128K`` 128KB 미만 파일 개수
   - ``U256K`` 256KB 미만 파일 개수
   - ``U512K`` 512KB 미만 파일 개수
   - ``U1M`` 1MB 미만 파일 개수    
   - ``U2M`` 2MB 미만 파일 개수    
   - ``U4M`` 4MB 미만 파일 개수    
   - ``U8M`` 8MB 미만 파일 개수    
   - ``U16M`` 16MB 미만 파일 개수  
   - ``U32M`` 32MB 미만 파일 개수  
   - ``U64M`` 64MB 미만 파일 개수  
   - ``U128M`` 128MB 미만 파일 개수
   - ``U256M`` 256MB 미만 파일 개수
   - ``U512M`` 512MB 미만 파일 개수
   - ``U1G`` 1GB 미만 파일 개수
   - ``U2G`` 2GB 미만 파일 개수
   - ``U4G`` 4GB 미만 파일 개수
   - ``U8G`` 8GB 미만 파일 개수
   - ``U16G`` 16GB 미만 파일 개수
   - ``O16G`` 16GB 이상 파일 개수


원본 통계
------------------------------

STON 미디어 서버와 원본서버 사이에 발생하는 트래픽을 프토콜별로 제공한다. ::

   "OriginTraffics":                             <OriginTraffics>
   {                                               <Http> ... </Http>
     "Http": { ... }                             </OriginTraffics>    ...
   }
                                                 

HTTP 세부항목은 다음과 같다. ::

   "Http":                                       <Http>                                             
   {                                               <ReqCount Sum="600">2</ReqCount>       
     "ReqCountSum":0,                              <ReqHeaderSize>3238</ReqHeaderSize>    
     "ReqCount":0,                                 <ReqBodySize>0</ReqBodySize>           
     "ReqHeaderSize":269,                          <ResHeaderSize>2020</ResHeaderSize>    
     "ReqBodySize":0,                              <ResBodySize>104894</ResBodySize>      
     "ResHeaderSize":169,                          <Response>                                     
     "ResBodySize":0,                                <ResTotal>                                   
     "Response":                                       <Count Sum="8100">13</Count>               
     {                                                 <Completed Sum="8100">12</Completed>       
       "ResTotal":                                     <TimeRes>1553</TimeRes>                    
       {                                               <TimeComplete>6630</TimeComplete>          
         "CountSum":0,                               </ResTotal>                                  
         "Count":1,                                  <Res2xx>                                     
         "CompletedSum":0,                             <Count Sum="8100">1</Count>                
         "Completed":1,                                <Completed Sum="8100">1</Completed>        
         "TimeRes":3300,                               <TimeRes>3300</TimeRes>                    
         "TimeComplete":3300                           <TimeComplete>69300</TimeComplete>         
       },                                            </Res2xx>                                    
       "Res2xx":                                     <Res3xx>                                     
       {                                               <Count Sum="8100">12</Count>               
         "CountSum":0,                                 <Completed Sum="8100">11</Completed>       
         "Count":0,                                    <TimeRes>1408</TimeRes>                    
         "CompletedSum":0,                             <TimeComplete>1408</TimeComplete>          
         "Completed":0,                              </Res3xx>                                    
         "TimeRes":0,                                <Res4xx>                                     
         "TimeComplete":0                              <Count Sum="8100">0</Count>                
       },                                              <Completed Sum="8100">0</Completed>        
       "Res3xx":                                       <TimeRes>0</TimeRes>                       
       {                                               <TimeComplete>0</TimeComplete>             
         "CountSum":0,                               </Res4xx>                                    
         "Count":1,                                  <Res5xx>                                     
         "CompletedSum":0,                             <Count Sum="8100">0</Count>                
         "Completed":1,                                <Completed Sum="8100">0</Completed>        
         "TimeRes":3300,                               <TimeRes>0</TimeRes>                       
         "TimeComplete":3300                           <TimeComplete>0</TimeComplete>             
       },                                            </Res5xx>                                    
       "Res4xx":                                     <ConnectTimeout Sum="8100">0</ConnectTimeout>
       {                                             <ReceiveTimeout Sum="8100">0</ReceiveTimeout>
         "CountSum":0,                               <Close Sum="8100">0</Close>                  
         "Count":0,                                </Response>                                    
         "CompletedSum":0,                         <Connect>                                      
         "Completed":0,                              <Count>0</Count>                             
         "TimeRes":0,                                <AvgDNSQueryTime>0</AvgDNSQueryTime>         
         "TimeComplete":0                            <AvgConnTime>0</AvgConnTime>                 
       },                                          </Connect>                                     
       "Res5xx":                                 </Http>                                 
       {
         "CountSum":0,
         "Count":0,
         "CompletedSum":0,
         "Completed":0,
         "TimeRes":0,
         "TimeComplete":0
       },
       "ConnectTimeoutSum":0,
       "ConnectTimeout":0,
       "ReceiveTimeoutSum":0,
       "ReceiveTimeout":0,
       "CloseSum":0,
       "Close":0
     },
     "Connect":
     {
       "Count":0,
       "AvgDNSQueryTime":0,
       "AvgConnTime":0
     }
   }, 
      
-  ``ReqCount`` 원본서버로 보낸 HTTP 요청 횟수
-  ``ReqHeaderSize (단위: Bytes)`` 원본서버로 보낸 HTTP 헤더 크기
-  ``ReqBodySize (단위: Bytes)`` 원본서버로 보낸 HTTP Body 크기
-  ``ResHeaderSize (단위: Bytes)`` 원본서버에서 받은 HTTP 헤더 크기
-  ``ResBodySize (단위: Bytes)`` 원본서버에서 받은 HTTP Body 크기
-  ``Response`` 원본서버에서 보낸 응답 (ResXXX)

   -  ``Count`` 응답횟수
   -  ``Completed`` 정상적으로 전송완료된 HTTP트랜잭션 횟수
   -  ``TimeRes`` HTTP 응답시간
   -  ``TimeComplete`` HTTP 트랜잭션 완료시간

-  ``Response`` 원본서버 연결에러
   
   -  ``ConnectTimeout`` 연결실패
   -  ``ReceiveTimeout`` 전송지연
   -  ``Close`` 전송 중 원본서버에서 먼저 소켓 종료
   
-  ``Connect`` 원본서버 접속통계

   -  ``Count`` 접속횟수
   -  ``AvgDNSQueryTime (단위: 0.01ms)`` 평균 DNS쿼리 시간
   -  ``AvgConnTime (단위: 0.01ms)`` 평균 접속시간 (TCP SYN전송 ~ TCP SYN ACK수신)

.. note::

   5분 통계에서만 제공되는 항목.
   
   -  ``HttpReqCountSum`` HTTP요청의 총 회수
   -  ``CountSum`` HTTP응답의 총 회수
   -  ``CompletedSum`` 완료된 HTTP 트랜잭션의 총 회수
   -  ``ConnectTimeoutSum`` 원본서버 접속실패 총 회수
   -  ``ReceiveTimeoutSum`` 원본서버 전송지연 총 회수
   -  ``CloseSum`` 원본서버에서 먼저 연결을 종료한 총 회수
   


.. _monitoring_stats_vhost_client:

클라이언트 통계
------------------------------

클라이언트 통계는 HTTP, HLS, RTMP로 구분되어 제공된다. ::

   "ClientTraffics":                             <ClientTraffics>
   {                                               <Http> ... </Http>
     "Http": { ... },                              <Hls> ... </Hls>
     "Hls": { ... },                               <Rtmp> ... </Rtmp>
     "Rtmp": { ... },                            </ClientTraffics>
   }

HTTP 클라이언트 통계는 다음과 같다. ::

   "Http":                                       <Http RequestHitRatio="9998">
   {                                               <ReqCount Sum="0">0</ReqCount>
     "RequestHitRatio":9998                        <ReqHeaderSize>4113</ReqHeaderSize>          
     "ReqCountSum":0,                              <ReqBodySize>0</ReqBodySize>
     "ReqCount":100,                               <ResHeaderSize>0</ResHeaderSize>
     "ReqHeaderSize":13968,                        <ResBodySize>892871</ResBodySize>
     "ReqBodySize":0,                              <Response> 
     "ResHeaderSize":5654,                           <ResTotal>                          
     "ResBodySize":104744,                             <Count Sum="0">18</Count>         
     "Response":                                       <Completed Sum="0">18</Completed>                       
     {                                                 <TimeRes>666</TimeRes>                                
       "ResTotal":                                     <TimeComplete>4377</TimeComplete>                     
       {                                             </ResTotal>
         "CountSum":0,                               <Res2xx>                                      
         "Count":52,                                   <Count Sum="0">10</Count>          
         "CompletedSum":0,                             <Completed Sum="0">10</Completed>                         
         "Completed":52,                               <TimeRes>1200</TimeRes>                               
         "TimeRes":94,                                 <TimeComplete>7870</TimeComplete>                     
         "TimeComplete":107                          </Res2xx>                                               
       },                                            <Res3xx>                                                
       "Res2xx":                                       <Count Sum="0">8</Count>                              
       {                                               <Completed Sum="0">8</Completed>                      
         "CountSum":0,                                 <TimeRes>0</TimeRes>                                  
         "Count":1,                                    <TimeComplete>12</TimeComplete>                       
         "CompletedSum":0,                           </Res3xx>                                               
         "Completed":1,                              <Res4xx>                                                
         "TimeRes":4700,                               <Count Sum="0">0</Count>                              
         "TimeComplete":4800                           <Completed Sum="0">0</Completed>                      
       },                                              <TimeRes>0</TimeRes>                                  
       "Res3xx":                                       <TimeComplete>0</TimeComplete>                        
       {                                             </Res4xx>                                               
         "CountSum":0,                               <Res5xx>                                                
         "Count":51,                                   <Count Sum="0">0</Count>                              
         "CompletedSum":0,                             <Completed Sum="0">0</Completed>                      
         "Completed":51,                               <TimeRes>0</TimeRes>                                  
         "TimeRes":3,                                  <TimeComplete>0</TimeComplete>                        
         "TimeComplete":15                           </Res5xx>
       },                                          </Response>          
       "Res4xx":                                   <RequestHit                                          
       {                                              TCP_NONE="0"                  
         "CountSum":0,                                TCP_HIT="0"                   
         "Count":0,                                   TCP_IMS_HIT="0"                                        
         "CompletedSum":0,                            TCP_REFRESH_HIT="0"                                    
         "Completed":0,                               TCP_REF_FAIL_HIT="0"                                   
         "TimeRes":0,                                 TCP_NEGATIVE_HIT="0"                                   
         "TimeComplete":0                             TCP_REDIRECT_HIT="0"                                   
       },                                             TCP_MISS="0"                                           
       "Res5xx":                                      TCP_REFRESH_MISS="0"                                   
       {                                              TCP_CLIENT_REFRESH_MISS="0"                            
         "CountSum":0,                                TCP_DENIED="0"                                         
         "Count":0,                                   TCP_ERROR="0" />                                        
         "CompletedSum":0,                         <RequestHitSum
         "Completed":0,                               TCP_NONE="0"                                         
         "TimeRes":0,                                 TCP_HIT="0"                                          
         "TimeComplete":0                             TCP_IMS_HIT="0"                   
       }                                              TCP_REFRESH_HIT="0"               
     },                                               TCP_REF_FAIL_HIT="0"              
     "RequestHit":                                    TCP_NEGATIVE_HIT="0"        
     {                                                TCP_REDIRECT_HIT="0"        
       "TCP_NONE":0,                                  TCP_MISS="0"                
       "TCP_HIT":0,                                   TCP_REFRESH_MISS="0"        
       "TCP_IMS_HIT":0,                               TCP_CLIENT_REFRESH_MISS="0" 
       "TCP_REFRESH_HIT":0,                           TCP_DENIED="0"              
       "TCP_REF_FAIL_HIT":0,                          TCP_ERROR="0" />
       "TCP_NEGATIVE_HIT":0,                     </Http>
       "TCP_REDIRECT_HIT":0,                     
       "TCP_MISS":0,                             
       "TCP_REFRESH_MISS":0,                     
       "TCP_CLIENT_REFRESH_MISS":0,              
       "TCP_DENIED":0,                           
       "TCP_ERROR":0                             
     },                                          
     "RequestHitSum":                            
     {                                           
       "TCP_NONE":0,                             
       "TCP_HIT":0,                              
       "TCP_IMS_HIT":0,                          
       "TCP_REFRESH_HIT":0,                      
       "TCP_REF_FAIL_HIT":0,                     
       "TCP_NEGATIVE_HIT":0,                     
       "TCP_REDIRECT_HIT":0,                     
       "TCP_MISS":0,                             
       "TCP_REFRESH_MISS":0,                     
       "TCP_CLIENT_REFRESH_MISS":0,              
       "TCP_DENIED":0,                           
       "TCP_ERROR":0                             
     }
   }

-  ``RequestHit`` 캐싱 HIT결과
-  ``ReqCount(단위: Bytes)`` 클라이언트가 보낸  요청수
-  ``ReqHeaderSize(단위: Bytes)`` 클라이언트가 보낸  요청 헤더 크기
-  ``ReqBodySize(단위: Bytes)`` 클라이언트가 보낸 요청 Body 크기
-  ``ResHeaderSize(단위: Bytes)`` STON이 보낸  응답 헤더 크기
-  ``ResBodySize(단위: Bytes)`` STON이 보낸 응답 Body 크기
-  ``Response`` STON이 보낸 응답
   
   -  ``Count`` 응답횟수
   -  ``Completed`` 정상적으로 전송완료된 트랜잭션 횟수
   -  ``TimeRes`` 응답에 소요된 시간 (ms)
   -  ``TimeComplete`` 트랜잭션 완료시간 (ms)

.. note::

   5분 통계에서만 제공되는 항목.
   
   -  ``ReqCountSum`` 클라이언트가 보낸 요청수 누적
   -  ``CountSum`` 응답의 총 회수
   -  ``CompletedSum`` 완료된 트랜잭션의 총 회수
   -  ``RequestHitSum`` 캐시 HIT 결과


HLS 클라이언트 통계는 HTTP 클라이언트 통계와 모든 필드/의미가 동일하다. ::

   "Hls": { ... }                                <Hls RequestHitRatio="0"> ... </Hls>


RTMP 통계는 다음과 같다. ::

   "Rtmp":                                       <Rtmp RequestHitRatio="9998">
   {                                               <ReqCount Sum="0">0</ReqCount>
     "RequestHitRatio":9998                        <ReqHeaderSize>4113</ReqHeaderSize>          
     "ReqCountSum":0,                              <ReqBodySize>0</ReqBodySize>
     "ReqCount":100,                               <ResHeaderSize>0</ResHeaderSize>
     "ReqHeaderSize":13968,                        <ResBodySize>892871</ResBodySize>
     "ReqBodySize":0,                              <NetConnection> 
     "ResHeaderSize":5654,                           <Connect>                          
     "ResBodySize":104744,                             <Success Sum="0">18</Count>
     "NetConnection":                                  <Fail Sum="0">18</Count>
     {                                                 <TimeRes>666</TimeRes>
       "Connect":                                    </Connect>
       {                                             <CreateStream>                    
         "Success":0,                                  <Success Sum="0">18</Count>
         "Fail":52,                                    <Fail Sum="0">18</Count>   
         "TimeRes":0,                                  <TimeRes>666</TimeRes>     
       },                                            </CreateStream>
       "CreateStream":                             </NetConnection>
       {                                           <NetStream>
         "Success":0,                                <Play>
         "Fail":52,                                    <Success Sum="0">18</Count>
         "TimeRes":0,                                  <Fail Sum="0">18</Count>                                 
       },                                              <TimeRes>666</TimeRes>
     },                                              </Play>
     "NetStream":                                    <Close>
     {                                                 <Success Sum="0">18</Count>
       "Play":                                         <Fail Sum="0">18</Count>   
       {                                               <TimeRes>666</TimeRes>     
         "Success":0,                                </Close>
         "Fail":52,                                  <Delete>
         "TimeRes":0,                                  <Success Sum="0">18</Count>
       },                                              <Fail Sum="0">18</Count>   
       "Close":                                        <TimeRes>666</TimeRes>     
       {                                             </Delete>
         "Success":0,                                <Seek>
         "Fail":52,                                    <Success Sum="0">18</Count>
         "TimeRes":0,                                  <Fail Sum="0">18</Count>   
       },                                              <TimeRes>666</TimeRes>     
       "Delete":                                     </Seek>
       {                                             <Pause>
         "Success":0,                                  <Success Sum="0">18</Count>
         "Fail":52,                                    <Fail Sum="0">18</Count>   
         "TimeRes":0,                                  <TimeRes>666</TimeRes>     
       },                                            </Pause>
       "Seek":                                     </NetStream>  
       {                                           <RequestHit                                                        
         "Success":0,                                 TCP_NONE="0"                                                   
         "Fail":52,                                   TCP_HIT="0"                                                    
         "TimeRes":0,                                 TCP_IMS_HIT="0"                                                
       },                                             TCP_REFRESH_HIT="0"                                            
       "Pause":                                       TCP_REF_FAIL_HIT="0"                                           
       {                                              TCP_NEGATIVE_HIT="0"                                           
         "Success":0,                                 TCP_REDIRECT_HIT="0"                                           
         "Fail":52,                                   TCP_MISS="0"                                                   
         "TimeRes":0,                                 TCP_REFRESH_MISS="0".                                          
       },                                             TCP_CLIENT_REFRESH_MISS="0"                                    
       "RequestHit":                                  TCP_DENIED="0"                                                 
       {                                              TCP_ERROR="0" />                                               
         "TCP_NONE":0,                             <RequestHitSum                                                    
         "TCP_HIT":0,                                 TCP_NONE="0"                                                   
         "TCP_IMS_HIT":0,                             TCP_HIT="0"                                                    
         "TCP_REFRESH_HIT":0,                         TCP_IMS_HIT="0"                                                
         "TCP_REF_FAIL_HIT":0,                        TCP_REFRESH_HIT="0"                                            
         "TCP_NEGATIVE_HIT":0,                        TCP_REF_FAIL_HIT="0"                                           
         "TCP_REDIRECT_HIT":0,                        TCP_NEGATIVE_HIT="0"                                           
         "TCP_MISS":0,                                TCP_REDIRECT_HIT="0"                                           
         "TCP_REFRESH_MISS":0,                        TCP_MISS="0"                                                   
         "TCP_CLIENT_REFRESH_MISS":0,                 TCP_REFRESH_MISS="0".                                          
         "TCP_DENIED":0,                              TCP_CLIENT_REFRESH_MISS="0"                                    
         "TCP_ERROR":0                                TCP_DENIED="0"                                                 
       },                                             TCP_ERROR="0" />                                               
       "RequestHitSum":                          </Rtmp>                 
       {                                           
         "TCP_NONE":0,                           
         "TCP_HIT":0,                                             
         "TCP_IMS_HIT":0,                                                                  
         "TCP_REFRESH_HIT":0,                                                              
         "TCP_REF_FAIL_HIT":0,                                                             
         "TCP_NEGATIVE_HIT":0,                                                             
         "TCP_REDIRECT_HIT":0,                                                             
         "TCP_MISS":0,                                                                     
         "TCP_REFRESH_MISS":0,                                                           
         "TCP_CLIENT_REFRESH_MISS":0,                                                    
         "TCP_DENIED":0,                                                                 
         "TCP_ERROR":0                                               
       }                                                    
     },                            


-  ``RequestHit`` 캐싱 HIT결과
-  ``ReqCount(단위: Bytes)`` 클라이언트가 보낸  요청수
-  ``ReqHeaderSize(단위: Bytes)`` 클라이언트가 보낸  요청 헤더 크기
-  ``ReqBodySize(단위: Bytes)`` 클라이언트가 보낸 요청 Body 크기
-  ``ResHeaderSize(단위: Bytes)`` STON이 보낸  응답 헤더 크기
-  ``ResBodySize(단위: Bytes)`` STON이 보낸 응답 Body 크기
-  ``NetConnection``
   -  ``Connect``, ``CreateStream`` 메소드 호출 응답
      -  ``Success`` 성공처리 횟수
      -  ``Fail`` 실패처리 횟수
      -  ``TimeRes`` 응답에 소요된 시간 (ms)

-  ``NetStream``
   -  ``Play``, ``Close``, ``Delete``, ``Seek``, ``Pause`` 메소드 호출 응답   
      -  ``Success`` 성공처리 횟수
      -  ``Fail`` 실패처리 횟수
      -  ``TimeRes`` 응답에 소요된 시간 (ms)


.. _api-monitoring-vhostlist:
   
가상호스트 목록조회
====================================

가상호스트 목록을 조회한다. ::

    http://127.0.0.1:20040/monitoring/vhostslist
    
결과는 JSON형식으로 제공된다. ::

    {
        "version": "2.0.0",
        "method": "vhostslist",
        "status": "OK",
        "result": [ "www.example.com","/foo", "www.example.com/bar" ] 
    }



.. _api-monitoring-fileinfo:
   
캐싱정보
====================================

캐싱하고 있는 파일상태를 모니터링한다.
일반적으로 파일은 URL로 구분되지만 같은 URL에 다른 옵션(i.e. Accept-Encoding등)이 
존재하는 경우 여러 개의 파일이 존재할 수 있다. ::

    http://127.0.0.1:20040/monitoring/fileinfo?url=example.com/trip.mp4
    
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



.. _api-monitoring-logtrace:
   
Log Trace
====================================

기록되는 로그를 실시간으로 받아본다. 
Access, Origin, Monitoing로그는 가상호스트(vhost)를 지정해야 한다. ::

    http://127.0.0.1:20040/monitoring/logtrace/info
    http://127.0.0.1:20040/monitoring/logtrace/deny
    http://127.0.0.1:20040/monitoring/logtrace/sys
    http://127.0.0.1:20040/monitoring/logtrace/originerror
    http://127.0.0.1:20040/monitoring/logtrace/access?vhost=www.site1.com
    http://127.0.0.1:20040/monitoring/logtrace/origin?vhost=www.site1.com
    http://127.0.0.1:20040/monitoring/logtrace/monitoring?vhost=www.site1.com


