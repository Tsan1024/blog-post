---
title:       "C++进程间通信"
subtitle:    ""
description: ""
date:        2024-05-30T16:38:45+08:00
author:      "tsand"
image:       ""
tags:        ["c++"]
categories:  ["Tech" ]
---

## 分类
* 管道（Pipes）

  - 命名管道（Named Pipes）：适用于无关进程之间的通信。命名管道有一个名称，可以在文件系统中进行标识。
  - 无名管道（Unnamed Pipes）：通常用于父子进程之间的通信，因为它们在创建时不具备名称。

* 消息队列（Message Queues）
    提供了一种通过消息发送和接收来进行进程间通信的方法。消息队列允许进程以一种异步方式传递数据。

* 共享内存（Shared Memory）
    允许多个进程共享一个内存段，从而实现最快速的进程通信。需要同步机制（如信号量）来避免数据竞争。

* 信号（Signals）
    主要用于通知进程某个事件的发生，可以用来处理简单的通知和中断。

* 信号量（Semaphores）
    通常用于控制对共享资源的访问。信号量可以是计数信号量，也可以是二元信号量（即互斥锁）。

* 套接字（Sockets）
    虽然通常用于网络通信，但也可以用于同一主机上不同进程之间的通信。套接字支持多种通信协议（如TCP和UDP）。
* 内存映射文件（Memory-Mapped Files）

    通过将文件映射到内存地址空间，实现文件内容的共享，进而实现进程间通信。

## 示例

### 管道
无名管道
```
#include <iostream>
#include <unistd.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char buffer[20];

    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();
    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // 子进程
        std::cout << "child: " << pid << std::endl;
        close(pipefd[1]); // 关闭写端
        read(pipefd[0], buffer, sizeof(buffer));
        std::cout << "Child received: " << buffer << std::endl;
        close(pipefd[0]);
    } else { // 父进程
        std::cout << "father: " << pid << std::endl;
        close(pipefd[0]); // 关闭读端
        write(pipefd[1], "Hello, world!", 14);
        close(pipefd[1]);
        wait(NULL); // 等待子进程结束
    }

    return 0;
}
```

有名管道
```
write.cpp
// writer.cpp
#include <iostream>
#include <fcntl.h>
#include <sys/stat.h>
#include <unistd.h>
#include <cstring>

int main() {
    const char *fifo_path = "/tmp/my_fifo";
    
    // 创建命名管道（如果已经存在则不会重新创建）mkfifo create fifo
    // if (mkfifo(fifo_path, 0666) == -1) {
    //     perror("mkfifo");
    //     return 1;
    // }

    // 打开命名管道
    int fd = open(fifo_path, O_WRONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // 写入消息到管道
    const char *message = "Hello from writer!";
    if (write(fd, message, strlen(message) + 1) == -1) {
        perror("write");
        close(fd);
        return 1;
    }

    // 关闭管道
    close(fd);

    return 0;
}
```

```
read.cpp
// reader.cpp
#include <iostream>
#include <fcntl.h>
#include <sys/stat.h>
#include <unistd.h>

int main() {
    const char *fifo_path = "/tmp/my_fifo";
    
    // 创建命名管道（如果已经存在则不会重新创建）
    if (mkfifo(fifo_path, 0666) == -1) {
        perror("mkfifo");
        return 1;
    }

    // 打开命名管道
    int fd = open(fifo_path, O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // 从管道读取消息
    char buffer[128];
    if (read(fd, buffer, sizeof(buffer)) == -1) {
        perror("read");
        close(fd);
        return 1;
    }

    // 输出读取的消息
    std::cout << "Reader received: " << buffer << std::endl;

    // 关闭管道
    close(fd);

    return 0;
}

```
### 消息队列
```
#include<stdio.h>
#include<unistd.h>
#include<sys/ipc.h>
#include<sys/msg.h>

#define KEY 0x9999

typedef struct{
    long msgType;
    char data[80];
}Msg;

// Msg可以随意定义但需要含有一个消息类型type，而且还必须是结构体的第一个成员。而结构体中的其他成员都被认为是要发送的消息体数据。

int main(void){
    pid_t pid;
    int msgid;
    int i = 0;
    int ret;
    Msg msg;

    pid = fork();
    if(pid == 0){ 
        msgid = msgget(KEY, IPC_CREAT | 0666);
        if(msgid == -1){
            perror("");
            return -1; 
        }   
        while(1){
            msg.msgType = 1;
            sprintf(msg.data, "Hello I am sender %d", i);
            msgsnd(msgid, &msg, sizeof(Msg) - sizeof(long), 0);
            i++;
            sleep(1);
        }
    }else if(pid > 0){
        msgid = msgget(KEY, IPC_CREAT | 0666);
        if(msgid == -1){
            perror("");
            return -1;
        }
        while(1){
            ret = msgrcv(msgid, &msg, sizeof(Msg) - sizeof(long), 1, 0);
            if(ret < 0){
                perror("");
                return -1;
            }
            printf("Father RCV :%s\n", msg.data);
        }


    }else{
        perror("");
        return -1;
    }   

    return 0;
}
```
