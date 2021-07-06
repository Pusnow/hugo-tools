---
title: 파일시스템의 종류 - 로컬, 네트워크, 분산
date: 2021-06-07
tags:
    - system
lang: ko-kr
---

Ceph dissertation (Ceph: Reliable, Scalable, and High-performance Distributed Storage, 2007)의 Related Work 부분을 요약 정리하였다.
당시에는 SSD가 대중화되어있지 않아서, 이 dissertation에서 다루는 디스크는 대부분 하드 디스크이다.

## 파일시스템의 분류

파일시스템은 크게 세 가지로 분류된다.

- 로컬 파일시스템: 로컬에 장착된 디스크와 상호 작용
- 네트워크 파일시스템 (클라이언트-서버 파일시스템): 원격 서버와 상호 작용
- 분산 파일시스템: 파일시스템의 데이터가 여러 호스트로 분산

## 로컬 파일시스템

수십 년간 파일시스템은 Unix 파일시스템과 BSD 유닉스의 Fast 파일시스템 (Fast File System, FFS)에 크게 영향을 받아왔다.
이의 인터페이스와 행동 시멘틱은 POSIX SUS (Single Unix Specification)으로 표준화되어서, 대부분의 모던 시스템과 애플리케이션에서 활용한다.

대부분의 로컬 파일시스템은 로컬에 장착되어있는 하드디스크(고정된 크기의 섹터와 블록으로 저장)에 맞추어 디자인되어있다.
예를 들면, 하드디스크는 탐색 (seek) 성능이 떨어지는데, 이를 만회하기 위하여 FFS는 실린더 그룹(cylinder groups)을 사용한다.
실린더 그룹은 데이터와 이와 연관된 메타데이터를 같은 실린더 영역에 저장하여, 메타데이터 탐색 시간을 최소화한다.

하드 디스크 대역폭 성능은 많이 개선되었지만, 탐색 성능은 이와 비교하면 매우 느리게 개선되었다.
이러한 하드웨어 변화상에 따라서 탐색에 소요되는 지연 시간을 더욱더 줄이기 위하여 Log-structured File System (LFS) 이 등장하였다 (Sprite networking OS).
LFS는 쓰기 워크로드에서 지연시간을 최소화하기 위하여, 새로운 데이터를 연속된 로그 형태로 디스크에 저장한다.
하지만, 이 방식은 주기적으로 "cleaner"가 로그의 삭제된 영역을 재배치하여 디스크의 유휴 공간을 확보해야 하는데, 이는 많은 워크로드에서 파일시스템 성능을 저하한다.

실린더 그룹의 컨셉은 수년간 정돈되어 왔지만, 이에 사용되는 기초적인 데이터 구조와 할당 전략은 현대의 파일시스템 (예: ext3) FFS와 크게 달라지지 않았다.
예를 들면, 디스크의 블록 크기는 초창기의 일반적인 파일 사이즈(4KB)에 맞추어 디자인되어 있지만, 더는 4KB의 파일이 일반적이지 않은 현대에도 여전히 쓰이고 있다.
이 점은 큰 파일을 저장할 때 매우 긴 블록 할당 리스트를 만들고 순회하게 된다.
XFS와 같은 현대의 파일시스템은 할당 리스트 대신 extents(시작점과 길이의 쌍)를 사용하여 효과적으로 디스크의 큰 영역을 기술한다.

현대의 파일시스템은 재해 발생시 빠른 복구를 수행하기 위해 신뢰성 기능을 추가하기도 한다.
XFS는 디스크 안에 저널 파일을 두어서 메타데이터 업데이트를 수행하기 전에 미리 기록하여 재해에 대비한다.
재해가 발생했을 때, 이 저널 파일을 이용하면 수행하려던 업데이트를 정확하게 반영하거나 부분적으로 수행하였으나 실패한 업데이트를 취소할 수 있다.
저널 기능은 성능상의 저하를 가져오지만, 재해 복구 시 수행되는 매우 비싼 정합성 체크를 회피할 수 있다.

