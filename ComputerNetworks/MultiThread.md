### ➡️pthread_create(&tid, NULL, &func, &arg);

스레드 생성 함수

- func으로 지정한 함수가 실행됨(시작함수)
- void * 타입의 인자 arg를 받을 수 있음
- void * 타입을 가리키는 포인터 리턴 가능
- 성공 시에 0 리턴

### ➡️pthread_self()

자신의 스레드 ID 확인

- 자신의 스레드 ID 리턴

### ➡️pthread_exit()

자신의 스레드 종료

- main에서 사용시 main 스레드 종료 후 자식 스레드들을 기다림
- 그냥exit()는 프로세스 종료임

### ➡️pthread_join(&tid, NULL)

자신이 생성한 자식 스레드가 종료할 때까지 기다림

- tid로 지정한 스레드가 종료될 때까지 대기
- NULL 위치에 자식의 종료 상태 저장 받기 가능 (void **타입)
- 주로 main 스레드에서 자식 스레드를 대기할 때 사용

### ➡️errno 관련

스레드 관련 함수들은 시스템 콜과 달리 errno 에러코드 저장X

전역 변수인 errno를 여러 스레드가 공유하므로 어떤 스레드가 현재의 errno를 발생시켰는지 구분 안 됨

그러나 시스템 콜은 여전히 errno 사용, 따라서 POSIX 스레드는 각 스레드별 errno를 제공하여 문제 해결(오버헤드가 큼)

이러한 이유로 pthread 관련 함수 호출시에 errno 변수나, perror 함수 사용하면 안 됨!

### mth_echoserv.c

```c
#include "netprog2.h"
#include <pthread.h>

#define MAXLINE 511
void *echo(void *arg);

int main(int argc, char *argv[]){
	struct sockaddr_in cliaddr;
	int listen_sock, sock, addrlen = sizeof(cliaddr);
	pthread_t tid;
	
	if(argc!=2){
		printf("Usage: %s port\n", argv[0]);
		exit(1);
	}
	
	listen_sock = tcp_listen(INADDR_ANY, atoi(argv[1]), 5);
	while(1){
		sock = accept(listen_sock, (struct sockaddr*)&cliaddr, &addrlen);
		pthread_create(&tid, NULL, echo, &sock);
	}
	return 0;
}

void *echo(void *arg){
	int nbyte, sock = *((int*)arg);
	char buf[MAXLINE + 1];
	printf("thread %lu started.\n", pthread_self());
	while(1){
		nbyte=read(sock, buf, MAXLINE);
		if(nbyte <= 0){
			close(sock);
			break;
		}
		buf[nbyte] = 0;
		printf("%lu : %s\n", pthread_self(), buf);
		write(sock, buf, nbyte);
	}
	printf("thread %lu stopped.\n", pthread_self());
}
```

### race.c <경쟁 상태>

```c
#include "netprog2.h"
#include <pthread.h>

#define MAX_THR 2
void *thrfunc(void *arg);
void prn_data(pthread_t me);
pthread_t who_run = 0;

int main(int argc, char **argv){
        pthread_t tid[MAX_THR];
        int i, status;
        for(i=0; i<MAX_THR; i++){
                if((status=pthread_create(&tid[i], NULL, &thrfunc, NULL)) !=0) {
                        printf("thread create error : %s\n", strerror(status));
                        exit(0);
                }
        }

        pthread_join(tid[0], NULL);
        return 0;
}

void *thrfunc(void *arg) {
        while(1){
				        //if문을 써서 who_run == 0 으로 해도 안 됨.(who_run이 0인 시점에 
				        //두 멀티스레드가 동시에 경쟁 상태에 놓임. 반드시 mutex 필요
                prn_data(pthread_self());
        }
        return NULL;
}

void prn_data(pthread_t me){
        who_run = me;
        if(who_run !=pthread_self()){
                printf("Error : %lu스레드 실행 중 who_run=%lu\n", me, who_run);
        } else {
                //printf("%lu\n", who_run);
        }
        who_run = 0;
}
```

---

### ➡️임계 구역(Critical Section)

둘 이상의 스레드가 동시에 실행하면 안 되는 코드 영역

하나의 공유 자원에 대한 둘 이상의 스레드의 경쟁 상태를 가지며 이는

공유 자원(전역 변수, 파일, 메모리 등)에 대한 **읽기/쓰기 충돌이 발생하게 함**

### ➡️뮤텍스 (Mutal Exclusion)

