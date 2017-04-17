.. _release:

Appendix B: 릴리스 노트
***********************
v1.0.x
====================================

1.0.1 (2017.4.18)
----------------------------

**기능개선/정책변경**

- :ref:`multi-protocol-apple-hls`
   - :ref:`multi-protocol-apple-hls-mp4segmentation` – 시간값(PCR, PTS, DTS)계산식 변경을 통한 플레이어 호환성 강화
   - :ref:`multi-protocol-apple-hls-mp3segmentation` – 분석과정 오류가 발생할 경우 정책 수정
        | **Before**. 404 Not Found 응답
        | **After**. 분석된 지점까지 서비스
   - 원본 통계 추가

- :ref:`multi-protocol-mpeg-dash` - 클라이언트/원본 통계 추가
- :ref:`wm`
   - 예제 URL에 RTMP 형식 추가
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



1.0.0 (2017.2.24)
----------------------------
  
- STON 미디어 서버 공식 릴리스

