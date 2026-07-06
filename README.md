# C9S-containerLAB

> Kubernetes 위에서 Arista cEOS를 이용한 가상 네트워크 랩 환경.  
> Clabernetes(containerlab + K8s)를 활용하여 토폴로지를 클라우드 네이티브 방식으로 구현.

---

## 개요

기존 containerlab은 단일 호스트에서만 동작하는 한계가 있습니다.  
이 프로젝트는 **Clabernetes**를 사용하여 containerlab 토폴로지를 Kubernetes 클러스터 위에서 실행하고,  
Arista cEOS 기반의 가상 네트워크 랩을 **클라우드 네이티브 환경에서 운영**합니다.

## 구현 토폴로지

```
# 노드와 링크를 자유롭게 수정하여 원하는 네트워크 구성 즉각 배포 #
# 이미지는 GHCR 기반 다운도르

Ex) yaml 예시 아래 내용을 필요에 맞도록 수정
            nodes:
              leaf1:
                kind: arista_ceos
                #image: ghcr.io/zpzg333/ceos:4.34.4F
              leaf2:           
                kind: arista_ceos
                #image: ghcr.io/zpzg333/ceos:4.34.4M
              client1:           
                kind: arista_ceos
                #image: ghcr.io/zpzg333/ceos:4.34.4M
              client2:           
                kind: arista_ceos
                #image: ghcr.io/zpzg333/ceos:4.34.4M
              
            links:
              #spine to leaf
              - endpoints: ["leaf1:eth8", "leaf2:eth8"]
              - endpoints: ["leaf1:eth1", "client1:eth1"]
              - endpoints: ["leaf2:eth1", "client2:eth1"]
```

## 구조

```
C9S-containerLAB/
└── 1.2node.yaml    ← Clabernetes Topology CR (Custom Resource)
```

## 실행 방법

```bash
# Clabernetes 설치 전제 (https://clabernetes.containerlab.dev)

# 토폴로지 배포
kubectl apply -f 1.2node.yaml -n test-network

# 상태 확인
kubectl get topology -n test-network

# cEOS 접속
kubectl exec -it <pod-name> -n test-network -- Cli
```

## 사용 기술 포인트

- **Clabernetes**: containerlab 토폴로지를 K8s Custom Resource(CR)로 정의하여 클러스터에서 실행
- **cEOS on K8s**: Arista cEOS 컨테이너를 Kubernetes Pod로 운영
- **Private Registry**: GHCR(GitHub Container Registry)에 커스텀 cEOS 이미지 호스팅

## ArgoCD 연동

이 프로젝트는 **ArgoCD(GitOps)** 와 연동되어 있습니다.  
토폴로지 정의 파일(`1.2node.yaml`)을 GitHub에 push하면 ArgoCD가 자동으로 감지하여  
Clabernetes Custom Resource를 Kubernetes 클러스터에 배포합니다.

```
GitHub push (1.2node.yaml 수정)
        ↓
ArgoCD 자동 감지 (auto-sync)
        ↓
Clabernetes Topology CR 배포
        ↓
cEOS Pod 자동 생성 (test-network namespace)
```


실무에서 접하는 EVPN 아키텍처를 가상 환경에서 직접 검증하고,  
Kubernetes 기반의 클라우드 네이티브 랩 환경 구축 역량을 쌓기 위한 프로젝트입니다.
