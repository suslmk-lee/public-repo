# Sys-Cafe Jupyter-notebook 배포


## admin-portal
admin-portal에서 앱개발환경을 추가합니다.
추가 시 buildpack은 'python_buildpack_jypyter' 를 선택합니다.
> admin-portal은 접속문제로 화면 덤프 없음.


## user-portal
admin-portal에서 앱개발환경을 추가하면 아래와 같이 추가한 앱개발환경을 선택할 수 있습니다.

선택 후 아래 내용에 따라 진행하십시오.
![image](https://user-images.githubusercontent.com/67575226/99601999-f23a1400-2a43-11eb-9478-882d3b33abc4.png)

![image](https://user-images.githubusercontent.com/67575226/99602217-75f40080-2a44-11eb-8c86-3f12f8d2c739.png)
메모리는 2G, 디스크는 1G 선택
사용자앱으로 선택 후 zip 파일을 등록 --> 생성

앱배포시간은 약 10분에서 15분이 소요됩니다.


앱배포 후 대쉬보드 화면입니다.
![image](https://user-images.githubusercontent.com/67575226/99602848-88bb0500-2a45-11eb-8c27-f1c7ea7e4158.png)

## 이슈
1. offline 배포시 문제
압축된 파일에는 관련된 소스파일이 없이 manifest로만 구성이 되어 있으며,
소스파일을 추가시 300Mbyte이상이 되어 UI 로 테스트시 UI에서 멈추는 현상이 발생하였습니다.

2. 포함된 plugin 문제
torch --> 포함 시 1G가 넘어버림.
keras
위 두개 앱 추가시 cf cli로도 현재는 정상적인 배포가 불가합니다.
