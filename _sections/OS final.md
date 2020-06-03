# Operating Systems Finals

## 7. Deadlock

- Necessary conditions for a deadlock (총 4개)
- Deadlock handling
  - Prevention
    - 각 necessary condition에 대해
  - Avoidance (사전정보 필요)
    - Banker's algorithm
  - Detect + Recover (사전정보 노필요)
    - Banker's에서 request => need 행렬으로 바뀜
    - safe seq 없으면 데드락
  - Ignore



## 8. Memory management

- Address, linking, shared library, swapping 등의 개념
- Partitioning
  - fixed
    - internal fragmentation
    - if proc too big => overlay (자기 메모리 안에서 덮어쓰고 덮어쓰고..)
    - equal size
    - unequal-size 
      - single queue, multi queue (minimize internal frag)
  - dynamic
    - external frag
    - compaction (sol to external frag)
  - issue of fragmentation
- Buddy System (사실 Virt mem + paging + segmentation이 나은데 커널쪽에 BS쓰긴 씀)
- Paging (페이지랑 프레임은 2^n 사이즈라 internal frag 가능)
  - logical address = (p, o) => 몇번째 페이지에서 얼마만큼.
    - **CPU는 logical addr만 본다!!**
    - OS, hardware가 translate
  - **Page Table**
    - RAM에 존재
    - 프로세스당 하나 (inverted의 경우 제외)
    - 런타임에 2번의 lookup 필요 (그래서 TLB가 나왔지..)
    - page sharing 가능 (inverted은 안됨)
  - Large page table
    - n-lvl paging
      - p1은 다음 PT의 *주소*, p2는 그다음 PT의 *주소*,..., 마지막 p가 프레임번호
    - hashed page tables
      - 해시해서 HT가면 linked list of (p, f)가 있음. 자기 페이지 찾으면 해당 f찾아감
    - inverted page tables
      - 물리프레임 기준. 따라서 PT 시스템상 하나
      - 프로세스 여러갠데 PT 하나면 pid, p 두개로 찾아야한다. 그걸로 (pid, p)를 찾으면 해당 인덱스가 f
  - 페이징을 빠르게 => TLB, 페이지테이블이 너무 큼 => multi lvl PT, hashed, inverted ...
- Segmentation
  - 페이징처럼 할 수 있는데 사이즈가 다양해서 dynamic allocation 해야함 => external frag
  - segment 안에 page를 쓰면 좋겠다!
  - 세그멘트별로 PT 가짐 (2 lvl paging이랑 비슷함 -- (p1,p2,o) 였던거에서 이제 (s,p,o)로 간다)
  - protection을 세그멘트별로 관리할 수 있어서 용이하고, shared segment PT도 가능



## 9. Virtual Memeory

- **logical memory와 physical memeory의 분리!!**
- virtual address => CPU는 logical addr를 발행한다 했는데, 그게 이제는 virtual addr임
- **Demand Paging** => **virtual memory의 구현은 demand paging으로 된다!!!**
  - 페이지는 필요할때만 물리메모리에 넣어라
  - 스와핑도 프로세스가 아니라 페이지 단위로!
  - 그럼 물리메모리가 프로세스들이 쓰는 메모리 캐시 느낌이 된다!
- page fault
  - valid/invalid bit 보고, invalid면 interrupt 걸어서 핸들러 돌려서 스왑파일에서 가져와서 교체
- page replacement
  - Optimal: 가장 나중에 쓸거
  - FIFO: 프레임이 더 많아졌는데 pf가 오히려 늘 수도 있음
  - LRU: counter나 스택같은걸로 구현해야함 => 둘다 오버헤드 짱짱
  - second chance
  - counting based
- Thrashing (프레임 부족해서 pf 계속 일어나는데 disk io하니까 cpu util low하고 OS는 cpu가 놀고있다 생각해서 프로세스 더 들여오고.. 악순환.)
  - sum(진짜 사용중인 페이지들) > 물리프레임수
