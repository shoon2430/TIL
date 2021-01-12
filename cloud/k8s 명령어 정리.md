# k8s 명령어

개인적으로 기본적인 명령어 이외에 새로 알게된 명령어들까지 정리해 두려고 한다.

### kubectl 명령어

| 명령어     | 설명                                                         |
| ---------- | ------------------------------------------------------------ |
| `apply`    | 원하는 상태를 적용합니다. 보통 `-f` 옵션으로 파일과 함께 사용합니다. |
| `get`      | 리소스 목록을 보여줍니다.                                    |
| `describe` | 리소스의 상태를 자세하게 보여줍니다.                         |
| `delete`   | 리소스를 제거합니다.                                         |
| `logs`     | 컨테이너의 로그를 봅니다.                                    |
| `exec`     | 컨테이너에 명령어를 전달합니다. 컨테이너에 접근할 때 주로 사용합니다. |
| `config`   | kubectl 설정을 관리합니다.                                   |



## 상태 설정하기 (apply)

원하는 리소스의 상태를 YAML로 작성하고 apply 명령어로 선언합니다.

##### 명령어

```
kubectl apply -f [파일명 또는 URL]
```



## 리소스 목록보기 (get)

쿠버네티스에 선언된 리소스를 확인하는 명령어는 다음과 같습니다.

##### 명령어

```
kubectl get [TYPE]
```

##### 사용 예시

```powershell
# Pod 조회
kubectl get pod

# 변경내역 조회하기(-w 옵션을 주어 변경될 때 마다 조회한다.)
kubectl get pod -w

# 줄임말(Shortname)과 복수형 사용가능
kubectl get pods
kubectl get po

# 여러 TYPE 입력
kubectl get pod,service
#
kubectl get po,svc

# Pod, ReplicaSet, Deployment, Service, Job 조회 => all
kubectl get all

# 결과 포멧 변경
kubectl get pod -o wide
kubectl get pod -o yaml
kubectl get pod -o json

# Label 조회
kubectl get pod --show-labels
```



## 리소스 상세 상태보기 (describe)

쿠버네티스에 선언된 리소스의 상세한 상태를 확인하는 명령어는 다음과 같습니다.

##### 명령어

```powershell
kubectl describe [TYPE]/[NAME] 또는 [TYPE] [NAME]
```

##### 사용 예시

```powershell
# Pod 조회로 이름 검색
kubectl get pod

# 조회한 이름으로 상세 확인
kubectl describe pod/wordpress-5f59577d4d-8t2dg # 환경마다 이름이 다릅니다
```



## 리소스 제거 (delete)

쿠버네티스에 선언된 리소스를 제거하는 명령어는 다음과 같습니다.

##### 명령어

```
kubectl delete [TYPE]/[NAME] 또는 [TYPE] [NAME]
```

##### 사용 예시

```powershell
# Pod 조회로 이름 검색
kubectl get pod

# 조회한 Pod 제거
kubectl delete pod/wordpress-5f59577d4d-8t2dg
```



## 컨테이너 로그 조회 (logs)

컨테이너의 로그를 확인하는 명령어는 다음과 같습니다.

##### 명령어

```powershell
kubectl logs [POD_NAME]
```

##### 사용 예시

```powershell
# Pod 조회로 이름 검색
kubectl get pod

# 조회한 Pod 로그조회
kubectl logs wordpress-5f59577d4d-8t2dg

# 실시간 로그 보기
kubectl logs -f wordpress-5f59577d4d-8t2dg
```



## 컨테이너 명령어 전달 (exec)



##### 명령어

```powershell
kubectl exec [-it] [POD_NAME] -- [COMMAND]
```

##### 사용 예시

```powershell
# Pod 조회로 이름 검색
kubectl get pod

# 조회한 Pod의 컨테이너에 접속
kubectl exec -it wordpress-5f59577d4d-8t2dg -- bash
```



## 설정 관리 (config)

```powershell
# 현재 컨텍스트 확인
kubectl config current-context

# 컨텍스트 설정
kubectl config use-context minikube
```



## 그외

```sh
# 전체 오브젝트 종류 확인
kubectl api-resources

# 특정 오브젝트 설명 보기
kubectl explain pod
```



```powershell
# pod의 label 삭제
 kubectl label po/echo-rs-75c6r tier-

# pod의 label 추가
 kubectl label po/echo-rs-75c6r tier=app
```

