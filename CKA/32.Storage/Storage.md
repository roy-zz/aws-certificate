# Storage

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "스토리지"에 대해서 자세하게 알아보도록 한다.

---

## Storage in Docker

### Docker 데이터 저장 위치

![1-file-system.png](images%2F1-file-system.png)

- Docker는 기본적으로 `/var/lib/docker` 디렉토리에 데이터를 저장한다.
- 이 디렉토리에는 `aufs`, `containers`, `image`, `volumes` 등의 하위 디렉토리가 있다.
  - `conatiners`: 컨테이너 관련 파일 저장
  - `image`: 이미지 관련 파일 저장
  - `volumes`: 볼륨 관련 파일 저장

### Docker 레이어 아키텍처

![2-layered-architecture-1.png](images%2F2-layered-architecture-1.png)

- Docker 이미지는 레이어 구조로 빌드된다.
- Dockerfile의 각 명령어에는 새로운 레이어를 생성하며, 각 레이어에는 이전 레이어의 변경 사항만 저장한다.
- 예시:
  - Ubuntu 기본 운영 체제 레이어
  - APT 패키지 설치 레이어
  - Python 패키지 설치 레이어
  - 소스 코드 복사 레이어
  - 이미지 엔트리 포인트 업데이트 레이어
- 레이어 구조의 장점:
  - 이미지 빌드 속도 향상: 동일한 레이어 재사용 (캐시 활용)
  - 디스크 공간 효율성 향상: 변경 사항만 저장
  - 이미지 업데이트 속도 향상: 변경된 레이어만 재빌드

### 이미지 레이어와 컨테이너 레이어

![3-layered-architecture-2.png](images%2F3-layered-architecture-2.png)

#### 이미지 레이어

- `docker build` 명령어로 생성된 레이어
- 읽기 전용(read-only)
- 이미지 빌드 후에는 수정 불가
- 여러 컨테이너에서 공유

#### 컨테이너 레이어

- `docker run` 명령어로 컨테이너 실행 시 생성되는 쓰기 가능(writable) 레이어
- 컨테이너에서 생성된 데이터(로그 파일, 임시 파일, 수정된 파일 등) 저장
- 컨테이너 삭제 시 함께 삭제

### Copy-on-Write 메커니즘

![4-copy-on-write.png](images%2F4-copy-on-write.png)

- 이미지 레이어의 파일 수정
  - 컨테이너 내부에서 이미지 레이어의 파일을 수정하면, Docker는 해당 파일의 복사본을 컨테이너 레이어에 생성한다.
  - 이후 수정은 컨테이너 레이어의 복사본으로 적용된다.
  - 이미지 레이어의 원본 파일은 변경되지 않는다.
- 장점
  - 이미지 레이어의 무결성 유지
  - 컨테이너별로 독립적인 파일 수정 가능

### 컨테이너 삭제 시 데이터 손실

- 컨테이너 삭제 시 컨테이너 레이어의 모든 데이터가 삭제된다.
- 수정된 팡리 및 새로 생성된 파일도 함께 삭제된다.
- 데이터 영속성을 위해 볼륨(volumes)을 사용해야 한다.

#### 데이터 영속성 (Persistent Data)

- **컨테이너 삭제 시 데이터 손실**: 컨테이너 레이어는 컨테이너 삭제 시 함께 삭제되므로 데이터가 손실된다.
- **데이터 영속성을 위한 볼륨(Volumes) 사용**: 컨테이너 삭제 후에도 데이터를 유지하려면 볼륨을 사용해야 한다.

### Docker 볼륨 생성 및 마운트

![5-volumes.png](images%2F5-volumes.png)

- **볼륨 생성**: `docker volume create <볼륨 이름>` 명령어를 사용하여 볼륨을 생성한다.
  - 예: `docker volume create data_volume`
  - 생성된 볼륨은 `/var/lib/docker/volumes/<볼륨 이름>/_data` 디렉토리에 저장된다.
- **컨테이너에 볼륨 마운트**: `docker run -v <볼륨 이름>:<컨테이너 내부 경로> <이미지 이름>` 명령어를 사용하여 컨테이너에 볼륨을 마운트한다.
  - 예: `docker run -v data_volume:/var/lib/mysql mysql`
  - 컨테이너 내부에 지정된 경로에 볼륨이 마운트되어 데이터가 저장된다.
