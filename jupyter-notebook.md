#Jupyter-notebook 배포

###▶BuildPack 다운로드

https://github.com/cloudfoundry/python-buildpack
![image1](https://user-images.githubusercontent.com/67575226/97148184-09237880-17ae-11eb-9df1-2258e978bc98.png)

다운로딩한 zip 파일을 cf 서버로 업로드한다.
```
$ cf create-buildpack {생성할 buildpack Name} {업로드한 zip 파일 경로} {position} --enable
(ex.) $ cf create-buildpack python_buildpack_jypyter ./python-buildpack-v1.7.24.zip 10 --enable

$ cf buildpacks
Getting buildpacks...
buildpack                  position   enabled   locked   filename                                       stack
staticfile_buildpack       1          true      false    staticfile_buildpack-cflinuxfs3-v1.4.42.zip    cflinuxfs3
java_buildpack             2          true      false    java-buildpack-cflinuxfs3-v4.19.zip            cflinuxfs3
ruby_buildpack             3          true      false    ruby_buildpack-cflinuxfs3-v1.7.38.zip          cflinuxfs3
dotnet_core_buildpack      4          true      false    dotnet-core_buildpack-cflinuxfs3-v2.2.11.zip   cflinuxfs3
nodejs_buildpack           5          true      false    nodejs_buildpack-cflinuxfs3-v1.6.49.zip        cflinuxfs3
go_buildpack               6          true      false    go_buildpack-cflinuxfs3-v1.8.39.zip            cflinuxfs3
python_buildpack           7          true      false    python_buildpack-cflinuxfs3-v1.6.32.zip        cflinuxfs3
php_buildpack              8          true      false    php_buildpack-cflinuxfs3-v4.3.76.zip           cflinuxfs3
nginx_buildpack            9          true      false    nginx_buildpack-cflinuxfs3-v1.0.11.zip         cflinuxfs3
python_buildpack_jypyter   10         true      false    python-buildpack-v1.7.24.zip
r_buildpack                11         true      false    r_buildpack-cflinuxfs3-v1.0.9.zip              cflinuxfs3
binary_buildpack           12         true      false    binary_buildpack-cflinuxfs3-v1.0.32.zip        cflinuxfs3
jupyter_python             13         true      false    python-buildpack.zip
```

###▶배포할 Jupyter-notebook 다운로드

```
$ git clone https://github.com/AbhishekGhosh/Bluemix-Jupyter-Notebook.git

```

###▶배포파일 수정

```
$ cd Bluemix-Jupyter-Notebook/

# requirement.txt의 내용 수정
$ vi requirements.txt
ipython[notebook]
numpy==1.16.6
pandas
matplotlib
tensorflow==2.0.0b1
torch
keras

# manifest.yml 파일 내용 수정
# vi manifest.yml
---
applications:
- name: ipython
  memory: 4G
  instances: 1
  host: ipython 
  domain: sys-cafe.com
  path: .
  buildpack: python_buildpack_jypyter
```

###▶패키지 다운로드

```
추가패키지를 Offline배포하기 위하여 vendor 폴더에 다운 받는다.
$ cd YOUR-APP-DIR  
$ mkdir -p vendor   
$ pip download -r requirements.txt --no-binary=:none: -d vendor
```

###▶배포

```
# disk 10G, timeout 15로 하여 배포해야 오류발생하지 않음.
# 기본 disk크기는 10g로 설정할 수 있게 사전에 변경해 놓는다.
$ cf push --no-start -k 10g t 15
$ cf start ipython
```

###▶접속
![image2](https://user-images.githubusercontent.com/67575226/97148230-222c2980-17ae-11eb-92a2-29464b9e642d.png)
