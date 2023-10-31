+++
author = "coucou"
title = "开发工具——Cjson"
date = "2023-08-01"
description = "开发工具专题之Cjson"
categories = [
    "开发工具"
]
tags = [
    "开发工具","Cjson"
]
+++

![](1.jpg)

## mqtt+esp8266+单片机通信

__注：这里特别说明，在传输数据的时候特别建议在数据末尾加上 "\r\n", 避免出现各种问题__

### 1. C语言直接编写

```c
// 传进来的参数pcRes需要申请空间，不然取值失败
char *pcRes = (char*)malloc(sizeof(char) * 100);  // 解析出的数据
int fingString(char *pcBuf, char *left, char *right, char *pcRes){
	char *pcBegin = NULL;
	char *pcEnd = NULL;

	pcBegin = strstr(pcBuf, left);

	pcEnd = strstr(pcBegin + strlen(left) + 3, right);

	if(pcBegin == NULL || pcEnd == NULL || pcBegin > pcEnd){
		return 0;
	}else{
		pcBegin += strlen(left);
		memcpy(pcRes, pcBegin + 3, pcEnd - pcBegin - 3);
		return 1;
	}
}
// 数据格式json
// 例：{"msg":"led_on","temp":"32" }

```

### 2. cJSON库

#### 2.1 cJSON数据封装

```c
// 直接上例程，需要运行源文件需包含cJSON.c以及cJSON.h
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "cJSON.h"

/* Used by some code below as an example datatype. */
struct record
{
    const char *precision;
    double lat;
    double lon;
    const char *address;
    const char *city;
    const char *state;
    const char *zip;
    const char *country;
};


/* Create a bunch of objects as demonstration. */
static int print_preallocated(cJSON *root)
{
    /* declarations */
    char *out = NULL;
    char *buf = NULL;
    char *buf_fail = NULL;
    size_t len = 0;
    size_t len_fail = 0;

    /* formatted print */
    out = cJSON_Print(root);

    /* create buffer to succeed */
    /* the extra 5 bytes are because of inaccuracies when reserving memory */
    len = strlen(out) + 5;
    buf = (char*)malloc(len);
    if (buf == NULL)
    {
        printf("Failed to allocate memory.\n");
        exit(1);
    }

    /* create buffer to fail */
    len_fail = strlen(out);
    buf_fail = (char*)malloc(len_fail);
    if (buf_fail == NULL)
    {
        printf("Failed to allocate memory.\n");
        exit(1);
    }

    /* Print to buffer */
    if (!cJSON_PrintPreallocated(root, buf, (int)len, 1)) {
        printf("cJSON_PrintPreallocated failed!\n");
        if (strcmp(out, buf) != 0) {
            printf("cJSON_PrintPreallocated not the same as cJSON_Print!\n");
            printf("cJSON_Print result:\n%s\n", out);
            printf("cJSON_PrintPreallocated result:\n%s\n", buf);
        }
        free(out);
        free(buf_fail);
        free(buf);
        return -1;
    }

    /* success */
    printf("%s\n", buf);

    /* force it to fail */
    if (cJSON_PrintPreallocated(root, buf_fail, (int)len_fail, 1)) {
        printf("cJSON_PrintPreallocated failed to show error with insufficient memory!\n");
        printf("cJSON_Print result:\n%s\n", out);
        printf("cJSON_PrintPreallocated result:\n%s\n", buf_fail);
        free(out);
        free(buf_fail);
        free(buf);
        return -1;
    }

    free(out);
    free(buf_fail);
    free(buf);
    return 0;
}

/* Create a bunch of objects as demonstration. */
static void create_objects(void)
{
    /* declare a few. */
    cJSON *root = NULL;
    cJSON *fmt = NULL;
    cJSON *img = NULL;
    cJSON *thm = NULL;
    cJSON *fld = NULL;
    int i = 0;

    /* Our "days of the week" array: */
    const char *strings[7] =
    {
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
    };
    /* Our matrix: */
    int numbers[3][3] =
    {
        {0, -1, 0},
        {1, 0, 0},
        {0 ,0, 1}
    };
    /* Our "gallery" item: */
    int ids[4] = { 116, 943, 234, 38793 };
    /* Our array of "records": */
    struct record fields[2] =
    {
        {
            "zip",
            37.7668,
            -1.223959e+2,
            "",
            "SAN FRANCISCO",
            "CA",
            "94107",
            "US"
        },
        {
            "zip",
            37.371991,
            -1.22026e+2,
            "",
            "SUNNYVALE",
            "CA",
            "94085",
            "US"
        }
    };
    volatile double zero = 0.0;

    /* Here we construct some JSON standards, from the JSON site. */

    /* Our "Video" datatype: */
    root = cJSON_CreateObject();
    cJSON_AddItemToObject(root, "name", cJSON_CreateString("Jack (\"Bee\") Nimble"));
    cJSON_AddItemToObject(root, "format", fmt = cJSON_CreateObject());
    cJSON_AddStringToObject(fmt, "type", "rect");
    cJSON_AddNumberToObject(fmt, "width", 1920);
    cJSON_AddNumberToObject(fmt, "height", 1080);
    cJSON_AddFalseToObject (fmt, "interlace");
    cJSON_AddNumberToObject(fmt, "frame rate", 24);

    /* Print to text */
    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);

    /* Our "days of the week" array: */
    root = cJSON_CreateStringArray(strings, 7);

    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);

    /* Our matrix: */
    root = cJSON_CreateArray();
    for (i = 0; i < 3; i++)
    {
        cJSON_AddItemToArray(root, cJSON_CreateIntArray(numbers[i], 3));
    }

    /* cJSON_ReplaceItemInArray(root, 1, cJSON_CreateString("Replacement")); */

    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);

    /* Our "gallery" item: */
    root = cJSON_CreateObject();
    cJSON_AddItemToObject(root, "Image", img = cJSON_CreateObject());
    cJSON_AddNumberToObject(img, "Width", 800);
    cJSON_AddNumberToObject(img, "Height", 600);
    cJSON_AddStringToObject(img, "Title", "View from 15th Floor");
    cJSON_AddItemToObject(img, "Thumbnail", thm = cJSON_CreateObject());
    cJSON_AddStringToObject(thm, "Url", "http:/*www.example.com/image/481989943");
    cJSON_AddNumberToObject(thm, "Height", 125);
    cJSON_AddStringToObject(thm, "Width", "100");
    cJSON_AddItemToObject(img, "IDs", cJSON_CreateIntArray(ids, 4));

    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);

    /* Our array of "records": */
    root = cJSON_CreateArray();
    for (i = 0; i < 2; i++)
    {
        cJSON_AddItemToArray(root, fld = cJSON_CreateObject());
        cJSON_AddStringToObject(fld, "precision", fields[i].precision);
        cJSON_AddNumberToObject(fld, "Latitude", fields[i].lat);
        cJSON_AddNumberToObject(fld, "Longitude", fields[i].lon);
        cJSON_AddStringToObject(fld, "Address", fields[i].address);
        cJSON_AddStringToObject(fld, "City", fields[i].city);
        cJSON_AddStringToObject(fld, "State", fields[i].state);
        cJSON_AddStringToObject(fld, "Zip", fields[i].zip);
        cJSON_AddStringToObject(fld, "Country", fields[i].country);
    }

    /* cJSON_ReplaceItemInObject(cJSON_GetArrayItem(root, 1), "City", cJSON_CreateIntArray(ids, 4)); */

    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);

    root = cJSON_CreateObject();
    cJSON_AddNumberToObject(root, "number", 1.0 / zero);

    if (print_preallocated(root) != 0) {
        cJSON_Delete(root);
        exit(EXIT_FAILURE);
    }
    cJSON_Delete(root);
}

int main(void)
{
    /* print the version */
    printf("Version: %s\n", cJSON_Version());

    /* Now some samplecode for building objects concisely: */
    create_objects();

    return 0;
}
```

