# Simplified molecular-input line-entry system \(SMILES\)

## Introduction

SMILES \(Simplified molecular-input line-entry system\)는 화합물을 String으로 표현하는 방법이다. 화합물을 표현하는데 가장 일반적으로 쓰인다.

예를 들면,

![](../.gitbook/assets/2880px-thiamin.svg.png)

비타민 B1은  `OCCc1c(C)[n+](cs1)Cc2cnc(C)nc2N`로 표시될 수 있다.

SMILES는 기본적으로 화합물을 그래프로 인식하고 그래프안의 모든 노드들을 [Depth First Search \(DFS\)](https://en.wikipedia.org/wiki/Depth-first_search)를 이용하여 탐색하면서 문자열을 생성하는 알고리즘이라고 생각하면 된다. 컴퓨터 공학과 학생이라면 그래프를 탐색하면서 문자열으로 만드는 과정은 잘 알고 있을테니 String 생성 관련해서는 설명은 생략하겠다.

그런데, 이렇게 문자열을 시작 노드를 무엇으로 정하는 가와 Branch가 생길 때 어느 쪽을 먼저 선택하는지에 따라서 여러 문자열이 생성될 수 있기때문에, 하나의 화합물이 여러 SMILES로 표현될 수 있다.

예를 들면, 에탄올은

* CCO
* OCC
* C\(O\)C

로 표현될 수 있다.

### Atoms

SMILES에서  Atom 하나 하나하나는 알파벳으로 표현된다.

이 때, 수소원자가 추가되면 꺽쇠 괄호가 추가되고 Charge를 추가한다. 예를 들면 암모니움 같은 경우 로 `[NH4+]`로 표기한다.

### Bonds

결합은 `. - = # $ : / \`등의 심볼로 표기된다.

일반적인 결합은 생략되고, 이중 결합, 삼중 결합, 사중결합은 `=, #, $` 로 표기된다.

따라서, 이산화탄소는 `O=C=O`로 표현된다.

 또한 결합이 없는 특수 상태가 있을 수 있는데 이는 `.`로 표현된다.

 액체안의 소금은 다음과 같이 표현된다 `[Na+].[Cl-]`

  `/ \`은 이성질체를 표현하기 위해 사용된다.

### Rings

그래프 상에서 DFS를 수행하려면 그래프 내부에 사이클이 없어야 한다. 고리 구조를 탐색 할 때 맨처음 진입한 노드를 기준으로 고리를 부순다. 그 뒤 숫자로 쓰여진 open label \(1부터 시작\)을 붙인다. 고리의 모든 원자를 탐색하면 같은 숫자를 가진 close label을 붙인다.

일반적인 육각 고리는  `C1CCCCC1`으로 표현될 수 있다.

또한, 하나의 고리를 탐색하는 중, 또 다른 고리를 탐색할 경우도 있는데, 이때는 기존 open label보다 하나 더큰 숫자를 붙여서 open-close를 표기해준다.  [decalin](https://en.wikipedia.org/wiki/Decalin)은 `C1CCCC2C1CCCC2`로 표현할 수 있다.

### Aromaticity

 Aromatic ring은 다음과 같은 방법으로 표현할 수 있다.

1.  소문자로 적는다. `c1ccccc1`
2.  일중결합과 이중 결합을 섞어서 사용한다. `C1=CC=CC=C1`
3.  `:`을 사용한다. \(거의 사용하지 않음\) `C:1:C:C:C:C:C1`

### Branching

DFS로 화합물을 탐색하다보면 필연적으로 Branch가 발생하게 되는데, 자식 가지에 진입할때 `(`를 열어서 브랜치를 표기해주고, 모두 탐색했을 때 `)`를 이용해서 브랜치를 빠져나가고 다음 자식 가지를 탐색한다.

## Canonicalization

위에서 언급했듯, 시작노드와 브랜치 선택에 따라서 하나의 화합물에 대해서 수많은 SMILES가 만들어 질 수 있다.  이런 1-to-N표현 문제를 해결하기 위해서 Canonicalization 을 하게 된다 \[2\].

Canonicalization을 하기 위해선

먼저, 다음 속성들을 이용하여 모든 원자에 대해서 Atom invariants를 구한다.

1. 연결된 아톰 갯수
2. Non-hydrogen 본드 갯수
3. Atom number
4. 전하 +/-
5. 전하값 \(절대값\)
6. 붙은 수소원자 개수

구해진 숫자를 붙여서 invariants를 만든다.

그렇다면, 화합물 모든 원자들이 숫자로 표시될 것이다.

이것을 기반으로 오름차순 랭크를 구한다.

`CCCCC`의 경우, invariants는

10106003-20206002-20206002-20206002-1006003

이 되고

이를 기반으로 랭크를 구하면

1-2-2-2-1

이 된다.

이 랭크를 기반으로 각각 아톰마다 그래프상 1보다 작거나 같은 거리에 있는 랭크중 가장 작은 랭크를 더해준다. 그러면 다음과 같은 랭크가 된다.

2-3-4-3-2

{% hint style="info" %}
첫 번째와 마지막 원자의 랭크는 자기 자신의 랭크\(거리 0\)을 더한것이다
{% endhint %}

이 랭크를, 다시 줄이면 1-2-3-2-1 이 된다.

이와 같은 과정을 원자들의 랭크가 더 이상 변하지 않을 때까지 반복한다.

이렇게 랭크를 구하고, SMILES 문자열을 만들 때, 시작노드는 가장 랭크가 작은 노드로부터, 가지를 선택할때도 가지 중 가장 낮은 랭크를 가진 가지를 선택하도록 탐색을 수행한다.

결과적으로, 하나의 화합물에 대해서 Canonicalization을 통해서 유니크한 SMILES를 얻을 수 있게 된다.

## References

1. Weininger, David. "[SMILES, a chemical language and information system. 1. Introduction to methodology and encoding rules.](https://pubs.acs.org/doi/abs/10.1021/ci00057a005)" Journal of chemical information and computer sciences 28.1 \(1988\): 31-36.
2. Weininger, David, Arthur Weininger, and Joseph L. Weininger. "[SMILES. 2. Algorithm for generation of unique SMILES notation.](https://pubs.acs.org/doi/10.1021/ci00062a008)" Journal of chemical information and computer sciences 29.2 \(1989\): 97-101.

