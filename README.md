# 쿠버네티스 로드밸런스를 온프레미스에서

### 발표자
조훈/인프라엔지니어그룹​

### 발표 소개

"온프레미스 쿠버네티스에 Pod들을 쿠버네티스 클러스터 외부로 서비스하기 위해서는 NodePort 서비스 또는 Ingress 오브젝트 등의 방법으로 노출해야 합니다. 만약 사용자가 온프레미스에서 기존에 구성 방식을 가능한 그대로 옮기고자 하는 경우에는 loadBalancer 서비스 타입이 가장 유효할 것입니다. (loadBalancer는 기존에 인프라 엔지니어들이 알고 있는 L4 loadbalacer와 매우 유사합니다.) 그렇지만 온프레미스 쿠버네티스에서를 native한 상태로 구성하였다면 loadBalancer에 대한 타입이 즉시 사용가능하지 않습니다. 이를 필요시 구현하도록 작성되어 있기 때문입니다. 이런 경우 손쉽게 native한 상태의 쿠버네티스에서도 loadBalancer 서비스 타입을 사용할 수 있게 해주는 프로젝트가 존재하는데, 이것이 이번 session에 소개해 드릴 Metallb입니다. Metallb는 매우 손쉽게 구현가능하고, 기본적인 설정만으로 강력한 기능을 제공합니다. 이 세션을 듣기 위해서 준비할 것은 따로 없습니다. 쿠버네티스와 Pod가 무엇인지 정도만 아신다면 타입별로 서비스가 노출되는 것들을 랩을 통해서 보여드리면서 설명드릴 예정입니다. 발표 세션 시간에 뵙겠습니다."

# 쿠버네티스 파드에 사용자가 접속하는 방법 = 서비스

1) ClusterIP: 내부 접속만 가능 => X
2) NodePort
3) LoadBalancer(지원 가능한 경우)
4) ExternalName: 복잡 어쩌구 저쩌구 => X

오늘은 2, 3번 케이스를 살펴볼 예정

# 발표자 Lab 환경
![environment](/images/environment.jpeg)

# 서비스 케이스
### Case1: NodePort 서비스 방식
![](/images/nodeport.PNG)
* 효율성이 떨어지는 요청 방식
* worker node가 죽으면 pod의 IP가 변경되기 때문에 k8s로 제대로 서비스 하기 어려운 부분이 있다
* production보다는 주로 개인, 테스트용으로 사용하는 서비스 방식

* 기본 포트 범위: 31000~32727

### Case2: LoadBalance 서비스 방식(바로 사용이 가능한 경우: Cloud)
![](/images/loadbalancer.PNG)

### Case3: LoadBalance 서비스 방식(바로 사용이 불가능한 경우: Onpremise) => MetalLB service
![](/images/matalLB.jpg)

#### deployment 생성
```
# kubectl create deployment lb-hname-pods --images=sysnet4admin/echo-hname
# kubectl get pod
```

#### metallb 설치
```
# kubectl apply -f https://raw.githubusercontent.com/sysnet4admin/IaC/master/manifests/svc/metallb.yaml
# kubectl get pod -n metallb-system
# kubectl get pod -n matallb-system -o wide
```
controller가 기능을 종합적으로 관리해주는 역할을 한다.

#### configMap을 사용하여 matallb 설정
```
# kubectl apply -f https://raw.githubusercontent.com/sysnet4admin/IaC/master/manifests/svc/metallb-l2config.yaml
```

#### service expose
```
# kubectl expose deployment lb-hname-pods --type=LoadBalancer --name=lb-hname-svc --port=80
# kubectl get svc
```

#### LB 테스트를 위해 replicaSet scale out
```
# kubectl get node -o wide
# kubectl scale deploy lb-hname-pods --replicas=6
```

* 설치가 매우 간단
* `configMap` 만 간단하게 설정하면 사용 가능
* 단, 기본 설정은 네트워크 관점에서는 효율적으로 동작하지 않으니(내부적으로 L2 switch 사용), 추후에 네트워크 엔지니어와 상담해서 구체적으로 구성을 잡아가세요.