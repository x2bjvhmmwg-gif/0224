
---

# Kubernetes & MSA 수업 총정리

## 1️⃣ 기본 개념

### 쿠버네티스(Kubernetes, k8s)

* 컨테이너 오케스트레이션 도구
* 여러 서버에 컨테이너 자동 배치
* 서버 장애 시 자동 복구
* 트래픽 자동 분산

> Docker = 컨테이너 하나 실행
> Kubernetes = 수백 개 컨테이너 관리

---

### 핵심 구성 요소

| 구성요소             | 역할                               |
| ---------------- | -------------------------------- |
| Pod              | 컨테이너 실행 단위                       |
| ReplicaSet       | Pod 여러 개 묶어 관리, 지정 수량 유지         |
| Deployment       | ReplicaSet 관리, 롤링 업데이트, 버전 관리 가능 |
| Service          | Pod 외부 연결, 로드밸런서 역할              |
| Namespace        | 리소스 그룹화, 논리적 폴더                  |
| Label / Selector | Pod를 논리적으로 선택, 서비스 연결 기준         |

---

## 2️⃣ 관계도

```
[외부 요청] → [Service(NodePort)]
                     │
                     ▼
                [Deployment]
                     │
                     ▼
                [ReplicaSet]
                     │
         ┌───┼────┬───┐
         ▼   ▼    ▼
      [Pod1][Pod2][Pod3]
         │    │     │
         ▼    ▼     ▼
      [Container 실행]
```

* Deployment → ReplicaSet → Pod 구조
* Label & Selector로 Service와 연결
* Service(NodePort)로 외부 요청을 Pod로 전달

---

## 3️⃣ Namespace

* 논리적 격리 단위 (폴더 개념)
* 예: `default`, `kube-system`, `n1`, `n2`
* 명령어 예시:

```bash
# Namespace 생성
kubectl create namespace n1

# Namespace 목록 확인
kubectl get namespaces

# 컨텍스트 생성 및 설정
kubectl config set-context n1-context --cluster=docker-desktop --user=docker-desktop --namespace=n1
kubectl config use-context n1-context
```

---

## 4️⃣ Pod 관리

* **Pod 이름 지정**

  * 직접 지정: `metadata.name`
  * 자동 생성: `[Deployment 이름]-[랜덤 문자열]`

* 명령어 예시:

```bash
# Pod 생성
kubectl run web --image=nginx:1.28 --port 80 -n n1

# Pod 확인
kubectl get pods -n n1
kubectl get pods --all-namespaces

# Pod 접속
kubectl exec -it <pod-name> -- /bin/bash

# Pod 비정상 테스트
docker ps
docker kill <container-ID>
kubectl get pods
```

---

## 5️⃣ Deployment & ReplicaSet

* **Deployment** = ReplicaSet 관리, 롤링 업데이트, 롤백 가능
* **ReplicaSet** = Pod 묶음 관리, 지정 수량 유지
* **Label / Selector** 중요 → Pod 선택 기준

### YAML 예제 (Deployment)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-dp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-label
  template:
    metadata:
      labels:
        app: app-label
    spec:
      containers:
      - image: nginx:1.28
        name: nginx
```

### 명령어 예시

```bash
# Deployment 생성 (dry-run으로 yaml 확인)
kubectl create deployment app-dp --replicas=2 --image=nginx:1.28 --dry-run=client -o yaml

# Deployment 적용
kubectl apply -f app-deployment.yaml

# 상태 확인
kubectl get deployment
kubectl get replicaset
kubectl get pods

# replicas 수정
kubectl edit deployment/app-dp
```

* Rollout: 점진적 업데이트
* Rollback: 이전 버전 복구 가능

---

## 6️⃣ Service

* 역할: Pod 외부 접속 + 로드밸런서 역할
* 종류:

  * `NodePort`: 외부 접속 가능
  * `ClusterIP`: 내부 통신만
  * `LoadBalancer`: 클라우드 로드밸런서 연결

### YAML 예제 (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-dp
spec:
  type: NodePort
  selector:
    app: app-label
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

### 명령어 예시

```bash
# Service 적용
kubectl apply -f app-service.yaml

# Service 확인
kubectl get service

# Service 삭제
kubectl delete service app-dp
```

> 외부 요청 → NodePort → Service → Selector → Pod

---

## 7️⃣ 자주 사용한 명령어 모음

```bash
# Context 확인
kubectl config get-contexts
kubectl config view
kubectl config use-context <context-name>

# Namespace
kubectl create namespace <name>
kubectl get namespaces

# Pod
kubectl get pods -n <namespace>
kubectl exec -it <pod-name> -- /bin/bash

# Deployment
kubectl apply -f <deployment.yaml>
kubectl edit deployment/<name>

# ReplicaSet
kubectl get replicaset

# Service
kubectl apply -f <service.yaml>
kubectl get service
```

---

## 8️⃣ MSA & Spring Cloud

* **Spring Cloud** = Java MSA 대표 프레임워크

* 역할:

  * 서비스 등록/발견: Eureka, Consul
  * 로드밸런싱: Ribbon, LoadBalancer
  * API Gateway: Zuul, Spring Cloud Gateway
  * 구성 관리: Config Server
  * 분산 트레이싱: Sleuth, Zipkin

* Kubernetes 연계

  * K8s: 컨테이너 배포, 스케일링, 복구
  * Spring Cloud: 서비스 연결, 호출, 로드밸런싱

---

## 9️⃣ 공식 문서

* [Kubernetes 공식 문서](https://kubernetes.io/docs/)
* [Spring Cloud 공식 문서](https://spring.io/projects/spring-cloud)

---


