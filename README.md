# Ti-CC1350-Uartecho
<br/>
CC1350 Uart介面程式說明，以下範例都是根據Ti的Resource Explorer<br/>
<br/>
首先最先執行的會是main()這個func，Ti driver裡的Ti-RTOS都會由main_tirtos.c開始執行並開啟一個叫mainThread的Pthread<br/>
<br/>
首先先載入所需要用到的函式庫<br/>

```
#include <stdint.h>   //載入基本函式庫stdint.h

/* POSIX Header files */
#include <pthread.h>  //Pthread函式庫

/* RTOS header files */
#include <ti/sysbios/BIOS.h>  //RTOS所需的函式庫

/* Example/Board Header files */
#include <ti/drivers/Board.h>  //開發版上各式腳位等定義函式庫

extern void *mainThread(void *arg0);  //宣告一個mainThread的func，之後的邏輯都會在這裡面

/* Stack size in bytes */
#define THREADSTACKSIZE    1024  //宣告一個變數用於設定執行緒的堆疊大小，如果沒有沒有設定會採用 ulimit -s 的預設值，預設是8M
```

<br/>
接下來就是main的主程式<br/>
<a href="https://blog.csdn.net/mijichui2153/article/details/82855925" target="_blank">pthread_create參數說明</a><br/>
<a href="https://shengyu7697.github.io/cpp-pthread_attr_setstacksize/" target="_blank">pthread_attr_setstacksize參數說明</a><br/>
<a href="https://www.796t.com/content/1549363165.html" target="_blank">pthread_attr_setdetachstate參數說明</a><br/>

```
/*
 *  ======== main ========
 */
int main(void)
{
    pthread_t           thread;  //宣告一個變數儲存線程
    pthread_attr_t      attrs;   //線程類型
    struct sched_param  priParam;  //宣告一個priParam來儲存線程的參數設定
    int                 retc;  //

    /* Call driver init functions */
    Board_init();  //開發版初始化

    /* Initialize the attributes structure with default values */
    pthread_attr_init(&attrs); //線程初始化

    /* Set priority, detach state, and stack size attributes */
    priParam.sched_priority = 1; //設定線程的優先順序 1~99，1最小 0最大
    retc = pthread_attr_setschedparam(&attrs, &priParam); //設定線程的類型
    retc |= pthread_attr_setdetachstate(&attrs, PTHREAD_CREATE_DETACHED); //執行緒自己自動釋放所有資源
    retc |= pthread_attr_setstacksize(&attrs, THREADSTACKSIZE); //宣告一個變數用於設定執行緒的堆疊大小，如果沒有沒有設定會採用 ulimit -s 的預設值，預設是8M
    if (retc != 0) {
        /* failed to set attributes 設定失敗 */
        while (1) {}
    }

    retc = pthread_create(&thread, &attrs, mainThread, NULL); //建立線程為mainThread
    if (retc != 0) {
        /* pthread_create() failed */
        while (1) {}
    }

    BIOS_start();

    return (0);
}
```
