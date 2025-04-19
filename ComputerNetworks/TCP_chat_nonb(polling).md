## tcp_chatserv_nonb.c

```c
#include "netprog2.h"
#include <fcntl.h>
#include <errno.h>

#define MAXLINE 511
#define MAX_SOCK 1024

char *EXIT_STRING = "exit";
char *START_STRING = "Connected to chat_server \n";

int num_chat=0;
int listen_sock, clisock_list[MAX_SOCK];

void addClient(int s, struct sockaddr_in *newcliaddr);
void removeClient(int s);
int set_nonblock(int sockfd);
int is_nonblock(int sockfd);
int getmax();

int main(int argc, char *argv[]) {
    struct sockaddr_in cliaddr;
    char buf[MAXLINE+1];

    int i, j, nbyte, maxfdp1, accp_sock, addrlen = sizeof(cliaddr);
    //fd_set read_fds; 셀렉트용 구조체

    if(argc != 2) {
        printf("사용법 : %s port\n", argv[0]);
        exit(0);
    }
    
    listen_sock = tcp_listen(INADDR_ANY, atoi(argv[1]), 5);
    if(listen_sock==-1) {
        errquit("tcp_listen fail");
    }
    //
    if(set_nonblock(listen_sock) == -1) {
        errquit("set_nonblock fail");
    }
    while(1) {
		    //
		    sleep(1);
		    putchar('.');
		    fflush(stdout);
	
		    accp_sock = accept(listen_sock, (struct sockaddr *)&cliaddr, &addrlen);
				
				//accept 연결이 제대로 안 되고, 논블록 모드의 즉시 리턴이 아닐 때
		    if(accp_sock == -1 && errno!=EWOULDBLOCK) {
		        errquit("accept fail");
		    } else if (accp_sock > 0) {
				    //연결 소켓이 논블록이 아니고, 논블록으로 설정할 수도 없을 때
		        if(is_nonblock(accp_sock) != 0 && set_nonblock(accp_sock) < 0 ) {
		            errquit("set_nonblock fail");
		        }
		        addClient(accp_sock, &cliaddr);
		        send(accp_sock, START_STRING, strlen(START_STRING), 0);
		        printf("%d번째 사용자 추가.\n", num_chat);
		   }
		   for(i=0; i < num_chat; i++) {
		        errno = 0;
		        nbyte = recv(clisock_list[i], buf, MAXLINE, 0);
		        if(nbyte==0) {
		            removeClient(i);
		            continue;
		        //논블록으로 인한 즉시 리턴일 때
		        } else if (nbyte == -1 && errno == EWOULDBLOCK) {
		            continue;
		        }
		        //buf에 EXIT_STRING 내용이 있을 경우
		        if(strstr(buf, EXIT_STRING) != NULL) {
		            removeClient(i);
		            continue;
		        }
						//null문자 추가 밎 클라이언트에게 출력
		        buf[nbyte] = 0;
		        for(j=0; j < num_chat; j++) {
		            send(clisock_list[j], buf, nbyte, 0);
		        }
		        printf("%s\n", buf);
		   }
    }
}
//아래 부분 암기
void addClient(int s, struct sockaddr_in *newcliaddr) {
    char buf[20];
    inet_ntop(AF_INET, &newcliaddr->sin_addr, buf, sizeof(buf));
    printf("new client: %s(%d)\n", buf, ntohs(newcliaddr->sin_port));

    clisock_list[num_chat] = s;
    num_chat++;
}

void removeClient(int s) {
    close(clisock_list[s]);
    if(s != num_chat-1) {
        clisock_list[s] = clisock_list[num_chat-1];
    }
    num_chat--;
    printf("채팅 참가자 1명 탈퇴. 현재 참가자 수 = %d\n", num_chat);
}

int getmax() {
    int max = listen_sock;
    int i;
    for(i = 0; i < num_chat; i++) {
        if(clisock_list[i] > max ) {
            max = clisock_list[i];
        }
    }
    return max;
}

int is_nonblock(int sockfd) {
    int val = fcntl(sockfd, F_GETFL, 0);
    if(val & O_NONBLOCK) {
        return 0;
    }
    return -1;
}

int set_nonblock(int sockfd) {
    int val = fcntl(sockfd, F_GETFL, 0);
    if(fcntl(sockfd, F_SETFL, val | O_NONBLOCK) == -1) {
        return -1;
    }
    return 0;
}

```