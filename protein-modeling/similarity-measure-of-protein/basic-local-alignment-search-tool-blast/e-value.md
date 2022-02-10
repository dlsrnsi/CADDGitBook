# e-value

## Alignment Score

![](../../../.gitbook/assets/smith-waterman-algorithm-example-en.gif)

두 개의 단백질 시퀀스를 alignment하는 작업을 통해 Score $$s$$를 구하는 작업을 생각해보자.\
일반적으로는 local alignment 알고리즘인 [Smith-Waterman 알고리즘](../smith-waterman-algorithm-sw.md)을 사용할 것이다.\
Random한 두개의 sequence를 alignment한다고 했을 때, SW알고리즘에서는 mismatch와 gap penalty에 대해서 -값을 주므로 거의 대부분 Score $$s$$에 대해서는 음수의 값을 줄 수 있을것이라고 생각할 수 있다.\
물론 Homologous한 두 시퀀스에 대해선 일치하는 부분이 존재하기 때문에 높은 $$s$$를 줄 것이다.

## Gumbel Distribution

Alignment score $$s$$에 대해서 생각해보자. $$s_i$$는 쿼리 시퀀스와 DB내의 시퀀스 $$i$$와의 Smith-Waterman alignment score이다.

SW 알고리즘 이 주는 스코어 값은 Alignment 스코어 중 최대값(maximum value)를 주게 된다.

일반적으로 Maximum value에 대한 분포로 검벨 분포([Gumbel distribution](https://en.wikipedia.org/wiki/Gumbel\_distribution))을 사용한다.\
검벨 분포의 PDF는 다음과 같다.

$$
\frac {1} {\beta}e^{-(x-\mu)/\beta+e^{-(x-\mu)/\beta}}
$$

$$\frac {1} {\beta} e^{-(x-\mu)/\beta}$$는 쉽게 볼 수 있는 정규분포꼴인데, 지수로 똑같은 $$\frac {1} {\beta} e^{-(x-\mu)/\beta}$$가 하나더 올라가있는 모습이다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Gumbel-Density.svg/1920px-Gumbel-Density.svg.png)

검벨 분포의 PDF를 그려보면 다음과 같은데 $$μ$$는 최빈값(mode)의 위치(location)를 $$β$$는 분포의 shape(scale)을 결정해주는 것을 볼 수 있으며, maximum값에 대한 분포 답게 오른꼬리분포임을 확인할 수 있다.

검벨분포의 CDF는 다음과 같다.

$$
e^{-e^{-(x-\mu)/\beta}}
$$

그렇다면 Alignment score $$s$$가 특정 값 $$S$$보다 클 확률 ($$S$$가 significance를 결정해줄 수 있는 특정 유의값이라고 생각하면 유의확률(p-value)이라고 생각할 수도 있다.)을 다음과 같이 생각할 수 있다.

$$
Pr(s\geq S)=1-e^{-t(s)}
$$

$$t(x)$$는 굼벨 분포에서 extreme value를 모델링하는 함수라고 생각하면 된다. (위의 CDF에서 $$t(x)=\frac {1} {\beta} e^{-(x-\mu)/\beta}$$)

검벨 분포의 평균은 $$μ+βγ$$이다. 여기서 $$γ$$는 [오일러-마스케로니 상수](https://en.wikipedia.org/wiki/Euler%E2%80%93Mascheroni\_constant)이다. 그리고 최빈값은 $$μ$$이다.

## Assessing the Statistical Significance

위와 같은 검벨 분포의 속성들을 활용해, 보수적으로 Score의 Significance를 측정할 수 있게 검벨분포를 디자인 해보도록 하자.

$$n$$개의 시퀀스를 가진 DB에 길이 $$m$$을 가진 쿼리 시퀀스를 alignment 했을 때, $$s$$보다 큰 값을 가진 alignment 결과 개수에 대한 예측값 Expected-value (E-value)를 근사적으로 모델링 하자.\
먼저 $$s$$가 크면 클수록 E-value값은 exponential하게 감소한다. Scaling factor $$λ$$를 넣어서 모델링하면 다음과 같이 쓸 수 있다.

$$
e^{-\lambda s}
$$

반면, 비교하는 쿼리 시퀀스의 길이 $$m$$가 증가할수록 Alignment score 값이 커질수 있다. 즉, $$m$$은 Expected-value와 정비례 한다. 또한, DB내의 비교시퀀스의 갯수 $$n$$은 Expected-value와 정비례 한다.

$$
mne^{-\lambda s}
$$

마지막으로 실제 데이터베이스에서 Score들을 구할 때, 실제값들의 분포로 보정해주는 $$K$$값을 곱해준다.

$$
E = Kmne^{-\lambda s}
$$

위의 식은 다음과 같이 표현할 수 있다.

$$
E=e^{-\lambda s + ln(Kmn)}=e^{- \lambda (s- \frac {ln(kmn)} {\lambda})}
$$

즉 해당 모델링은 location은 $$\frac {ln(Kmn)} {\lambda}$$ 이고 scale은 $$\frac {1} {\lambda}$$인 굼벨 분포로 모델링 할 수 있을을 확인 할 수 있다.\
따라서, 해당 스코어의 유의확률은 다음과 같이 구할 수 있다.

$$
Pr(x\geq s)=1-e^{-E}
$$

위에 언급하였듯, Alignment Score의 경우에는 쿼리 시퀀스의 길이에 따라 가변적이므로, $$E$$값을 통하여 해당 Alignment의 유의성을 확인할 수 있다.\
다만, 실제 E-value를 위하여 DB bias K와 scaling factor $$λ$$를 피팅하여야 한다.\
즉, $$E=10$$**는 내가 어떤 Alignment score** $$s$$**를 결과로 얻었을때, 길이** $$m$$**의 길이를 가진 쿼리 시퀀스를** $$n$$**개의 시퀀스를 가진 DB내에서 랜덤으로 alignment score를 구해보면 해당** $$s$$**보다 같거나 큰 케이스가 10건이 존재할 것을 의미**한다.

추가적으로 E-value의 특이점은 E-value가 충분히 작을 경우 p-value와 같이 사용 될 수 있다. ($$λ$$가 충분히 작을 때, 포아송분포의 p-value와 근사하게 된다.)

| E-value | p-value               |
| ------- | --------------------- |
| 10      | 0.999985460           |
| 5       | 0.99326205            |
| 2       | 0.86466472            |
| 1       | 0.63212056            |
| 0.1     | 0.09516258 (대략 0.1)   |
| 0.05    | 0.04877058 (대략 0.05)  |
| 0.001   | 0.00099950 (대략 0.001) |
| 0.0001  | 0.0001000             |
