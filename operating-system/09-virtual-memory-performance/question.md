# 가상 메모리 성능 질문 정리

 <details>
  <summary><strong>스래싱이란 무엇인가요?</strong></summary>

시스템이 너무 많은 페이지 부재(page fault)를 경험하면서, 실제 작업보다 페이지 교체(스와핑)에 소요되는 시간이 지나치게 많아져 전체 성능이 급격히 저하되는 현상  
메모리 과부하로 인해 실제 프로세스 작업보다 페이지를 메모리로 불러오고 내보내는 작업에 집중하게 됨

 <details>
  <summary><strong>스래싱은 어떻게 감지할 수 있을까요?</strong></summary>

- **높은 페이지 부재율:**
  - 프로세스들이 빈번하게 페이지 부재 예외를 발생시키는지 모니터링
- **CPU 활용률 대비 낮은 실제 작업량:**
  - CPU가 페이지 교체 작업에 대부분의 시간을 소비하여, 실제 프로세스 실행에 할애되는 시간이 극히 낮은 경우 스래싱 의심
- **시스템 성능 저하:**

  - 응답 시간 지연, 처리량 감소, 시스템의 전반적인 성능 하락 등을 통해 간접적으로 감지
  </details>

   <details>
  <summary><strong>스래싱이 발생할 때 어떻게 완화할 수 있을까요?</strong></summary>

**페이지 교체 알고리즘 최적화:**

- LRU, LFU와 같은 효율적인 페이지 교체 정책을 사용하여, 자주 사용하는 페이지들이 메모리에 오래 머무를 수 있게 함.

**프로세스 수 조절 (Working Set 관리):**

- 동시에 실행되는 프로세스 수를 줄여 각 프로세스가 충분한 메모리를 사용할 수 있게 함.
- 작업 집합(Working Set)을 관리하여, 필요한 페이지들이 메모리에 유지되도록 함.

  </details>

</details>

<br/>

 <details>
  <summary><strong>TLB는 무엇인가요?</strong></summary>

TLB는 최근에 사용된 가상 주소 → 물리 주소 변환 정보를 저장하는 고속 캐시 메모리
MMU 내 또는 근접한 하드웨어에 통합되어 있어, 주소 변환 과정을 가속화

 <details>
  <summary><strong>TLB Hit Ratio가 무엇이고, 이것의 중요성에 대해 설명해주세요.</strong></summary>

- **TLB Hit Ratio:**
  - 메모리 접근 시 TLB에 필요한 변환 정보가 존재하는 비율
- **중요성:**

  - 높은 TLB Hit Ratio는 자주 참조되는 페이지의 주소 변환을 캐시에서 빠르게 처리할 수 있어, 메모리 접근 지연을 크게 줄이고 전체 시스템 성능 향상
  </details>

   <details>
  <summary><strong>TLB를 쓰면 왜 속도가 빨라질까요?</strong></summary>

  TLB는 페이지 테이블에 접근할 필요 없이, 자주 사용하는 가상 주소의 물리 주소를 즉시 제공함으로써, 메모리 접근 시간을 단축

  </details>

  <details>
  <summary><strong>TLB와 MMU는 어디에 위치할까요?</strong></summary>

- **TLB:** CPU 내부의 MMU 내에 존재하거나, MMU 근처에 통합되어 있어, 빠른 접근과 변환이 가능
- **MMU:** CPU와 메모리 사이에 위치한 하드웨어 장치로, 가상 주소를 물리 주소로 변환하는 역할을 담당.

  </details>

  <details>
  <summary><strong>코어가 여러 개일 때 TLB는 어떻게 동기화될 수 있을까요?</strong></summary>

일반적으로 각 코어는 자체 TLB를 보유하며, 동일한 프로세스의 가상 주소 매핑 정보를 저장

- **TLB Shootdown:**

  - 페이지 테이블에 변경이 발생할 때, 운영체제는 해당 변경사항을 모든 코어에 전달하여 관련 TLB 엔트리를 무효화
  - 이를 통해 각 코어의 TLB가 최신의 페이지 매핑 정보를 반영하도록 동기화

  </details>

</details>

