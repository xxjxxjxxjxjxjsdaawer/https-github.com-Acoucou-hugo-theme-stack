+++
author = "coucou"
title = "数据结构——实践"
date = "2023-08-01"
description = "数据结构专题之实践"
categories = [
    "数据结构"
]
tags = [
    "数据结构","实践"
]
+++

![](1.png)

## 数据结构——程序

**说明**： 以下的程序为数据结构基础测试，比如排序、栈、队列、链表的简单操作程序，还有其他一下简单算法，面试可能会考的简单题目，一般都是简单算法题目，相信难不倒大家，哈哈哈

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAXDATA 10

void sort_test01();
void sort_test02();
void stack_test();
void queue_test();
void list_test();
void string_test();

/* 冒泡排序-------------------------------------------- */
void sort_test01()
{
    int temp = 0;
    int arr[] = {3, 5, 6, 1, 2, MAXDATA, 14, 9, 0, 7};
    for(int i=0; i<MAXDATA-1; i++)
    {
        for(int j=i+1; j<MAXDATA; j++)
        {
            if(arr[i] > arr[j])
            {
                temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
    printf("sort_test01: ");
    for(int i=0; i<MAXDATA; i++)
    {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

/* 插入排序-------------------------------------------- */
void sort_test02()
{
    int arr[] = {3, 5, 6, 1, 2, 10, 14, 9, 0, 7};

    for(int i=1; i<MAXDATA; i++)
    {
        int temp = arr[i];
        int j;
        for(j=i-1; j>=0, arr[j]>temp; j--)
        {
            arr[j+1] = arr[j];
        }
        arr[j+1] = temp;
    }
    printf("sort_test02: ");
    for(int i=0; i<MAXDATA; i++)
    {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

/* 队列-------------------------------------------- */
typedef struct QUEUE{
    int data[MAXDATA];
    int head;
    int rear;
}MyQueue;
void queue_push(MyQueue *queue, int data)
{
    if((queue->rear + 1) % MAXDATA == queue->head)
    {
        printf("queue is full");
    }
    queue->data[queue->rear] = data;
    queue->rear = (queue->rear + 1) % MAXDATA;
}
int queue_pop(MyQueue *queue)
{
    int data = 0;
    if(queue->head == queue->rear)
    {
        return 0;
    }
    
    data = queue->data[queue->head];
    queue->head = (queue->head - 1) % MAXDATA;

    return data;
}
void queue_test()
{
    int data = 0;
    MyQueue queue;
    queue.head = 0;
    queue.rear = 0;

    queue_push(&queue, 1);
    queue_push(&queue, 8);

    data = queue_pop(&queue);
    printf("queue_test: %d\n", data);
}

/* 栈-------------------------------------------- */
typedef struct STACK{
    int data[MAXDATA];
    int top;
}MyStack;
void stack_push(MyStack *st, int data)
{
    if(st->top == MAXDATA-1)
    {
        printf("stack is full");
    }
    st->top++;
    st->data[st->top] = data;
}
int stack_pop(MyStack *st)
{
    int ret = 0;
    if(st->top == -1)
    {
        return 0;
    }
    ret = st->data[st->top];
    st->top--;

    return ret;
}

void stack_test()
{
    int data = 0;
    MyStack stack;
    stack.top = -1;

    stack_push(&stack, 3);
    stack_push(&stack, 4);

    data = stack_pop(&stack);

    printf("stack_test: %d\n", data);
}

/* 链表-------------------------------------------- */
typedef struct NODE{
    int data;
    struct NODE *next;
}MyList;
MyList *list_insert(MyList *head, int data)
{
    MyList *currentNode = head;
    MyList *newnode = (MyList *)malloc(sizeof(MyList));
    if(newnode == NULL)
    {
        exit(1);
    }

    newnode->data = data;
    newnode->next = NULL;

    if(head == NULL)
    {
        return newnode;
    }

    while(currentNode->next != NULL)
    {
        currentNode = currentNode->next;
    }

    currentNode->next = newnode;

    return head;
}

void print_list(MyList *list)
{
    MyList *temp_list = list;
    while(temp_list != NULL)
    {
        printf("%d ", temp_list->data);
        temp_list = temp_list->next;
    }
}

void list_test()
{
    MyList *list = NULL;

    list = list_insert(list, 2);
    list = list_insert(list, 4);

    printf("list_test:");
    print_list(list);
    printf("\n");
}

/* 大小端-------------------------------------------- */
union BYTE
{
    short data;
    char ch[2];
}byte;

void byte_test()
{   
    byte.data = 0x1234;
    if(byte.ch[0] == 0x12 && byte.ch[1] == 0x34)
    {
        printf("byte_test1: This is big type\n");
    }
    else if(byte.ch[0] == 0x34 && byte.ch[1] == 0x12)
    {
        printf("byte_test1: This is litlle type\n");
    }

    short data = 0x1234;
    char *ch = (char *)&data;
    if(*ch == 0x12)
    {
        printf("byte_test2: This is big type\n");
    }
    else if(*ch == 0x34)
    {
        printf("byte_test2: This is litlle type\n");
    }
    // ch = ch + 1;
    // if(*ch == 0x12)
    // {
    //     printf("really\n");
    // }
}

/* 字符串内存占用-------------------------------------------- */
void StrSize_test()
{
    char ch[] = {'1', 'a', 'c', 'd'};
    char ch2[] = "abcd1";
    char ch3[5] = "abcd1";
    char *str = "1";

    printf("StrSize_test: ch>> %d ch2>> %d ch3>> %d str>> %d\n", \
            sizeof(ch), sizeof(ch2), sizeof(ch3), sizeof(str));
}
/* 字符串查找 -------------------------------------------- */
void StrFind_test()
{
    char *str = "aabsafdcaadd";
    char *str1 = strchr(str, 'f');
    char *str2 = strstr(str, "saf");
    int index = str2 - str;

    printf("StrFind_test: str1>> %s str2>> %s idx: %d \n", str1, str2, index);

}

/* 指针 -------------------------------------------- */
void point_test()
{
    int *p = NULL;
    int a[4]={1,2,3,4};

    p=a;
    printf("point_test: *p++>>%d ",*p++);

    p=a;
    printf("*(p++)>>%d ",*(p++));

    p=a;
    printf("(*p)++>>%d ",(*p)++);

    p=a;
    printf("*++p>>%d ",*++p);

    p=a;
    printf("++*p>>%d ", ++*p);  
}

void memery_copy(char *dest, char *src)
{
    if(src == NULL)
    {
        dest = NULL;
    }
    while (*src != '\0')
    {
        *dest++ = *src++;
    }
    *dest = '\0';
}
/* 链表简单操作
bool linkring(link *head)
{
    link *p = head;
    link *p2 = head;

    while(p->next && p2->next)
    {
        p = p->next;
        p2 = p2->next->next;

        if(p == p2)
        {
            return true;
        }
    }

    return false;
}

void deletelink(link **head, link *p)
{
    p->pre->next = p->next;
    p->next->pre = p->pre;

    if(p == *head)
    {
        *head = p->next;
    }
    free(p);
}

void insertlink(link *p, int data)
{
    link *next = NULL;
    next = (link *)malloc(sizeof(link));
    next->data = data;

    
    next->pre = p;

    next->next = p->next;
    p->next->pre = next;
    
    p->next = next;
}

void reservelink(link *head)
{
    link *pre = head;
    link *next = head->next;

    while(next)
    {
        pre->next = next->next;
        next = hand;
        head = next;
        next = pre->next;
    }
}

*/
int main()
{
    sort_test01();
    sort_test02();
    stack_test();
    queue_test();
    list_test();

    byte_test();  // 大小端判断
    StrSize_test();
    StrFind_test();
    point_test();

    return 0;
}
```

