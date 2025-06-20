# 10장 : 파이프

## 이름 없는 파이프

### 10-3 이름 없는 파이프 통신

```c
int main(){
        int fd[2];
        pid_t pid;
        char buf[257];
        int len, status;

        if(pipe(fd) == -1){
                perror("pipe");
                exit(1);
        }

        switch (pid=fork()){
                case -1:
                        perror("fork");
                        exit(1);
                        break;
                case 0:
                        close(fd[1]);
                        write(1, "Child Process:", 15);
                        len = read(fd[0], buf, 256);
                        write(1, buf, len);
                        close(fd[0]);
                        break;
                default:
                        close(fd[0]);
                        write(fd[1], "Test Message\n", 14);
                        close(fd[1]);
                        waitpid(pid, &status, 0);
                        break;
        }
}
```

### 10-5 이름없는 파이프 양방향 활용

```c
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(){
        int fd1[2], fd2[2];
        pid_t pid;
        char buf[257];
        int len, status;

        if(pipe(fd1) == -1){
                perror("pipe");
                exit(1);
        }

        if(pipe(fd2) == -1){
                perror("pipe");
                exit(1);
        }

        switch(pid = fork()){
                case -1:
                        perror("fork");
                        exit(1);
                        break;
                case 0:
                        close(fd1[1]);
                        close(fd2[0]);
                        len = read(fd1[0], buf, 256);
                        write(1, "Child Process:", 15);
                        write(1, buf, len);

                        strcpy(buf, "Good\n");
                        write(fd2[1], buf, strlen(buf));
                        break;
                default:
                        close(fd1[0]);
                        close(fd2[1]);

                        write(fd1[1], "Hello\n", 6);

                        len = read(fd2[0], buf, 256);
                        write(1, "Parent Process:", 15);
                        write(1, buf, len);
                        wait(NULL);
                        break;
        }
}
```

---

## 이름 있는 파이프

### FIFO 파일 생성 : mkfifo(3)

함수 원형 : int mkfifo(const char *pathname, mode_t mode);

Server.c

```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>

int main(){
        int pd, n;
        char msg[] = "Hello, FIFO";

        printf("Server ==========\n");

        if(mkfifo("./S_FIFO", 0666) ==-1 && errno != EEXIST){
                perror("mkfifo");
                exit(1);
        }

        if((pd = open("./S_FIFO", O_WRONLY)) == -1){
                perror("open");
                exit(1);
        }

        printf("To Client : %s\n", msg);

        n = write(pd, msg, strlen(msg)+1);
        if(n==-1){
                perror("write");
                exit(1);
        }
        close(pd);
}
```

Client.c

```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(){
        int pd, n;
        char imsg[80];
        
        if((pd = open("./S_FIFO", O_RDONLY)) == -1){
                perror("open");
                exit(1);
        }       
        
        printf("Client ============\n");
        write(1, "From Server :", 13);

        while((n=read(pd, imsg, 80)) > 0){
                write(1, imsg, n);
        }

        if(n==-1){
                perror("read");
                exit(1);
        }       
        write(1, "\n", 1);
        close(pd);
}
```