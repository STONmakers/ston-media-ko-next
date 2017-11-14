.. _release:

Appendix B: 릴리스 노트
***********************

v17
====================================

v17.11.0 (2017.11.13)
----------------------------

**기능개선/정책변경**

 -  AAC 확장자 지원

**버그수정**

 - [WM] :ref:`multi-protocol-http-ps-modifyheader` 설정이 초기화되는 증상



v17.10.0 (2017.10.24)
----------------------------

- HTTP OPTIONS Method 지원

**버그수정**

 - 설정이 정상적으로 백업되지 않을 때 SNMP 관련 설정이 반영되지 않던 문제 수정
 - :ref:`multi-protocol-vod-apple-hls-packetizing` - 캐싱된 MP3 콘텐츠가 갱신될 경우 비정상 종료되는 문제 수정



v17.09.0  (2017.9.11)
----------------------------

- :ref:`multi-protocol-apple-hls` - MP4 Tracks을 QueryString을 통해 선택적으로 :ref:`multi-protocol-vod-apple-hls-packetizing` 한다.



v17.07.0 (2017.7.31)
----------------------------

- :ref:`multi-protocol-apple-hls` - MP4 Tracks(오디오 또는 비디오)를 선택적으로 :ref:`multi-protocol-vod-apple-hls-packetizing` 한다.
- HEVC/H.265를 지원한다.
- :ref:`adv_topics_memory_only` 를 지원한다.



v17.04.0 (2017.4.17)
----------------------------

**기능개선/정책변경**

- :ref:`multi-protocol-apple-hls` 
   - :ref:`multi-protocol-vod-apple-hls-packetizing`  – 시간값(PCR, PTS, DTS)계산식 변경을 통한 플레이어 호환성 강화

   - :ref:`multi-protocol-vod-apple-hls-packetizing`  – 분석과정 오류가 발생할 경우 정책 수정
      | **Before**. 404 Not Found 응답
      | **After**. 분석된 지점까지 서비스

   - 원본 통계 추가
- :ref:`multi-protocol-mpeg-dash` - 클라이언트/원본 통계 추가
- :ref:`wm`
   - :ref:`multi-protocol-adobe-rtmp` - 예제 URL 추가
   - :ref:`multi-protocol-apple-hls` - 원본 통계 추가
   - :ref:`multi-protocol-mpeg-dash` - 클라이언트/원본 통계 추가

**버그수정**  

 - 낮은 확률로 로그 정리 시 비정상 종료 되는 증상
 - 낮은 확률로 404응답이 메모리에서 Swap 될 때 비정상 종료 되는 문제
 - 로그 압축 기능 사용시 로그가 일부 누락 될 수 있는 문제
 - 시스템 시간 변경 시 5분 통계가 1시간 동안 누락되는 문제
 - :ref:`wm`
    – User-Agent 값을 STON Media Server가 아니라 STON으로 기록하던 문제
    – HTTP Listen을 OFF로 설정 할 경우 적용 되지 않는 문제



v17.02.0 (2017.2.24)
----------------------------
  
- STON 미디어 서버 공식 릴리스 

