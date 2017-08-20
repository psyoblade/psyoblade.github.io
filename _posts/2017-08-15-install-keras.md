---
layout: post
title:  Installing openai gym
date:   2017-08-15 00:15:22 +0900
categories: psyoblade update
---
## openai gym 설치하기

### 설치
* Anaconda 통해서 rl35 가상환경 생성 및 필수 패키지 설치
```bash
git clone https://github.com/rlcode/reinforcement-learning-kr.git
cd reinforcement-learning-kr
source activate rl35
pip install requirements.txt
```
* OpenAI Gym 패키지 설치
```bash
git clone https://github.com/openai/gym.git
cd gym

brew install zlib cmake boost boost-python sdl2 swig wget
pip install -e '.[all]'

export MACOSX_DEPLOYMENT_TARGET=10.12
pip install -e '.[all]'
...
Successfully installed Box2D-kengz-2.3.3 Pillow-4.2.1 atari-py-0.1.1 gym imageio-2.2.0 keras-2.0.6 mujoco-py-0.5.7 olefile-0.44 pachi-py-0.0.21 pyyaml-3.12 scipy-0.19.1 theano-0.9.0

```