<br/>
 <details>
  <summary><strong>페이지 교체 알고리즘이 왜 필요할까요?</strong></summary>

- **한정된 메모리 관리:**
  - 실제 메모리 용량은 제한적이므로, 새로운 페이지를 로드할 때 어느 페이지를 교체할지 결정해야 함
- **시스템 성능 최적화:**
  - 효율적인 페이지 교체 정책을 통해 페이지 부재율을 줄이고, 전체 시스템의 응답 시간과 처리 성능 개선

 <details>
  <summary><strong>페이지 교체 알고리즘 중 LRU 알고리즘은 어떤 특성을 이용한 알고리즘인가요?</strong></summary>

최근에 사용되지 않은 페이지는 앞으로도 사용될 가능성이 낮다는 가정(지역성 원칙)을 기반으로, 가장 오랫동안 참조되지 않은 페이지를 교체 대상으로 선택

**시간적 지역성 (Temporal Locality) 이용:**

- 최근에 사용된 데이터나 명령어는 가까운 미래에도 다시 사용될 가능성이 높다는 원칙
  </details>

   <details>
  <summary><strong>LRU 알고리즘을 구현한다면 어떻게 구현할 수 있을까요?</strong></summary>

**이중 연결 리스트 + 해시맵:**

- **구현 방식:**
  - 이중 연결 리스트는 페이지의 순서를 유지하여, 가장 최근에 사용된 페이지를 리스트 앞쪽에, 가장 오래된 페이지를 리스트 뒷쪽에 배치
  - 해시맵은 각 페이지에 대한 빠른 접근을 위해, 페이지 번호와 리스트 내 노드 포인터를 저장
- **동작:**
  - 페이지 접근 시, 해당 페이지를 해시맵을 통해 찾아내고, 이중 연결 리스트에서 해당 노드를 가장 앞쪽으로 이동
  - 페이지 교체가 필요하면, 리스트의 마지막 노드를 제거

**타임스탬프 기록:**

- 각 페이지의 마지막 접근 시간을 기록하고, 가장 오래된 타임스탬프를 가진 페이지를 교체하는 방식
  </details>

    <details>
    <summary><strong>LRU 알고리즘 단점이 있다면 무엇일까요? 단점을 어떻게 해결할 수 있을까요?</strong></summary>

- 구현 복잡도와 유지 비용 증가.
- 정확한 LRU 구현은 오버헤드가 크기 때문에 근사 알고리즘(예: Clock 알고리즘)이 많이 사용됨
- **Clock 알고리즘**: 원형 큐를 사용하여 LRU의 근사치를 제공하는 알고리즘.

  </details>

</details>

<br/>

 <details>
  <summary><strong>가상 메모리 성능을 최적화하기 위한 방법으로는 무엇이 있을까요?</strong></summary>

**선페이징 (Prepaging)**

- 프로세스 실행 전에 예상되는 페이지들을 미리 로드하여, 초기 페이지 폴트 발생을 줄이는 기법.
- 프로그램 시작 시의 지연 감소 및 성능 향상.

**페이지 크기 최적화**

- 시스템 특성과 애플리케이션의 메모리 접근 패턴을 고려하여 최적의 페이지 크기 결정.
- 너무 작은 페이지는 페이지 테이블의 오버헤드를 증가시키고, 너무 큰 페이지는 내부 단편화를 유발.

 <details>
  <summary><strong>시간적/공간적 지역성에 대해 설명해주세요.</strong></summary>

- **시간적 지역성 (Temporal Locality):**
  - 최근에 사용된 데이터나 명령어는 가까운 미래에도 다시 사용될 가능성이 높다는 원칙
  - 예를 들어, 루프 내부의 변수나 반복 호출되는 함수는 시간적 지역성의 대표적인 사례로, 캐시나 메모리에서 빠르게 재사용됨
- **공간적 지역성 (Spatial Locality):**

  - 특정 주소에 접근하면, 그 주변의 인접한 메모리 영역도 곧 사용될 가능성이 높은 원칙
  - 예를 들어, 배열이나 연속된 데이터 블록은 공간적 지역성이 뛰어나, 인접 데이터를 미리 로드해두면 페이지 부재를 줄이고 성능 향상

  </details>

</details>

<br/>
