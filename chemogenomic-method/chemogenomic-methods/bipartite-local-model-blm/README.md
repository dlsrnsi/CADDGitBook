# Bipartite Local Model \(BLM\)

전 문서인, [Chemogenomic Methods](../)에서는 Pharmacological space가 굉장이 복잡\(coarse\)한 hyperplane을 형성할 수 있고, 이로 인해서 예측모델의 성능이 하락할 수 있음을 지적한바 있다.

이러한 문제점을 해결하기 위해서 고안된 모델이 Bipartite Local Model \(BLM\)이다. BLM에서는 DTI 쌍을 Bipartite 그래프로 표현하고, 이에 bipartite 그래프라는 구조에 좀더 중점을 준다.

먼저 Bipartite graph에 대해서 알아보자. 

## Bipartite Graph

Bipartite 그래프는 서로 다른 두 그룹 U,V가 있고 U와 V사이에서 edge만이 존재하는 그래프들을 의미한다. 즉 U와 V그룹 내부의 엣지는 존재하지 않는다.

![&#xCD9C;&#xCC98;: https://en.wikipedia.org/wiki/Bipartite\_graph ](../../../.gitbook/assets/simple-bipartite-graph.svg.png)

아래와 같은 그림에서 볼 수 있듯이, 약물-단백질 상호작용은 약물 그룹과 단백질 그룹의 Bipartite 그래프로 표현될 수 있다 \[1\].  

![&#xCD9C;&#xCC98;: https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0230726](../../../.gitbook/assets/pone.0230726.g001.png_m.png)

이렇게 약물-단백질 상호작용이 Bipartite 그래프로 표현될 경우에 좋은 점은, 한 그룹내의 노드 입장에서 같은 그룹의 다른 노드를 생각할 필요가 없고\(엣지가 없기 때문에\), 상대 그룹만 신경쓰면 된다는 것이다. 약물-표적 단백질 상호작용 입장에서는 단백질 관점에서는 약물 쪽만 보면 되고, 약물 입장에서는 단백질 쪽만 보면 된다.

## Bipartite Local Model \(BML\)

\_\_[_Bleakley et al._](https://academic.oup.com/bioinformatics/article/25/18/2397/197654) __\[2\] __ 에서는 이러한 Bipartite 그래프의 특성을 이용하여 Bipartite local model \(BML\)을 만들었다. 위에서 언급하였듯, Bipartite 그래프의 장점은 한 노드 관점에서 상대 그룹만을 신경쓰면 된다는 것이다.

BLM에서는 각각 약물, 그리고 단백질 하나하나 마다 상대 그룹에 대한 분류 모델을 만든다.  
예를 들면, 단백질 a에 대해서 반대 그룹인 약물 그룹들의 약물들을 feature로 표현하고, 이 feature들을 이용해서 단백질 a와 상호작용 알려진 약물들에 대해서는 1로 예측하도록, 아닌 약물들은 -1로 예측하도록 SVM을 학습한다.

그렇다면, 단백질 하나 입장에서 약물 feature로만 분류하게 되므로 분류 공간이 훨씬 덜 복잡하게 만들어지게 된다.

![&#xC57D;&#xBB3C; feature&#xB85C;&#xB9CC; &#xB9CC;&#xB4E4;&#xC5B4;&#xC9C4; &#xBD84;&#xB958; &#xACF5;&#xAC04;&#xC758; &#xC608;. &#xCD9C;&#xCC98;: https://www.petersincak.com/news/why-i-do-not-believe-in-error-backpropagation/ ](../../../.gitbook/assets/uno-research.jpg)

이렇게 하나의 단백질, 혹은 약물의 관점에서 분류모델을 만들기 때문에 Local model 이라고 부를 수 있고, Bipartite 그래프 관점을 적용하였기 때문에, Bipartite Local Model이라고 부르게 된다.

이 때, 약물과 단백질 feature는 유사도 벡터를 사용한다. 즉 특정 단백질 a에 레이블이 1인 약물에 대해서 비슷한 약물은 예측시 스코어가 1에 더 가깝게 예측 될 것이다.

또한 약물-표적 상호작용은 bipartite 그래프의 관점에서 양방향으로 볼 수 있으므로, 약물-표적 상호작용을 예측하는데 약물쪽에서 예측한 것과 단백질 쪽에서 예측한 스코어를 종합하여 예측하였다.

## BLM의 중요성과 한계점.

BLM은 각각 단백질과 약물마다 모델을 만들면서도\(local\), 기존의 Pharmacological model 보다 좋은 성능을 보여준다. 또한 local model을 만들기 때문에 예측속도가 타 모델들\(Pharmacological space, Matrix factorization\)보다 학습 및 예측이 훨씬 빠르다는 장점이 있다. 따라서 [_Ezzat et al._](https://academic.oup.com/bib/article/20/4/1337/4824712) \[3\] 같은 논문에서는 Matrix factorization 모델을 돌리기 전, 선행적으로 BLM 모델들로 예측으로 수행하는 것을 추천하고 있다.

하지만 BLM도 한계점을 가지고 있는데, BLM은 한쪽 그룹의 노드 관점에서 상대 그룹을 평가하는 것이기 때문에, 양쪽 그룹 모두에 없는 약물-단백질 쌍에 대해서는 예측을 할수가 없다. 약물, 혹은 단백질 쪽에서도 feature를 생성하여 학습을 진행할 수 없기 때문이다.

반면 Pharmacological model은 성능이 떨어질지언정 약물과 단백질이 모두 학습데이터에 없어도 예측할 수 있다 \(단백질 및 약물 feature가 함께 pharmacological space로 매핑되기 때문에\).

## BLM 기반의 모델들.

어찌되었든, BLM은 약물-표적 상호작용 예측모델에 있어서 중요한 마일스톤 논문임에는 틀림 없다. BLM이후로 BLM의 약점들을 보완한 모델들이 많이 나왔다. BLM을 기반으로 한 모델이 꽤 많음으로, 대표적인 모델 몇 개만 소개하도록 한다.

* [LaplRLS \(2010\)](laprls-and-netlaprls.md)
* [Gaussian Interaction Profile \(GIP, 2011\)](gaussian-interaction-profile-gip.md)

## Reference

1. Buza, Krisztian, Ladislav Peška, and Júlia Koller. "[Modified linear regression predicts drug-target interactions accurately.](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0230726)" PloS one 15.4 \(2020\): e0230726.
2. Bleakley, Kevin, and Yoshihiro Yamanishi. "[Supervised prediction of drug–target interactions using bipartite local models.](https://academic.oup.com/bioinformatics/article/25/18/2397/197654)" Bioinformatics 25.18 \(2009\): 2397-2403.
3. Ezzat, Ali, et al. "[Computational prediction of drug–target interactions using chemogenomic approaches: an empirical survey.](https://academic.oup.com/bib/article/20/4/1337/4824712)" Briefings in bioinformatics 20.4 \(2019\): 1337-1357.



