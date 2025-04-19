## tcp_talkserv.c

```c
#include "netprog.h"

char *EXIT_STRING = "exit";
int recv_and_print(int sd);
int input_and_send(int sd);

#define MAXLINE 511

int main(int argc, char* argv[]) {
    struct sockaddr_in cliaddr, servaddr;
    int listen_sock, accp_sock,
        addrlen = sizeof(cliaddr);
    //부모 프로세스(키보드 입력 대기)와 자식 프로세스(상대 채팅 출력)을 구분짓기 위해 필요
    pid_t pid; 

    if(argc != 2) {
        printf("사용법 : %s port\n", argv[0]);
        exit(0);
    }
    //소켓 생성
    if((listen_sock=socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        errquit("socket fail");
    }
    
    bzero((char*)&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(atoi(argv[1]));
    // bind를 통해서 소켓 디스크립터와 실제 서버구조체 연결
    if(bind(listen_sock, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0) {
            errquit("bind fail");
    }
    puts("서버가 클라이언트를 기다리고 있습니다");
    //listen을 통해서 클라이언트 연결 대기
    listen(listen_sock, 1);
    //accept 연결 성공시 아래 fork를 통해 두 프로세스로 나뉘어 대기에 들어감
    if((accp_sock = accept(listen_sock, (struct sockaddr *)&cliaddr, &addrlen)) < 0) {
        errquit("accept fail");
    }

    puts("클라이언트가 연결 되었습니다.");
    // 부모 프로세스(키보드 입력 대기)
    if((pid=fork()) > 0) {
        input_and_send(accp_sock);
    // 자식 프로세스(상대 채팅 출력)
    } else if (pid==0) {
        recv_and_print(accp_sock);
    }
    close(listen_sock);
    close(accp_sock);
    return 0;
}

int input_and_send(int sd) {
    char buf[MAXLINE+1];

    while(fgets(buf, sizeof(buf), stdin) != NULL ) {
        write(sd, buf, strlen(buf));
				//buf의 내용에서 EXIT_STRING이 있다면 종료
        if(strstr(buf, EXIT_STRING) != NULL ) {
            puts("Good bye.");
            close(sd);
            exit(0);
        }
    }
    return 0;
}

int recv_and_print(int sd) {
    char buf[MAXLINE+1];
    int nbyte;

    while(1) {
        if((nbyte = read(sd, buf, MAXLINE)) < 0) {
            perror("read fail");
            close(sd);
            exit(0);
        }
        buf[nbyte] = 0;

        if(strstr(buf, EXIT_STRING) != NULL ) {
            break;
        }
        printf("%s", buf);
    }
    return 0;
}
```

## tcp_talkcli.c

```c
#include "netprog.h"

char *EXIT_STRING = "exit";
int recv_and_print(int sd);
int input_and_send(int sd);

#define MAXLINE 511

int main(int argc, char* argv[]) {
        static struct sockaddr_in servaddr;
        static int s;
        pid_t pid;

        if(argc != 3) {
                printf("사용법 : %s server_ip port\n", argv[0]);
                exit(0);
        }
        //소켓 생성
        if((s=socket(PF_INET, SOCK_STREAM, 0)) < 0) {
                errquit("socket fail");
        }
        //클라이언트의 소켓 주소 구조체 servaddr 0으로 초기화
        bzero((char*)&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        inet_pton(AF_INET, argv[1], &servaddr.sin_addr);
        servaddr.sin_port = htons(atoi(argv[2]));
        
        //connect : s소켓 디스크립터로 servaddr과 연결
        if(connect(s, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0) {
                errquit("connect fail");
        }
				//연결 성공시 부모 프로세스(키보드 입력 대기) 자식 프로세스(상대 채팅 출력 대기)
        if((pid=fork()) > 0) {
                input_and_send(s);
        } else if (pid==0) {
                recv_and_print(s);
        }
        close(s);
        return 0;
}

int input_and_send(int sd) {
        char buf[MAXLINE+1];

        while(fgets(buf, sizeof(buf), stdin) != NULL ) {
                write(sd, buf, strlen(buf));

                if(strstr(buf, EXIT_STRING) != NULL ) {
                        puts("Good bye.");
                        close(sd);
                        exit(0);
                }
        }
        return 0;
}

int recv_and_print(int sd) {
        char buf[MAXLINE+1];
        int nbyte;

        while(1) {
                if((nbyte = read(sd, buf, MAXLINE)) < 0) {
                        perror("read fail");
                        close(sd);
                        exit(0);
                }
                buf[nbyte] = 0;

                if(strstr(buf, EXIT_STRING) != NULL ) {
                        break;
                }
                printf("%s", buf);
        }
        return 0;
}

```

## ***exec 방식을 사용한 클라이언트

### tcp_talkcli_exec.c

```c
#include "netprog.h"

char *EXIT_STRING = "exit";
int recv_and_print(int sd);
int input_and_send(int sd);

#define MAXLINE 511

int main(int argc, char* argv[]) {
        static struct sockaddr_in servaddr;
        static int s;
        pid_t pid;

        if(argc != 3) {
                printf("사용법 : %s server_ip port\n", argv[0]);
                exit(0);
        }
        //소켓 생성
        if((s=socket(PF_INET, SOCK_STREAM, 0)) < 0) {
                errquit("socket fail");
        }
        //클라이언트의 소켓 주소 구조체 servaddr 0으로 초기화
        bzero((char*)&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        inet_pton(AF_INET, argv[1], &servaddr.sin_addr);
        servaddr.sin_port = htons(atoi(argv[2]));
        if(connect(s, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0) {
                errquit("connect fail");
        }

        if((pid=fork()) > 0) {
                input_and_send(s);
        } else if (pid==0) {
        //execl은 문자열 형태로 인자를 넘기기 때문에 line 변수 만들기.
        //sprintf : line 변수에 %d 정수형의 s소켓 디스크립터를 문자열로 만들어 저장
        //execl : "tcp_talkexec" 실행파일로 자식 프로세스를 바꾸고
        // 첫 번째 인자 : "tcp_talkexec", 두 번째 인자 : line(소켓 디스크립터)
                char line[80];
                sprintf(line, "%d", s);
                execl("tcp_talkexec", "tcp_talkexec", line);
        }
        close(s);
        return 0;
}

int input_and_send(int sd) {
        char buf[MAXLINE+1];

        while(fgets(buf, sizeof(buf), stdin) != NULL ) {
                write(sd, buf, strlen(buf));

                if(strstr(buf, EXIT_STRING) != NULL ) {
                        puts("Good bye.");
                        close(sd);
                        exit(0);
                }
        }
        return 0;
}

```

### tcp_talkexec.c

```c
#include "netprog.h"
char *EXIT_STRING = "exit";
#define MAXLINE 511

int main(int argc, char *argv[]) {
        char buf[MAXLINE+1];
        int nbyte, sd;
				//atoi : 문자열을 정수로
        sd = atoi(argv[1]);

        while (1) {
                if ((nbyte = read(sd, buf, MAXLINE)) < 0){
                        close(sd);
                        errquit("read fail");
                }
                buf[nbyte] = 0;

                if (strstr(buf, EXIT_STRING) != NULL )
                        break;
                printf("%s", buf);
        }
        close(sd);
}
```