<img src="https://capsule-render.vercel.app/api?type=waving&color=00C3FF&height=150&section=header" width="1000" />

<div align="center">
<h1 style="font-size: 36px;">🌱 Docker & K8s를 활용한<br> SpringBoot App 배포</h1>
</div>
</br>

## 목차
1. [🙆🏻‍♂️ 팀원](#%EF%B8%8F-팀원)
2. [🌱 프로젝트 개요: Docker & K8s를 활용한 SpringBoot App 배포](#-프로젝트-개요-docker--k8s를-활용한-springboot-app-배포)
3. [#️⃣ 실습 과정](#%EF%B8%8F⃣-실습-과정)
4. [📖 배운 점](#-배운-점)
5. [💜 회고](#-회고)

## 🙆🏻‍♂️ 팀원

#### 팀명 : 아프지말아조
우리FISA 4기 클라우드 엔지니어링 아프지말아조팀

|<img src="https://avatars.githubusercontent.com/u/150774446?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/55776421?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/179544856?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/129985846?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|:-:|
|김예진<br/>[@yeejkim](https://github.com/yeejkim)|이슬기<br/>[@seulg2027](https://github.com/seulg2027)|이은준<br/>[@2EunJun](https://github.com/2EunJun)|정파란<br/>[@BlueRedOrange](https://github.com/BlueRedOrange)|

---

# 🌱 프로젝트 개요: Docker & K8s를 활용한 SpringBoot App 배포

Spring Boot 애플리케이션을 Docker로 컨테이너화하고, Kubernetes 환경에 NodePort와 LoadBalancer 방식으로 배포하는 실습 레포지토리입니다.

<br>

## 📌 주요 목표

- ✅ Spring Boot 애플리케이션 Docker 이미지화
- ✅ Docker Hub에 이미지 Push
- ✅ Kubernetes에 NodePort 방식으로 배포
- ✅ Kubernetes에 LoadBalancer 방식으로 배포

<br>

## #️⃣ 실습 과정


### 🔅 실습 환경

| 사용 도구 | 내용 |
| --- | --- |
| 클러스터 환경 | Minikube |
| 애플리케이션 프레임워크 | Spring Boot |
| 컨테이너 빌드 | Docker  |
| 이미지 저장소 | Docker Hub |
| 쿠버네티스 CLI | kubectl |
| 테스트 도구 | curl, 브라우저 |


### ⚙️ 1. Spring Boot 애플리케이션 Docker 이미지 빌드 & 푸시

- 아래 Dockerfile을 활용하여 애플리케이션을 이미지화 하고, Docker Hub에 푸시 진행

  <details>
    <summary>📦 Dockerfile</summary>
  
    ```dockerfile
    # 1. 가벼운 JRE 기반 이미지 사용 (Alpine 기반)
    FROM eclipse-temurin:17-jre-alpine
  
    # 2. 작업 디렉토리 설정
    WORKDIR /app
  
    # 3. JAR 파일 복사
    COPY step07_cicd-0.0.1-SNAPSHOT.jar app.jar
  
    # 4. 실행 명령어 설정 (JVM 최적화 옵션 추가)
    ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
  ```
  </details>



```bash

# 이미지 build
$docker build -t <dockerhub-id>/<app-name>:latest .

# docker hub login
$docker login
Username:
Password:

# image push 
$docker push <dockerhub-id>/<app-name>
```

<img src="https://github.com/user-attachments/assets/b566fe37-34fe-472a-8026-6e7a39f6def8" width="700">


### 💫 2. Kubernetes 배포

### 🔹 NodePort 방식 배포

<details>
  <summary>📄 cicd_nodeport.yaml</summary>

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: step07-nodeport
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: step07
    template:
      metadata:
        labels:
          app: step07
      spec:
        containers:
          - name: step07
            image: yejin00/step07-cicd
            ports:
              - containerPort: 8081

  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: step07-nodeport-service
  spec:
    selector:
      app: step07
    ports:
      - protocol: TCP
        port: 8081
        targetPort: 8081
        nodePort: 30080  # NodePort 범위 내에서 선택
    type: NodePort
```
</details>


```bash
# yaml 파일 실행 
$kubectl apply -f <yaml-file-name>

# NodePort 확인
kubectl get svc

# 접속 확인
$curl http://<minikube-IP>:<Port-number>
```


### 🔹 LoadBalancer 방식 배포

<details>
  <summary>📄 cicd_loadbalancer.yaml</summary>

  ```yaml
  apiVersion: apps/v1
kind: Deployment
metadata:
  name: step07-loadbalancer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: step07-lb
  template:
    metadata:
      labels:
        app: step07-lb
    spec:
      containers:
        - name: step07
          image: yejin00/step07-cicd
          ports:
            - containerPort: 8081

---
apiVersion: v1
kind: Service
metadata:
  name: step07-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: step07-lb
  ports:
    - protocol: TCP
      port: 8081        # 외부에서 접속할 포트
      targetPort: 8081  # 컨테이너에서 열려 있는 포트
```
</details>


```bash
# yaml 파일 실행
$kubectl apply -f <yaml-file-name>

# LoadBalancer 확인
$kubectl get svc

# LoadBalancer IP tunneling
$minikube tunnel

# 접속 확인
$curl http://<minikube-IP>:<Port-number>
```


### 🚀 3. 접속 확인 결과
### ✅ NodePort 방식

```yaml
spec:
  ...
  ports:
    - protocol: TCP
      port: 8081       # Client 가 접근하는 외부 포트
      targetPort: 8081 # 실제 어플리케이션이 실행되는 포트
      nodePort: 30100  # 30000~32767 사이의 값 선택 가능
  type: NodePort
```

- Type을 NodePort로 설정 후, `ports` 에 nodePort를 할당

```bash
$kubectl apply -f nginx-nodeport.yaml # yaml 파일 적용
ubuntu@miniserver:~/02.kubernetes$ kubectl get svc # ip 확인
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
kubernetes            ClusterIP      10.96.0.1        <none>         443/TCP          28h
spring-load-service   LoadBalancer   10.106.10.98     10.106.10.98   8081:30951/TCP   17m
spring-service        NodePort       10.110.146.221   <none>         8081:30100/TCP   32m
```

<img src="https://github.com/user-attachments/assets/7d5e2f94-7b81-4234-9676-2cd7bc4603df" width="700">

### ✅ LoadBalancer 방식

<img src="https://github.com/user-attachments/assets/d6a66017-3530-4952-ab1f-220ef90254a8" width="700">



### 🔍 4. Minikube 환경에서 외부 접속
### ✅ NodePort 방식

<img src="https://github.com/user-attachments/assets/e12081d4-1367-44b5-80d8-db133fc8993b" width="700">


### ✅ LoadBalancer 방식

<img src="https://github.com/user-attachments/assets/e3293537-a88c-4f0c-964a-8f40135e0ecd" width="700">



## 📖 배운 점

**📌 Spring Boot 애플리케이션 컨테이너화**

- JAR 파일을 `Dockerfile`을 통해 컨테이너 이미지로 빌드
- `eclipse-temurin:17-jre-alpine` 기반으로 가벼운 실행 환경 구성

**📌 Docker 이미지 푸시 & 공유**

- 로컬에서 빌드한 이미지를 Docker Hub에 업로드하고, 외부에서 당겨 사용

**📌 Kubernetes 배포 기본**

- `Deployment`, `Service` 리소스를 정의하는 YAML 파일 생성
- `kubectl apply`, `kubectl get all`, `kubectl logs` 등을 활용한 리소스 관리

**📌 NodePort vs LoadBalancer 이해**

- NodePort는 Minikube 노드 IP와 외부 포트로 접근
- LoadBalancer는 클라우드 환경에서는 외부 IP를 자동 할당, Minikube에선 `minikube service`로 접근


## 💜 회고

- 처음에는 단순히 "JAR → 이미지" 정도만 알고 있었는데, 쿠버네티스까지 연결해서 **전체 배포 파이프라인 흐름을 직접 체험**해볼 수 있어 좋았다.
- 특히 `ImagePullBackOff` 같은 에러를 직접 마주치고 해결하면서 **실전에서 자주 겪는 문제 해결 능력**을 키울 수 있었다.
- NodePort와 LoadBalancer의 차이를 직접 실습으로 비교해보며 **K8s 네트워크 구조**에 대한 감을 잡을 수 있었다.
- 단순히 따라치는 것이 아니라, **오류 상황마다 원인 분석 → 로그 확인 → 해결**을 반복하며 문제 해결력을 기를 수 있었다. 🔥
