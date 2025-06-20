# 11장 : 시스템 V의 프로세스 간 통신

### 키 생성하기 : ftok (3)

함수 원형 : key_t ftok(const char *pathname, int proj_id);

경로명 + 임의의 번호를 통해서 IPC 객체 생성할 때 쓰이는 키 값 생성 후 반환

### 메시지 큐 식별자 생성 : msgget (2)

함수 원형 : int msgget(key_t key, int msgflg);

### 메시지 전송 : msgsnd (2)

함수 원형 : int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);

```c
typedef struct mymsgbuf{
        long mtype;
        char mtext[80];
}Message; //이렇게 만들어야함

int main(){
        key_t key;
        int msgid;
        Message msg;

        key = ftok("keyfile", 1);
        msgid = msgget(key, IPC_CREAT | 0644);
        if(msgid == -1){
                perror("msgget");
                exit(1);
        }

        msg.mtype = 1;
        strcpy(msg.mtext, "Message Q Test");

        if(msgsnd(msgid, (void *)&msg, 80, IPC_NOWAIT) == -1){
                perror("msgmd");
                exit(1);
        }
        return 0;
}
```

---

### 메시지 수신 : msgrcv()

함수 원형 : ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

msgtyp이 0일시 메시지 큐 가장 앞 메시지 읽어오기, 양수일시 msgtyp과 같은 유형의 메시지 읽어오기.

msgflg값이 0일시 메시지가 올 때까지 기다림. IPC_NOWAIT의 경우 기다리지 않고 없을시 오류 리턴

```c
struct mymsgbuf{
        long mtype;
        char mtext[80];
};

int main(){
        struct mymsgbuf inmsg;
        key_t key;
        int msgid, len;

        key = ftok("keyfile", 1);
        if((msgid = msgget(key, 0)) < 0) {
                perror("msgget");
                exit(1);
        }

        len = msgrcv(msgid, &inmsg, 80, 0, 0);
        printf("Received Msg = %s, Len=%d\n", inmsg.mtext, len);
}
```

---

### 메시지 제어 : msgctl (2)

함수 원형 : msgctl(int msqid, int cmd, struct msqid_ds * buf);

```c
int main(){
        key_t key;
        int msgid;

        key = ftok("keyfile", 1);
        msgid = msgget(key, IPC_CREAT | 0644);
        if(msgid == -1){
                perror("msgget");
                exit(1);
        }

        printf("Before IPC_RMID\n");
        system("ipcs -q");
        msgctl(msgid, IPC_RMID, (struct msqid_ds *)NULL);
        printf("Atfer IPC_RMID\n");
        system("ipcs -q");
}
```

### << 제어 cmd 종류 작성해가기

---

## 공유 메모리 & 세마포어

### 공유 메모리 식별자 생성 : shmget (2)

함수 원형 : int shmget(key_t key, size_t size, int shmflg);

```c
int main(){
        key_t key;
        int shmid;

        key = ftok("shmfile", 1);

        shmid = shmget(key, 1024, IPC_CREAT|0644);
        if(shmid == -1){
                perror("shmget");
                exit(1);
        }
}
```

### 공유 메모리 연결 : shmat (2)

함수 원형 : void *stmat(int shmid, const void *shmaddr, int shmflg);

### 공유 메모리 연결 해제 : shmdt (2)

함수 원형 : int shmdt(cons void *shmaddr);

### 공유 메모리 제어 : shmctl (2)

함수 원형 : int shmctl(int shmid, int cmd, struct shmid_ds *buf);

### << 제어 종류 작성해가기

### 공유 메모리 예제 11-5

```c
int main(){
        int shmid, i;
        char *shmaddr, *shmaddr2;

        shmid = shmget(IPC_PRIVATE, 20, IPC_CREAT | 0644);
        if(shmid == -1){
                perror("shmget");
                exit(1);
        }

        switch(fork()){
                case -1:
                        perror("fork");
                        exit(1);
                        break;
                case 0:
                        shmaddr = (char *)shmat(shmid, (char *)NULL, 0);
                        printf("Child Process =====\n");
                        for(i=0; i<10; i++){
                                shmaddr[i] = 'a' + i;
                        }
                        shmdt((char *)shmaddr);
                        exit(0);
                        break;
                default:
				                wait(NULL);
                        shmaddr2 = (char *)shmat(shmid, (char *)NULL, 0);
                        printf("Parent Process =====\n");
                        for(i=0; i<10; i++){
                                printf("%c ", shmaddr2[i]);
                        }
                        printf("\n");
                        sleep(5);
                        shmdt((char *)shmaddr2);
                        shmctl(shmid, IPC_RMID, (struct shmid_ds *)NULL);
                        break;
        }
}
```