#### 2.2 cJSON数据解析

##### 2.2.1 cJSON结构体

```c
// cJSON结构体
typedef struct cJSON 
{     //cJSON结构体
       struct cJSON *next,*prev;               /* 遍历数组或对象链的前向或后向链表指针*/
       struct cJSON *child;                   /* 数组或对象的孩子节点*/
       int type;                              /* key的类型*/
       char *valuestring;                     /* 字符串值*/
       int valueint;                          /* 整数值*/
       double valuedouble;                    /* 浮点数值*/
       char *string;                          /* key的名字*/
} cJSON;
```

##### 2.2.2 cJSON常用函数

```c
cJSON *cJSON_Parse(const char *value);
// 作用：将一个JSON数据包，按照cJSON结构体的结构序列化整个数据包，并在堆中开辟一块内存存储cJSON结构体
// 返回值：成功返回一个指向内存块中的cJSON的指针，失败返回NULL

cJSON *cJSON_GetObjectItem(cJSON *object,const char *string);
// 作用:获取JSON字符串字段值
// 返回值：成功返回一个指向cJSON类型的结构体指针，失败返回NULL

char  *cJSON_Print(cJSON *item);
// 作用：将cJSON数据解析成JSON字符串，并在堆中开辟一块char*的内存空间存储JSON字符串
// 返回值：成功返回一个char*指针该指针指向位于堆中JSON字符串，失败返回NULL
    
void  cJSON_Delete(cJSON *c);
// 作用：释放位于堆中cJSON结构体内存
// 返回值：无
```

##### 2.2.3 例程1

