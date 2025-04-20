# 4장 : 어휘 분석과 구문 분석

## 🎯서론

언어 프로세서의 구문 분석은 거의 두 부분으로 구성

1. 어휘 분석기 : 이름과 수치 리터럴 같은 작은 규모의 언어 구조를 다룸
2. 구문 분석기 : 식, 문장, 프로그램 단위 같은 큰 규모의 언어 구조를 다룸
- 두 부분으로 구분하는 이유
1. 단순성 : 어휘 분석에 덜 복잡한 접근 가능, 파서 간소화
2. 효율성 : 분리를 통해 어휘 분석기 최적화
3. 이식성 : 어휘 분석기의 일부는 이식 불가, 구문 분석기는 항상 가능

## 🎯어휘 분석(lexical analysis)

주어진 문자열에서 주어진 문자 패턴과 일치하는 부분 문자열을 찾는 패턴 매칭기이다.

parser의 이전 단계

[토큰 예시 확인]

## 🎯파싱 문제

Parser = 구문 분석기

- 모든 systax error를 찾아낸다.
- parse tree를 생성한다.
1. Top down 파싱 : root부터 시작해서 parse tree 생성(ex-재귀하강파싱)
2. Bottom up 파싱 : leaves부터 시작해서 parse tree 생성(ex-LR파싱)

[예시 참고]

## 🎯재귀-하강 파싱

각각의 논터미널이 각 subprogram을 이룬다.

EBNF가 논터미널 개수를 최소화 했기 때문에 재귀하강파서에 적합

[BNF 표현 및  예시 참고]

## 🎯상향식 파싱

1. 유도 과정 중에 나타나는 이전의 우문장 형태로 감축하는데, 적당한 핸들을 찾는다
2. 핸들에 해당하는 RHS를 rule에서 찾아 논터미널로 대체한다.
- Shift-Reduce Algorithms
- shift : next token을 parse stack의 top에 넣는 action
- reduce : parse stack의 top에 있는 handle을 해당하는 LHS로 대체하는 action

[예시 참고]