# 8장 : 제어 구조

## 📌8.2 선택 구문

선택 구문이란 프로그램이 실행 경로를 선택하는 것

➡️**two-way : if then else**

ex) if else문 (고려해야할 점)

- **제어 표현식의 형태와 타입**
- 절의 형태
- 중첩 선택문의 의미

**선택문이 다수 중첩될 때 어떤 if문에 속하는지 모호해지는 danglin else 문제 발생**

➡️mutiple-section : 다중 선택문

ex) switch문 (고려해야할 점)

- 제어 표현식의 형태와 타입
- 선택 가능한 세그먼트 지정 (switch, elif)
- 세그먼트 형태
- 선택 후 제어 전달

switch문은 **값 비교에 유용, 복잡한 조건 선택에는 한계** 존재

if-elif-else 구문과 같은 구조는 조건 기반 **다중 선택을 간결하고 명확히함. (깊게 중첩되면 가독성 저하)**

## 📌8.3 반복문

➡️계수기 제어 루프, 루프 변수를 이용하여 반복 횟수 제어

- 초기값
- 종료값
- 증가 크기

위 세 조건을 이용해 사용(꼭 모두 사용할 필요는 없음)

➡️논리 제어 루프, 반복 횟수가 아니라 논리적 참/거짓을 기준으로 반복여부 결정

- 사전 검사 루프 : while (구문을 실행하기 전에 사전 검사를 거치고 구문 루프 시작)
- 사후 검사 루프 : do-while (구문을 한 번 수행한 후에 검사를 거침)

➡️사용자 지정 루프. 루프 제어 위치를 사용자가 직접 지정하는 방식

중첩된 루프에서 탈출 가능(break, continue 등)

## 📌8.4 - 8.6 무조건 분기, 보호 명령

프로그램의 지정된 위치로 강제로 실행 제어 전달 (goto문)

goto는 구조적, 순서적 프로그래밍 원칙을 깰 수 있음

**보호명령 : 다익스트라가 제안한 선택 루프 구조 << 중요**

프로그램이 모두 작성된 후 검증하는 게 아닌, **개발 도중 정확성 보장**을 위해서

**기존의 if-else 문 : 참이 되는 조건 중 첫번째 문장만 실행**

**보호명령** : 참이되는 조건이 여러개라면 **비결정적으로 하나만 선택**

**if i=0 → sum :=sum +1
[] i >j → sum := sum+2
[] j > i → sum := sum+3
fi**

경우1 : i=0, j>0인경우 → **조건이 두 문장에 대해 비결정적으로 실행 (1, 3번 중 하나 실행)**

경우2 : i=j이고 i가 0이 아닌 경우 → **모두 만족하지 않으므로 실행 보장 안 됨(문제 발생)**

- **보호 명령의 문제점 (비결정성 관련)**

> 조건이 여러 개 참이면 **어떤 문장이 실행될지 알 수 없음(예측 불가**), **조건에 만족하지 않으면 아무것도 실행되지 않아 보호명령을 사용하는 의미가 사라짐**
>
>
> → 오류가 있어도 **“피해서 실행”되면 드러나지 않음 (비결정적으로 하나만 실행하기 때문)**
>
> → 해결 방법: 모델 검사(model checking)로 경로 전체 탐색 <<
>