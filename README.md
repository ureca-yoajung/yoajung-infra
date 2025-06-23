## 프로젝트 개요
- Spring Boot 기반의 `yoajung` 프로젝트를 Kubernetes 환경에 배포하기 위한 인프라 구성입니다. 
- 오브젝트를 디렉토리 단위로 구분하여 유지보수가 용이하도록 구성했습니다.

## 팀원 소개
| 이름   | 역할      | 주요 구현 내용                          | GitHub                                             |
|------- |-------------|-----------------------------------|----------------------------------------------------|
| 이재윤 | Infra     | Kubernetes 기반 인프라 구성 및 배포 설정 | <a href="https://github.com/iju42829"><img src="https://avatars.githubusercontent.com/u/116072376?v=4" width="100" height="100" alt="iju42829" /></a>         |

## Kubernetes 설계
| Object           | 설명                                                                 |
|------------------|----------------------------------------------------------------------|
| 메인 서버         | 메인 서버는 Deployment가 관리하는 ReplicaSet을 통해 구성되며, 해당 ReplicaSet은 3개의 Spring Boot 애플리케이션을 유지합니다.<br>외부 요청은 Service 객체를 통해 Pod들과 연결되며, 이 Service는 NodePort 방식으로 30000번 포트를 개방해 외부 통신을 가능하게 하고, 내부적으로는 3개의 애플리케이션에 대해 로드밸런싱을 수행합니다.       |
| 관리자 서버       | 관리자 서버는 Deployment와 ReplicaSet, Service를 통해 구성되며, ReplicaSet에는 1개의 Spring Boot 애플리케이션(Pod)만 유지됩니다.<br> Service는 NodePort 방식으로 30001번 포트를 개방해 외부에서 접근할 수 있도록 설정되어 있습니다. |
| mysql            | MySQL은 Deployment, ReplicaSet, Service로 구성되며, 1개의 MySQL Pod를 유지합니다.<br> Service는 ClusterIP로 설정되어 외부 접근이 차단되며, Spring Boot 애플리케이션 내부에서만 접근할 수 있도록 구성되었습니다.<br>또한, 데이터의 영속성을 보장하기 위해 PersistentVolume(PV)을 연결하여 데이터를 유지할 수 있도록 설정했습니다.|
| redis            | Redis는 Deployment와 ReplicaSet, Service를 통해 구성되며, ReplicaSet에는 1개의 Redis Pod만 유지합니다.<br> 외부 접근을 차단하기 위해 Service는 ClusterIP 타입으로 설정되어 있으며, Spring Boot 애플리케이션 내부에서만 접근이 가능하도록 구성되었습니다.|
## 실행 방법
1. yoajung-admin, yoajung-server docker build
- 각 리포지토리를 클론한 후, 해당 디렉토리에서 아래 명령어를 실행하여 Docker 이미지를 빌드합니다.
```bash
docker build -t yoajung-admin .
docker build -t yoajung-server .
````

2. `server-secret.yaml` 파일 생성 (필수)

- 프로젝트 실행에 필요한 정보를 담은 Secret 오브젝트를 **프로젝트 루트 경로**에 생성합니다.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: server-secret
stringData:
  mysql-root-password: ""
  db-url: ""
  db-username: ""
  db-password: ""
  mysql-database: ""

  s3-access: ""
  s3-secret: ""

  open-ai-key: ""
  redis-host: ""
  redis-port: ""
  mail-username: ""
  mail-password: ""
  rest-api-key: ""     # (카카오 OAuth)
  redirect-url: ""     # (카카오 OAuth)
  dify-key: ""
```

3. secret object 등록
```bash
kubectl apply -f server-secret.yaml
```

4. mysql object 등록
```bash
kubectl apply -f mysql/mysql-pv.yaml
kubectl apply -f mysql/mysql-pvc.yaml
kubectl apply -f mysql/mysql-deployment.yaml
kubectl apply -f mysql/mysql-service.yaml
```

5. redis object 등록
```bash
kubectl apply -f redis/redis-deployment.yaml
kubectl apply -f redis/redis-service.yaml
```

6. admin spring object 등록
```bash
kubectl apply -f admin/admin-spring-deployment.yaml
kubectl apply -f admin/admin-spring-service.yaml
```

7. main spring object 등록
```bash
kubectl apply -f main/spring-deployment.yaml
kubectl apply -f main/spring-service.yaml
```
