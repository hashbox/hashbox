---
layout: post
current: post
cover: False
navigation: True
title: SSH환경에서 X11 Forwarding을 이용한 Matplotlib 사용
date: 2018-11-15 28:10:00
tags: develop solution
class: post-template
subclass: 'post tag-develop'
logo: assets/images/slaysd.png
author: slaysd
---

![출처 : https://uechi.io/blog/x11forward](https://uechi.io/uploads/x11-plot.png)

> `Matplotlib.pyplot`을 이용하여 plot을 사용을 자주 하는데, SSH GPU서버에 원격으로 붙어 사용할 경우 `plt.show()`을 이용하여 볼 수 있는 방법이 없을까 싶어 찾은 해결방법입니다.

Python을 사용하다보면 `Matplotlib.pyplot` 라이브러리를 이용하여 생성된 이미지나 그래프를 확인하는 경우가 생각보다 많습니다. 로컬 환경에서 이를 사용할 경우에는 크게 문제가 없는 반면에 저와 같이 GPU서버 또는 외부 서버에 SSH를 이용하여 원격으로 서버를 사용하는 경우 이미지나 그래프를 확인하기 위해 폴더에 저장하여 보곤 했습니다. 원격으로 `plt.show()`와 같은 명령어를 통해 이미지를 보고 싶은 경우에 이 글이 해결책이 될것이라 생각됩니다.

# on macOS
macOS에서는 다음과 같은 처리가 필요합니다.

## X11 Forwarding
우선적으로 원격에서 생성되는 GUI를 로컬머신에서도 지원할 수 있도록 도움을 주는 프로그램인 X11-Forwarding이 설치되어있어야합니다. macOS에서 이를 사용하기 위해서는 X11-Forwarding 응용프로그램인 [XQuartz](https://www.xquartz.org)이 설치되어 있어야합니다. macOS에서 원래 기본적으로 설치되어있던 응용프로그램이었지만 Lion으로 넘어오면서 필요한 경우 직접 설치해야합니다. 설치를 위해서는 홈페이지를 통해 직접 설치하시거나 `brew`를 이용하여 설치가 가능합니다. 현재 버전으로 2.7.11으로 macOS Mojave에서도 문제 없이 작동됨을 확인하였습니다.

### brew
``` bash
brew cask install xquartz
```

# on Remote server (Ubuntu 16.04)
원격서버에서는 다음과 같은 처리가 필요하며, 필자가 사용한 버전은 Ubuntu 16.04입니다.

``` bash
sudo apt install -y xorg xauth openssh
sudo sed -i '/ForwardX11/s/.*/ForwardX11 yes/' /etc/ssh/sshd_config
sudo service ssh restart
```

## matplotlib 설정
X11-Forwarding을 통해 `matplotlib`을 사용하기 위해서는 설정을 바꿔주어야 합니다. 이 설정을 바꾸기 위해서는 두가지 방법으로 전역적으로 기본 설정값을 바꾸는 방법과 해당 코드내에서만 바꾸는 방법으로 원하시는 방법을 이용하여 사용하시면 됩니다.

### 기본 설정값 변경
`~/config/matplotlib/matplotlibrc`
``` vim
backend: TkAgg
```

### 코드내 변경
``` python
import matplotlib
matplotlib.use('TkAgg')
import matplotlib.pyplot as plt
...
```

# 사용
이제 모든 설정이 완료되었습니다. 설정이 완료되신 후 `XQuartz`를 설치함으로써 환경변수들이 적용될 수 있도록 macOS는 재부팅이 필요합니다. 재부팅 후 터미널에서 `echo $DISPLAY`를 통해 제대로된 값이 적용됬는지 확인이 가능합니다.

``` bash
ssh -X username@hostname
```
X11-Forwarding을 사용하는 SSH 접속을 위해서는 `-X`옵션을 꼭 붙여주셔야 합니다. 자 이제 원격에서도 GUI가 사용한 환경이 모두 갖춰졌습니다!
