---
layout: post
title:  "Cost Function vs Distance Function"
date:   2017-07-30 11:16:22 +0900
categories: psyoblade update
---
## 

검색엔진 혹은 고전적인 기계학습 영역에서의 거리함수distance function은 상당히 많은 부분에서 사용되는데, 예를 들면 clustering 에서 각 클러스터간의 유사도를 계산한다거나, 검색 시에 검색 키워드와 문서의 유사도 계산 시에 문서와 문서의 유사도relevance rank를 계산하는 데에도 거리함수를 사용하게 된다.
맨하탄manhattan distance, 유클리디안euclidean distance, 마할라노비스mahalanobis distance 등이 존재하는데 어떤 거리함수를 사용하는 지에 따라서 유사도similarity 측정이 달라지고 전체적 성능에 큰 영향을 주게 된다.

딥러닝을 공부하면서 이러한 거리함수와 유사한 개념이 비용함수cost function 개념이 나오는데 비교해 보면 재미있을 것 같아 한번 생각해 보았다.
우선 거리함수는 위와 같이 두 데이터(점, 선, 면, 벡터, 하이퍼 플레인)간의 거리를 통해 유사도를 측정하는 데에 사용되는 metric이다.
반면 비용함수는 우리가 추측한 경우의 데이터(가중치 벡터, 편향 벡터)와 정답인 그것과의 차이를 측정하는 데에 사용되는 metric이다. 

결국 어떠한 형태로 표현되는 데이터와 데이터 간의 차이를 비용 혹은 거리라는 metric으로 표현하는 관점에서 본다면 비슷하다는 생각을 하게 되었다.

최근 공부한 뉴럴 네트워크의 학습에서 mnist 데이터를 통한 숫자 이미지를 학습supervised learning을 통해 새로운 숫자 이미지를 추론inference 하는 과정이 있는데,
학습 과정에서 역전파back propagation 단계에서 추측한 결과 즉, 0~9까지의 숫자일 확률 혹은 가장 높은 확률의 숫자가 정답(one hot encdoing인 경우)과 실제 정답과 차이를 비교할 때에 평균제곱오차mean squared error 를 사용하기도 하고 교체 엔트로피 오차cross entropy error 를 사용하기도 하는데 왜 뜬금 없이 이런 함수들을 사용하는 지 궁금했다.

이를 새로운 관점에서 보았을 때에 내가 추정한 초기의 값과 정답과 얼마나 먼 거리에 있는지를 계산하는 것으로 생각해 보았다.
즉, 정답 데이터와 추정 데이터 간의 유사도를 측정한다고 말이다. 이러한 관점에서 보니 왜 MSE와 CEE가 나오는지 대충 감이 왔다.

거리함수의 경우 데이터의 특성에 따라 거리함수를 달리 가져가야 좋은 성능을 보이는데, 마찬가지로 비용함수의 경우에도 그런 것 같다.
정답의 종류가 Boolean, Categorical probability 혹은 Random variables 인 지에 따라 다른 metric을 쓴다고 보면 어떨까?

boolean : 추정 확률 간의 비교이므로 단순 확률 값의 차이만으로 거리, 비용을 계산할 수 있으므로 Mean Seqaured Error 사용가능
categorical or numerical probability: 각 추정한 값들의 오차의 제곱에 대한 평균값으로 거리, 비용을 하나의 숫자로 표현할 수 있다
random variables: 

정리하다 보니 데이터의 특성이라기 보다는 다양한 방법으로 유사도를 계산하는 방법이 아닐까 하는 생각이 드는데...
* 추측한 데이터간의 차이의 제곱의 평균을 통해 기하학적인 면적의 크기를 비용으로 보는 관점과, - mean squared error
* 반면에 추측한 데이터를 random variables 로 보고, 확률분포가 유사한지를 보는 관점으로 볼 수도 있겠다.


이제와서 보니... 


질문
* mean square 는 최초에 어떤 상황에서 이러한 오류 혹은 metric이 만들어진걸까? 땅 넓이?
* cross entropy 는 또 어떤가? 어떤 히스토리가 있을까?



