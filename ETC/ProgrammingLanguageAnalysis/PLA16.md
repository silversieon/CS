# 16장 : 논리형 프로그래밍 언어

## 1. 논리형 언어의 특징

**선언전(비절차적) 언어,** 무엇을 기술, 어떻게는 기술하지 않음

기호 논리 기반, 특히 술어 논리 사용

## 2. Prolog 문장의 3가지 구성(사실, 규칙, 목표)

1. 사실 : 참인 명제 선언 → father(bill, jake).
2. 규칙 : if-then 형태, 조건부 명제 → parent(X, Y) :- fahter(X, Y).
3. 목표 : 질의 문장 → ?- parent(bill, jake).

## 3. 호른절

- 논리 증명에 사용되는 문장 활용식
- Headed : A:- B1, B2 (조건 → 결론, A는 B1, B2일 때 성립)
- Headless :    :- B1, B2 (사실 형태, 제약을 둘 때 사용 (B1과 B2는 동시에 성립 불가))

## 4. Prolog의 추론 방식

- 후방 연쇄 사용 : 목표에서 시작 → 사실/규칙을 따라 추론
- 깊이 우선 탐색 < 백트래킹 전략 : 실패시 이전 선택 지점으로 되돌아가는 백트래킹 전략을 쓸 수 있는 깊이 우선 탐색을 통해 모든 해를 빠짐없이 탐색함

## 5. 단일화 & 인스턴스화

- 단일화 : 두 항을 같게 만들기 위해 변수와 값을 매칭하는 과정
- 인스턴스화 : 단일화 중 변수에 값을 **임시로** 바인딩하는 것
- 단일화가 실패하면 더 진행할 수 없으므로 **백트래킹**하여 다른 경로를 시도

### 6. **리스트 처리 및 재귀**

- 리스트 구조: `[Head | Tail]`
- 주요 리스트 함수 예시:

```prolog
append([], L, L).
append([H|T], L2, [H|R]) :- append(T, L2, R).

reverse([], []).
reverse([H|T], R) :- reverse(T, RT), append(RT, [H], R).

member(X, [X|_]).
member(X, [_|T]) :- member(X, T).

```

### 7. **산술 계산**

- `is` 연산자 사용: `X is Y + Z`
- 대입 아님! 오른쪽은 반드시 계산 가능한 수식이어야 함
- `X is X + 1`은 **불법!!**

### 8. **Prolog의 단점**

- **분해 순서 제어 어려움** (비결정성)
- **폐쇄 세계 가정**: DB에 없는 정보는 false로 간주
- **부정 처리 어려움**: not 표현이 직접적이지 않음
- **실행 제어 미비**: 정렬 등 절차적 처리가 어려움

```prolog
% --work1
parent(hs, sieon). %hs is parent
parent(hs, seongyeon).
parent(ym, sieon).
parent(ym, seongyeon).
parent(sj, hs).
parent(bs, hs).
parent(ph, ym).
parent(sy, ym).
female(seongyeon).
female(ym).
female(sy).
male(sieon).
male(hs).
male(ph).
mother(X,Y):-parent(X,Y), female(X).
father(X,Y):-parent(X,Y), male(X).
grandmother(X,Z):-parent(X,Y), parent(Y,Z), female(X).
grandfather(X,Z):-parent(X,Y), parent(Y,Z), male(X).
sister(X,Y):-parent(Z,X), parent(Z,Y), female(X), X\=Y.
brother(X,Y):-parent(Z,X), parent(Z,Y), male(X), X\=Y.
% --work2
life_stage(A, child):- A<13, !.
life_stage(A, teen):- A>=13, A<20, !.
life_stage(A, adult):- A>=20, A<65, !.
life_stage(A, senior):- A>=65.
% --work3
sum_list([], 0).
sum_list([Head|Tail], Sum) :-sum_list(Tail, Sum2), Sum is Head + Sum2.
```