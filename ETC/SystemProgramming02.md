## ✅개요

리눅스에서는 디렉터리도 파일의 한 종류로 취급.

1. 일반 파일 : 텍스트 파일, 실행 파일 등 바이너리 형태의 파일
2. 특수 파일 : **💡장치 관련 특수 파일(데이터 블록 사용x)** **대신 장치 번호를 inode에 저장**
    - 블록 장치 파일 : 블록 단위로 데이터 읽고 씀
    - 문자 장치 파일 : 섹터 단위로 읽고 씀

   ls -l 시에 일반 파일과 달리 10, 175 형태로 파일의 크기가 출력되는데

   10은 장치 종류, 175는 장치의 개체 번호

3. 디렉터리 : 해당 디렉터리에 속한 파일을 관리하는 파일

파일 : 파일명, inode, 데이터 블록으로 구성

파일명 : 사용자가 파일에 접근할 때 사용

inode : 외부적으로 번호로 표시, 데이터 블록의 위치를 나타내는 주소들 저장

데이터 블록 : 실제 데이터가 저장되는 하드디스크 공간

### ➡️inode

1. 파일명 → inode 번호
2. 파일 정보 → 파일 종류, 접근 권한, 크기, 소유자, 소유 그룹 등
3. 데이터 블록 위치 → 데이터 블록 주소(→데이터 블록)

## ✅디렉터리 기본 명령어

### ➡️디렉터리 생성 : mkdir(2)

함수 원형 : int mkdir(const char *pathname, mode_t mode);

예시 :

```c
#include <sys/stat.h>

if(mkdir("han", 0755) == -1)
//성공시 0, 실패시 -1 리턴
//7(소유자 권한) 5(그룹 권한) 5(기타 권한)
```

### ➡️디렉터리 삭제 : rmdir(2)

함수 원형 : int rmdir(const char *pathname);

예시 :

```c
#include <unistd.h>

if(rmdir("han") == -1)
//성공시 0, 실패시 -1 리턴
```

### ➡️현재 작업 디렉터리 위치 검색 1 : getcwd(3)

함수 원형 : char *getcwd(char *buf, size_t size);

예시 :

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main() {
	char *cwd;
	char wd1[BUFSIZ];
	char wd2[10];
	
	//1. buf에 경로를 저장할 충분한 메모리 할당, 크기를 size에 지정
	getcwd(wd1, BUFSIZ);
	
	//2. buf에 NULL 지정, 할당이 필요한 메모리 크기를 size에 지정
	cwd = getcwd(NULL, BUFSIZ);
	free(cwd);
	
	//3. buf에 NULL 지정, size 0 지정
	cwd = getcwd(NULL, 0);
	free(cwd);
	
	//문자열 수 + \0(NULL 문자) 까지 사이즈 필요!+
```

### ➡️디렉터리명 변경 : rename(2)

함수 원형 : int rename(const char **oldpath, const char* *newpath);

예시 :

```c
#include <stdio.h>

if(rename("han", "bit") == -1 )
//성공시 0, 실패시 -1 리턴
```

### ➡️디렉터리 이동1 : chdir(2)

함수 원형 : int chdir(const char *path);

예시 :

```c
#include <unistd.h>

chdir("bit");
//성공시 0, 실패시 -1 리턴
//중요 : 쉘이 자신의 데이터를 바꾸는 게 아님.
//★실행 파일의 프로세스 상태에서만 수행★
```

### ➡️디렉터리 이동2 : fchdir(2)

함수 원형 : int fchdir(int fd);

예시 :

```c
#include <fcntl.h>
#include <unistd.h>

fd = open("bit", O_RDONLY);
fchdir(fd);

close(fd);
```

## ✅디렉터리 내용 읽기

### ➡️디렉터리 열기, 닫기, 읽기 : opendir(3) closedir(3) readdir(3)

함수 원형 : DIR *opendir(const char *name);

함수 원형 : int closedir(DIR *dirp);

함수 원형 : struct dirent *readdir(DIR *dirp);

- dirent 구조체

```c
struct dirent {
	ino_t d_ino; //inode 번호
	off_t d_off; 
	unsigned short d_reclen;
	unsigned char d_type; //파일의 종류
	char d_name[256]; //항목의 이름
};
```

예시 :

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

int main() {
	DIR *dp;
	struct dirent *dent;
	
	dp = opendir("."); //디렉터리 포인터 반환
	
	while((dent = readdir(dp))) { //dirent 구조체 반환
		printf("Name : %s", dent->d_name);
		printf("Inode : %d\n", (int)dent->d_ino); //★
	}
	
	closedir(dp); //성공시 0, 실패시 -1 반환
}
```

### ➡️디렉터리 내용을 읽는 위치 변경 : telldir/seekdir/rewinddir(3)

함수 원형 : long telldir(DIR *dirp);

(디렉터리 스트림에서 현재 위치 리턴, 오류시 -1 리턴)

함수 원형 : void seekdir(DIR *dirp, long loc);

(디렉터리 스트림에서 readdir() 함수가 다음 항목을 읽을 수 있는 위치로 오프셋 이동. loc은 telldir()이 리턴한 값이어야 함.)

함수 원형 : void rewinddir(DIR *dirp);

(디렉터리 스트림 위치를 시작 지점으로 이동)

예시 :

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

int main() {
	DIR *dp;
	struct dirent *dent;
	
	long loc;
	
	dp = opendir(".");
	
	printf("Start Position : %ld\n", telldir(dp));
	while((dent = readdir(dp))) {
		printf("Read : %s -> ", dent->d_name);
		printf("Cur Position : %ld\n", telldir(dp));
	}
	
	printf("** Directory Position Rewind **\n");
	rewinddir(dp);
	printf("Cur Position : %ld\n", telldir(dp));
	
	printf("** Move Directory Pointer **\n");
	readdir(dp);
	loc = telldir(dp);
	seekdir(dp, loc);
	printf("Cur Position : %ld\n", telldir(dp));
	
	dent = readdir(dp);
	printf("Read : %s \n", dent->d_name);
	
	closedir(dp);
}
```