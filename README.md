# 지능형 드론 개발 환경 구축 방법을 알아 봅시다.



## 기본 환경 구성

### OS

* Ubuntu 18.04 LTS 설치하기.

* GPU 드라이버 설치하기.
  ```
  $ sudo add-apt-repository ppa:graphics-drivers/ppa
  $ sudo apt-get update
  $ sudo apt-get install nvidia-driver-410
  ```
  
* 기본 패키지 설치하기.

  ```
  $ sudo apt-get install unzip
  ```



### docker

* [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 설치하기.
  
- container와 image가 저장되는 디렉토리 수정하기. [[참고사이트](http://dveamer.github.io/backend/DockerImageDirectory.html)]
  - Docker 프로세스 중지
    ```
    $ sudo service docker stop
    ```
  - container나 image를 저장할 폴더 생성
    ```
    $ mkdir -p /home/USER/DOCKERFOLDER
    ```
  - Docker 환경정보 파일 열기
    ```
    $ sudo gedit /etc/default/docker
    ```
  - Docker_OPT 변경
    ```
    $ DOCKER_OPTS="-g /home/USER/DOCKERFOLDER"
    ```
  - Docker 프로세스 시작
    ```
    $ sudo service docker start
    ```

* sudo 명령 없이 docker를 실행할 수 있도록 권한 수정하기. [참고사이트](https://docs.docker.com/install/linux/linux-postinstall/)

  ```
  $ sudo groupadd docker
  $ sudo usermod -aG docker $USER
  ```

* 만약 `WARNING: Error loading config file: /home/user/.docker/config.json: stat /home/user/.docker/config.json: permission denied`와 같은 에러가 발생한다면 아래 코드를 실행한다.

  ```
  $ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
  $ sudo chmod g+rwx "$HOME/.docker" -R
  ```



### NVIDIA docker

* [nvidia-docker2](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)) 설치하기.
- container와 image가 저장되는 디렉토리 수정하기. [[참고 사이트](https://docs.docker.com/config/daemon/systemd/#start-automatically-at-system-boot)]
  * Docker 서비스 중지 및 폴더 생성
    ```
    $ sudo service docker stop
    ```
    
  * container나 image를 저장할 폴더 생성 (앞서 이미 했으므로 무시 가능함.)
    
    ```
    $ mkdir -p /home/USER/DOCKERFOLDER
    ```
    
  * Docker daemon 설정 파일 생성(또는 수정)
    
    ```
    $ sudo gedit /etc/docker/daemon.json
    ```
    * 아래 구문 추가
      ```
      {
      	"data-root": "/home/teoac/docker",
      	"storage-driver": "overlay2",
      		"runtimes": {
      			"nvidia": {
      				"path": "nvidia-container-runtime",
      				"runtimeArgs": [] 
      			}
      		}
      }
      ```
    
  * Docker 서비스 재실행
    ```
    sudo service docker start
    ```
  * 확인
  
    ```
    sudo lsof | grep /home/USER/DOCKERFOLDER
    ```

* 정상 설치 확인하기  
  ```
  $ docker run --runtime=nvidia --rm nvidia/cuda:10.0-base nvidia-smi
  ```

* [NVIDIA GPU CLOUD](https://ngc.nvidia.com/registry/)에 로그인하기.
  
* 가입 후 개인 API Key를 얻어야 각종 도커 이미지를 다운로드할 수 있다.
  
* 해당 컴퓨터로 1회 해놓으면 향후 별도 로그인은 불필요하다.
  
  * 터미널창 명령
    ```
    sudo docker login nvcr.io
    Username: $oauthtoken
    Password: NGC에 가입하면 얻을 수 있는 API Key 입력
    ```
  



### AirSim (Binaries)

* [참고사이트](https://github.com/Microsoft/AirSim/blob/master/docs/docker_ubuntu.md#binaries)

* AirSim git 복제하기.

  ```
  $ git clone https://github.com/microsoft/AirSim.git
  ```

* docker image 받기.

  ```
  $ cd AirSim/docker
  $ python build_airsim_image.py \
     --base_image=nvidia/cudagl:10.0-devel-ubuntu16.04 \
     --target_image=airsim_binary:10.0-devel-ubuntu16.04
  ```

* Scene 파일 다운로드 받고 압축풀기.

  ```
  $ wget -c https://github.com/Microsoft/AirSim/releases/download/v1.2.0Linux/SCENE.zip
  $ unzip SCENE.zip
  $ rm SCENE.zip
  ```

  * 위 SCENE에는 리눅스에서 사용가능한 Unreal environment 이름이 들어가야 한다. 아래는 v1.2.0에서 가능한 이름과 이를 실행하기 위한 sh 파일 목록이다. [다운로드 링크](https://github.com/microsoft/AirSim/releases/tag/v1.2.0Linux)

    ```
    Africa				| Africa/Africa_001.sh
    Blocks				| Blocks/Blocks.sh
    City
    Forest
    LandscapeMountains
    Neighborhood		| Neighborhood/AirSimNH.sh
    SimpleMaze
    SubT
    TrapCam
    Warehouse
    ZhangJiaJie
    ```

* docker container 내에서 Unreal 실행하기.

  * 지정 창크기로 실행하기

    ```
    $ ./run_airsim_image_binary.sh airsim_binary:10.0-devel-ubuntu16.04 SCENE/SCENE.sh -windowed -ResX=1080 -ResY=720
    ```
    
  * SCENE/SCENE.sh에 가능한 목록

    * Blocks/Blocks.sh
    * Neighborhood/AirSimNH.sh
    * 

  * 만약 `xauth: file /tmp/.docker.xauth does not exist`와 같은 에러가 발생하면 [링크](https://github.com/lbeaucourt/Object-detection/issues/7)를 참고하여 해결한다.



## ROS 환경 구성

* 참고사이트: [ROS docker #1](http://wiki.ros.org/docker/Tutorials/Docker) | [ROS docker #2](https://hub.docker.com/_/ros) | [ROS docker #3]([https://medium.com/@rookiecj/%EA%B0%91%EC%9E%90%EA%B8%B0-ros-%EA%B7%B8%EB%A6%AC%EA%B3%A0-docker%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD-5b941c9ff098](https://medium.com/@rookiecj/갑자기-ros-그리고-docker개발환경-5b941c9ff098)) | [ROS kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) | [AirSim w/ ROS](https://github.com/microsoft/AirSim/blob/master/docs/ros.md)

* ROS kinetic 설치하기.

  * ROS kinetic (for Ubuntu 16.04) docker image 다운로드하기.
  
    ```
    $ docker pull osrf/ros:kinetic-desktop-full
    ```
    
  * docker container 생성 및 내부 터미널 명령이 가능하도록 실행하기.
  
    ```
    $ docker run -it osrf/ros:kinetic-desktop-full
    ```
    
  * 필수 패키지 설치하기.
  
    ```
    $ apt-get update
    $ apt-get install -y tmux curl wget vim sudo libgl1-mesa-glx libgl1-mesa-dri \
        mesa-utils unzip locales ntp whois
    ```
  
  * 기존 docker에 있던 기본 ros 환경 삭제하기.
  
    ```
    $ apt-get autoremove --purge --yes
    $ rm -rf /var/lib/apt/lists/*
    $ rm -rf /etc/ros/rosdep/sources.list.d/20-default.list
    ```
    
  * rosdep 초기화하기.
  
    ```
    $ rosdep init
    $ rosdep update
    ```
    
  * ROS 관련 패키지 설치하기.
  
    ```
    $ apt install python-rosinstall python-rosinstall-generator python-wstool build-essential
    ```

* ROS 작업공간 생성하기.

  * catkin 작업공간 만들기.

    ```
    $ mkdir -p /catkin_ws/src
    $ cd /catkin_ws/
    $ catkin_make
    $ source devel/setup.bash
    $ echo $ROS_PACKAGE_PATH/catkin_ws/src:/opt/ros/kinetic/share
    ```

    




## Jetson TX2 환경 구성

* Jetpack 4.3 [다운로드](https://developer.nvidia.com/embedded/jetpack)하기.