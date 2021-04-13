---
description: Introduction
---

# Chemogenomic Methods

Chemogenomic method는 약물과 타겟 단백질을 특정 공간에 배치시킨 후, 이를 이용해서 상호작용을 예측하는 방법들을 말한다.

이러한 방법론에서는 약물과 단백질은 특정 feature으로 표현되며, 이 feature을 Pharmacological space로 병합하여 약물-표적 상호작용을 예측하게 된다.

$$
DTI_{d_i,t_j}=f_p\big(f_c(d_i),f_g(t_j)\big)
$$

수식으로는 다음과 같이 표현될 수 있으며, 논문마다 각자 다른 약물, 단백질, 상호작용 표현 방법들을 사용할 수 있다.

그림으로 표현하면 다음과 같다.

![&#xCD9C;&#xCC98;: https://academic.oup.com/bioinformatics/article/24/13/i232/231871](../../.gitbook/assets/pharmacological_space-1024x456.jpeg)

약물과 단백질이 각각 feature로 변환이 되어 각각 chemical space와 genomic space상에 존재하게 되고, 이를 pharmacological space상으로 매핑함수 \( $$f_c, f_g$$\) 를 이용하여 Pharmacological space상의 DTI쌍으로 표현하는 것을 보여주고 있다. __이렇게 Pharmacological space로 매핑한 후에는 여러 분류기법 \(RF, SVM, KNN 등등\)을 이용하여 상호작용하는 약물-단백질쌍을 학습하고 예측할 수 있게 된다.



## 참조논문

1. Yamanishi, Yoshihiro, et al. "[Prediction of drug–target interaction networks from the integration of chemical and genomic spaces.](https://academic.oup.com/bioinformatics/article/24/13/i232/231871)" Bioinformatics 24.13 \(2008\): i232-i240.

