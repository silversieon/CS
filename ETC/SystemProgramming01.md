## ✅기본 명령

### ➡️리눅스 접속, 해제

- telnet, ssh : 리눅스 시스템에 접속
- eixt, logout : 리눅스 시스템 접속 해제

### ➡️파일, 디렉터리

- pwd : 현재 디렉터리 경로 출력
- ls : 디렉터리 내용 출력 (-a, -l)
- cd : 디렉터리 이동
- cp : 파일 복사, 디렉터리 복사 (-r)
- mv : 파일명/디렉터리명 변경, 파일/디렉터리 이동
- rm : 파일 삭제 (-r 디렉터리 삭제)
- cat, more : 파일 내용 출력
- chmod : 파일/디렉터리 접근 권한 변경

### ➡️프로세스, 기타

- ps : 현재 실행 중인 프로세스 정보
- kill : 강제 종료
- su : 사용자 계정 변경

## ✅동적 메모리 할당

1. malloc : void *malloc(size_t size)
2. calloc : void *calloc(size_t nmemb, size_t size)
3. realloc : void *realloc(void *ptr, size_t size)