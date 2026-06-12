# UE4 비대칭 PvP 숨바꼭질

**UE4 리슨 서버 기반 리플리케이션과 RPC를 활용한 킬러 vs 생존자 비대칭 멀티플레이어 숨바꼭질 게임**

[![Unreal Engine](https://img.shields.io/badge/Unreal%20Engine%204-0E1128?style=flat&logo=unrealengine&logoColor=white)](https://www.unrealengine.com/)
[![C++](https://img.shields.io/badge/C++-00599C?style=flat&logo=cplusplus&logoColor=white)](https://isocpp.org/)

[![게임플레이 영상](https://img.youtube.com/vi/oPROX41re1A/0.jpg)](https://youtu.be/oPROX41re1A?si=e3I2GQQJsZ8XT4Nq)

> 이미지를 클릭하면 게임플레이 영상을 볼 수 있습니다.
[![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=flat&logo=youtube&logoColor=white)](https://youtu.be/oPROX41re1A?si=e3I2GQQJsZ8WT4Nq)

---

## 프로젝트 정보

| 항목 | 내용 |
|---|---|
| 장르 | 비대칭 PvP 멀티플레이어 (킬러 vs 생존자) |
| 개발 기간 | 2022.04 ~ 2022.08 |
| 팀 구성 | 1인 개발 |
| 사용 기술 | Unreal Engine 4, C++, UE4 Replication, DataTable |

**본인 담당 파트:** 전체

---

## 프로젝트 개요

킬러 1명이 생존자 여러 명을 추격하고, 생존자는 사물로 변신해 숨으면서 발전기를 수리해 탈출하는 비대칭 PvP 게임입니다. Dead by Daylight 장르를 참고하여 제작했습니다.

UE4 리슨 서버 구조 위에서 `bReplicates`, Server/Multicast RPC를 활용하여
킬러의 공격, 생존자의 변신, 발전기 수리 진행도를 모든 클라이언트에 동기화했습니다.

---

## 코드 상세

### 플레이어 (`Source/MyProp/Player/`)

| 파일 | 역할 |
|---|---|
| [`Player/MyCharacter.h`](./MyProp/Source/MyProp/Player/MyCharacter.h) / [`MyCharacter.cpp`](./MyProp/Source/MyProp/Player/MyCharacter.cpp) | 킬러, 생존자 공통 베이스 클래스. 카메라(`SpringArm`, `Camera`), 3인칭 회전, `EPLAYER_STATE` Enum 기반 상태 관리 |
| [`Player/Survivor/Survivor.h`](./MyProp/Source/MyProp/Player/Survivor/Survivor.h) / [`Survivor.cpp`](./MyProp/Source/MyProp/Player/Survivor/Survivor.cpp) | 생존자 메인 클래스. 사물 변신용 `StaticMeshComponent` 부착, 킬러 접근 감지 시 심박음/추격음 전환, 피격 시 랜덤 SFX 재생, `bReplicates = true` 리플리케이션 설정 |
| [`Player/Survivor/Survivor_Change.cpp`](./MyProp/Source/MyProp/Player/Survivor/Survivor_Change.cpp) | 생존자 변신 로직. `LineTraceSingleByChannel`로 전방 사물 감지, 아웃라인 렌더(`SetRenderCustomDepth`), `ChangeToObject` Server RPC로 메시/스케일 동기화 |
| [`Player/Survivor/Survivor_Move.h`](./MyProp/Source/MyProp/Player/Survivor/Survivor_Move.h) / [`Survivor_Move.cpp`](./MyProp/Source/MyProp/Player/Survivor/Survivor_Move.cpp) | 생존자 이동 입력 처리 및 대시 로직 |
| [`Player/Survivor/Multi/Survivor_Multi.h`](./MyProp/Source/MyProp/Player/Survivor/Multi/Survivor_Multi.h) / [`Survivor_Multi.cpp`](./MyProp/Source/MyProp/Player/Survivor/Multi/Survivor_Multi.cpp) | 생존자 멀티플레이어 전용 리플리케이션 처리 (Server, Multicast RPC) |
| [`Player/Killer/Killer.h`](./MyProp/Source/MyProp/Player/Killer/Killer.h) / [`Killer.cpp`](./MyProp/Source/MyProp/Player/Killer/Killer.cpp) | 킬러 메인 클래스. 근접 공격(`Attack`), 범위 공격(`RangeAttack`), 원거리 RC 발사체 공격(`RCAttack`), E키 특수 공격. 헤드 `BoxComponent` 오버랩으로 피격 판정 |
| [`Player/Killer/Multi/Killer_Multi.h`](./MyProp/Source/MyProp/Player/Killer/Multi/Killer_Multi.h) / [`Killer_Multi.cpp`](./MyProp/Source/MyProp/Player/Killer/Multi/Killer_Multi.cpp) | 킬러 멀티플레이어 전용 리플리케이션 처리 |
| [`Player/Killer/Projectile/MyProjectile.h`](./MyProp/Source/MyProp/Player/Killer/Projectile/MyProjectile.h) / [`MyProjectile.cpp`](./MyProp/Source/MyProp/Player/Killer/Projectile/MyProjectile.cpp) | 킬러 투사체 베이스 클래스 |
| [`Player/Killer/Projectile/KillerRCProjectile.h`](./MyProp/Source/MyProp/Player/Killer/Projectile/KillerRCProjectile.h) / [`KillerRCProjectile.cpp`](./MyProp/Source/MyProp/Player/Killer/Projectile/KillerRCProjectile.cpp) | RC(원격조종) 투사체. 생존자 충돌 시 데미지 처리 |
| [`Player/Anim/MyAnimInstance.h`](./MyProp/Source/MyProp/Player/Anim/MyAnimInstance.h) / [`MyAnimInstance.cpp`](./MyProp/Source/MyProp/Player/Anim/MyAnimInstance.cpp) | 캐릭터 애니메이션 인스턴스. 이동 속도, 상태 기반 애니메이션 전환 |
| [`Player/Effect/MyEffect.h`](./MyProp/Source/MyProp/Player/Effect/MyEffect.h) / [`MyEffect.cpp`](./MyProp/Source/MyProp/Player/Effect/MyEffect.cpp) | 개별 이펙트 액터 |
| [`Player/Effect/MyEffectManager.h`](./MyProp/Source/MyProp/Player/Effect/MyEffectManager.h) / [`MyEffectManager.cpp`](./MyProp/Source/MyProp/Player/Effect/MyEffectManager.cpp) | 이펙트 풀 관리 및 스폰 |
| [`Player/Common/MyCharacterState.h`](./MyProp/Source/MyProp/Player/Common/MyCharacterState.h) | `EPLAYER_STATE` enum 정의 (IDLE, OBJECT, REPAIR 등) |
| [`Player/Common/MyInfo.h`](./MyProp/Source/MyProp/Player/Common/MyInfo.h) | 킬러/생존자 스탯 구조체 (`FSurvivorInfo`, `FKillerInfo`) |

### 게임 오브젝트 / 발전기 (`Source/MyProp/`)

| 파일 | 역할 |
|---|---|
| [`Machine/MyMachine.h`](./MyProp/Source/MyProp/Machine/MyMachine.h) / [`MyMachine.cpp`](./MyProp/Source/MyProp/Machine/MyMachine.cpp) | 발전기 액터. `RepairBox`(BoxComponent) 오버랩으로 수리 가능 범위 감지. `Tick`에서 `CurRepairTime` 누적, `RepairTime`(5초) 달성 시 완료 처리. 완료 시 메시 교체 및 포인트 라이트 점등, `GameMode`에 완료 통보 |
| [`Object/MyPlayerObject.h`](./MyProp/Source/MyProp/Object/MyPlayerObject.h) / [`MyPlayerObject.cpp`](./MyProp/Source/MyProp/Object/MyPlayerObject.cpp) | 생존자가 변신할 수 있는 사물 액터. `SelectObj` 트레이스 채널로 탐지 대상 지정 |

### 게임 모드 / 매칭 (`Source/MyProp/Mode/`)

| 파일 | 역할 |
|---|---|
| [`Mode/MyPropGameModeBase.h`](./MyProp/Source/MyProp/Mode/MyPropGameModeBase.h) / [`MyPropGameModeBase.cpp`](./MyProp/Source/MyProp/Mode/MyPropGameModeBase.cpp) | 인게임 GameMode. 발전기 완료 수 집계, 전체 완료 시 게임 종료 처리. 킬러/생존자 스폰 관리 |
| [`Mode/MyMatchingModeBase.h`](./MyProp/Source/MyProp/Mode/MyMatchingModeBase.h) / [`MyMatchingModeBase.cpp`](./MyProp/Source/MyProp/Mode/MyMatchingModeBase.cpp) | 매칭 GameMode. 플레이어 접속 감지, 인원 충족 시 인게임 씬으로 전환 |
| [`Mode/MyStartModeBase.h`](./MyProp/Source/MyProp/Mode/MyStartModeBase.h) / [`MyStartModeBase.cpp`](./MyProp/Source/MyProp/Mode/MyStartModeBase.cpp) | 시작 씬 GameMode |

### 컨트롤러 / 게임 인스턴스 (`Source/MyProp/`)

| 파일 | 역할 |
|---|---|
| [`Controller/MyPlayerController.h`](./MyProp/Source/MyProp/Controller/MyPlayerController.h) / [`MyPlayerController.cpp`](./MyProp/Source/MyProp/Controller/MyPlayerController.cpp) | 플레이어 컨트롤러. UI 표시/숨김, 발전기 수리 진행도 갱신, 킬러/생존자 역할별 HUD 분기 |
| [`Controller/MyMatchingController.h`](./MyProp/Source/MyProp/Controller/MyMatchingController.h) / [`MyMatchingController.cpp`](./MyProp/Source/MyProp/Controller/MyMatchingController.cpp) | 매칭 씬 전용 컨트롤러. 매칭 대기 중 플레이어 목록 관리 |
| [`GameInstance/MyGameInstance.h`](./MyProp/Source/MyProp/GameInstance/MyGameInstance.h) / [`MyGameInstance.cpp`](./MyProp/Source/MyProp/GameInstance/MyGameInstance.cpp) | 씬 전환 간 유지되는 글로벌 인스턴스. `DataTable`에서 킬러/생존자 스탯 로드(`GetKillerInfo`, `GetSurvivorInfo`). 킬러/생존자 Pawn 클래스, HUD 위젯 클래스 참조 보관 |

### UI (`Source/MyProp/UI/`)

| 파일 | 역할 |
|---|---|
| [`UI/MyMainHUD.h`](./MyProp/Source/MyProp/UI/MyMainHUD.h) / [`MyMainHUD.cpp`](./MyProp/Source/MyProp/UI/MyMainHUD.cpp) | 생존자 인게임 HUD |
| [`UI/Killer/MyKillerMainHUD.h`](./MyProp/Source/MyProp/UI/Killer/MyKillerMainHUD.h) / [`MyKillerMainHUD.cpp`](./MyProp/Source/MyProp/UI/Killer/MyKillerMainHUD.cpp) | 킬러 인게임 HUD |
| [`UI/MyMachineWidget.h`](./MyProp/Source/MyProp/UI/MyMachineWidget.h) / [`MyMachineWidget.cpp`](./MyProp/Source/MyProp/UI/MyMachineWidget.cpp) | 발전기 수리 진행도 UI |
| [`UI/MyHPBarWidget.h`](./MyProp/Source/MyProp/UI/MyHPBarWidget.h) / [`MyHPBarWidget.cpp`](./MyProp/Source/MyProp/UI/MyHPBarWidget.cpp) | 체력바 위젯 |
| [`UI/Common/MyTimerWidget.h`](./MyProp/Source/MyProp/UI/Common/MyTimerWidget.h) / [`MyTimerWidget.cpp`](./MyProp/Source/MyProp/UI/Common/MyTimerWidget.cpp) | 인게임 타이머 UI |
| [`UI/Common/MyMatchingHUD.h`](./MyProp/Source/MyProp/UI/Common/MyMatchingHUD.h) / [`MyMatchingHUD.cpp`](./MyProp/Source/MyProp/UI/Common/MyMatchingHUD.cpp) | 매칭 대기 화면 HUD |
| [`UI/Common/MyOtherPlayerWidget.h`](./MyProp/Source/MyProp/UI/Common/MyOtherPlayerWidget.h) / [`MyOtherPlayerWidget.cpp`](./MyProp/Source/MyProp/UI/Common/MyOtherPlayerWidget.cpp) | 다른 플레이어 상태 표시 위젯 |
| [`UI/Common/MyEndingHUD.h`](./MyProp/Source/MyProp/UI/Common/MyEndingHUD.h) / [`MyEndingHUD.cpp`](./MyProp/Source/MyProp/UI/Common/MyEndingHUD.cpp) | 게임 종료 결과 화면 |
| [`UI/Function/SelectCharacter.h`](./MyProp/Source/MyProp/UI/Function/SelectCharacter.h) / [`SelectCharacter.cpp`](./MyProp/Source/MyProp/UI/Function/SelectCharacter.cpp) | 킬러/생존자 캐릭터 선택 UI |
| [`UI/MySPWidget.h`](./MyProp/Source/MyProp/UI/MySPWidget.h) / [`MySPWidget.cpp`](./MyProp/Source/MyProp/UI/MySPWidget.cpp) | 특수 능력 게이지 위젯 |
| [`UI/MyStartGameWidget.h`](./MyProp/Source/MyProp/UI/MyStartGameWidget.h) / [`MyStartGameWidget.cpp`](./MyProp/Source/MyProp/UI/MyStartGameWidget.cpp) | 게임 시작 화면 위젯 |
| [`UI/MyStartHUD.h`](./MyProp/Source/MyProp/UI/MyStartHUD.h) / [`MyStartHUD.cpp`](./MyProp/Source/MyProp/UI/MyStartHUD.cpp) | 시작 씬 HUD |

---

## 핵심 구현

### UE4 C++ 리플리케이션 직접 구현

UE4 리슨 서버 구조에서 `bReplicates = true` 설정과  Server/Multicast RPC를 활용하여 킬러 공격, 생존자 변신 메시 교체, 발전기 수리 진행도를 모든 클라이언트에 동기화했습니다.

### 생존자 변신 시스템

`Survivor_Change`에서 `LineTraceSingleByChannel`로 전방 1000 유닛 이내의 사물을 탐지합니다. 감지된 사물에는 `SetRenderCustomDepth`로 아웃라인을 렌더링하여 시각 피드백을 제공합니다. 변신 시 생존자 캐릭터 메시를 숨기고 사물의 `StaticMesh`와 스케일을 복사해 덮어쓰며, 이 처리 전체를 Server RPC -> Multicast RPC 순으로 전파하여 모든 클라이언트에서 동기화합니다.

### 발전기 수리 시스템

`MyMachine`의 `RepairBox`(BoxComponent) 범위 안에 생존자가 진입하면 수리가 시작됩니다. `Tick`에서 `CurRepairTime`을 누적하여 `RepairTime`(5초) 달성 시 완료 처리하고, 메시를 완료 메시로 교체하며 포인트 라이트를 점등합니다. 완료 정보는 `GameMode`로 전달되어 전체 발전기 완료 수를 집계하고 탈출 조건을 판단합니다.

### DataTable 기반 캐릭터 스탯 관리

킬러/생존자 스탯을 UE4 DataTable(`DT_Killer`, `DT_Survivor`)로 관리합니다. `MyGameInstance`가 `FindRow`로 스탯을 로드하여 `BeginPlay` 시 각 캐릭터에 주입하므로, 코드 수정 없이 에디터에서 수치를 조정할 수 있습니다.

### 킬러 3종 공격 시스템

근접 공격(좌클릭), 범위 공격(Q키), RC 투사체 공격(우클릭), E키 특수 공격으로 구성됩니다. 각 공격은 독립적인 쿨타임 플래그(`bAttackEnable`, `bRangeAttackEnable`, `bRCAttackEnable`)로 관리됩니다. RC 투사체(`KillerRCProjectile`)는 `SpawnActor`로 생성되며 생존자 충돌 시 데미지를 처리합니다.

---

## 폴더 구조

- `MyProp/Source/MyProp/Player/` : 킬러, 생존자 캐릭터 및 공통 베이스
- `MyProp/Source/MyProp/Player/Killer/` : 킬러 로직, 투사체
- `MyProp/Source/MyProp/Player/Survivor/` : 생존자 로직, 변신, 이동
- `MyProp/Source/MyProp/Machine/` : 발전기 액터
- `MyProp/Source/MyProp/Object/` : 변신 대상 사물 액터
- `MyProp/Source/MyProp/Mode/` : 인게임/매칭/시작 GameMode
- `MyProp/Source/MyProp/Controller/` : 플레이어 컨트롤러, 매칭 컨트롤러
- `MyProp/Source/MyProp/GameInstance/` : 글로벌 게임 인스턴스, DataTable 로드
- `MyProp/Source/MyProp/UI/` : 킬러/생존자/공통/매칭/결과 UI
- `MyProp/Content/Blueprints/` : Blueprint 에셋
