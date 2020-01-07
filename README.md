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
* [nvidia-docker2](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)) 설치하기.
* [ue4-docker](https://adamrehn.com/docs/ue4-docker/configuration/configuring-linux) 
