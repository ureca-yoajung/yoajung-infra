# yoajung-infra

## 프로젝트 개요
- Spring Boot 기반의 `yoajung` 프로젝트를 Kubernetes 환경에 배포하기 위한 인프라 구성입니다. 
- 오브젝트를 디렉토리 단위로 구분하여 유지보수가 용이하도록 구성했습니다.

## 아키텍처
![architecture](https://github.com/user-attachments/assets/c99ff7a9-e885-406a-b757-828563c17a88)

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