- **자동 볼륨 생성**: `docker run` 명령어 실행 시 존재하지 않는 볼륨을 지정하면 Docker가 자동으로 볼륨을 생성하고 마운트한다.

#### 바인드 마운트 (Bind Mount)

- **호스트 시스템의 디렉토리 마운트**: 호스트 시스템의 특정 디렉토리를 컨테이너에 마운트한다.
- **명령어**: `docker run -v <호스트 디렉토리 경로>:<컨테이너 내부 경로> <이미지 이름>`
  - 예: `docker run -v /host/data:/container/data nginx`
  - 호스트 시스템의 `/data/mysql` 디렉토리가 컨테이너의 `/var/lib/mysql` 디렉토리에 마운트된다.
- **볼륨 마운트 vs 바인드 마운트**:
  - 볼륨 마운트: Docker가 관리하는 볼륨 사용 (`/var/lib/docker/volumes`)
  - 바인드 마운트: 호스트 시스템의 임의 디렉토리 사용

#### `--mount` 옵션 (권장)

- **`-v` 옵션의 대체**: `--mount` 옵션은 `-v` 옵션보다 명시적이고 유연하다.
- **명령어**: `docker run --mount type=<mount type>,source=<source>,target=<target> <이미지 이름>`
  - `type`: `volume` 또는 `bind`
  - `source`: 볼륨 이름 또는 호스트 디렉토리 경로
  - `target`: 컨테이너 내부 경로
- **예시 (바인드 마운트)**: `docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`

### 스토리지 드라이버 (Storage Drivers)

![6-storage-drivers.png](images%2F6-storage-drivers.png)

- **역할**: 레이어 아키텍처 관리, 쓰기 가능 레이어 생성, Copy-on-Write 구현 등
- **종류**: AUFS, VTRFS, Device Mapper, Overlay, Overlay2 등
- **자동 선택**: Docker는 운영 체제에 따라 최적의 스토리지 드라이버를 자동으로 선택한다.
- **성능 및 안정성**: 스토리지 드라이버마다 성능 및 안정성 특성이 다르므로 애플리케이션 요구 사항에 맞는 드라이버를 선택해야 한다.

---

## Storage Driver

### 스토리지 드라이버와 볼륨 드라이버의 차이

![8-storagedrivers-volumedrivers.png](images%2F8-storagedrivers-volumedrivers.png)

#### 스토리지 드라이버 (Storage Driver)

- 이미지와 컨테이너의 스토리지 관리를 담당한다.
- 레이어 아키텍처, 쓰기 가능 레이어, Copy-on-Write 등을 처리한다.

#### 볼륨 드라이버 플러그인 (Volume Driver Plugin)

- 볼륨 관리를 담당한다.
- 데이터 영속성을 위한 볼륨 생성 및 관리를 처리한다.
- 스토리지 드라이버와 볼륨 드라이버는 서로 다른 역할을 수행한다.

### 기본 볼륨 드라이버 플로그인 (Local)

- **기본 플러그인**: Docker 설치 시 기본적으로 제공되는 플러그인이다.
- **기능**: Docker 호스트에 볼륨을 생성하고 데이터를 `/var/lib/docker/volumes` 디렉토리에 저장한다.

### 다양한 볼륨 드라이버 플러그인

- **외부 스토리지 솔루션 지원**: 다양한 볼륨 드라이버 플러그인을 사용하여 Azure File Storage, Convoy, DigitalOcean Block Storage, Google Compute Persistent Disks 등과 같은 외부 스토리지 솔류션에 볼륨을 생성할 수 있다.
- **예시**:
  - Azure File Storage
  - Convoy
  - DigitalOcean Block Storage
  - Flocker
  - Google Compute Persistent Disks
  - Cluster FS
  - NetApp
  - REX-Ray
  - Portworx
  - VMware vSphere Storage

- **다양한 스토리지 제공업체 지원**: 일부 볼륨 드라이버는 여러 스토리지 제공업체를 지원한다.
  - REX-Ray: AWS EBS, S3, EMC Isilon, ScaleIO, Google Persistent Disk, OpenStack Cinder 등 지원

### 외부 볼륨 드라이버 플러그인 사용

![9-volume-drivers.png](images%2F9-volume-drivers.png)

