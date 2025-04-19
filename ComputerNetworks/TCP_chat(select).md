## tcp_chatserv.c

```c
#include "netprog2.h"
#define MAXLINE 511
#define MAX_SOCK 1024

char *EXIT_STRING = "exit";
char *START_STRING = "Connected to char_server \n";

int num_chat = 0;
int listen_sock, clisock_list[MAX_SOCK];

void addClient(int s, struct sockaddr_in *newcliaddr);
void removeClient(int s);
int getmax();

int main(int argc, char *argv[]) {
	struct sockaddr_in cliaddr;
	char buf[MAXLINE+1];
	
	int i, j, nbyte, maxfdp1, accp_sock, addrlen = sizeof(struct sockaddr_in);
	fd_set read_fds;
	
	if(argc != 2) {
		printf("사용법 : %s port\n", argv[0]);
		exit(0);
	}
	listen_sock = tcp_listen(INADDR_ANY, atoi(argv[1]), 5);
	
	while(1) {
		//빠져나간 클라이언트가 있을 수 있음(1비트 배열 모두 0으로 초기화)
		FD_ZERO(&read_fds);
		//새로 accept 요청하는 클라이언트 1로 설정
		FD_SET(listen_sock, &read_fds);
		//감시할 클라이언트 목록 1로 설정
		for(i = 0; i<num_chat; i++) {
			FD_SET(clisock_list[i], &read_fds);
		}
		//가장 큰 소켓 디스크립터 + 1
		maxfdp1 = getmax() + 1;
		puts("wait for client");
		//read_fds에 1로 초기화된 소켓 중 I/O 작업이 들어오면 select 리턴
		//I/O 작업이 발생한 소켓만 1로 초기화하여 리턴
		if(select(maxfdp1, &read_fds, NULL, NULL, NULL) < 0 ) {
			errquit("select fail");
		}
		
		//만약 새로 accept 요청이라면
		if(FD_ISSET(listen_sock, &read_fds)) {
			accp_sock = accept(listen_sock, (struct sockaddr*)&cliaddr, &addrlen);
			if(accp_sock == -1) {
				errquit("accept fail");
			}
			addClient(accp_sock, &cliaddr);
			send(accp_sock, START_STRING, strlen(START_STRING), 0);
			printf("%d번째 사용자 추가.\n", num_chat);
		}
		
		//감시할 클라이언트 I/O작업(전송한 게 있는지)확인
		for(i=0; i < num_chat; i++) {
			if(FD_ISSET(clisock_list[i], &read_fds) {
				nbyte = recv(clisock_list[i], buf, MAXLINE, 0);
				//클라이언트 측 소켓이 close 한 경우
				if(nbyte <= 0) { 
					removeClient(i);
					continue;
				}
				//null문자 추가
				buf[nbyte] = 0;
				//exit를 입력하여 종료한 경우
				if(strstr(buf, EXIT_STRING) != NULL) {
					removeClient(i);
					continue;
				}
				//종료한 경우가 아니라면 모든 클라이언트에게 send
				for(j=0; j < num_chat; j++) {
					send(clisock_list[j], buf, nbyte, 0);
				}
				//서버에도 출력
				printf("%s\n", buf);
			}
		}
	}
	return 0;
}

void addClient(int s, struct sockaddr_in *newcliaddr) {
	char buf[20];
	//네트워크 바이트 순서의 IP주소를 문자열로 buf에 저장
	inet_ntop(AF_INET, &newcliaddr->sin_addr, buf, sizeof(buf));
	printf("new client: %s(%d)\n", buf, ntohs(newcliaddr->sin_port));
	
	clisock_list[num_chat] = s;
	num_chat++;
}

void removeClient(int s) {
	close(clisock_list[s]);
	//만약 마지막 클라이언트가 아니라면, 마지막 클라이언트를 종료한 클라이언트에 위치
	if(s != num_chat-1) {
		clisock_list[s] = clisock_list[num_chat-1];
	}
	num_chat--;
	printf("채팅 참가자 1명 탈퇴. 현재 참가자 수 = %d\n", num_chat);
}

int getmax() {
	int max = listen_sock;
	int i;
	//가장 큰 소켓 디스크립터 찾기
	for(i = 0; i < num_chat; i++) {
		if(clisock_list[i] > max ) {
			max = clisock_list[i];
		}
	}
	return max;
}
```

## tcp_chatcli.c

```c
#include "netprog2.h"
#define MAXLINE 1000
#define NAME_LEN 20

char *EXIT_STRING = "exit";

int main(int argc, char *argv[]) {
    char bufall[MAXLINE+NAME_LEN], *bufmsg;
    int maxfdp1, s, namelen;
    fd_set read_fds;

    if(argc != 4) {
        printf("사용법 : %s server_ip port username \n", argv[0]);
        exit(0);
    }
		//이름이 sieon이라면 "[sieon] : "을 bufall에 저장
    sprintf(bufall, "[%s] : ", argv[3]);
		
		//bufmsg 포인터가 bufall 다음을 가리키도록(메세지 출력)설정
    namelen = strlen(bufall);
    bufmsg = bufall+namelen;

    s = tcp_connect(AF_INET, argv[1], atoi(argv[2]));
    if(s == -1) {
        errquit("tcp_connect fail");
    }

    puts("서버에 접속 되었습니다.");
    maxfdp1 = s + 1;
    FD_ZERO(&read_fds);

    while(1) {
		    //표준 입력과 s소켓디스크립터(통신용 소켓) 1로 초기화
        FD_SET(0, &read_fds);
        FD_SET(s, &read_fds);

        if(select(maxfdp1, &read_fds, NULL, NULL, NULL) < 0 ) {
		        errquit("select fail");
        }
        //통신용 소켓에 전송작업이 올 시
        if(FD_ISSET(s, &read_fds)) {
		        int nbyte;
		        //bufmsg 위치에 상대 메세지 덮어씌우기
            if((nbyte = recv(s, bufmsg, MAXLINE, 0)) > 0 ) {
		            //null문자 추가후 출력
                bufmsg[nbyte] = 0;
                printf("%s \n", bufmsg);
            }
	      }
	      //표준 입력(키보드)에 작업이 발생할 시
        if(FD_ISSET(0, &read_fds)) {
		        //bufmsg 위치에 내 메세지 쓰기
		        if(fgets(bufmsg, MAXLINE, stdin)) {
		            if(send(s, bufall, namelen + strlen(bufmsg), 0 ) < 0 ) {
				            puts("Error : Write error on socket");
                }
                if(strstr(bufmsg, EXIT_STRING) != NULL) {
		                puts("Good Bye.");
                    close(s);
                    exit(0);
               }
           }
       }
  }
}

```