### 세마포어 생성 : semget (2)

함수 원형 : int semget(key_t key, int nsems, int semflg);

### 세마포어 제어 : semctl (2)

함수 원형 : int semctl(int semid, int semnum, int cmd, …);

### << 제어 종류 작성해가기

### 세마포어 연산 : semop (2)

함수 원형 : semop(int semid, struct sembuf *sops, size_t nsops);

```c
struct sembuf{
	unsigned short sem_num; //세마포어 번호
	short semp_op; // 세마포어 연산
	short sem_flg; //연산 플래그, IPC_NOWAIT 등
}
```

sem_op < 0 : 세마포어 잠금

sem_op > 0 : 세마포어 잠금 해제

### 사용 예제 11-7

```c
union semun {
        int val;
        struct semid_ds *buf;
        unsigned short *array;
};

int initsem(key_t semkey){
        union semun semunarg;
        int status = 0, semid;

        semid = semget(semkey, 1, IPC_CREAT | IPC_EXCL | 0600);
        if(semid == -1){
                if(errno == EEXIST){
                        semid = semget(semkey, 1, 0);
                }
        } else{
                semunarg.val = 1;
                status = semctl(semid, 0, SETVAL, semunarg);
        }
        return semid;
}

int semlock(int semid){
        struct sembuf buf;

        buf.sem_num = 0;
        buf.sem_op = -1;
        buf.sem_flg =SEM_UNDO;
        if(semop(semid, &buf, 1) == -1){
                perror("semlock failed");
                exit(1);
        }
        return 0;
}
int semunlock(int semid){
        struct sembuf buf;

        buf.sem_num = 0;
        buf.sem_op = 1;
        buf.sem_flg = SEM_UNDO;
        if(semop(semid, &buf, 1) == -1){
                perror("semunlock failed");
                exit(1);
        }
        return 0;
}

void semhandle(){
        int semid;
        pid_t pid = getpid();

        if((semid = initsem(1)) < 0)
                exit(1);

        semlock(semid);
        printf("Lock : Process %d\n", (int)pid);
        printf("** Lock Mode : Critical Section\n");
        sleep(1);
        printf("UnLock : Process %d\n", (int)pid);
        semunlock(semid);

        exit(0);
}

int main(){
        int a;
        for(a=0; a<3; a++)
                if(fork() == 0) semhandle();
}
```

### 쉬운 세마포어 통신 예제

1. Writer

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <stdlib.h>
#include <string.h>
#include "ipc_comm.h"

union semun {
    int val;
};

int main() {
    int shmid = shmget(SHM_KEY, SHM_SIZE, IPC_CREAT | 0666);
    int semid = semget(SEM_KEY, 1, IPC_CREAT | 0666);

    if (shmid == -1 || semid == -1) {
        perror("IPC setup failed");
        exit(1);
    }

    // 세마포어 초기화: 값 = 0 (잠금)
    union semun arg;
    arg.val = 0;
    semctl(semid, 0, SETVAL, arg);

    char *shmaddr = (char *)shmat(shmid, NULL, 0);

    printf("Writer: Enter message: ");
    fgets(shmaddr, SHM_SIZE, stdin);

    // 세마포어 unlock (값 1 증가)
    struct sembuf sem_unlock = {0, 1, 0};
    semop(semid, &sem_unlock, 1);

    printf("Writer: Message written to shared memory.\n");

    shmdt(shmaddr);
    return 0;
}

```

1. Reader

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <stdlib.h>
#include "ipc_comm.h"

int main() {
    int shmid = shmget(SHM_KEY, SHM_SIZE, 0666);
    int semid = semget(SEM_KEY, 1, 0666);

    if (shmid == -1 || semid == -1) {
        perror("IPC access failed");
        exit(1);
    }

    // 세마포어 lock (값이 0이상이 될 때까지 대기)
    struct sembuf sem_lock = {0, -1, 0};
    semop(semid, &sem_lock, 1);

    char *shmaddr = (char *)shmat(shmid, NULL, 0);
    printf("Reader: Message received: %s", shmaddr);

    shmdt(shmaddr);

    // cleanup: 공유 자원 제거 (보통 마지막 프로세스가 함)
    shmctl(shmid, IPC_RMID, NULL);
    semctl(semid, 0, IPC_RMID);

    return 0;
}

```