- **특정 볼륨 드라이버 지정**: `docker run` 명령어 실행 시 특정 볼륨 드라이버를 지정하여 외부 스토리지에 볼륨을 생성하고 컨테이너에 연결할 수 있다.
- **예시**: REX-Ray EBS 볼륨 드라이버를 사용하여 AWS EBS에서 볼륨을 프로비저닝하고 컨테이너에 연결한다.
- **데이터 영속성 보장**: 컨테이너 종료 후에도 외부 스토리지에 데이터가 안전하게 보관된다.

---

## Container Storage Interface

### 배경 및 필요성

- **과거 쿠버네티스와 Docker**: 과거에는 쿠버네티스가 Docker만을 컨테이너 런타임 엔진으로 사용했으며, Docker 관련 코드가 쿠버네티스 소스 코드에 내장되어 있었다.

![10-container-runtime-interface-1.png](images%2F10-container-runtime-interface-1.png)

- **다양한 컨테이너 런타임 등장**: RKT, CRI-O 등 다양한 컨테이너 런타임이 등장하면서 쿠버네티스는 다양한 런타임을 지원하고 독립성을 유지해야 했다.
- **CRI (Container Runtime Interface)**: 쿠버네티스와 같은 오케스트레이션 솔루션이 Docker와 같은 컨테이너 런타임과 통신하는 방식을 정의하는 표준이다.
  - 새로운 컨테이너 런타임은 CRI 표준을 준수하여 쿠버네티스 개발팀의 도움 없이 쿠버네티스와 연동될 수 있다.

![11-container-network-interface.png](images%2F11-container-network-interface.png)

- **CNI (Container Networking Interface)**: 다양한 네트워킹 솔루션을 지원하기 위해 도입되었다.
  - 새로운 네트워킹 공급업체는 CNI 표준을 기반으로 플러그인을 개발하여 쿠버네티스와 연동할 수 있다.

![12-container-storage-interface-1.png](images%2F12-container-storage-interface-1.png)

- **CSI (Container Storage Interface)**: 다양한 스토리지 솔류션을 지원하기 위해 개발되었다.

### CSI (Container Storage Interface)

![13-container-storage-interface-2.png](images%2F13-container-storage-interface-2.png)

- **목적**: 쿠버네티스와 다양한 스토리지 벤더의 솔루션을 연동하기 위한 범용 표준이다.
- **CSI 드라이버**: 각 스토리지 벤더(Portworx, Amazon EBS, Azure Disk, Dell EMC Isilon, NetApp 등)는 자체 CSI 드라이버를 제공한다.
- **범용 표준**: CSI는 쿠버네티스 특정 표준이 아니며, 컨테이너 오케스트레이션 도구와 스토리지 벤더를 연결하는 범용 표준이다.
  - 쿠버네티스, Cloud Foundry, Mesos 등이 CSI를 지원한다.

### CSI 작동 방식

- **RPC (Remote Procedure Calls) 정의**: CSI는 컨테이너 오케스트레이터가 호출해야 하는 RPC 집합을 정의한다.
- **스토리지 드라이버 구현**: 스토리지 드라이버는 CSI에서 정의된 RPC를 구현해야 한다.
- **예시**:
  - **CreateVolume RPC**: 파드 생성 시 볼륨이 필요한 경우 쿠버네티스는 CreateVolume RPC를 호출하고 볼륨 이름 등의 정보를 전달한다. 스토리지 드라이버는 이 요청을 처리하고 스토리지 어레이에 새 볼륨을 프로비저닝한 후 결과를 반환한다.
  - **DeleteVOlume RPC**: 볼륨 삭제 시 쿠버네티스는 DeleteVolume RPC를 호출한다. 스토리지 드라이버는 이 요청을 처리하고 스토리지 어레이에서 볼륨을 삭제한다.
- **명세**: CSI 명세는 호출자가 전달해야 하는 매개변수, 솔루션이 수신해야 하는 매개변수, 교환해야 하는 오류 코드 등을 상세히 정의한다.

---

## Volumes

### 쿠버네티스 볼륨의 기본 개념

#### Docker 볼륨과의 유사성

![14-volumes-1.png](images%2F14-volumes-1.png)

- Docker 컨테이너와 마찬가지로 쿠버네티스 파드도 일시적인 성격을 가진다.
- 파드가 삭제되면 파드 내부의 데이터도 함께 삭제된다.
- 데이터를 영구적으로 유지하기 위해 볼륨을 파드에 연결한다.

#### 파드 데이터의 영속성

