![20220721_131133_1](https://user-images.githubusercontent.com/68460391/180128409-b4deb644-84ba-43c9-80a7-27db589f1ff3.png)
- 로비 화면

![20220726_162302_1](https://user-images.githubusercontent.com/68460391/182110745-ee58a3fb-c991-4464-b563-d74174cb79b0.png)
- 매칭 대기 화면
  
![20220721_131133_3](https://user-images.githubusercontent.com/68460391/180128414-3beedc45-ff66-4060-ade6-c224baebc3d0.png)
- 생존자 플레이 화면 (발전기 수리 진행)

![20220721_131133_4](https://user-images.githubusercontent.com/68460391/180128417-a66abdf3-93af-450d-bdb4-9440f8d43197.png)
- 생존자 플레이 화면 (사물 변신)

![20220721_131133_5](https://user-images.githubusercontent.com/68460391/180128419-4fd54a7e-51ad-4553-9a37-775f2382a3f9.png)
- 킬러 플레이 화면 (공격 스킬 사용)

---

## 📋 프로젝트 정보

| 항목 | 내용 |
|------|------|
| **프로젝트명** | UE4PvPGame |
| **장르** | 멀티플레이 PvP 숨바꼭질 게임 |
| **플랫폼** | PC (Windows) |
| **엔진** | Unreal Engine 4 |
| **언어** | C++, Blueprint |
| **개발 기간** | 2022.04 ~ 2022.08 |
| **인원** | 1인 |
| **최대 플레이어 수** | 5명 (킬러 1명 + 생존자 4명) |
| **모티브 게임** | 프롭나이트, Dead by Daylight |

---

## ⚙ 핵심 기술 및 구현 내용

### 🎮 비대칭 게임플레이 구조
- **생존자 (4인):** 맵에 배치된 머신(발전기) 4개를 수리해 탈출 조건을 달성해야 함. 수리 중 킬러에게 발각되지 않도록 소품(Prop)으로 변신해 은신 가능
- **킬러 (1인):** 제한 시간 내에 모든 생존자를 처치해야 함. 메인 공격, 원거리 공격(Q), 특수기(E), 원격 조작 투사체(RC) 등 4가지 공격 기술 보유
- **승리 조건:** 생존자는 모든 머신 수리 후 탈출 / 킬러는 시간 내 전원 처치

### 🌐 멀티플레이어 네트워크 시스템
- UE4 내장 `Replication` 시스템 기반의 클라이언트-서버 아키텍처
- `UFUNCTION(Server, Reliable)`, `UFUNCTION(NetMulticast, Reliable)`, `UFUNCTION(Client, Reliable)` RPC를 역할별로 명확히 구분
- `UPROPERTY(Replicated)` / `ReplicatedUsing = OnRep_*` 를 사용한 상태 자동 동기화
- `GetLifetimeReplicatedProps()` 오버라이드를 통한 세밀한 복제 속성 관리

### 🔄 매칭 시스템
- `AMyMatchingModeBase` + `UMyGameInstance` 조합으로 플레이어 입장 감지 및 카운트 관리
- `PostLogin` 이벤트에서 입장한 플레이어를 `MatchingPCArr`에 등록하고 실시간 UI 반영
- 5명 정원 충족 시 타이머 기반으로 자동 게임 레벨 전환 (`OpenLevel("GameMap")`)
- 매칭 UI(`BP_MyMatchingHUD`)에 실시간 인원 현황 표시

### 🎭 Prop 변신 시스템 (생존자 핵심 고유 기술)
- 생존자가 맵 내 소품(Prop)으로 위장 변신 가능 (`ChangeToObject` / `ChangeToPlayer` RPC)
- 변신 시 캐릭터 메시 숨김 + 스태틱 메시(소품) 활성화 + 물리 시뮬레이션 전환
- 변신 전 위치·회전값 저장 (`FVChange`, `FRChange`) 후 복귀 시 원위치 복원
- 쿨타임 시스템(`bChangeEnable`)으로 남용 방지

### 💓 킬러 근접 감지 & 공포 연출 시스템
- 킬러가 500 유닛 이내 접근 시 `ShowVinetting` 시스템 발동
- 카메라 포스트 프로세스로 Vignette(0~2.0), 필름 그레인(0~1.0), 지터 효과를 동적 조절
- 심장 소리(`AC_HeartBeat`) 및 추적 음악(`AC_Chase`) 자동 재생으로 긴장감 연출
- 거리 기반 음량 제어 및 오디오 컴포넌트 생존자 메시에 부착

### 🔧 머신(발전기) 수리 시스템
- `AMyMachine` 클래스: `RepairBox` 충돌 감지로 수리 가능 영역 진입 여부 판단
- 복수 생존자 동시 수리 시 진행 속도 중첩
- 수리 완료 시 조명 강도 변경(10000), 완료 메시로 스왑하여 시각적 피드백 제공
- 완료 머신 수(`DoneMachineNum`) 서버에서 관리 후 모든 클라이언트에 멀티캐스트 동기화

### 🎯 킬러 전투 시스템
- 메인 공격: Sweep Trace 충돌 감지 (ECC_GameTraceChannel3), 30 데미지, 2초 쿨타임
- 원거리 Q 공격: 7 데미지, 애니메이션 노티파이(`AnimNotify_RangeAttackAction`)로 타이밍 정밀 제어
- RC 투사체 공격: `AnimNotify_RCAttackAction` 호출 시 투사체 스폰
- 모든 쿨타임 값 서버에서 계산 후 `UpdateUI_Server` → 클라이언트 UI 실시간 반영

### 🎬 애니메이션 시스템
- `UMyAnimInstance`: C++ 기반 애니메이션 인스턴스로 상태 기반 애니메이션 전환
- `EPLAYER_STATE` 열거형으로 `IDLE`, `MOVE`, `JUMP`, `DASH`, `MACHINE`, `ATTACK`, `DEAD` 등 세분화
- 애니메이션 노티파이로 공격 판정, 투사체 스폰, 점프 정점 이벤트 연동

### 📊 플레이어 데이터 관리
- `DataTable` 기반 캐릭터 스탯 관리 (`DT_Survivor`, `DT_Killer`)
- `FSurvivorInfo` 구조체: 최대/현재 HP, SP, 수리 머신 수, 스턴 횟수, 코인
- `FKillerInfo` 구조체: 처치 수, 명중률, 각 스킬별 쿨타임 잔여 시간
- 구조체 전체를 `Replicated`로 선언해 클라이언트 UI와 자동 연동

### 🖥 HUD & UI 시스템
- 생존자 HUD: HP 바, SP 바, 머신 수리 진행도, 게임 타이머 + 머신 완료 아이콘 4개
- 킬러 HUD: Q/E/RC 스킬 쿨타임 프로그레스 바 + 수치 텍스트, 생존자 목록(이름·상태)
- 타이머 UI: MM:SS 포맷, 제로 패딩 처리, 매 프레임 `UpdateTimerUI` RPC로 동기화
- 맞은 캐릭터 머티리얼 빨간색 전환 후 일정 시간 뒤 원본 복구 (`TurnRed` / `TurnOriginalColor`)

### ✨ 이펙트 시스템
- `MyEffectManager` 싱글턴 패턴으로 파티클 이펙트 생성·관리
- `EKillerEffect` 열거형으로 ATTACK, Q, E, RIGHTCLICK, PARTICLEHIT 이펙트 분류
- `AMyEffect` 액터: 파티클 완료 콜백(`OnFinish`) 수신 후 자동 Destroy

---

## 🔑 주요 구현 포인트

- 3단계 게임 흐름(시작맵 → 매칭 로비 → 인게임)을 각각의 `GameMode` 클래스로 분리하여 명확한 책임 분리
- `UMyGameInstance`를 싱글턴 상태 컨테이너로 활용해 레벨 전환 간 플레이어 정보 유지
- 킬러와 생존자를 단일 `AMyCharacter` 베이스 클래스로 통합하고 역할별 서브클래스(`AKiller`, `ASurvivor`)로 확장
- `PostLogin` 이벤트에서 플레이어 타입(킬러/생존자)에 따라 별도 스폰 위치에 캐릭터 스폰
- 모든 움직임 관련 입력(`UpDown`, `LeftRight`, `MyJump`)을 Server RPC → Multicast 패턴으로 권한 서버 처리
- `OnRep_State` 콜백으로 상태 변경 시 클라이언트 측 애니메이션 인스턴스 자동 갱신
- 소품 변신 시 물리 시뮬레이션을 동적으로 활성화·비활성화해 소품처럼 자연스럽게 넘어지는 연출 구현
- 생존자 여러 명이 동일 머신을 동시 수리할 때 `surArr` 배열로 참여자를 추적하고 진행도를 누적 합산
- 킬러의 공격 판정을 `AnimNotify`와 연동해 애니메이션 타이밍에 정확히 일치하는 히트 판정 구현
- `DataTable`로 캐릭터 스탯을 외부에서 관리해 코드 수정 없이 밸런스 조정 가능
- 카메라 포스트 프로세스 파라미터(Vignette, Grain)를 거리 함수로 실시간 변조해 공포 몰입감 강화
- `HeadBox` 별도 컴포넌트로 머리 전용 충돌 레이어를 분리해 부위별 피격 판정 지원
- 쿨타임 잔여 시간을 서버에서 계산 후 `Client` RPC로 해당 플레이어에게만 전달해 불필요한 네트워크 트래픽 최소화
- 매칭 완료 감지를 `PostLogin` 카운트 방식으로 구현해 플레이어 수에 따른 자동 레벨 전환을 신뢰성 있게 처리