한 시점에 오직 하나의 스레드만 임계 구역을 실행할 수 있도록 보장하는 락, 언락

여러 스레드가 동시에 공유 자원을 건드리지 못 하게 하는 커널이 제공하는 플래그 장치

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&lock);

pthread_mutex_unlock(&lock);
```

### mth_chatserv.c

```c
#include "netprog2.h"
#include <pthread.h>

#define MAXLINE 511
#define MAX_SOCK 100

void addClient(int sock);
void removeClient(int sock);
void *recv_and_send(void *arg);

int socklist[MAX_SOCK];
int num_chat = 0;

//뮤텍스 변수 정의
pthread_mutex_t socklist_lock = PTHREAD_MUTEX_INITIALIZER;

int main(int argc, char* argv[]) {
		//클라이언트 정보를 담아둘 구조체
    struct sockaddr_in cliaddr;
    int listen_sock, sock, addrlen = sizeof(cliaddr);

    if(argc != 2) { printf("usage : %s port\n", argv[0]); exit(1); }
		//tcp_listen(INADDR_ANY, 포트 번호, 5)를 통한 listen 소켓 연결
    listen_sock = tcp_listen(INADDR_ANY, atoi(argv[1]), 5);

    while(1) {
		    //listen_sock을 통해 커넥트 요청이 오면 sock과 연결.
        sock = accept(listen_sock, (struct sockaddr*)&cliaddr, &addrlen);
        addClient(sock);
    }
    return 0;
}

void addClient(int sock) {
    pthread_t tid;
		//뮤텍스를 이용하여 임계 구역(전역 변수 사용)을 잠근 후 접근
    pthread_mutex_lock(&socklist_lock);
    socklist[num_chat++] = sock;
    //임계 구역을 빠져나올 때 뮤텍스 잠금 해제
    pthread_mutex_unlock(&socklist_lock);
		
		//클라이언트로부터 메시지를 받아 모든 클라이언트에게 보내줄 스레드 생성
    pthread_create(&tid, NULL, recv_and_send, &sock);
}

void removeClient(int sock) {
    int i;
    //socklist 사용하기 때문에 임계 구역 잠금
    pthread_mutex_lock(&socklist_lock);
    for(i = 0; i < num_chat; i++)
        if(socklist[i] == sock) {
		        //만약 삭제할 소켓이 배열의 끝에 있는 것이 아니면 삭제할 소켓의 위치에
		        //맨 끝의 소켓 번호 가져오기
            if(i != num_chat - 1)
                socklist[i] = socklist[num_chat-1];
            num_chat--;
        }
    pthread_mutex_unlock(&socklist_lock);
}

void *recv_and_send(void *arg) {
    int i, nbyte, sock = *((int*)arg);
    char buf[MAXLINE+1];

    printf("thread %lu started.\n", pthread_self());
    while(1) {
		    //소켓으로 메시지가 왔을 시에 읽어들임
        nbyte = read(sock, buf, MAXLINE);
        if(nbyte <= 0) {
            close(sock);
            removeClient(sock);
            break;
        }

        buf[nbyte] = 0;
				//공유 자원 socklist를 사용하기 때문에 임계 구역 잠금
        pthread_mutex_lock(&socklist_lock);
        //모든 클라이언트에게 메시지 전달
        for(i = 0; i < num_chat; i++)
            write(socklist[i], buf, nbyte);
        pthread_mutex_unlock(&socklist_lock);
    }
    printf("thread %lu stopped.\n", pthread_self());
}
```

### mth_chatcli.c

```c
#include "netprog2.h"
#include <pthread.h>

#define MAXLINE 1000
#define NAME_LEN 20

char *EXIT_STRING= "exit";
void *recv_and_print(void *arg);

