# 지능형 드론 개발 환경 구축 방법을 알아 봅시다.

## 기본 OS 및 환경 구성
* Ubuntu 18.04 LTS 설치하기.
* GPU 드라이버 설치하기.
  ```
  sudo add-apt-repository ppa:graphics-drivers/ppa
  sudo apt-get update
  sudo apt-get install nvidia-driver-410
  ```
* [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 설치하기.
  - 도커 컨테이너 및 이미지 저장 디렉토리 수정 필요함. [[참고사이트](http://dveamer.github.io/backend/DockerImageDirectory.html)]
    - Docker 프로세스 중지
      ```
      sudo service docker stop
      ```
    - Docker 이미지나 컨테이너를 저장할 폴더 생성
      ```
      mkdir -p /home/USER/DOCKERFOLDER
      ```
    - Docker 환경정보 파일 열기
      ```
      sudo gedit /etc/default/docker
      ```
    - Docker_OPT 변경
      ```
      DOCKER_OPTS="-g /home/USER/DOCKERFOLDER"
      ```
    - Docker 프로세스 시작
      ```
      sudo service docker start
      ```
* [nvidia-docker2](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)) 설치하기.
  - Docker 이미지, 컨테이너 설치 폴더 변경 [[참고 사이트](https://docs.docker.com/config/daemon/systemd/#start-automatically-at-system-boot)]
    * Docker 서비스 중지 및 폴더 생성 (디렉토리 생성은 앞서 했으므로 무시)
      ```
      sudo serviece docker stop
      mkdir -p /home/USER/DOCKERFOLDER
      ```
    * Docker daemon 설정 파일 생성(또는 수정)
      ```
      sudo gedit /etc/docker/daemon.json
      ```
      * 아래 구문 추가
      ```
      {
        "data-root": "/home/USER/DOCKERFOLDER",
        "storage-driver": "overlay",
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
  * [NVIDIA GPU CLOUD](https://ngc.nvidia.com/registry/)에 로그인
    * 가입 후 개인 API Key를 얻어야 각종 도커 이미지를 다운로드할 수 있다.
    * 해당 컴퓨터로 1회 해놓으면 향후 별도 로그인은 불필요하다.
    * 터미널창 명령
      ```
      sudo docker login nvcr.io
      Username: $oauthtoken
      Password: NGC에 가입하면 얻을 수 있는 API Key 입력
      ```
* [ue4-docker](https://adamrehn.com/docs/ue4-docker/configuration/configuring-linux) 
