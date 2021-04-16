# Kernelized Bayesian matrix factorization with twin kernels \(KBMF2K\)

Kernelized Baysian matrix factorization with twin kernels \(KBM2K\) 는 [Pharmacological space](../pharmacologcial-space.md) 기법을 Matriax Factorization으로 해석한 논문이다.

## Pharmacological Space and Matrix Factorization

KBMBF모델의 개요는 다음과 같다.

![&#xCD9C;&#xCC98;: &#xBCF8;&#xC778;&#xC81C;&#xC791;](../../../.gitbook/assets/kbmf2k.png)

먼저, Pharmacological space 기법에서 사용한 방법으로 약물과 단백질을 유사도 커널을 사용하여 약물공간과 단백질 공간 $$K_d,K_t $$ 의 벡터로 표현한다.

그후, 약물과 단백질에 대해서 $$A_d, A_t$$ 인 매핑 함수를 사용하여 약물공간과 단백질 공간에서 Pharmacological space로 매핑한다. 매핑된 Pharmacologcial space상의 약물과 단백질을 $$G_d (d_i), G_t (t_j)$$ 라고 한다.

이후 $$G_d(d_i), G_t(t_j)$$ 간의 pharmacological space상에서의 유사도를 재는데, 이 때 내적을 사용한다. $$F(d_i, t_j)=G_d(d_i)\cdot G_t(t_j)$$ 로 표현될 수 있으며, 이 유사도를 $$\nu$$ 기반의 분류함수를 통하여 1 혹은 -1로 분류하게 된다.

위에 언급된 과정을 $$N_d,N_t$$ 개의 약물과 단백질에 대해서 한번에 수행하는 것은 Matrix muliplication으로 수행할 수 있다. 반대로 예측값을 두개의 별개의 행렬로 만드는 Matrix factorization으로 표현될 수 있다.

![Matrix multiplication&#xC758; &#xC608;&#xC2DC;. &#xCD9C;&#xCC98;: https://academic.oup.com/bib/article/20/4/1337/4824712](../../../.gitbook/assets/m_bby002f3.png)

$$
F=G_d^TG_t=(A_d^TK_d)^T(A_t^TK_t)=K_d^TA_dA_t^TK_t
$$

위의 과정을 잘 따라온 사람이라면 알겠지만, 사실 모델에 있어서 학습될 수 있는 파라미터는 매핑 함수인 $$A_d, A_t$$ 밖에 없다. KBMF2K는 두함수의 파라미터들에 대해서 베이지언 추론을 하는 방법이다.

## Model and Bayesian Inference

$$A_d,A_t$$ 의 파라미터들을 [감마 분포](https://en.wikipedia.org/wiki/Gamma_distribution)를 사전 확률 분포\(Prior distribution\)으로 가진 사후 확률 분포 \(Posterior distribution\)으로 가정한다. $$A_d,A_t$$ 는 기본적으로 $$\mathbb {N}(0,1)$$ 을 Likelihood로 가지되, 사전 확률 분포로 감마 분포를 [Conjugate piror](https://en.wikipedia.org/wiki/Conjugate_prior)로 가져서 분산의 역인 precision $$\tau$$ 를 감마 분포로부터 샘플링 하는 사후 분포가 된다. 이때 감마 분포의 하이퍼 파라미터 $$\alpha,\beta$$ 는 사용자가 정의해준다.

$$
\lambda_{d,s}^i \sim \mathbb {G}(\lambda_{d,s}^i;\alpha_{\lambda}, \beta_{\lambda}) 
\\ 
\\
a^i_{d,s}|\lambda_{d,s}^i \sim \mathbb {N}(a^i_{d,s};0, (\lambda_{d,s}^i)^{-1})
$$

단백질은 $$d\rightarrow t$$ 로 바꾸면 된다.

위의 사후 분포 $$A_d,A_t$$ 를 기반으로 $$G_d,G_t$$ 는 다음과 같이 정의 될 수 있다.

$$
g_{d,i}^s|a_{d,s},k_{d,i}\sim \mathbb{N}(g_{d,i}^i; a_{d,s}^T k_{d,i}, \sigma^2_g)
$$

$$G_d,G_t$$ 를 기반으로 $$F$$ 는 다음과 같이 정의 될 수 있다.

$$
f^i_j|g_{d,i},g_{t,j} \sim \mathbb {N}(f^i_j;g_{d,i}^Tg_{t,j}, 1)
$$

### Variational Inference \(변분 추론\)

우리의 목표는 입력값 $$k_{d,i},k_{t,j}$$ 으로부터 $$Y$$ 를 예측하는 확률값을 최대화 하는 것이다. 이 때, $$\Theta: \Lambda_d, \Lambda_t$$ \(사전 분포 세트\)이고 $$\Xi:A_d,A_t,G_d,G_t$$ \(나머지 변수들\)이다.

$$
\log {\int p(Y|K_d,K_t)}p(\Theta,\Xi)d(\Theta,\Xi)
$$

그러나, 사후확률 분포들은 너무 구하기가 힘들다. 따라서 사후 확률들은 다루기 쉬운 확률분포 $$q$$ 로 근사하게 된다.

$$
q(z)\sim p(z|x)
$$

이는, 근사 분포 $$q(z)$$와 $$p(z|x)$$ 의 KL-divergence를 최소화 하는 방식으로 진행되게 된다. 즉,$$D_{KL}(q(z)||p(z|x))$$를 최소화 하는 것이다.

