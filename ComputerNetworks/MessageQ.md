### msgsnd.c

```c
#include "netprog2.h"
#include <sys/ipc.h>
#include <sys/msg.h>

#define BUFSIZ 512

typedef struct _msg{
	long msg_type; //반드시 long형
	char msg_text[BUFSIZ];
}msg_t;

void message_send(int qid, long type, const char *text){
	msg_t pmsg;
	pmsg.msg_type = type;
	int len = strlen(text);
	strncpy(pmsg.msg_text, text, len); // ★첫 인자에 두 번째 인자를 len만큼 복사(문자열 복사)
	pmsg.msg_text[len] = 0;
	msgsnd(qid, &pmsg, len, 0); //메시지 큐 아이디, 메시지 구조체 주소, text의 길이
}

int main(int argc, char **argv){
	if(argc!=2){
		printf("Usage: %s msqkey\n", argv[0]);
		exit(1);
	}
	
	key_t key = atoi(argv[1]);
	int qid = msgget(key, IPC_CREAT | 0600);
	if(qid < 0){
		errquit("msgget");
	}
	
	char text[BUFSIZ];
	puts("Enter message to post : ");
	fgets(text, BUFSIZ, stdin);
	message_send(qid, 3, text);
	
	puts("Enter message to post : ");
	fgets(text, BUFSIZ, stdin);
	message_send(qid, 3, text);
	
	puts("Enter message to post : ");
	fgets(text, BUFSIZ, stdin);
	message_send(qid, 3, text);
	
	return 0;
}
	
```

### msgrcv.c

```c
#include "netprog2.h"
#include <sys/ipc.h>
#include <sys/msg.h>

#define BUFSIZ 512

typedef struct _msg{
	long msg_type;
	char msg_text[BUFSIZ];
}msg_t;

void message_receive(int qid, long type){
	msg_t rmsg;
	int nbytes = msgrcv(qid, &rmsg, sizeof(rmsg.msg_text), type, 0);
	printf("recv = %d\n", nbytes);
	printf("msg_type = %ld\n", rmsg.msg_type); //long형은 %ld 잊지말기
	printf("msg_text = %s\n", rmsg.msg_text);
}

int main(int argc, char *argv[]){
	if (argc != 2){
        printf("Usage: %s <message queue id>\n", argv[0]);
        exit(EXIT_FAILURE);
  }
  key_t key = atoi(argv[1]);
  
  int qid = msgget(key, IPC_CREAT|0666);
  if(qid<0){
	  perror("msgget");
	  exit(1);
	 }
	 message_receive(qid, -3);
	 message_receive(qid, -3);
	 message_receive(qid, -3);
	 
	 return 0;
}
```

### msg_calcserv.c