int main(int argc, char *argv[]){
    char bufall[MAXLINE+NAME_LEN], *bufmsg;
    int s, namelen;
    pthread_t tid;

    if(argc != 4) {
        printf("사용법 : %s server_ip port username \n", argv[0]);
        exit(0);
    }
    //bufall에 [이름] 저장
    sprintf(bufall, "[%s] : ", argv[3]);
    namelen = strlen(bufall);
    //bufmsg의 시작점 bufall에서 namelen길이만큼 간 곳
    bufmsg = bufall+namelen;
		
		//tcp_connect로 해당 주소 포트번호로 소켓 연결
    s = tcp_connect(AF_INET, argv[1], atoi(argv[2]));
    if(s == -1) {
        errquit("tcp_connect fail");
    }
    //main문에서는 메시지 보내기
    //스레드에서는 메시지 받고 출력( &s을 인자로 넘기기)
    pthread_create(&tid, NULL, recv_and_print, &s);

    puts("서버에 접속 되었습니다.");
    while(1) {
        if(fgets(bufmsg, MAXLINE, stdin)) {
		        //send(소켓, 내용, 내용의 크기, 0)
            if(send(s, bufall, namelen+strlen(bufmsg), 0) < 0)
                puts("Write error on socket");
            if(strstr(bufmsg, EXIT_STRING) !=NULL){
                puts("Good bye.");
                close(s);
                exit(0);
            }
  }
}
}

void *recv_and_print(void *arg){
		//소켓을 받을 때 s = *((int*)arg); 주의
    int nbyte, s = *((int*)arg);
    char buf[MAXLINE+1];
    while(1){
			  //recv(소켓, 받을 곳, 받는 곳 크기, 0)
        if((nbyte = recv(s, buf, MAXLINE, 0)) > 0) {
            buf[nbyte] = 0;
            printf("%s \n", buf);
        }
    }
}
```

## ⭐과제로 낸 멀티스레드 채팅 꼭 보기!!!

---

### ➡️조건 변수 (Condition Variable)

스레드 간에 시그널을 주고 받으며 특정 조건이 만족될 때까지 기다렸다가 깨어나는 동기화 수단

뮤텍스와 함께 사용되어 공유 자원의 상태 변화를 기다리는 스레드를 효율적으로 관리

스레드 간 생산자-소비자 패턴에서 주로 사용

- 생산자 : 데이터를 만들어 메시지 큐에 삽입
- 소비자 : 메시지 큐에서 데이터를 꺼냄
- 어떤 스레드가 먼저 수행될지 모르고, 소비자 스레드가 두 번 이상 실행되면 시간 낭비가 됨

따라서 조건 변수를 통해서 다른 스레드에게 공유 자원의 변화를 알려줌

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

//뮤텍스를 얻은 상태에서 써야하며, 자동으로 뮤텍스 해제 후
//블록 상태에서 대기, 조건 알림을 받으면 깨어나면서 뮤텍스 얻음
pthread_mutex_lock(&lock);
while(공유 자원 상태){ //여러 스레드가 접근할 수 있으므로
	pthread_cond_wait(&cond, &lock);
}
...
...
...
pthread_mutex_unlock(&lock);

//뮤텍스를 얻은 상태에서 써야하며, 임계 구역을 나가기 직전
//다른 스레드에게 시그널을 보내 공유 자원의 사용 끝을 알림
pthread_mutex_lock(&lock);
...
...
...
pthread_cond_broadcast(&cond);
pthread_mutex_unlock(&lock);
```

### producer_consumer.c

```c
#include "netprog2.h"
#include <pthread.h>

int resource_count = 0;
void *consumer_func(void *arg);
void *producer_func(void *arg);

pthread_mutex_t resource_lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t resource_cond = PTHREAD_COND_INITIALIZER;

#define NUM_CONSUMER 10

int main(int argc, char *argv[]){
        int i, id[NUM_CONSUMER];
        pthread_t consumer_thread[NUM_CONSUMER], producer_thread;

        for(i=0; i<NUM_CONSUMER; i++){
            id[i] = i;
            pthread_create(&consumer_thread[i], NULL, &consumer_func, &id[i]);
        }

        pthread_create(&producer_thread, NULL, &producer_func, NULL);
        pthread_join(producer_thread, NULL);

        return 0;
}

void *consumer_func(void *arg){
        int trial = 0, success = 0, id = *((int*)arg);

        while(1){
                pthread_mutex_lock(&resource_lock);

                while(resource_count == 0){
                        pthread_cond_wait(&resource_cond, &resource_lock);
                }
                trial++;
                if(resource_count > 0){
                        resource_count--;
                        success++;
                }
                pthread_mutex_unlock(&resource_lock);
                printf("%2d: trial = %d success = %d\n", id, trial, success);
                sleep(1);
        }
        return NULL;
}

void *producer_func(void *arg){
        while(1){
                pthread_mutex_lock(&resource_lock);
                resource_count++;

                pthread_cond_broadcast(&resource_cond);
                pthread_mutex_unlock(&resource_lock);
                sleep(1);
        }
        return NULL;
}
```