![15-volumes-2.png](images%2F15-volumes-2.png)

- 볼륨에 데이터를 저장하면 파드가 삭제되어도 데이터는 유지된다.

### 쿠버네티스 볼륨 구현 예시

![16-volumes-mounts-1.png](images%2F16-volumes-mounts-1.png)

#### 단일 노드 클러스터

- 1 ~ 100 사이의 난수를 생성하여 `/opt/number.out` 파일에 기록하는 파드를 생성한다.
- 파드 삭제 시 난수 데이터도 함께 삭제된다.
- 데이터 유지를 위해 볼륨을 생성한다.

#### 볼륨 스토리지 설정

- 볼륨 스토리지는 다양한 방식으로 설정할 수 있다.
- 단순화를 위해 호스트 노드의 디렉토리(`/data`)를 스토리지로 사용한다.

#### 볼륨 마운트

- 파드 컨테이너 내부의 `/opt` 디렉토리에 볼륨을 마운트한다.
- 난수는 컨테이너 내부의 `/opt` 디렉토리에 저장되고, 이는 호스트 노드의 `/data` 디렉토리에 실제로 저장된다.

#### 파드 삭제 후 데이터 유지

- 파드가 삭제되어도 난수 파일은 호스트 노드의 `/data` 디렉토리에 남아있다.

### 볼륨 스토리지 옵션

#### 호스트 경로 (hostPath)

![17-volume-types-1.png](images%2F17-volume-types-1.png)

- 호스트 노드의 디렉토리를 볼륨 스토리지로 사용한다.

![18-volume-types-2.png](images%2F18-volume-types-2.png)

- 단일 노드 클러스터에는 작동하지만 다중 노드 클러스터에서는 권장되지 않는다.

![19-volume-types-3.png](images%2F19-volume-types-3.png)

- 다중 노드 클러스터에서는 파드가 모든 노드의 `/data` 디렉토리를 사용하므로 데이터 불일치가 발생할 수 있다.

#### 다양한 스토리지 솔루션 지원

![20-volume-types-4.png](images%2F20-volume-types-4.png)

- 쿠버네티스는 NFS, ClusterFS, Flocker, Fiber Channel, CephFS, ScaleIO와 같은 다양한 스토리지 솔루션을 지원한다.
- AWS EBS, Azure Disk, Google Persistent Disk와 같은 퍼블릭 클라우드 스토리지 솔류션도 지원한다.

### AWS EBS 볼륨 설정 예시

![21-volume-types-5.png](images%2F21-volume-types-5.png)

- 볼륨의 `hostPath` 필드를 `awsElasticBlockStore` 필드로 대체한다.
- 볼륨 ID와 파일시스템 유형을 지정한다.
- 볼륨 스토리지는 이제 AWS EBS에 위치한다.

---

## Persistent Volume

### 볼륨의 한계와 영구 볼륨 (Persistent Volume)의 필요성

![22-persistent-volume-1.png](images%2F22-persistent-volume-1.png)

- 기존 볼륨의 설정: 파드 정의 파일 내에 볼륨 설정을 직접 구성했다.

![23-persistent-volume-2.png](images%2F23-persistent-volume-2.png)

- **대규모 환경의 문제점**
  - 많은 사용자가 다양한 파드를 배포하는 대규모 환경에서는 각 파드마다 스토리지 설정을 반복해야 한다.
  - 스토리지 솔루션 변경 시 모든 파드 정의 파일을 수정해야 한다.
  - 중앙 집중적인 스토리지 관리의 필요성이 대두된다.

![24-persistent-volume-3.png](images%2F24-persistent-volume-3.png)

- **Persistent Volume의 역할**
  - 관리자가 클러스터 전체에서 사용할 수 있는 스토리지 풀을 미리 구성한다.
  - 사용자는 영구 볼륨 클레임(Persistent Volume Claims, VPC)을 통해 필요한 스토리지를 할당받는다.
  - 중앙 집중적인 스토리지 관리 및 사용자 편의 성을 제공한다.

### 영구 볼륨 (Persistent Volume, PV) 생성

![25-persistent-volume-4.png](images%2F25-persistent-volume-4.png)

- **기본 템플릿 및 설정**
  - API 버전, 종류(PersistentVolume), 이름(pv-vol1), 용량, 액세스 모드, 스토리지 클래스 등을 설정한다.
  - `spec` 영역에서 세부 설정을 정의한다.