```c
#include "netprog2.h"
#include <pthread.h>
#include <sys/ipc.h>
#include <sys/msg.h>

#define MSGCALC_H
#define REQUEST 0
#define RESPONSE 1
#define SUCCESS 0
#define FAIL 1

typedef struct _calcdata { //연산을 위한 구조체
    char type, flag, op;
    float op1, op2, result;
} calcdata_t;

typedef struct _msg { //메시지 큐를 담을 구조체
    long type;
    calcdata_t data;
    struct sockaddr_in addr;
} msg_t;

int msgqid, msgsize;
void *dequeue_and_send(void *arg);
void calculate(calcdata_t *data);

int main(int argc, char *argv[]){
    struct sockaddr_in cliaddr;
    int nbyte, sock, addrlen = sizeof(cliaddr);
    pthread_t tid;
    calcdata_t data;
    msg_t msg;
    msg.type = 1;
    msgsize = sizeof(msg_t) - sizeof(long); //타입을 제외한 크기
    if(argc!=2){
        printf("Usage : %s port\n", argv[0]);
        exit(0);
    }
		//IPC_PRIVATE을 통해서 키 생성 후 본인 프로세스 내부에서 사용
    if((msgqid = msgget(IPC_PRIVATE, IPC_CREAT | 0600)) < 0){
        errquit("msgget");
    }
    //udp_server_socket(INADDR_ANY, 포트 번호)를 통한 서버 소켓 생성
    if((sock = udp_server_socket(INADDR_ANY, atoi(argv[1]))) < 0){
        errquit("bind");
    }
    //dequeue_and_send를 멀티 스레드로 메시지를 전송하도록 작업하도록 구성
    pthread_create(&tid, NULL, dequeue_and_send, &sock);
		//main문에서는 클라이언트로부터 수식 전달받기
    while(1){
		    //sock을 통해서 데이터를 어디에 받을지와 크기, 어디서 받았는지와 받은 곳의 크기 주소★
        nbyte = recvfrom(sock, &data, sizeof(calcdata_t), 0, (struct sockaddr*)&cliaddr, &addrlen);
        if(nbyte == sizeof(calcdata_t)){
		        //메시지큐 구조체에 받은 데이터와 클라이언트 정보를 담아 메시지 큐 삽입
            msg.data = data;
            msg.addr = cliaddr;
            if(msgsnd(msgqid, &msg, msgsize, 0) < 0){
                errquit("msgsnd");
            }
            //msg.addr.sin_addr 주소를 변환하여 addr에 문자열로 저장 및 출력
            char addr[80];
            inet_ntop(AF_INET, &msg.addr.sin_addr, addr, sizeof(addr));
            printf("recv from %s:%d (%f %c %f)\n", addr, ntohs(msg.addr.sin_port), data.op1, data.op, data.op2);
        } else{
            errquit("wrong message format\n");
        }
    }
    //메시지 큐 제거
    msgctl(msgqid, IPC_RMID, 0);
    return 0;
}

void *dequeue_and_send(void *arg){
    int sock = *((int *)arg), addrlen = sizeof(struct sockaddr_in);
    calcdata_t data;
    msg_t msg;
    char addr[80];
		
    data.type = RESPONSE;
    while(1){
		    //메시지 구조체를 통해서 메시지 받음
        if(msgrcv(msgqid, (void*)&msg, msgsize, 0, 0) < 0){
            errquit("msgrcv");
        }
        //data 에 메시지로부터 받은 데이터 저장 및 계산
        data = msg.data;
        //포인터로 받아 주소를 전달하여 값에 영향을 주게함
        calculate(&data);
        inet_ntop(AF_INET, &msg.addr.sin_addr, addr, sizeof(addr));
        printf("sendto %s:%d (%f %c %f = %f)\n", addr, ntohs(msg.addr.sin_port), data.op1, data.op, data.op2, data.result);
        if(sendto(sock, &data, sizeof(calcdata_t), 0, (struct sockaddr*)&msg.addr, addrlen) < 0) {
            errquit("sendto");
        }
    }
}

void calculate(calcdata_t *data){
    (*data).flag = SUCCESS;
    if((*data).op == '+')
        (*data).result = (*data).op1 + (*data).op2;
    else if ((*data).op == '-')
        (*data).result = (*data).op1 - (*data).op2;
    else if ((*data).op == '*')
        (*data).result = (*data).op1 * (*data).op2;
    else if ((*data).op == '/')
        if((*data).op2 !=0){
            (*data).result = (*data).op1 / (*data).op2;
        } else {
            (*data).flag = FAIL;
            (*data).result = 0;
        }
}
```

### msg_calccli.c

```c
#include "netprog2.h"
#include <pthread.h>

#define MSGCALC_H
#define REQUEST 0
#define RESPONSE 1
#define SUCCESS 0
#define FAIL 1

typedef struct _calcdata { //연산을 위한 구조체 calcdata_t
    char type, flag, op;
    float op1, op2, result;
} calcdata_t;

typedef struct _msg { //메시지 큐를 담을 구조체 msg_t
    long type;
    calcdata_t data;
    struct sockaddr_in addr; //
} msg_t;

void *recv_and_print(void *arg);

int main(int argc, char* argv[]) {
    struct sockaddr_in servaddr; //서버 소켓 주소
    int sock, addrlen = sizeof(servaddr);
    calcdata_t data;
    pthread_t tid;

    if(argc != 3) { printf("usage : %s ipaddr port\n", argv[0]); exit(EXIT_FAILURE); }
		
		//udp_client_socket(서버 주소, 포트 번호, 서버 정보 담을 곳)을 통한 통신 소켓 만들기
    if((sock = udp_client_socket(argv[1], atoi(argv[2]), &servaddr)) < 0) errquit("socket");
    //main함수는 사용자로부터 입력 받기, recv_and_print는 서버로부터 받은 메시지 출력
    pthread_create(&tid, NULL, recv_and_print, &sock); 
    printf("1.2 + 2.3 형식으로 입력(지원연산 + - * /)\n");
    while(1) {
        data.type = REQUEST;
        scanf("%f %c %f", &data.op1, &data.op, &data.op2);
        if(data.op == '+' || data.op == '-' || data.op == '*' || data.op == '/') {
            if(sendto(sock, &data, sizeof(calcdata_t), 0, (struct sockaddr*)&servaddr, addrlen) < 0) errquit("sendto");
        }
        else printf("지원하지 않는 연산자\n");
    }
    return 0;
}

void *recv_and_print(void *arg) {
    struct sockaddr_in servaddr;
    int sock = *((int*)arg), nbyte, addrlen = sizeof(servaddr);
    calcdata_t data;
    char op, flag;

    while(1) {
        if((nbyte = recvfrom(sock, &data, sizeof(calcdata_t), 0, (struct sockaddr*)&servaddr, &addrlen)) <0) errquit("recvfrom");
        if(nbyte == sizeof(calcdata_t)) {
            if(data.flag == SUCCESS)
                printf("RESULT: %f %c %f = %f\n", data.op1, data.op, data.op2, data.result);
            else
                printf("RESULT: %f %c %f = ERROR\n", data.op1, data.op, data.op2);
        }
        else errquit("wrong message format\n");
    }
}
```