- Working set model = set(진짜 사용중인 페이지들)
  - $\Delta$ = working set window (사전정의한 n instruction => 이동안 reference 된 페이지의 집합은 working set)
  - sampled LRU처럼 가능
- Other issues
  - prepaging - working set 기억해뒀다가 프로세스 스왑인 될때 한번에 불러옴
  - page size - 크면 PT 작고 PF도 작지만 internal frag 문제
  - TLB reach - TLB size * page size인데, 보통 한 프로세스의 working set은 다 케어 할 수 있게끔 한다
  - program structure - 컴파일러가 이제 알아서 해주지만 루프 돌릴때 이론적으로 한 행을 이너 루프로 돌리는게 PF가 적게 난다
  - IO interlock - 프레임 교체해야되는데 지금 IO 중이심..
- Benefits
  - Copy on write - 부모자식이 페이지 공유하고, write가 발생할때만 카피
    - write가 발생했는지 어떻게 암? write하면 trap해서 운체가 페이지 카피떠서 PT 업뎃하고 write 재수행
  - memory mapped files
    - 파일을 page 사이즈만큼 따와서 메모리에 넣음. 이후 IO는 걍 메모리 작업이 됨. 끝에 저장할때 실제로 디스크로 감



## 10. File System

- Files: long term storage를 위해 필요하지?
  - types - 윈도우는 extension이 실제로 중요하고, 유닉스 계열은 그닥..
  - attributes - stat 해보면 보이는것들
  - operations - read/write/open/close...
    - open-file table을 시스템와이드, 각 프로세스에 대해 가지고 있음. 각 프로세스의 oft는 시스템꺼 포인터 가지고 있고 fd는 인덱스 번호. 시스템와이드껀 FCB를 가지고 있음
    - open하면 시스템 와이드에 FCB넣고 그 포인터를 프로세스의 oft에 줌. 그럼 그 인덱스 번호를 fd로 리턴.
- Directories: 네이밍 인터페이스 줘서 logical file org랑 physical file placement랑 분리시킴!
  - internals: 걍 메타데이터를 담고있는 파일이다. [(file name, file attr), ..., (name, attr)] 형식.
    - 디렉토리 구조랑 파일들이 전부 디스크에 저장돼있다.
  - pathname: /로 구분된 디렉토리들을 실제로 열고 찾고.. 하면서 open한다. 그래서 open이랑 read/write 등이 분리돼있음
  - operations: 똑같이 파일임. 근데 추가적인 시스템콜이 몇개 있긴 함
- Organizing directories - 트리로 만들수도 있고 acyclic directed graph로 표현도 가능 (심볼릭 링크하면 부모 공유 가능)
  - 부모 공유 가능하면 둘 중 하나 지우면 나머지는 어떻게? 보통 유닉스 체계에서는 댕글링 포인터 유저가 알아서 지워줘야함.
- Implementation
  - File System: file은 데이터 묶음이고 FS도 secondary storage에 삶. FCB가 파일에 대한 정보를 담음.
    - On disk에 어떻게 저장?
      - Boot control block (**boot block**)
      - Partition control block (**super block**) - 파티션 디테일: 블록사이즈, free block, FCB count etc
      - Individual files (**inode**)
    - 메모리에는 어떻게 띄움?
      - partition table: 마운트 되어있는 각 파티션에 대한 정보를 담고있는 테이블
      - directory structure: 최근에 액세스한 디렉토리를 캐시 느낌으로 가지고 있음
      - system wide open file table: FCB 카피해서 가지고 있음
      - per process open file table: system wide꺼 포인터
      - open()을 하면, path 분석해서 해당 디렉토리의 파일을 찾은 후, FCB를 sys wide open file table에 복사하고, 포인터를 요청한 프로세스의 open file table에 넘겨줌. 그럼 그걸 저장하고 인덱스 번호를 fd로 유저한테 줌.
  - Directory: 디렉토리 안에 파일 및 다른 디렉토리 찾는게 주 업문데, 그럼 디렉토리는 어떻게 구성하는게 맞나?
    - linear list
    - hash table
    - 메타데이터는 어떻게 구성할까?
      - dentry에 넣거나
      - inode로 빼거나
      - 하이브리드
