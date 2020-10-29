# 보고서

1. [OCP란 무엇인가?](#1-ocp란-무엇인가)<br>
    1. [Red Hat OpenShift의 특징](#11-red-hat-openshift의-특징)<br>
2. [OCP 4.5 클러스터 아키텍처](#2-ocp-45-클러스터-아키텍처)<br>
    1. [Bootstrap](#21-bootstrap)<br>
    2. [Master](#22-master)<br>
    3. [Worker](#23-worker)<br>
    4. [Bastion](#24-bastion)<br>
    5. [HAProxy](#25-haproxy)<br>
    6. [DNS](#26-dns)<br>
    7. [PXE](#27-pxe)<br>
        1. [DHCP](#271-dhcp)<br>
        2. [TFTP](#272-tftp)<br>
3. [Helper Node를 이용한 Bare-metal에 클러스터 구축](#3-helper-node를-이용한-bare-metal에-클러스터-구축)<br>
    1. [Helper Node](#31-helper-node)<br>
    2. [구축 절차 오류 과정](#32-구축-절차-오류-과정)<br>
        1. [Bare-metal에 구축 불가](#321-bare-metal에-구축-불가)<br>
        2. [Wi-Fi 접속 불가](#322-wi-fi-접속-불가)<br>
        3. [DHCP 서버: 공유기 vs Helper](#323-dhcp-서버-공유기-vs-helper)<br>
        4. [Ubuntu 물리 네트워크 가상 Bridge 연결 오류](#324-ubuntu-물리-네트워크-가상-bridge-연결-오류)<br>
        5. [Helper DNS 설정 문제](#325-helper-dns-설정-문제)<br>
    3. [최종 구축 과정](#33-최종-구축-과정)<br>
        1. [물리머신 환경](#331-믈리머신-환경)<br>
        2. [가상머신 환경](#332-가상머신-환경)<br>
	3. [클러스터 아키텍처](#333-클러스터-아키텍처)<br>
	4. [Helper Node 구성](#334-helper-node-구성)<br>
	        1. [Playbook 설정&실행](#playbook-설정실행)<br>
	        2. [Ignition config 파일 생성](#ignition-config-파일-생성)<br>
	5. [Bootstrap/Master/Worker Node 구성](#335-bootstrapmasterworker-node-구성)<br>
	        1. [물리머신에 각 Node의 가상머신 준비](#물리머신에-각-node의-가상머신-준비)<br>
	        2. [가상머신 설치](#가상머신-설치)<br>
    4. [클러스터 구성 완료 후 작업](#34-클러스터-구성-완료-후-작업)<br>
        1. [Web console에 Login](#web-console에-login)<br>
	2. [기본적인 openshift setting](#기본적인-openshift-setting)<br>
4. [서비스 소개](#4-서비스-소개)<br>
    1. [서비스 아키텍처](#서비스-아키텍처)<br>
5. [서비스 구축과정](#5-서비스-구축과정)<br>
    1. [Jupyter Notebook](#51-jupyter-notebook)<br>
        1. [Importing Minimal Notebook](#511-importing-minimal-notebook)<br>
        2. [Making Minimal Notebook](#512-making-minimal-notebook)<br>
        3. [Deploying Minimal Notebook](#513-deploying-minimal-notebook)<br>
    2. [Metric 생성](#52-metric-생성)<br>
        1. [Kaggle 데이터를 이용한 데이터 분석](#521-kaggle-데이터를-이용한-데이터-분석)<br>
    3. [Prometheus](#53-prometheus)<br>
        1. [Prometheus metric library](#531-prometheus-metric-library)<br>
        2. [Prometheus metric extraction](#532-prometheus-metric-extraction)<br>
        3. [Install Library](#533-install-library)<br>
    4. [Preprocessing](#54-preprocessing)<br>
    5. [Metric to pdf](#55-metric-to-pdf)<br>
6. [서비스 시연 결과물](#6-서비스-시연-결과물)

# 1. OCP란 무엇인가?

Red Hat OpensShift Container Platform

하이브리드 클라우드로, 더 나은 애플리케이션을 더 빠르게 빌드하고 제공하기 위한 기업형 쿠버네티 플랫폼

![openshift architecture](https://github.com/FalcoN1995/project6/blob/master/images/1.png)

## 1.1 Red Hat OpenShift의 특징

- 컨테이너 호스트와 런타임
    - 컨테이너 호스트 운영체제로 Kubernetes의 Master용 RHCOS와 Worker용 RHEL를 포함
    - 컨테이너 엔진으로는 표준 Docker와 CRI-O 컨테이너 런타임을 지원(OCP 4.5는 CRi-O 컨테이너 엔진 사용)
- 통합된 컨테이너 레지스트리
    - 통합된 프라이빗 컨테이너 저장소를 제공(Kubernetes의 일부분으로 설치되거나 유연성을 높이기 위해 독단적으로 설치)
- 기업형 Kubernetes
    - Kubernetes의 모든 릴리즈에 대한 업스트립의 결함, 보안, 성능 문제에 대한 수백개의 수정을 포함
    - 수십개의 기술들로 테스트되고 9년의 생명주기에 걸쳐 지원되는 통합된 플랫폼
- 개발자 워크플로우
    - 젠킨스 파이프라인과 애플리케이션의 코드에서 컨테이너로 직행하는 S2I 기술을 기본으로 포함하여 더 빠른 개발을 돕는 간소화된 워크플로우를 제공
    - Istio와 Knative와 같은 새 프레임워크로 확장할 수 있다.
- 인증된 통합
    - 소프트웨어적으로 정의된 네트워크를 포함하고 추가적인 일반적 네트워크 솔루션을 인증
    - 모든 릴리즈에서 수많은 스토리지와 third-party 플러그인을 인증
- 쉬운 서비스 접근
    - 서비스 중개자(AWS 서비스의 직접적 접근 등), 검증된 third-party 솔루션, 내장된 OperatorHub를 통한 Kubernetes 오퍼레이터로 관리자를 돕고 애플리케이션 팀을 지원
- 신뢰할 수 있는 플랫폼
    - 애플리케이션의 라이프사이클 전반에 지속적인 보안검사를 컨테이너 스택에 구축
- 통합 생태계
    - 수백개의 파트너 사와 협력하여 오픈시프트의 기술 통합을 검증
- 내장된 모니터링
    - 네이티브 클라우드 클러스터와 애플리케이션 모니터링의 표준 Prometheus를 포함
    - 시각화 도구로 Grafana 대시보드 사용
- 중앙집중식 정책관리
    - 관리자에게 모든 오픈시프트 클러스터에 걸쳐 통합된 콘솔을 이용해 여러 팀에서 정책을 구현하고 수행 할 수 있는 하나의 공간을 제공
- 주문형 환경
    - 중앙집중식 관리와 제어로 승인된 서비스와 인프라구조를 제공, 사용자가 원하는 환경을 직접 구성하는 셀프 서비스

# 2. OCP 4.5 클러스터 아키텍처

## 2.1 Bootstrap

OCP는 Master 노드에 필요한 정보를 제공하기 위한 초기 설정동안 일시적인 Bootstrap 노드를 사용한다. 클러스터를 어떻게 생성할지 명세한 Ignition config 파일을 이용해 부팅을 시작한다. Bootstrap 노드가 ignition config 파일을 바탕으로 Master 노드를 구성하고, Master 노드가 Worker 노드를 생성한다. Bootstrap 머신은 OS로 Red Hat CoreOS(RHCOS)를 사용한다.

![bootstrap](https://github.com/FalcoN1995/project6/blob/master/images/2.1.png)

## 2.2 Master 

 Control plane 역할을 하는 모든 머신들이 Master 머신들이기 때문에, 용어를 설명하는데 'Master'와 'Control plane'은 같은 의미의 용어로 사용된다.

 Master 노드는 전체 클러스터 환경의 제어기능을 수행하는 서버이다. OpenShift와 관련된 모든 오브젝트의 생성 및 관리, 스케줄링을 담당한다. 클러스터를 관리하기 위한 Kubernetes 서비스들(API 서버, etcd, Controller Manager 서버 등)을 하나의 OpenShift 바이너리에 포함한다. OCP 4.5의 모든 Control plane 머신에는 운영체제로 반드시 RHCOS를 사용해야한다.

> 3대의 Master 노드를 사용한다. 비록 이론적으로 어떤 수의 master 노드를 사용 가능하더라도, master의 static 파드와 etcd static 파드가 같은 호스트에서 작동하기에 etcd 쿼럼에 의해 그 수가 제한된다.

## 2.3 Worker

 OCP 4.5 릴리스에서 Compute 머신의 유일한 기본 유형이 Worker 머신이기 때문에 'Worker 머신'과 'Compute 머신' 은 같은 의미로 사용된다.

 Worker 노드는 사용자에 의해 요청된 실제 워크로드가 동작하고 관리되는 곳이다. CRI-O(컨테이너 엔진), kubelet(컨테이너의 워크로드의 동작과 정지 요청을 수용하고 만족시키는 서비스), 그리고 kube-proxy(파드와 Worker 간의 의사소통을 관리)와 같은 중요한 서비스들이 각 Worker 노드에서 작동한다. Worker 역할의 머신들은 autoscale되는 명세된 머신 풀에 의해 관리되는 컴퓨팅 워크로드를 구동한다. Compute 머신에는 운영체제로 RHCOS 뿐만 아니라 Red Hat Enterprise Linux(RHEL)도 사용 할 수 있다.

## 2.4 Bastion

 OpenShift 클러스터의 주요 배포&관리 서버 역할을 수행하도록 배정된 노드이다. 클러스터의 관리자 시스템 배포와 기능 관리를 수행하기 위한 로그온 노드로 사용된다. OCP의 수동 또는 자동 배포를 위한 OpenShift 설치 파일이 Bastion 노드에서 동작한다. Bastion 노드는 리눅스 KVM 패키지가 설치된 RHEL 8.1에서 동작한다.

## 2.5 HAProxy

 OpenShift Load Balance 노드는 Keepalived와 HAProxy 라우터와 같은 로드밸런싱 서비스를 작동시킨다. HAProxy 라우터는 OpenShift 애플리케이션에 라우팅 기능을 제공한다. 최근에는 Server Name Indication (SNI)를 이용한 HTTP(S) 트래픽과 TLS-enabled 트래픽을 지원한다. OpenShift Load Balance 노드에 추가적인 애플리케이션과 서비스를 배포될 수 있다. OpenShift Load Balance 노드는 RHEL 8.1 서버를 실행한다.

## 2.6 DNS

OCP에서 DNS 서버의 역할은 중요하다. 일반적으로 IP 주소가 변경될 경우가 거의 없는 가상 머신과 달리, 컨테이너는 짧은 라이프사이클이 순환할 때마다 IP 주소가 변경되어 특정하기 어렵다. 따라서 클러스터 내부 객체 간 내부 통신에 IP 주소가 아닌 변하지 않는 호스트 이름으로 통신한다. 동적 DNS를 사용하기 때문에 서비스 개체가 재생성될 때마다 DNS가 새 레코드로 업데이트한다. 이 기능을 사용하여 각 서비스 개체 호스트 이름은 고유하고 동적인 서비스 IP 주소를 갖는다.

## 2.7 PXE

< Preboot Execution Environment ><br>
로컬 드라이브에서 부팅하는 대신 네트워크에서 노드를 부팅하는 방법이다.PXE 네트워크 부팅은 클라이언트-서버 프로토콜을 사용하여 수행된다.

### 2.7.1 DHCP

< Dynamic Host Configuration Protocol > <br>
DHCP 서버는 클라이언트에 IP 네트워크 구성을 제공한다. 명세된 IP 대역 내 pool 범위에서 IP 주소를 클라이언트에 임대해준다.
PXE의 경우 부팅 파일을 다운로드할 서버의 IP 주소를 포함하는 옵션.

클라이언트는 네트워크 구성을 요청하는 브로드 캐스트 형식으로 'discover'패킷을 보내고, DHCP 서버가 이 패킷을 수신받는다.
DHCP 서버에서 클라이언트로 'offer'패킷이 전송된다. 'offer'를 분석 한 후 클라이언트에는 IP 주소, 서브넷 마스크 등과 같은 네트워크 매개 변수가 할당된다.

### 2.7.2 TFTP

< Trivial File Transfer Protocol > <br>
TFTP는 인증이나 권한 부여가 없는 FTP(File Transfer Protocol)의 기본 경량 버전이다. 파일을 가져오거나 보내기 위한 간단한 UDP 기반 프로토콜로, 커널과 같이 부팅에 필요한 파일을 불러오는 역할을 한다. 파일을 불러오기 위한 서버로 FTP, HTTP, NFS도 사용 할 수 있다.

# 3. Helper Node를 이용한 Bare-Metal에 클러스터 구축

> 물리머신 환경
> - Ubuntu 18.04
> - 500 GB disk
> - 8 vCPUs
> - 20 GB of RAM
> - 192.168.20.0/24 (wifi)

## 3.1 Helper Node

OpenShift 4 클러스터를 구축하기위해 필요한 모든 서비스가 함축된 "All in One" 노드. 플레이북 하나로 간단하게 구축

![helper node architecture](https://github.com/FalcoN1995/project6/blob/master/images/hn.png)

> 최소 요구사항
> - CentOS/RHEL 7 or 8
> - 50 GB disk
> - 4 vCPUs
> - 8 GB of RAM

## 3.2 구축 절차 오류 과정

### 3.2.1 Bare-metal에 구축 불가

Bare-metal에 노드를 바로 구축하고 머신을 같은 대역의 유선 네트워크로 연결해 클러스터를 구성하는 방법을 구상했지만, 대여한 노트북이 UEFI 부팅만 가능하기에 실패했다. OCP를 설치하기 위한 RHCOS가 legacy 방식인 BIOS 부팅 방식만 지원하기 때문에 구축이 아예 불가능했다. 따라서 Helper 노드를 제외한 클러스터 구성 노드는 호스트에 OS를 설치하고 가상머신으로 생성한 bare-metal에 역클러스터를 구축하는 방법으로 변경하였다.

> BIOS vs UEFI

a.  BIOS(Basic Input/Output System)

- 운영체제 중 가장 기본적인 컴퓨터의 입출력을 처리하는 소프트웨어
- 하드웨어 부품을 초기화하고 검사하는 역할
- 부트로더 또는 대용량 저장장치에 저장된 OS를 RAM으로 읽어오는 기능 수행
- 현재 UEFI 방식이 나오고나서 Legacy 방식으로 전락

b. UEFI (Unified Extensible Firmware Interface)

- OS와 플랫폼 펌웨어 사이의 소프트웨어 인터페이스를 정의하는 규격
- 인텔 사의 EFI(Extensible Firmware Interface) 규격 기반

![https://lh4.googleusercontent.com/dIRWaMswFEjRYAriD3QaVIaF60cf2DAAmBxuhIqzIhZaHKiOHBdNlQ5IzWyph6c9xOkjUMXT-V4dt42gc-XKkbES_diEXrU2hYUjBbdsqxTnxLX88D0Gjp-1puQTTb12tCo-YbsC](https://lh4.googleusercontent.com/dIRWaMswFEjRYAriD3QaVIaF60cf2DAAmBxuhIqzIhZaHKiOHBdNlQ5IzWyph6c9xOkjUMXT-V4dt42gc-XKkbES_diEXrU2hYUjBbdsqxTnxLX88D0Gjp-1puQTTb12tCo-YbsC)

- EFI는 boot 서비스와 runtime 서비스를 지원
- boot 서비스

펌웨어가 플랫폼을 소유하는 동안에만 이용 가능 (ExitBootServices 호출 이전)

다양한 장치, 버스, 블록, 파일 서비스 상의 텍스트와 그래픽 콘솔을 포함

- runtime 서비스

OS가 실행 중인 동안에 접근 가능

날짜, 시간, NVRAM 접근 서비스

### 3.2.2 Wi-Fi 접속 불가

Host OS를 올린 이후 노트북에 내장된 무선 네트워크 인터페이스와 드라이버가 맞지 않아 Wi-Fi에 접속 할 수 없었다. 따라서 호환되는 드라이버를 직접 설치해야 했다.

Helper 머신의 운영체제로 CentOS 7 또는 8을 사용하기에 CentOS에 드라이버 설치를 시도했다. 하지만 대여한 노트북의 NIC에 맞는 드라이버가 CentOS를 지원하지 않았다. 따라서 내부적인 코드를 변경, 드라이버 설치는 성공했으나 Wi-Fi 드라이버가 반복적으로 죽어 Wi-Fi에 접속할 수 없었다. 대여 노트북의 NIC와 맞는 드라이버가 Ubuntu에서는 정상적으로 작동하였다. 결국 다른 Node들과 같이 helper Node도 Host OS(Unbuntu) 위에 kvm을 이용해 가상머신으로 구축하기로 변경하였다.

#### OS 별 드라이버 설치 방법
> CentOS 8

```docker
sudo apt install -y bc module-assistant build-essential dkms git 
sudo m-a prepare 
git clone https://github.com/tomaspinho/rtl8821ce cd rtl8821ce
sudo vi os.intfs.c
# modify this block 
# to match the fields in your kernel headers.
#if (LINUX_VERSION_CODE >= KERNEL_VERSION(4,19,0))
static u16 rtw_select_queue(struct net_device *dev, struct sk_buff *skb
    , struct net_device *sb_dev
    #if (LINUX_VERSION_CODE < KERNEL_VERSION(5,2,0))
    , select_queue_fallback_t fallback
    #endif
)
#else
static u16 rtw_select_queue(struct net_device *dev, struct sk_buff *skb
#if LINUX_VERSION_CODE >= KERNEL_VERSION(3, 13, 0)
	, void *accel_priv
	#if LINUX_VERSION_CODE >= KERNEL_VERSION(3, 14, 0)
	, select_queue_fallback_t fallback
	#endif
#endif
vi rtw.android.c
# remove the VERIFY_READ (first argu)
	if (!access_ok(priv_cmd.buf, priv_cmd.total_len)) {
#endif
		RTW_INFO("%s: failed to access memory\n", __FUNCTION__);
		ret = -EFAULT;
		goto exit;
	}
sudo ./dkms-install.sh 
lsmod | grep 8821
```

> Ubuntu OS

```docker
sudo apt install -y bc module-assistant build-essential dkms git 
sudo m-a prepare 
git clone https://github.com/tomaspinho/rtl8821ce cd rtl8821ce
sudo ./dkms-install.sh 
lsmod | grep 8821
```

### 3.2.3 DHCP 서버: 공유기 vs Helper

처음 구상은 모든 공유기의 dhcp 기능을 끄고 Helper Node에 dhcp 서버를 구축하는 방법이었다. 그러나 코로나 사태로 인해 강의 장소가 원경에서 강의장으로 변경되면서, 클러스터 구성 환경이 팀원의 개인집에서 강의장으로 달라졌다. 강의장의 공유기 설정을 변경 할 수 없었기에 다른 방법을 찾아야 했다. 차안으로 공유기가 DHCP 서버 역할을 수행하도록 설정, (Helper Node에 DHCP 서버 동시 구축 시 충돌 또는 인식이 불가), 노드에 정적으로 IP를 넣어 구축하는 방식을 선택했다. 하지만 이 방식은 BIOS 부팅 방식만 가능했고, 앞서 말했듯이 BIOS 부팅이 불가능한 환경이었기에 역시 다른 대책이 필요했다. 가상머신으로 클러스터 구축을 하면서 해결했다.

### 3.2.4 Ubuntu 물리 네트워크 가상 Bridge 연결 오류

다른 호스트의 가상머신끼리 통신하기 위해 물리 네트워크를 가상 브릿지로 연결해 가상머신에 연결해야 했다. Ubuntu 18.04의 네트워크 설정 netplan과 NetworkManager 두 방법이 충돌, 물리 네트워크를 가상 브릿지로 연결해 가상머신에 연결 할 수 없었다.

KVM으로 가상머신 생성 설정에서 물리 네트워크를 bridge로 연결하는 설정을 이용해 해결하였다.

> netplan yaml파일로 네트워크 설정

```docker
sudo apt-get install net-tools

# check the Physical Interface card 
ifconfig

cat /etc/netplan/01-network-manager-all.yaml
network:
  version: 2 
  renderer: networkd
  ethernets:
    enx...: 
      dhcp4: false
  bridges: 
    br0: 
      dhcp4: true
      interfaces: [enx...]

# apply the changed yaml file
sudo netplan apply 
cat /etc/br0.xml
<network>
 		<name>br0</name>
 		<forward mode='bridge'/>
 		<bridge name='br0'/> 
</network> 

# apply xml file 
sudo virsh net-define /etc/br0.xml 
sudo virsh net-start br0 
sudo virsh net-autostart br0 

# check the virsh net list 
sudo virsh net-list -all
```

### 3.2.5 Helper DNS 설정 문제

Helper의 웹 서버에 Ignition config 파일과 구축에 필요한 파일들(커널 등)이 저장되어 있다. Helper에 dns 주소를 8.8.8.8과 같이 외부 주소로 설정했을 때 PXE 부팅 과정에서 Node 구축에 필요한 설정과 파일을 읽어오지 못하는 문제가 발생했다. Node들은 부팅 과정에서 Helper를 바라보고 dns 주소를 따라 구성을 시도하기 때문이었다. 따라서 Helper의 DNS 주소를 Helper의 IP 주소로 설정하여 해결하였다.

## 3.3 최종 구축 과정&결과

### 3.3.1 물리머신 환경

- Ubuntu 18.04
- virt-manager 설치

    ```docker
    sudo apt update 
    sudo apt install -y qemu-kvm libvirt-bin bridge-utils virtinst virt-manager 
    sudo systemctl enable libvirtd 
    reboot
    ```

- 무선 네트워크 드라이버 설치

    ```docker
    sudo apt install -y bc module-assistant build-essential dkms git
    sudo m-a prepare 
    git clone https://github.com/tomaspinho/rtl8821ce 
    cd rtl8821ce 
    sudo ./dkms-install.sh 
    lsmod | grep 8821
    ```

- 가상머신 원격접속을 위한 openssh 설치

### 3.3.2 가상머신 환경

<최소 요구사항>
|    Node   |     Operating SYS    | vCPUs |   RAM  | Disk Storage |
|:---------:|:--------------------:|:-----:|:------:|:------------:|
| Helper    | CentOS/RHEL 7 or 8   | 4     |   8 GB | 50 GB        |
| Bootstrap | RHCOS                | 4     | 16 GB  | 120 GB       |
| Master    | RHCOS                | 4     | 16 GB  | 120 GB       |
| Worker    | RHCOS or RHEL 7 or 8 | 2     |   8 GB | 120 GB       |

<IP 주소 설정>
|   Node   |   IP address    |
|:---------|:----------------|
|Helper    |192.168.20.111   |
|Bootstrap |192.168.20.150   |
|Master0   |192.168.20.151   |
|Master1   |192.168.20.152   |
|Master2   |192.168.20.153   |
|Worker0   |192.168.20.161   |
|Worker1   |192.168.20.162   |

### 3.3.3 클러스터 아키텍처

![cluster architecture](https://github.com/FalcoN1995/project6/blob/master/images/3.3.3.png)

### 3.3.4 Helper Node 구성

Helper에 dns를 패키지를 설치하는 등 외부접속이 필요한 때는 공인 IP를 넣어주고 이후에는 자신의 IP 주소를 넣어준다. Node가 부팅 시 dns를 따라가 파일을 불러오기 때문

#### Playbook 설정&실행

1. 필요 패키지 설치

    ```docker
    # EPEL 설치
    yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm

    # ansible과 git 설치
    yum -y install ansible git

    # 플레이북 git에서 가져오기
    git clone https://github.com/RedHatOfficial/ocp4-helpernode
    cd ocp4-helpernode
    ```

2. vars.yaml 파일을 환경에 맞게 수정

    ```docker
    cp docs/examples/vars-static.yaml .
    vars-static.yaml
    ---
    staticips: true
    helper:
      name: "helpnode"
      ipaddr: "192.168.20.111"
    dns:
      domain: "example.com"
      clusterid: "ocp4"
      forwarder1: "8.8.8.8"
      forwarder2: "8.8.4.4"
    bootstrap:
      name: "bootstrap"
      ipaddr: "192.168.20.150"
    masters:
      - name: "master0"
        ipaddr: "192.168.20.151"
      - name: "master1"
        ipaddr: "192.168.20.152"
      - name: "master2"
        ipaddr: "192.168.20.153"
    workers: 
      - name: "worker0"
        ipaddr: "192.168.20.161"
      - name: "worker1"
        ipaddr: "192.168.20.162"
    ...
    ```

3. playbook 실행

    ```docker
    ansible-playbook -e @vars-static.yaml -e staticips=true tasks/main.yml
    ```

플레이북 실행 후 Helper Node의 리소스가 제대로 생성되었는지 확인하는 helpernodecheck 명령어
/usr/local/bin/helpernodecheck 로 확인

#### Ignition config 파일 생성

1. 작업 디렉토리 생성&이동

    ```docker
    mkdir ~/ocp4
    cd ~/ocp4
    ```

2. 시크릿 파일 생성

    ```docker
    mkdir -p ~/.openshift
    cat <<EOF > ~/.openshift/pull-secret
    # https://cloud.redhat.com/openshift/install/metal에서 secret 복사&붙여넣기
    EOF
    ```

    이 플레이북은 ~/.ssh/helper_rsa 에 sshkey를 생성한다. 다른 키를 사용하고 싶으면 ~/.ssh/config 를 수정한다.

3. install-config.yaml 파일 생성

    ```docker
    cat <<EOF > install-config.yaml
    apiVersion: v1
    baseDomain: example.com
    compute:
    - hyperthreading: Enabled
      name: worker
      replicas: 0
    controlPlane:
      hyperthreading: Enabled
      name: master
      replicas: 3
    metadata:
      name: ocp4
    networking:
      clusterNetworks:
      - cidr: 10.254.0.0/16
        hostPrefix: 24
      networkType: OpenShiftSDN
      serviceNetwork:
      - 172.30.0.0/16
    platform:
      none: {}
    pullSecret: '$(< ~/.openshift/pull-secret)'
    sshKey: '$(< ~/.ssh/helper_rsa.pub)'
    EOF
    ```

4. installation manifest 생성&수정

    ```docker
    openshift-install create manifests
    ```

    master 노드에 파드 스케줄링을 막기 위해 mastersSchedulable 의 값 수정
    master에 파드를 배치하려면 파일 수정은 건너뛴다.

    ```docker
    # 파일 수정
    sed -i 's/mastersSchedulable: true/mastersSchedulable: false/g' manifests/cluster-scheduler-02-config.yml

    cat manifests/cluster-scheduler-02-config.yml
    apiVersion: config.openshift.io/v1
    kind: Scheduler
    metadata:
      creationTimestamp: null
      name: cluster
    spec:
      mastersSchedulable: false    # 이 부분 확인
      policy:
        name: ""
    status: {}
    ```

5. ignition config 생성

    ```docker
    openshift-install create ignition-configs

    #가상머신의 네트워크 부팅에 쓰이는 8080포트의 경로로 ignition 파일을 복사
    cp ~/ocp4/.ign /var/www/html/ignition/

    #selinux context 복구 & 권한 추가
    restorecon -vR /var/www/html/
    chmod o+r /var/www/html/ignition/.ign
    ```

### 3.3.5 Bootstrap/Master/Worker Node 구성

#### 물리머신에 각 Node의 가상머신 준비

- 모든 물리&가상머신은 동일한 무선 네트워크 대역(192.168.20.0) 사용
- 물리머신의 무선 네트워크를 가상머신의 bridge로 연결 (NIC : virtio)
- IP주소를 정적으로 설정
- Bootstrap -> Master -> Worker 순으로 가상머신 Set Up

#### 가상머신 설치

RHCOS ISO Installer을 사용하여 인스턴스를 부팅한다.

![installation screen](https://github.com/FalcoN1995/project6/blob/master/images/installation.png)

1. booting이 시작되면 boot menu에서 tab을 누른다.

![boot screen](https://github.com/FalcoN1995/project6/blob/master/images/3.3.5.png)

2. 각 Node에 맞는 정적ip와 coreOS 설정을 한 줄로 입력한다. (각 필드는 space로 구분)

    Bootstrap 입력 예시

    ```docker
    ip=192.168.20.150::192.168.20.1:255.255.255.0:bootstrap.ocp4.example.com:ens3:none
    nameserver=192.168.20.111
    coreos.inst.install_dev=sda
    coreos.inst.image_url=http://192.168.20.111:8080/install/bios.raw.gz
    coreos.inst.ignition_url=http://192.168.20.111:8080/ignition/bootstrap.ign
    ```

    형식

    ```docker
    영구적용되는 정적 IP 설정
    [ip=<ipaddr>::<defaultgw>:<netmask>:<hostname>:<iface>:none] → 인터페이스 ens3
    DNS 서버 설정: 여러번 입력 가능
    [nameserver=<dnsserver>] → Helper의 IP주소
    ```

    Bootstrap -> Masters -> Worker 순으로 입력 후 가동

3. helper node 에서 다음 명령어로 설치 시작

    ```docker
    openshift-install wait-for bootstrap-complete --log-level debug
    ```

웹 브라우저에서 Helper의 HAProxy(9000번 포트)를 통해 Node의 상태를 볼 수 있다 http://192.168.20.111:9000

Master Node가 모두 올라오면 Bootstrap Node를 삭제해도 된다.

## 3.4 클러스터 구성 완료 후 작업

설치가 끝난 후 웹 콘솔에 접속하기까지의 과정이다.

1. Login to cluster

    ```docker
    export KUBECONFIG=/root/ocp4/auth/kubeconfig
    ```

2. worker node의 승인을 위해 CSR 확인

    ```docker
    oc get csr
    ```

3. pending 되어 있는 모든 CSR을 한번에 Approved 한다. (각 node마다 csr 두 쌍씩 존재)

    ```docker
    oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
    ```

4. Registry를 설정하려면 클러스터 managementState를 Managed로 설정해야한다.

    ```docker
    oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
    ```

5. emptyDIr 사용하기

    ```docker
    oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
    ```

6. 설치 프로세스를 완료하기

    ```docker
    openshift-install wait-for install-complete
    ```

> oc 명령어 자동완성
```docker
# Bash completion 코드를 파일에 저장
oc completion bash > oc_bash_completion
# 파일을 /etc/bash_completion.d/.로 복사
sudo cp oc_bash_completion /etc/bash_completion.d/
# 셸을 새로 연다
bash
```

#### Web console에 Login

Web console은 Helper node에서만 login이 가능하다. 따라서 helpernode에 그래픽 인터페이스인 GNOME을 설치한다.

```docker
yum grouplist -v
sudo yum groupinstall -y "GNOME Desktop"
```

#### 기본적인 openshift setting

> OCP의 RBAC

![service account](https://github.com/FalcoN1995/project6/blob/master/images/3.5.1.png)

1. project 만들기
2. user 만들기

    ```docker
    mkdir /etc/htpasswd
    htpasswd -c -B -b /etc/htpasswd/user.htpasswd inchoi password
    #-c 옵션은 처음에만 넣는다. (create)
    htpasswd --B -b /etc/htpasswd/user.htpasswd user1 user1
    htpasswd  -B -b /etc/htpasswd/user.htpasswd user2 user2
    htpasswd  -B -b /etc/htpasswd/user.htpasswd user3 user3
    htpasswd  -B -b /etc/htpasswd/user.htpasswd user4 user4
    ```

3. secret 생성

    ```docker
    oc create secret generic htpass-secret --from-file=htpasswd=/etc/htpasswd/user.htpasswd -n openshift-config
    ```

4. HTPasswdCR yaml 파일 생성

    ```docker
    mkdir -p yaml/cr
    vi yaml/cr/htpass_cr.yml

    apiVersion: config.openshift.io/v1
    kind: OAuth
    metadata:
      name: cluster
    spec:
      identityProviders:
      - name: HTPass
        mappingMethod: claim 
        type: HTPasswd
        htpasswd:
          fileData:
            name: htpass-secret
    ```

5. 정의된 CR을 적용한다

    ```docker
    oc apply -f /yaml/cr/htpass_cr.yml
    ```

6. 사용자로 log in

    ```docker
    oc login -u user1
    ```

7. 사용자가 성공적으로 로그인했는지 확인, 사용자 이름 확인

    ```docker
    oc whoami
    ```

# 4. Service Introduction

Openshift의 Prometheus의 metirc값으로 PaaS의 사용량을 측정하고 이를 토대로 비용을 계산, 자동으로 pdf 고지서를 만드는 서비스를 개발

## 서비스 아키텍처

![service architecture](https://github.com/FalcoN1995/project6/blob/master/images/4.0.png)


# 5. 서비스 구축 과정

## 5.1 Jupyter Notebook

Source-to-Image 빌드 프로세스를 사용하여 minimal Jupyter notebook 이미지를 만든다.

### 5.1.1 Importing Minimal Notebook

CentOS를 기반으로하는 미리 빌드 된 minimal notebook 버전은 quay.io의 다음 위치에서 찾을 수 있다.

[https://quay.io/organization/jupyteronopenshift](https://quay.io/organization/jupyteronopenshift)

테스트하지 않은 이미지의 최신 버전으로 변경되지 않게 하기 위하여 위의 이미지를 사용하여 제공된 이미지 스트림 정의를 사용하여 로드한다.

```docker
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/image-streams/s2i-minimal-notebook.json
```

### 5.1.2 Making Mnimal Notebook

1. Openshift 클러스터의 소스 코드에서 minimal notebook 이미지를 빌드한다.

    ```docker
    oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/build-configs/s2i-minimal-notebook.json
    ```

2. 빌드 진행 상황을 확인한다.

    ```docker
    oc logs --follow bc/s2i-minimal-notebook-py36
    ```

### 5.1.3 Deploying Minimal Notebook

1. minimal notebook image 배포

    ```docker
    oc new-app s2i-minimal-notebook:3.6 --name minimal-notebook \
        --env JUPYTER_NOTEBOOK_PASSWORD=mypassword
    ```

2. 배포 진행률을 확인한다.

    ```docker
    oc rollout status dc/minimal-notebook
    ```

3. 노트북 인스턴스는 기본적으로 public 네트워크에 노출되지 않으므로 노출시킨다.

    ```docker
    oc create route edge minimal-notebook --service minimal-notebook \
        --insecure-policy Redirect
    ```

4. 노트북 인스턴스에 지정된 호스트 이름을 확인한다

    ```docker
    oc get route/minimal-notebook
    ```

5. 브라우저를 사용하여 호스트 이름에 엑세스하고 위에서 사용한 비밀번호를 입력한다.

## 5.2 Metric 생성

### 5.2.1 Kaggle 데이터를 이용한 데이터 분석

1. Titanic Data
    1. Create the Jupyter container in OpenShift 4
    2. Cteate the Virtual ENV

        ```docker
        pip install virtualenv
        virtualenv metricCreator
        cd metricCreator
        source bin/activate
        ```

    3. Install the Kaggle API

        ```docker
        pip install kaggle --upgrade
        ```

    4. Input the Kaggle ID auth Token
        1. Open the kaggle site([www.kaggle.com](http://www.kaggle.com/))
        2. Login
        3. My Account Tab
        4. API -> Create the New API Token
        5. Input the Token File in Jupyter Node (in .kaggle dir)

        ```docker
        cat /opt/app-root/src/.kaggle/kaggle.json
        {"username":"yourUserName","key":"KeyValue"}
        ```

    5. Install the Titanic Data

        ```docker
        kaggle competitions download -c titanic
        ```

    6. Data preprocessing & Analysis (using ML)

## 5.3 Prometheus

### 5.3.1 Prometheus metric library

서비스를 모니터링하려면 먼저 Prometheus 클라이언트 라이브러리 중 하나를 통해 코드에 계측을 추가해야한다. 이들은 Prometheus 측정을 구현한다 .

Python Prometheus 클라이언트 라이브러리를 사용하여 애플리케이션 인스턴스의 HTTP 엔드 포인트를 통해 내부 측정 항목을 정의하고 노출 할 수 있다.

### 5.3.2 Prometheus metric extraction

> Openshift Oauth

유저가 ocp 클러스터에 접근하려면 OAuth서버를 통해 인증을 받아야 한다.

API나 웹 콘솔을 통해 OAuth서버의 인증을 요청할 수 있고, OAuth토큰을 받게 된다.

그리고 그 토큰은 유저의 API Request에 포함되어 해당 사용자의 자격증명을 가능하게 해준다.

보통 한번 이 토큰을 받게되면 24시간동안 유지되고 그 전에 로그아웃 하더라도 만료된다. 그 이후에는 다시 토큰을 받아야 한다.

프로메테우스에 접근하기 위한 권한을 확인하기 위해 Oauth token이 필요하다.

> Oauth Token 확인

```
oc get --user=user1 oauthaccesstokens
```

해당 사용자의 oauth token을 확인한다

Oauth Token을 활용하여 prometheus에 접근

### 5.3.3 Install Library

1. Jupyter notebook에서 필요한 library를 설치한다.

    ```python
    !pip install prometheus-api-clinet
    !pip install prometheus-client
    ```

2. prometheus metric을 추출하기 위한 library 사용

    ```python
    from prometheus_api_client import Metric, MetricsList, PrometheusConnect
    from prometheus_api_client.utils import parse_datetime
    from datetime import timedelta, datetime

    import pandas as pd
    import os
    ```

3. prometheus URL 생성

    ```python
    prom_url = "http://prometheus-k8s-openshift-monitoring.apps.ocp4.example.com/"
    print("Prometheus url: ", prom_url)
    ```

    Prometheus url: [http://prometheus-k8s-openshift-monitoring.apps.ocp4.example.com/](http://prometheus-k8s-openshift-monitoring.apps.ocp4.example.com/)

4. 사용자의 token 저장 스크립트

    ```python
    os.system("oc login -u user1 -p user1")
    token = os.popen("oc whoami -t")
    r_token = token.read()
    print(r_token.rstrip())
    pc = PrometheusConnect(url=prom_url, headers={"Authorization": "bearer " + r_token.rstrip()}, disable_ssl=True)
    ```

    SSL 인증은 하지 않는다.

5. 모든 metric list

    ```python
    pc.all_metrics()
    
    [':kube_pod_info_node_count:',
   ':node_memory_MemAvailable_bytes:sum',
   'ALERTS',
   'ALERTS_FOR_STATE',
   'aggregator_openapi_v2_regeneration_count',
   'aggregator_openapi_v2_regeneration_duration',
   'aggregator_unavailable_apiservice',
   ... 
   'node_memory_HugePages_Surp',
   'node_memory_HugePages_Total',
   'node_memory_Hugepagesize_bytes',
   'node_memory_Hugetlb_bytes',
   'node_memory_Inactive_anon_bytes',
 ...]
    ```

6. 쿼리문 사용

    ```python
    pc.custom_query(query="container_cpu_usage_seconds_total")
    ```

## 5.4 Preprocessing

dataframe을 정제하는 전처리 과정

1. 추출 데이터의 시계열 지정
    ```python
    start_time = parse_datetime("3d")
    end_time = parse_datetime("now")
    chunk_size = timedelta(days=3)
    ```

2. memory 사용량 metric 추출
    ```python
    mem_metric_data = pc.get_metric_range_data(
        "container_memory_rss{namespace='default'}",  # this is the metric name and label config
        start_time=start_time,
        end_time=end_time,
        chunk_size=chunk_size,
    )

    mem_metrics_object_list = MetricsList(mem_metric_data)
    
    mem_metric_object = mem_metrics_object_list[0] # one of the metrics from the list

    mem_metric_object_chunk_list = []
    
    for raw_metric in mem_metric_data:
        mem_metric_object_chunk_list.append(Metric(raw_metric))
    ```

3. cpu 사용량 metric 추출
    ```python
    cpu_metric_data = pc.get_metric_range_data(
        "namespace:container_cpu_usage:sum{namespace='default'}",  # this is the metric name and label config
        start_time=start_time,
        end_time=end_time,
        chunk_size=chunk_size,
    )
    
    cpu_metrics_object_list = MetricsList(cpu_metric_data)
    
    cpu_metric_object = cpu_metrics_object_list[0] # one of the metrics from the list

    cpu_metric_object_chunk_list = []
    
    for raw_metric in cpu_metric_data:
        cpu_metric_object_chunk_list.append(Metric(raw_metric))
    ```

4. network 사용량 metric 추출
    ```python
    network_metric_data = pc.get_metric_range_data(
        "container_network_transmit_bytes_total{namespace='default'}",  # this is the metric name and label config
        start_time=start_time,
        end_time=end_time,
        chunk_size=chunk_size,
    )
    
    network_metrics_object_list = MetricsList(network_metric_data)
    
    network_metric_object = network_metrics_object_list[0] # one of the metrics from the list

    network_metric_object_chunk_list = []
    for raw_metric in network_metric_data:
        network_metric_object_chunk_list.append(Metric(raw_metric))
    ```
5. dataframe 생성
    ```python
    ini_df = pd.DataFrame(data = cpu_metric_object.metric_values)
    ini_df.rename(columns = {"y": "cpu"}, inplace=True)
    ini_df['mem'] = mem_metric_object.metric_values.y
    ini_df['network'] = network_metric_object.metric_values.y
    ini_df
    ```
    ![dataframe1](https://github.com/FalcoN1995/project6/blob/master/images/dataframe1.png)

6. 값이 없는 부분 제거
    ```python
    sec_df = ini_df[['ds', 'cpu', 'mem', 'network']].dropna(axis=0)
    sec_df
    ```
    ![dataframe2](https://github.com/FalcoN1995/project6/blob/master/images/dataframe2.png)

7. 시계열을 주 단위로 정의
    ```python
    def get_date(y, m, d):
        s = f'{y:04d}-{m:02d}-{d:02d}'
        return datetime.strptime(s, '%Y-%m-%d')

    def get_week_no(df):
        y = int(df['ds'].year)
        m = int(df['ds'].month)
        d = int(df['ds'].day)
        target = get_date(y,m,d)
        firstday = target.replace(day=1)
        if firstday.weekday() == 6:
            origin = firstday
        elif firstday.weekday() < 3:
            origin = firstday - timedelta(days=firstday.weekday() + 1)
        else:
            origin = firstday + timedelta(days=6-firstday.weekday())
        return (target - origin).days // 7 + 1

    sec_df['weekly'] = sec_df.apply(get_week_no, axis=1)
    sec_df
    ```
    
    ![dataframe3](https://github.com/FalcoN1995/project6/blob/master/images/dataframe3.png)

8. ???
    ```python
    cpu_first_data = sec['cpu'][-1:] - ini_df['cpu'][2875]
    cpu_first_data
    ```
    
9. 시계열을 주로 묶어 출력
    ```python
    table_weekly = pd.pivot_table(ini_df, values = ['cpu', 'mem', 'network'], index=['weekly'], aggfunc = np.mean)
    table_weekly
    ```
    ![dataframe4](https://github.com/FalcoN1995/project6/blob/master/images/dataframe4.png)
    
10. 모자란 한 달간의 데이터 임의 입력
    ```python
    fake_metric_data = {
        'weekly': [1,2,3,4],
        'cpu': [0.124,0.123, 0.17, 0.19],
        'mem': [234213142,5552423, 7513246, 8646534],
        'network': [312432.2134, 123111.8321, 123332.8321, 123556.8321]
    }
    fake_metric_df = fake_metric_df.set_index('weekly')
    
    gen_weekly_data = table_weekly.append(fake_metric_df)
    gen_weekly_data = gen_weekly_data.sort_index()
    gen_weekly_data
    ```
    ![dataframe5](https://github.com/FalcoN1995/project6/blob/master/images/dataframe5.png)
    

## 5.5 Metric to pdf

0. Prototype
![prototype](https://github.com/FalcoN1995/project6/blob/master/images/prototype.png)

1. 빈 PDF 생성
    ```python
    from reportlab.platypus import SimpleDocTemplate
    from reportlab.lib.pagesizes import letter
    
    fileName = 'pdfTable.pdf'

    pdf = SimpleDocTemplate(fileName,pagesize=letter)
    
    from reportlab.graphics.charts.piecharts import (
	Pie
    )
    from reportlab.graphics.charts.legends import (
	Legend
    )
    from reportlab.lib.validators import Auto
    ```
    
2. 원 그래프 PDF에 그리기
    ```python
    def getPieChart(data1, data2, data3):
	data = [data1, data2, data3]
	chart = Pie()
	chart.data = data
	chart.x = 50
	chart.y = 5

	chart.labels = ['cpu_cost','mem_cost','net_cost']

	chart.sideLabels = True

	chart.slices[0].fillColor = colors.red
	#chart.slices[0].popout = 8

	title = String(
		50, 110, 
		'Cost ratio', 
		fontSize = 12
	)	

	legend = Legend()
	legend.x = 220
	legend.y = 80
	legend.alignment = 'right' 	

	legend.colorNamePairs = Auto(obj=chart)

	drawing = Drawing(0, 500)
	drawing.add(title)
	drawing.add(chart)
	drawing.add(legend)

	return drawing
    ```

3. 선 그래프 PDF에 그리기
   ```python
   from reportlab.graphics.charts.linecharts import(
	HorizontalLineChart
    )

    def getLineChart(title, data, data_max):
	chart = HorizontalLineChart()
	chart.data = data
	chart.x = 5
	chart.y = 5
	chart.height = 100
	chart.width = 150

	chart.categoryAxis.categoryNames = [
		'week_1', 'week_2', 'week_3', 'week_4'
	]

	title = String(
		35, 120, 
		title +' usage per week', 
		fontSize = 12
	)	
    #cpu y axis range
	chart.valueAxis.valueMin = 0
	chart.valueAxis.valueMax = data_max
	chart.valueAxis.valueStep = round(data_max,2)/5
	chart.lines[0].strokeWidth = 3.5
	chart.lines[0].strokeColor = colors.purple

	drawing = Drawing(170, 450) 
	drawing.add(title)
	drawing.add(chart)
	return drawing
    ```

4. dataframe을 표(이미지 형태)로 저장
    ```python
    def get_table(dataframe):
        table_fig = plt.figure(figsize=(9,2))
        ax = plt.subplot(111)
        ax.axis('off')
        ax.table(cellText=dataframe.values, colLabels=dataframe.columns, bbox=[0,0,1,1])
        return table_fig
	
    from reportlab.graphics.shapes import Drawing
    from reportlab.lib import colors
    from reportlab.platypus import Table
    from reportlab.graphics.shapes import String
    import matplotlib.pyplot as plt
    
    cpu_title = 'cpu'
    mem_title = 'mem'
    net_title = 'net'

    cpu_data = [tuple(x for x in gen_weekly_data["cpu"].values.tolist())]
    mem_data = [tuple(x for x in gen_weekly_data["mem"].values.tolist())]
    net_data = [tuple(x for x in gen_weekly_data["network"].values.tolist())]
    
    #metric value max
    cpu_max = max(cpu_data[0])
    mem_max = max(mem_data[0])
    net_max = max(net_data[0])

    #metric value sum
    cpu_sum = sum(cpu_data[0]) * 10000
    mem_sum = sum(mem_data[0]) / 10000
    net_sum = sum(net_data[0])
    
    #total cost
    total_sum = round(cpu_sum + mem_sum + net_sum)

    pieChart = getPieChart(cpu_sum, mem_sum, net_sum)
    cpu_lineChart = getLineChart(cpu_title, cpu_data, cpu_max)
    mem_lineChart = getLineChart(mem_title, mem_data, mem_max)
    net_lineChart = getLineChart(net_title, net_data, net_max)
    
    dataframe_fig = get_table(gen_weekly_data)

    dataframe_fig.savefig('dataframe.png')

    #chart 위치
    table = Table([
	[cpu_lineChart,mem_lineChart,net_lineChart]
    ], 190, 200)

    pie_table = Table([
	[pieChart]
    ], 550, 350)
    ```
5. ???
    ```python
    table.setStyle([
	#('INNERGRID',(0,0),(-1,-1),1,colors.white),
	("VALIGN",(0,0),(-1,-1),"BOTTOM"),
    ("ALIGN",(0,0),(-1,-1),"CENTER"),	
   ])

   pie_table.setStyle([
	('INNERGRID',(0,0),(-1,-1),1,colors.transparent),
	("VALIGN",(0,0),(-1,-1),"BOTTOM"),
    ("ALIGN",(0,0),(-1,-1),"LEFT"),	
   ])
   
   elems = []
    elems.extend([table, pie_table])
    pdf.build(elems)
    ```

6. 고지서 제목과 내용을 입력한 PDF를 생성하고 그래프 PDF와 병합
    ```python
    from PyPDF2 import PdfFileWriter, PdfFileReader
    import io

    import datetime
    import calendar

    now = datetime.datetime.today()
    month = calendar.month_name[now.month]

    from reportlab.pdfgen import canvas

    pdf_data = io.BytesIO()
    # create a new PDF with Reportlab
    pdf_canvas = canvas.Canvas(pdf_data, pagesize=letter)
    # 제목
    pdf_canvas.setFont('Helvetica', 20)
    pdf_canvas.drawString(200,725,month+" Cloud Usage Bill")
    # 가격 산정 제목
    pdf_canvas.setFont('Helvetica', 18)
    pdf_canvas.drawString(330,270,"< Price Calculation Method >")

    pdf_canvas.setFont('Helvetica', 9)
    # CPU 가격 산정 방식
    pdf_canvas.drawString(350,240,"CPU: sum of container cpu usage, dollar per 0.1")
    # Memory 가격 산정 방식
    pdf_canvas.drawString(350,220,"Memory: container memory RSS, dollar per MB")
    # Network 가격 산정 방식
    pdf_canvas.drawString(350,200,"Network: container network transmit bytes total, dollar per MB")

    pdf_canvas.setFont('Helvetica', 18)
    pdf_canvas.drawString(350,100,"total cost :  " + str(total_sum) + "won")

    #draw image
    pdf_canvas.drawImage('logo.png', x=70, y=30, width=90, height=60)
    pdf_canvas.drawImage('dataframe.png', x=-20, y=300, width=648, height=144)
    pdf_canvas.save()

    os.system("rm -rf logo.png dataframe.png")
    ```

# 6. 서비스 시연 결과물
![service_bill](https://github.com/FalcoN1995/project6/blob/master/images/bill.png)