많은 시스템(WAFL, EBOFS, 등)은 또한 soft updates 기능을 구현하기도 한다.
Soft updates는 파일시스템 수정을 항상 디스크의 유휴 공간에 작성하여 디스크의 이미지가 항상 정합성을 유지할 수 있게 한다.

## 클라이언트-서버 파일시스템

네트워크 환경에서 스토리지 자원을 공유하기 위하여 클라이언트-서버 파일시스템이 등장하였다.
NFS와 CIFS 파일시스템이 대표적인 네트워크 파일시스템으로, 이 시스템들은 중앙 서버가 로컬 파일시스템을 "export" 하는 것을 허용한다.
클라이언트는 export된 파일시스템들을 각각의 네임 스페이스에 매핑하여 사용한다.
중앙 스토리지는 NAS로 대중화되어 최적화되고 고성능의 스토리지 시스템 제공된다.

하지만, 중앙 서버 디자인은 확장성(scalability)에 제한을 두게 된다.
모든 파일시스템 작업이 하나의 서버에서만 수행되어야 하기 때문이다.
이 때문에, 대규모의 클러스터에서는 일반적으로 전체 데이터를 여러 부분 집합으로 나누어 각 부분을 담당하는 여러 서버로 나누어 저장하게 된다.
이 방법은 효과적이고 널리 사용되지만, 특정 데이터 부분 집합의 크기가 커지면 다른 서버로의 데이터 이전을 고려해야 하고, 이 점은 관리상의 비용 증가로 이어진다.

네트워크 파일시스템은 높은 캐시 성능을 위하여 정합성 시멘틱을 완화한다.
예를 들면, NFS 클라이언트는 파일 데이터를 서버에 비동기적으로 전송하여, 다른 클라이언트가 가장 최근의 내용이 아닌 이전 내용을 읽을 수도 있다.
이와 같은 완화된 정합성은 많은 애플리케이션에서 문제들을 발생시킬 수 있으며 이 애플리케이션들이 NFS-기반 환경에서의 작동하지 못하게 한다.

## 분산 파일시스템

분산 파일시스템은 클라이언트-서버 파일시스템의 로드 밸런싱과 스케일링 문제를 해결하려 등장했다.
초창기의 분산 파일시스템(예: AFS, Coda, Sprite)은 같은 데이터에 대한 접근을 제어하여 정합성을 보장하기 위해 중앙 서버가 파일시스템 접근을 조정(coordinate)한다.
중앙 서버는 *leases*를 발급하여 클라이언트가 캐시를 유효하게 사용할 수 있는 기간을 보장하고, 만일 특정 데이터에 대한 접근이 충돌한다면, 이어지는 *callbacks*를 통하여 보장 기간을 무효로 한다.
구체적으로는 Sprite의 파일시스템은 단순히 쓰기가 동시에 발생하는 경우 캐시를 비활성화하고, AFS는 파일 open/close 기반의 정합성 모델을 제공한다.

분산 파일시스템은 데이터를 여러 서버에 분산하여 저장하게 된다.
Sprite는 네임스페이스를 "도메인"으로 파티션하고 동적으로 이 도메인을 서버와 매핑하게 된다.
AFS는 전체 데이터를 파일 이름과 볼륨 식별자에 기반하여 파티션하고 이에 기반하여 서버에 저장한다.
이런 시스템에서는 파티션간 *link* 혹은 *atomic rename*이 지원하지 않는다.

분산 구조는 신뢰성과 가용성을 위하여 데이터를 여러 서버에 중복하여 저장하게 된다.
Harp 파일시스템은 primary-copy 복제 기법과 write-ahead 로그를 통하여 재해 발생시 빠른 복구를 지원한다.
많은 수의 스토리지 서브시스템(예: Petal, FAB)은 이와 유사한 신뢰성 블록-기반 접근 기능을 제공한다.
또한, 로컬 파일시스템이 신뢰성을 위해 RAID를 사용하는 것처럼 Frangipani와 같은 시스템은 분산 스토리지-레이어 추상계층을 사용하여 분산 파일시스템을 구현한다.