- Allocation - 파일을 디스크에 어떻게 할당할까
  - Contiguous - 랜덤액세스 빠르고 좋은데 external frag, 생성 후 파일 사이즈 변경 불가 등의 매우 큰 단점이 있음
  - Linked - waste space 없는데 랜덤액세스 느리고 포인터 추가 저장해야되고 포인터 하나 깨지면 뒤에 애들 다 놓침.
  - Indexed - 디스크 안에 한 블록이 인덱스 역할을 해서 데이터 어디어디에 있는지 가리킴. 인덱스 테이블 필요함
- Free space management
  - Bitmap: 파티션이 총 n블록이라면 n사이즈 벡터로 관리
  - Linked
  - Group: index block 느낌으로 free space도 한 블록에서 관리를 하고, 마지막놈이 다음 그룹 가리킴
  - Counting: locally contiguous + 마지막애가 다음으로 locally cont한 애들 어디있는지 포인터로 가리킴
- Virtual File System (VFS)
  - underlying FS가 무엇이던 사용 가능하도록 시스템콜로 추상화! 커널과 유저스페이스 사이에서 징검다리 역할을 한다.
  - 리눅스에서는 4 obj 필요!
    - superblock: mounted fs info
    - dentry: directory entry <=> file을 묶어주는 역할
    - inode: 파일의 이름과 데이터를 제외한 제너럴한 정보들
    - file: 열린 파일과 프로세스간의 인터랙션
- Disk caching
  - 파일 읽고 쓸 때 locality있으니까 파일 블록을 메모리에 캐시하자 (시스템 와이드, MMIO처럼 작동)
  - 근데 virtual memory랑 경쟁해야되고 교체정책 생각해야됨
  - 그래서 디스크블록단위 말고 페이지단위로 캐싱
- Ext2 File System



## 11. IO Systems

- IO device랑 소통하는데 3가지 방법이 있다: polling, interrupt, DMA
- IO architecture (connected to IO device via:)
  - IO port (IO bus에 연결돼있는 dev들은 각자 IO port를 가짐)
  - IO interface (포트에 들어온 값을 명령으로 변환)
  - IO controller (aka device controller)
- Polling: 씨퓨가 계속 너 괜찮아? 물어봄 (소프트웨어로 구현가능)
- Interrupt: 하드웨어가 인터럽트 거는 방식
- DMA: CPU 안거치고 메모리 <=> IO dev (데이터가 크면 씨퓨 안거쳐도 되도록 해놓음)
  - Cycle stealing: CPU의 버스 사이클 몇개 훔쳐서 전송
  - Burst mode: 버스 꽉 쥐고 한번에 쫙 보냄
- IO software
  - IO마다 속도가 다른데 어떻게 지원하나? 공통된 인터페이스 어떻게?
  - Device driver라는 abstraction layer를 사이에 두자!
- Device drivers (dev dependent)
  - OS가 사용하는 API를 만들어주면 됨. 표준화된 드라이버 인터페이스가 정의돼있음
- Device independent IO software
  - UNIX에선 디바이스도 file임. 따라서 open, read, write, close 똑같이 씀.
- User space IO software - 정말 간단하게 몇개의 시스템콜만 내준다!



## 12. Mass Storage

- Disk addressing (cylinder, surface, track)
- Disk scheduling
  - FCFS
  - SSTF
  - (C - ) SCAN
  - (C - ) LOOK
- Swap-space management
  - swap file - 스왑파티션 따로 안잡고 그냥 스왑할때 파일로 읽고쓰고.. => 느림!
  - swap partition - 스왑파티션 따로 잡아서 빠르게!
- RAID
  - Redundancy: mirroring + parity data
  - RAID 0: striped disks
  - RAID 1: disk mirroring
  - RAID 3: striped disks with dedicated parity disk (small write problem)
  - RAID 5: striped disk + distributed parity
- Tertiary storage
  - 싸고 reliable, 근데 느림
- Traditional architecture vs NAS vs SAN: 버리자