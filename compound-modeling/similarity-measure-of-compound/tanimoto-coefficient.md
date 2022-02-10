# Tanimoto Coefficient

Tanimoto coefficient (similarity)는 화합물이 이진벡터(Binary vector)로 표현될 때, 화합물간의 유사도 계수이다.

Tanimoto coefficient는 다음과 같이 정의 된다.

$$
S_T(A, B) = \frac {A\cap B} {A \cup B}
$$

핑거프린트에서 각각 인덱스는 substructure를 의미하기 때문에,  타니모토 유사도의 분자는 두 화합물이 가지고있는 공통구조를 의미한다. 즉 타니모토 유사도는 공통되는 하부구조를 전체 하부구조로 나누어준 값이다. &#x20;

혹은 [jaccard index](https://en.wikipedia.org/wiki/Jaccard\_index)의 화합물 하부구조 버전으로 생각할 수 있다.
