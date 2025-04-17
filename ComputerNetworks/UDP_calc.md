## udp_calccli.c

```c
#include "netprog.h"
#define MAXLINE 511

struct calcmsg {
     char type; //0 : REQ, 1 : RSP
     char status; // 0 : SUCCESS, 1 : DIVIDE_BY_ZERO, 2 : OVERFLOW
     char op;// '+' : ADD, '-' : SUBTRACT, '*' : MULTYPLY, '/' : DIVIDE
     char padding;
     int32_t op1, op2, result;
};

int main(int argc, char *argv[]) {
    struct sockaddr_in servaddr;
    int s, nbyte, addrlen = sizeof(servaddr);
    char buf[MAXLINE+1];

    if(argc != 3) {
        printf("usage : %s ip_address port_number\n", argv[0]);
        exit(0);
    }

    if((s = socket(PF_INET, SOCK_DGRAM, 0)) < 0) {
        errquit("socket fail");
    }

    bzero((char *)&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    inet_pton(AF_INET, argv[1], &servaddr.sin_addr);
    servaddr.sin_port = htons(atoi(argv[2]));
    
    printf("입력 : ");
    char op;
    int32_t op1, op2;
    scanf("%d %c %d", &op1, &op, &op2);

    struct calcmsg reqmsg;
    reqmsg.type = 0;
    reqmsg.status = 0;
    reqmsg.op = op;
    reqmsg.op1 = htonl(op1);
    reqmsg.op2 = htonl(op2);
    if(sendto(s, &reqmsg , sizeof(reqmsg), 0, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0 ){
        errquit("sendto fail");
    }
    
    struct calcmsg rspmsg;
    if((nbyte = recvfrom(s, &rspmsg, sizeof(rspmsg), 0, (struct sockaddr*)&servaddr, &addrlen)) < 0 ) {
        errquit("recvfrom fail");
    }
    if(rspmsg.status==0) {
        printf("출력 : %d %c %d = %d\n", op1, op, op2, rspmsg.result);
    } else {
        printf("0으로 나눌 수 없습니다.\n");
    }
    
    close(s);
}

```

## udp_calcserv.c

```c
#include "netprog.h"
#define MAXLINE 511

struct calcmsg {
    char type; //0 : REQ, 1 : RSP
    char status; // 0 : SUCCESS, 1 : DIVIDE_BY_ZERO, 2 : OVERFLOW
    char op;// 0 : ADD, 1 : SUBTRACT, 2 : MULTYPLY, 3 : DIVIDE
    char padding;
    //int형보다 환경에 따라 바뀌지 4바이트 int32_t
    int32_t op1, op2, result;
};

int main(int argc, char*argv[]) {
    struct sockaddr_in servaddr, cliaddr;
    int s, nbyte, addrlen = sizeof(cliaddr);
    char buf[MAXLINE+1];

    if(argc !=2) {
        printf("usage : %s port\n", argv[0]);
        exit(0);
    }
    if((s = socket(PF_INET, SOCK_DGRAM, 0)) < 0 )
        errquit("socket fail");

    bzero((char *)&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(atoi(argv[1]));

    if(bind(s, (struct sockaddr*)&servaddr, addrlen) < 0)
        errquit("bind fail");
    
    struct calcmsg reqmsg;
    while(1) {
        puts("Server : waiting request.");
        if((nbyte = recvfrom(s, &reqmsg, sizeof(reqmsg), 0, (struct sockaddr*)&cliaddr, &addrlen)) < 0)
            errquit("recvfrom fail");
        struct calcmsg sendmsg;
        sendmsg.op = reqmsg.op;
        sendmsg.op1 = ntohl(reqmsg.op1);
        sendmsg.op2 = ntohl(reqmsg.op2);
        sendmsg.status = 0;
        switch(sendmsg.op){
            case '+' : sendmsg.result = sendmsg.op1 + sendmsg.op2; break;
            case '-' : sendmsg.result = sendmsg.op1 - sendmsg.op2; break;
            case '*' : sendmsg.result = sendmsg.op1 * sendmsg.op2; break;
            case '/' :
                      if(sendmsg.op2==0){
                           sendmsg.status = 2;
                           break;
                      }
                      else{
                           sendmsg.result = sendmsg.op1 / sendmsg.op2;
                           break;
                      }
            default : break;
        }

        if(sendto(s, &sendmsg, sizeof(sendmsg), 0, (struct sockaddr*)&cliaddr, addrlen) < 0) {
            errquit("sendto fail");
        }
        printf("sendto complete\n");
    }
}

```