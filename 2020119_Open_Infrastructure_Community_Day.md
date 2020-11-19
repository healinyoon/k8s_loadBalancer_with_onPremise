# Crunchbase Rank?
= 투자 가치가 있다.

# Porter vs Metallb
우리는 경쟁자로 인식한다(=우리는 1위 없체가 아니다).

# Porter vs Metallb 비교
* 구성 환경

## Metallb

* 구조

* Metallb 동작 확인
- pod deployment
- namespace 생성 + 배포
- Metallb speaker와 controller 통신 위한 secret 설정
- configmap 배포
- Metallb 서비스 배포
- speacker와 controller 통신 확인
    - speacker는 일하고 controller는 일 거의 안함

## Porter

* 구조

* portal 동작 확인
- pod deployment: helm 으로 진행
    - 생성된 portal 확인
- eip라는 오브젝트를 만들어서 진행(metallb는 configmap을 사용하는 점이 다르다.)
- portal을 이용해서 service expose
- kubetail로 확인
    - default namespace 사용
    - masterdpsms agent 설치 안함
    - monitoring하면 agent는 거의 일 안하고 manager가 많이 함


# 결론
- metallb: 데몬셋으로 각 워커 노드에 배포된 스피커가 열심히 일함
- porter: 데몬셋으로 배포된 에이전트는 일 안하고 포터 매니저가 열일함

에이전트가 열일하는 것이 쿠버네티스 다움
기술, 비기술적으로 보았을 때도 metallb가 더 안정적

- 약간의 기술이 다를 뿐, L2, L3가 동작하는 방식은 거의 똑같다.

결론: 네트워크 엔지니어의 상담을 받으세요 ^_^