- **접근 모드(accessMode)**
  - 볼륨을 호스트에 마운트하는 방식(읽기 전용, 읽기/쓰기 등)을 정의한다.
  - `readOnlyMany`, `ReadWriteOnce`, `ReadWriteMany` 등의 값을 사용할 수 있다.
- **용량(capacity)**
  - 영구 볼륨에 할당할 스토리지 용량을 지정한다. (예: 1GB)
- **볼륨 유형(volumeType)**
  - 볼륨 스토리지의 유형을 설정한다.
  - `hostPath` 옵션을 사용하여 노드의 로컬 디렉토리를 사용할 수 있다. (개발/테스트 환경에서만 권장)
- **영구 볼륨 생성 및 조회**
  - `kubectl create` 명령어를 사용하여 영구 볼륨을 생성한다.
  - `kubectl get persistentvolume` 명령어를 사용하여 생성된 영구 볼륨을 조회한다.
- **스토리지 솔루션 변경**
  - `hostPath` 옵션을 지원되는 다른 스토리지 솔루션(AWS EBS 등)으로 변경하여 실제 스토리지 환경에 맞게 구성한다.

---

## Persistent Volume Claim

### 영구 볼륨 클레임 (Persistent Volume Claim, PVC)의 역할

![26-persistent-volume-claim-1.png](images%2F26-persistent-volume-claim-1.png)

- **영구 볼륨(PV)과의 관계**: 영구 볼륨과 영구 볼륨 클레임은 쿠버네티스 네임스페이스의 별도 객체다.
- **관리자와 사용자의 역할 분담**: 관리자는 영구 볼륨을 생성하고, 사용자는 영구 볼륨 클레임을 생성하여 스토리지를 사용한다.
- **영구 볼륨 바인딩**: 쿠버네티스는 클레임의 요청과 볼륨의 속성을 기반으로 영구 볼륨을 클레임에 바인딩한다.
- **1:1 관계**: 각 영구 볼륨 클레임은 하나의 영구 볼륨에 바인딩된다.

### 영구 볼륨 클레임 바인딩 과정

![27-binding-1.png](images%2F27-binding-1.png)

- **용량 확인**: 쿠버네티스는 클레임에서 요청한 용량과 일치하거나 충분한 용량을 가진 영구 볼륨을 찾는다.

![28-binding-2.png](images%2F28-binding-2.png)

- **속성 확인**: 접근 모드, 볼륨 모드, 스토리지 클래스 등 클레임의 다른 요청과 볼륨의 속성을 비교한다.

![29-binding-3.png](images%2F29-binding-3.png)

- **레이블 및 선택기**: 여러 후보 볼륨이 있는 경우 레이블과 선택기를 사용하여 특정 볼륨을 선택할 수 있다.
- **용량 초과 바인딩**: 다른 모든 조건이 충족되고 더 나은 옵션이 없는 경우, 더 작은 클레임이 더 큰 볼륨에 바인딩될 수 있다.

![30-binding-4.png](images%2F30-binding-4.png)

- **클레임 대기 상태**: 사용 가능한 볼륨이 없는 경우 클레임은 대기 상태로 유지되며, 새 볼륨이 생성되면 자동으로 바인딩된다.

### 영구 볼륨 클레임 생성

![31-persistent-volume-claim-2.png](images%2F31-persistent-volume-claim-2.png)

![32-persistent-volume-claim-3.png](images%2F32-persistent-volume-claim-3.png)

- **기존 템플릿 및 설정**
  - API 버전, 종류(PersistentVolumeClaim), 이름(My-Claim)을 설정한다.
  - `spec` 영역에서 세부 설정을 정의한다.
- **접근 모드(accessModes)**: 읽기/쓰기 한번(`ReadWriteOnce`)으로 설정한다.
- **리소스 요청(resources.requests)**: 500MB의 스토리지를 요청한다.

- **클레임 생성 및 조회**
  - `kubectl create` 명령어를 사용하여 영구 볼륨 클레임을 생성한다.
  - `kubectl get persistentvolumeclaim` 명령어를 사용하여 생성된 영구 볼륨 클레임을 조회한다.

![33-view-pvc.png](images%2F33-view-pvc.png)

- **바인딩 상태 확인**: `kubectl get pv` 명령어를 실행하여 클레임이 영구 볼륨에 바인딩되었는지 확인한다.