### Wide-area 파일시스템

많은 시스템은 파일시스템 구성요소를 WAN을 통하여 분산하는 시도를 하였다.

- xFS: 무효화-기반 캐시 정합성 프로토콜을 사용한 공격적인 클라이언트 캐싱을 사용하고, 피어의 캐시 읽기를 통하여 WAN을 통한 데이터 통신을 감소시켰다.
- OceanStore: erasure 코드와 위치-독립적인 라우팅 추상화를 통하여 전역적으로 가용한 파일 스토리지를 구현하였다.
- Pangaea: 강한 정합성이 필요하지 않은 경우 replica의 정합성을 완화하고, 공격적으로 파일 내용을 복제하여 WAN 기반 파일시스템을 제공한다.

### SAN 파일시스템

다른 파일에 대한 I/O는 서로 큰 관계가 없음으로 대부분의 파일 I/O는 동시에 수행이 가능하다.
따라서, 가장 최신의 "cluster" 파일시스템은 기반 스토리지 장치에 대한 공유된 접근 방식에 기반한다.
SAN (storage area network) 파일시스템은 파이버 채널과 같은 네트워크를 통하여 통신하는 하드 디스크 혹은 RAID 컨트롤러를 활용한다.
이 방식은 임의의 호스트가 임의의 연결된 디스크에 명령을 주는 것을 허용한다.

대부분의 SAN 파일시스템은 분산 락 관리자(distributed lock manager, DLM)을 사용하여 공유 블록 디바이스에 대한 접근을 조정한다.
추가로 GPFS와 같은 시스템들은 락 컨텐션을 줄이기 위한 최적화를 진행하였고, StorageTank는 metadata에 대한 락 컨텐션이 빈번하게 발생하는 것에서 착안하여 메타데이터를 위한 보조 서버 클러스터를 구성하였다.

### 객체 및 Brick-기반 파일시스템

최근에는 다양한 파일시스템과 플랫폼(예: FAB, PVFS, pNFS)이 NAS 클러스터를 기반으로 디자인되고 있다.
StorageTank 모델처럼 메타데이터를 독립적인 서버에서 수행하지만 파일 I/O는 스토리지 서버 클러스터("bricks"라 불리기도 함)에 수행하는 디자인이 제안되었다.

NASD가 대중화한 객체-기반 스토리지 패러다임은 Lustre, Panasas 파일시스템, zFS, Sorrento, Ursa Minor, Kybos와 같은 파일시스템을 제안하였다.
이 시스템은 I/O를 작고 고정된 크기의 블록을 단위호 하는 것이 아니라 이름을 가진 가변 길이의 객체로 저장하는 방식을 취한다.
객체-기반의 스토리지는 저수준의 작업(블록 할당, 보안)을 각 장치에서 수행하여, 메타데이터 관리에 가해지는 컨텐션을 줄이고 전체 시스템의 확장성을 증가시킨다.

### 비-POSIX 파일시스템

많은 수의 분산 파일시스템은 비-POSIX 파일시스템 인터페이스를 제공한다.

- Farsite: 많은 수의 비신뢰 워크스테이션을 비잔틴 결함-내성 분산 파일시스템으로 구성한다.
- Sorrento: 완화된 정합성 모델을 통하여 메타데이터 관리와 공유 쓰기의 업데이트 정합성을 간소화한다.
- Cedar: 정합성 문제를 풀기 위하여 공유된 파일을 불변하게 하고 버저닝을 도입한다.
- Venti: Cedar와 유사하지만 모든 파일이 불변하다.
- Google 파일시스템: 구조적으로 객체-기반 시스템과 유사하지만, 표준 파일시스템 인터페이스를 사용하지 않고, 매우 큰 파일들과 매우 큰 읽기 및 파일 내용 추가(append) 워크로드에 최적화 되어 있다.