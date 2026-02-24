* 공식 문서 예시 포함
* 오늘 배운 명령어 포함
* Kubernetes 핵심 개념, YAML, Deployment, Service, Namespace 등 전체 흐름 정리

---

# Kubernetes 총정리 (2026-02-24 수업)

## 1️⃣ Kubernetes 개요

* **Kubernetes(k8s)**: 컨테이너 오케스트레이션 도구

  * 수백 개의 컨테이너를 여러 서버에 자동 배치
  * 서버 장애 발생 시 자동 복구
  * 트래픽 자동 분산

* **Docker vs Kubernetes**
  | 항목 | Docker | Kubernetes |
  |------|--------|------------|
  | 실행 단위 | 컨테이너 하나 | Pod, Deployment 등 관리 |
  | 관리 | 단일 서버 | 여러 서버, 수백 개 컨테이너 |
  | 기능 | 실행/종료 | 배포, 스케일링, 로드밸런싱, 업데이트, 롤백 |

* **기본 네트워크**

  * IP, Port, DNS 지원
  * Pod 내부 통신 및 외부 접근 가능

---

## 2️⃣ 핵심 개념

| 구성요소                 | 역할            | 설명                                        |
| -------------------- | ------------- | ----------------------------------------- |
| **Pod**              | 컨테이너 실행 단위    | Pod 1개에 컨테이너 1~여러 개, 실무에선 1:1 사용 많음       |
| **Deployment**       | ReplicaSet 관리 | 롤링 업데이트, 롤백 가능, 그룹 단위 버전 관리               |
| **ReplicaSet(RS)**   | Pod 그룹 관리     | Pod 수 유지, Pod 강제 종료 시 자동 재생성              |
| **Service**          | 외부 연결 제공      | NodePort, ClusterIP, LoadBalancer, 트래픽 전달 |
| **Label / Selector** | Pod 선택 기준     | RS/Service가 Pod를 선택할 때 사용                 |
| **Namespace**        | 리소스 격리        | 여러 프로젝트/팀 간 격리 가능, 컨텍스트 전환 가능             |

---

## 3️⃣ Namespace

* **Namespace 생성**

```bash
kubectl create namespace n1
```

* **Namespace 확인**

```bash
kubectl get namespaces
kubectl get pods -n n1
kubectl get pods --all-namespaces
```

* **Namespace 설정**

```bash
kubectl config set-context n1-context --cluster=docker-desktop --user=docker-desktop --namespace=n1
kubectl config use-context n1-context
```

---

## 4️⃣ Pod 관리

* **Pod 생성 (Run)**

```bash
kubectl run web --image=nginx:1.28 --port=80 -n n1
```

* **Pod 확인**

```bash
kubectl get pods
kubectl get pods -n n1
kubectl get pods --all-namespaces
```

* **Pod 접속**

```bash
kubectl exec -it web-xxxx -- /bin/bash
```

* **Pod 제거**

```bash
kubectl delete pod [pod이름]
```

* **Pod YAML 확인**

```bash
kubectl run web --image=nginx:1.28 --port=80 --dry-run=client -o yaml
kubectl create -f web.yaml
```

* **Labels**

  * ReplicaSet과 Service가 Pod 선택 시 사용
  * `kubectl explain pod.metadata.labels` 확인 가능

---

## 5️⃣ Deployment 관리

* **Deployment 생성**

```bash
kubectl create deployment app-dp --replicas=2 --image=nginx:1.28 --dry-run=client -o yaml
kubectl apply -f app-dp.yaml
```

* **Deployment 구조 예시 (YAML)**

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

* **Deployment 확인**

```bash
kubectl get deployment
kubectl get replicaset
kubectl get pods
```

* **Deployment 편집**

```bash
kubectl edit deployment app-dp
```

* **ReplicaSet 역할**

  * Pod의 복제본 관리
  * Deployment가 Pod 업데이트 시 RS를 관리

* **롤링 업데이트**

  1. 새 ReplicaSet 생성
  2. 새 Pod 1개 생성
  3. 기존 Pod 1개 삭제
  4. 순차 교체

  * 오류 발생 시 롤백 가능

---

## 6️⃣ Service 관리

* **Service YAML 예시**

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

* **Service 적용**

```bash
kubectl apply -f app-dp-service.yaml
```

* **Service 확인**

```bash
kubectl get service
kubectl describe service app-dp
```

* **포트 제한**

  * NodePort 외부 포트: 30000~32767
  * ClusterIP: 내부 통신 전용
  * LoadBalancer: 외부 로드밸런서 연결

* **Service 역할**

  * Pod 접속 자동화
  * NodePort를 통한 외부 트래픽 분산
  * Service 종료 시 연결된 Pod는 외부 접근 불가 (“죽은 것과 같음”)

---

## 7️⃣ 전체 흐름도

```
[외부 요청] ──> [Service(NodePort)]
                     │
                     ▼
                [Deployment]
                     │
                     ▼
                [ReplicaSet]
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
      [Pod1]       [Pod2]      [Pod3]
         │           │           │
         ▼           ▼           ▼
   [Container 실행] [Container 실행] [Container 실행]
```

**설명**

1. 외부 요청은 NodePort Service로 들어감
2. Service는 Selector(Label) 기준으로 Pod 연결
3. Deployment는 ReplicaSet 관리, 롤링 업데이트 가능
4. ReplicaSet은 Pod 수량 유지 및 재생성
5. Pod 안에서 실제 컨테이너 실행

---

## 8️⃣ 주요 명령어 총정리

| 기능            | 명령어                                                                 |
| ------------- | ------------------------------------------------------------------- |
| Namespace 생성  | `kubectl create namespace n1`                                       |
| Namespace 확인  | `kubectl get namespaces`                                            |
| Context 확인    | `kubectl config get-contexts`                                       |
| Context 변경    | `kubectl config use-context n1-context`                             |
| Pod 생성        | `kubectl run web --image=nginx:1.28 -n n1`                          |
| Pod YAML 확인   | `kubectl run web --image=nginx:1.28 -n n1 --dry-run=client -o yaml` |
| Pod 삭제        | `kubectl delete pod [pod이름]`                                        |
| Deployment 생성 | `kubectl create deployment app-dp --replicas=2 --image=nginx:1.28`  |
| Deployment 확인 | `kubectl get deployment`                                            |
| ReplicaSet 확인 | `kubectl get replicaset`                                            |
| Pod 확인        | `kubectl get pods -n n1`                                            |
| Deployment 수정 | `kubectl edit deployment app-dp`                                    |
| Service 생성    | `kubectl apply -f app-dp-service.yaml`                              |
| Service 확인    | `kubectl get service`                                               |
| Pod 접속        | `kubectl exec -it pod이름 -- /bin/bash`                               |

---

## 9️⃣ 참고 공식 문서

* Kubernetes 공식: [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/)
* Deployment: [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Service: [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

---