### 영구 볼륨 클레임 삭제 시 영구 볼륨 처리

![34-delete-pvc.png](images%2F34-delete-pvc.png)

- **삭제 명령어**: `kubectl delete persistentvolumeclaim` 명령어를 사용하여 클레임을 삭제한다.
- **삭제 시 영구 볼륨 처리 옵션**:
  - **유지(Retain)**: 기본 설정으로 관리자가 수동으로 삭제할 때까지 영구 볼륨이 유지되며 다른 클레임에서 재사용할 수 없다.
  - **삭제(Delete)**: 클레임 삭제 시 영구 볼륨도 자동으로 삭제되어 스토리지 장치의 공간을 확보한다.
  - **재활용(Recycle)**: 데이터 볼륨의 스크럽(삭제)하여 다른 클레임에서 사용할 수 있도록 한다.

### Using PVCs in Pods

- PVC를 생성한 후에는 다음과 같은 볼륨 섹션의 `persistentVolumeClaim` 섹션에서 PVC 이름을 지정하여 Pod 정의 파일에서 사용할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

- ReplicaSet 또는 Deployment도 동일하게 파드 템플릿 영역에 PVC를 지정하여 사용할 수 있다.

---

## Storage Class

![35-pv-pvc.png](images%2F35-pv-pvc.png)

### 정적 프로비저닝 (Static Provisioning)

![36-static-provisioning.png](images%2F36-static-provisioning.png)

- 이전에는 PV를 먼저 수동으로 생성하고 PVC를 통해 이를 클레임하는 방식이었다.
- GCP에서 영구 디스크를 사용하려면 먼저 GCP 콘솔에서 디스크를 생성해야 했다.
- 그 다음, 생선한 디스크와 동일한 이름을 사용하여 PV 정의 파일을 수동으로 작성해야 했다.
- 이런한 방식은 애플리케이션이 스토리지를 필요로 할 때마다 수동으로 디스크를 프로비저닝해야 하는 번거로움이 있다.

### 동적 프로비저닝 (Dynamic Provisioning)과 스토리지 클래스

![37-dynamic-provisioning-1.png](images%2F37-dynamic-provisioning-1.png)

- 스토리지 클래스를 사용하면 애플리케이션이 스토리지를 필요로 할 때 자동으로 볼륨을 프로비저닝할 수 있다.
- 스토리지 클래스는 프로비저너(Provisioner)를 정의하여 자동으로 스토리지 프로비저닝을 수행한다.
- GCP의 경우 `kubernetes.io/gce-pd` 프로비저너를 사용하여 GCP 영구 디스크를 자동으로 생성할 수 있다.

![38-dynamic-provisioning-2.png](images%2F38-dynamic-provisioning-2.png)

- PV 정의 파일은 더 이상 필요하지 않다. 스토리지 클래스가 PV와 실제 스토리지를 자동으로 생성하기 때문이다.
- PVC 정의 파일에서 `storageClassName` 필드를 지정하여 사용할 스토리지 클래스를 선택한다.
- PVC가 생성되면 스토리지 클래스는 정의된 프로비저너를 사용하여 GCP에 필요한 크기의 새 디스크를 프로비저닝하고, PV를 생성한 다음 PVC를 해당 볼륨에 바인딩한다.

### 다양한 프로비저너와 매개변수

![39-storage-class-1.png](images%2F39-storage-class-1.png)

- GCP 외에도 AWS EBS, Azure File, Azure Disk, CephFS, Portworx, ScaleIO 등 다양한 프로비저너를 사용할 수 있다.
- 각 프로비저너는 디스크 유형, 복제 모드 등 추가 매개변수를 설정할 수 있다.
- GCP 영구 디스크의 경우 `type` (standard, SSD) 및 `replication mode` (none, regional PD)를 지정할 수 있다.

### 스토리지 클래스의 활용

![40-storage-class-2.png](images%2F40-storage-class-2.png)

- 스토리지 클래스를 사용하여 다양한 서비스 등급(예: silver, gold, platinum)을 정의할 수 있다.
- 각 등급은 달느 유형의 디스크와 설정을 사용하여 성능과 안정성을 조절할 수 있다.
- PVC를 생성할 때 필요한 스토리지 클래스를 지정하여 원하는 서비스 등급의 볼륨을 사용할 수 있다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
- [Claim as Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)