```c
#include <stdio.h>
#include "cJSON.h"
 
int main()
{
	char json_string[] ="{	\"cmd\":12,				\
	 						\"device\":\"lamp\",	\
	 						\"power\":1,			\
	 						\"brightness\":50		\
	 					}";									
	 //JSON字符串到cJSON格式
	cJSON* cjson = cJSON_Parse(json_string); 
	//判断cJSON_Parse函数返回值确定是否打包成功
	if(cjson == NULL)
	{
	printf("json pack into cjson error...");
	}
	else
	{//打包成功调用cJSON_Print打印输出
	    printf("%s\r\n",cJSON_Print(cjson));
	}
 
	//获取字段值
	//cJSON_GetObjectltem返回的是一个cJSON结构体所以我们可以通过函数返回结构体的方式选择返回类型！
	int test_1_string = cJSON_GetObjectItem(cjson,"cmd")->valueint;
	char* test_2_string = cJSON_GetObjectItem(cjson,"device")->valuestring;
	int test_3_string = cJSON_GetObjectItem(cjson,"power")->valueint;
	int test_4_string = cJSON_GetObjectItem(cjson,"brightness")->valueint;
 
	//打印输出
	printf("%d\r\n",test_1_string);
	printf("%s\r\n",test_2_string);
	printf("%d\r\n",test_3_string);
	printf("%d\r\n",test_4_string);
	 
	//delete cjson
	cJSON_Delete(cjson);
}
```

##### 2.2.3 例程2（解析数组）

```c
#include <stdio.h>
#include "cJSON.h"
 
int main()
{
	char json_arr_string[]="{\"test_arr\":[{\"test_1\":\"arr_1\",\"test_2\":\"arr_2\",\"test_3\":\"arr_3\"},{\"test_1\":\"1\",\"test_2\":\"2\",\"test_3\":\"3\"}]}";	
 
	cJSON* cjson = cJSON_Parse(json_arr_string);
	if(cjson == NULL)
	{
		printf("cjson error...\r\n");
	}
	else
	{//打包成功调用cJSON_Print打印输出
		printf("%s\r\n",cJSON_Print(cjson));
	}
	cJSON* test_arr = cJSON_GetObjectItem(cjson,"test_arr");
	int arr_size = cJSON_GetArraySize(test_arr);//return arr_size 2
	cJSON* arr_item = test_arr->child;//子对象
 
	for(int i = 0;i <=(arr_size-1)/*0*/;++i)
	{
		printf("%s\r\n",cJSON_Print(cJSON_GetObjectItem(arr_item,"test_1")));
		printf("%s\r\n",cJSON_Print(cJSON_GetObjectItem(arr_item,"test_2")));
		printf("%s\r\n",cJSON_Print(cJSON_GetObjectItem(arr_item,"test_3")));
		arr_item = arr_item->next;//下一个子对象
	}
	cJSON_Delete(cjson);
}
 
```

#### 2.3 cJSON基础

1、构造json对象；2、向对象中添加元素；3、获取对象中元素；4、获取元素的值；5、json对象与数组互转；6、释放json对象空间；

从官网直接下载cJSON源文件，添加到工程中即可。

char jssd_def[256];

##### 2.3.1 构造json对象

```c
cJSON* root = NULL;
cJSON* item = NULL;
item = cJSON_CreateObject();
root = cJSON_CreateObject();
```

##### 2.3.2 向对象中添加元素

```c
// 1、将对象作为元素进行添加：
cJSON_AddItemToObject(item, "name", cJSON_CreateString("xiaopeng"));
cJSON_AddItemToObject(item,"age",cJSON_CreateNumber(21));
cJSON_AddItemToObject(root,"root",item);

// 2、直接添加元素：
cJSON_AddItemToObject(root,"ID",cJSON_CreateNumber(1));
```

##### 2.3.3 获取对象中元素

```c
root = cJSON_Parse(jssd_def);
item = cJSON_GetObjectItem(root, "root");
```

##### 2.3.4 获取元素的值

```c
cJSON* age = NULL;
age = cJSON_GetObjectItem(item, "age");
printf( "age=%d\r\n", age->valueint );
```

##### 2.3.5 json对象与数组互转

```c
// 1、json对象转字符串
sprintf(jssd_def, "%s", cJSON_Print(root));//带格式化\n\t
sprintf(jssd_def, "%s", cJSON_PrintUnformatted(root));//不带格式

// 2、字符串转json对象
root = cJSON_Parse(jssd_def);
```

##### 2.3.6 释放json对象空间

 ````c
// 如果使用cJSON_Delete(cJSON *c);进行对象释放：当一个对象是另一个对象的元素时，只调用外层对象释放，否则会出现硬件错误；如果内层对象被释放，再释放外层对象，也会出现硬件错误。如果不放心，则可以直接使用free进行多次释放。

//cJSON_Delete( root );等价于{free( item );free( root );}
//警告：使用cJSON_Delete( root );则不能使用cJSON_Delete( item );
cJSON_Delete( root );
 ````

