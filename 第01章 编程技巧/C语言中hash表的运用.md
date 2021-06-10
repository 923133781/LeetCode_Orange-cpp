



uthash 是C的比较优秀的开源代码，它实现了常见的hash操作函数，例如查找、插入、删除等待。该套开源代码采用宏的方式实现hash函数的相关功能，支持C语言的任意数据结构最为key值，甚至可以采用多个值作为key，无论是自定义的struct还是基本数据类型，需要注意的是不同类型的key其操作接口方式略有不通。

   使用uthash代码时只需要包含头文件"uthash.h"即可。由于该代码采用宏的方式实现，所有的实现代码都在uthash.h文件中，因此只需要在自己的代码中包含该头文件即可。可以通过下面两种方式获取源代码：

- 通过官方下载链接：

https://github.com/troydhanson/uthash

- 另外，uthash的英文使用文档介绍可从下面网址获得：

http://troydhanson.github.io/uthash/userguide.html#_add_item

**1．uthash的效率**

   uthash的插入、查找、删除的操作时间都是常量，当然这个常量的值收到key以及所选择的hash函数的影响，uthash共提供了7中函数函数，一般情况下选择默认的即可。如果对效率要求特别高时，可以再根据自己的需求选择适合自己的hash函数。

**2、uthash的使用**

   在hash操作中，都是按照“键-值“对的方式进行插、查等操作，在uthash中，其基本数据结构就是一个包含“键-值“对的结构体，另外，该结构体中还包含一个uthash内部使用的hash处理句柄，如下代码所示：

```cpp
#include"uthash.h"  

struct my_struct {  
    int id;                    /* key */  
    char name[10];  
    UT_hash_handle hh;         /* makes this structure hashable */  
};  
```

其中：

- id是键（key）；
- name是值，即自己要保存的数据域，这里可以根据自己的需要让它变成结构体指针或者其他类型都可以；
- hh是内部使用的hash处理句柄，在使用过程中，只需要在结构体中定义一个UT_hash_handle类型的变量即可，**不需要为该句柄变量赋值，但必须在该结构体中定义该变量**。
-    Uthash所实现的hash表中可以提供类似于双向链表的操作，可以通过结构体成员hh的 `hh.prev`和hh.next获取当前节点的上一个节点或者下一个节点。

 

**3．Key类型为int的简单示例**

```C
1）定义一个键为int类型的hash结构体：
#include "uthash.h"  
 
struct my_struct {  
    int ikey;                    /* key */  
    char value[10];  
	UT_hash_handle hh;           
};  

struct my_struct *g_users = NULL;  
这里需要注意:
```

- key的类型为int，key的类型不一样，后面的插入、查找调用的接口函数就不一样，因此要求确保key的类型与uthash的接口函数一致。
- 必须提供`UT_hash_handle`变量`hh`，无需为其初始化。
- 定义一个hash结构的空指针`users`，用于指向保存数据的hash表，必须初始化为空，在后面的查、插等操作中，uthash内部会根据其是否为空而进行不同的操作。

**2）实现自己的查找接口函数：**

```cpp
struct my_struct *find_user(int ikey) {  
    struct my_struct *s;  
	HASH_FIND_INT(g_users, &ikey, s );  
	return s;  
}  
其实现过程就是先定义一个hash结构体指针变量，然后通过HASH_FIND_INT接口找到该key所对应的hash结构体。这里需要注意：
```

- Uthash为整型key提供的查找接口为`HASH_FIND_INT`；
- 传给接口`HASH_FIND_INT`的第一个参数就是在1）中定义的指向hash表的指针，传入的第二个参数是整型变量ikey的地址。

**3）实现自己的插入接口函数：**

```cpp
void add_user(int ikey, char *value_buf) {  
    struct my_struct *s;  
    HASH_FIND_INT(g_users, &ikey, s);  /* 插入前先查看key值是否已经在hash表g_users里面了 */  
    if (s==NULL) {  
		s = (struct my_struct*)malloc(sizeof(struct my_struct));  
		s->ikey = ikey;  
      	HASH_ADD_INT(g_users, ikey, s );  /* 这里必须明确告诉插入函数，自己定义的hash结构体中键变量的名字 */  
    }  
    strcpy(s-> value, value_buf);  
}  
由于uthash要求键（key）必须唯一，而uthash内部未对key值得唯一性进行很好的处理，因此它要求外部在插入操作时要确保其key值不在当前的hash表中，这就需要，在插入操作时，先查找hash表看其值是否已经存在，不存在在时再进行插入操作，在这里需要特别注意以下两点：
```

- **插入时，先查找，当键不在当前的hash表中时再进行插入，以确保键的唯一性。**
- **需调用插入接口函数时需要明确告诉接口函数，自己定义的键变量的名字是什么。**

**4）实现删除接口**

```cpp
void delete_user(int ikey) {  
    struct my_struct *s = NULL;  
    HASH_FIND_INT(g_users, &ikey, s);  
    if (s!=NULL) {  
	    HASH_DEL(g_users, s);   
		free(s);              
    }  
}  
```

   删除操作的接口函数为`HASH_DEL`，只需要告诉该接口要释放哪个hash表（这里是`g_users`）里的哪个节点（这里是s），需要注意：释放申请的hash结构体变量，uthash函数只将结构体从hash表中移除，并未释放该结构体所占据的内存。

**5）清空hash表**

```cpp
void delete_all() {  
	struct my_struct *current_user, *tmp;  
	HASH_ITER(hh, users, current_user, tmp) {  
    	HASH_DEL(g_users,current_user);    
		free(current_user);              
	}  
}  
这里需要注意：uthash内部提供了另外一个清空函数:
HASH_CLEAR(hh, g_users);
```

函数，但它不释放各节点的内存，因此尽量不要使用它，

**6）统计hash表中的已经存在的元素数**

```C
该操作使用函数HASH_COUNT即可获取到当前hash表中的元素数，其用法为：
unsigned int num_users;  

num_users = HASH_COUNT(g_users);  

printf("there are %u items\n", num_users);  
7、遍历元素
      在开发过程中，可能需要对整个hash表进行遍历，这里可以通过hh.next获取当前元素的下一个元素。具体遍历方法为：
struct my_struct *s, *tmp;  

HASH_ITER(hh, g_users, s, tmp) {  
    printf("user ikey %d: value %s\n", s->ikey, s->value);  
    /* ... it is safe to delete and free s here */  
}  
另外还有一种不安全的删除方法，尽量避免使用它：
void print_users() {  
    struct my_struct *s;  
    for(s=g_users; s != NULL; s=s->hh.next) {  
        printf("user ikey %d: value %s\n", s->ikey, s->value);  
   }  
}  


4． 其他类型key的使用
       本节主要关于key值类型为其他任意类型，例如整型、字符串、指针、结构体等时的用法。
注意：在使用key值为浮点类型时，由于浮点类型的比较受到精度的影响，例如：1.0000000002被认为与1相等，这些问题在uthash中也存在。
4.1． int类型key
前面就是以int类型的key作为示例，总结int类型key使用方法，可以看到其查找和插入分别使用专用接口:HASH_FIND_INT和ASH_ADD_INT。
4.2． 字符指针char*类型key与字符数组char key[100]类型key
特别注意在Strting类型中，uthash对指针char*和字符数组（例如char key[100]）做了区分，这两种情况下使用的接口函数时不一样的。在添加的时候，key的类型为指针时使用接口函数HASH_ADD_KEYPTR，key的类型为字符数组时，使用接口函数HASH_ADD_STR，除了添加的接口不一样外，其他的查找、删除、变量等接口函数都是一样的。
4.3．使用地址作为key
          在uthash中也可使用地址做key进行hash操作，使用地址作为key值时，其类型为void*，这样它就可以支持任意类型的地址了。在使用地址作为key时，插入和查找的专用接口函数为HASH_ADD_PTR和HASH_FIND_PTR，其余接口是一样的。
4.3．其他非常用类型key
在uthash中还可使用结构体作为key，甚至可以采用组合的方式让多个值作为key，这些在其官方的网站张均有较详细的使用示例。在使用uthash需要注意以下几点：
```

-  在定义hash结构体时不要忘记定义UT_hash_handle的变量
-  需确保key值唯一，如果插入key-value对时，key值已经存在，再插入的时候就会出错。
- 不同的key值，其增加和查找调用的接口函数不一样，具体可见第4节。一般情况下，不通类型的key，其插入和查找接口函数是不一样的，删除、遍历、元素统计接口是通用的，特殊情况下，字符数组和字符串作为key值时，其插入接口函数不一样，但是查找接口是一样的。

**5．完整程序例子**

**5.1．key类型为int的完整的例子**

```cpp
#include <stdio.h>   /* gets */  
#include <stdlib.h>  /* atoi, malloc */  
#include <string.h>  /* strcpy */  
#include "uthash.h"  

struct my_struct {  
    int ikey;                    /* key */  
    char value[10];  
    UT_hash_handle hh;         /* makes this structure hashable */  
};  

static struct my_struct *g_users = NULL;  
void add_user(int mykey, char *value) {  
    struct my_struct *s;  
    HASH_FIND_INT(users, &mykey, s);  /* mykey already in the hash? */  
    if (s==NULL) {  
    	s = (struct my_struct*)malloc(sizeof(struct my_struct));  
	    s->ikey = mykey;  
	    HASH_ADD_INT( users, ikey, s );  /* ikey: name of key field */  
    }  
    strcpy(s->value, value);  
}  

struct my_struct *find_user(int mykey) {  
    struct my_struct *s;  
    HASH_FIND_INT( users, &mykey, s );  /* s: output pointer */  
    return s;  
}  

void delete_user(struct my_struct *user) {  
    HASH_DEL( users, user);  /* user: pointer to deletee */  
    free(user);  
}  

void delete_all() {  
	struct my_struct *current_user, *tmp;  
	HASH_ITER(hh, users, current_user, tmp) {  
	    HASH_DEL(users,current_user);  /* delete it (users advances to next) */  
		free(current_user);            /* free it */  
	}  
}  

void print_users() {  
	struct my_struct *s;  
    for(s=users; s != NULL; s=(struct my_struct*)(s->hh.next)) {  
        printf("user ikey %d: value %s\n", s->ikey, s->value);  
    }  
}  


int name_sort(struct my_struct *a, struct my_struct *b) {  
    return strcmp(a->value,b->value);  
}  

int id_sort(struct my_struct *a, struct my_struct *b) {  
    return (a->ikey - b->ikey);  
}  

void sort_by_name() {  
    HASH_SORT(users, name_sort);  
}  

void sort_by_id() {  
    HASH_SORT(users, id_sort);  
}

int main(int argc, char *argv[]) {  
    char in[10];  
    int ikey=1, running=1;  
    struct my_struct *s;  
    unsigned num_users;  

    while (running) {  
        printf(" 1. add user\n");  
        printf(" 2. add/rename user by id\n");  
        printf(" 3. find user\n");  
        printf(" 4. delete user\n");  
        printf(" 5. delete all users\n");  
        printf(" 6. sort items by name\n");  
        printf(" 7. sort items by id\n");  
        printf(" 8. print users\n");  
        printf(" 9. count users\n");  
        printf("10. quit\n");  
        gets(in);  
        switch(atoi(in)) {  
            case 1:  
                printf("name?\n");  
                add_user(ikey++, gets(in));  
                break;  
            case 2:  
                printf("id?\n");  
                gets(in); ikey = atoi(in);  
                printf("name?\n");  
                add_user(ikey, gets(in));  
                break;  
            case 3:  
                printf("id?\n");  
                s = find_user(atoi(gets(in)));  
                printf("user: %s\n", s ? s->value : "unknown");  
                break;  
            case 4:  
                printf("id?\n");  
                s = find_user(atoi(gets(in)));  
                if (s) delete_user(s);  
                else printf("id unknown\n");  
                break;  
            case 5:  
                delete_all();  
                break;  
            case 6:  
                sort_by_name();  
                break;  
            case 7:  
                sort_by_id();  
                break;  
            case 8:  
                print_users();  
                break;  
            case 9:  
                num_users=HASH_COUNT(users);  
                printf("there are %u users\n", num_users);  
                break;  
            case 10:  
                running=0;  
                break;  
        }  
    }  

    delete_all();  /* free any structures */  

    return 0;  
}  
```

**5.2．key类型为字符数组的完整的例子**

```cpp
#include <string.h>  /* strcpy */  
#include <stdlib.h>  /* malloc */  
#include <stdio.h>   /* printf */  
#include "uthash.h"  

struct my_struct {  
    char name[10];             /* key (string is WITHIN the structure) */  
    int id;  
    UT_hash_handle hh;         /* makes this structure hashable */  
};  

int main(int argc, char *argv[]) {  

    const char **n, *names[] = { "joe", "bob", "betty", NULL };  
    struct my_struct *s, *tmp, *users = NULL;  
    int i=0;  

    for (n = names; *n != NULL; n++) {  
        s = (struct my_struct*)malloc(sizeof(struct my_struct));  
        strncpy(s->name, *n,10);  
        s->id = i++;  
        HASH_ADD_STR( users, name, s );  
    }  
    HASH_FIND_STR( users, "betty", s);  
    if (s) printf("betty's id is %d\n", s->id);  
    /* free the hash table contents */  
    HASH_ITER(hh, users, s, tmp) {  
		HASH_DEL(users, s);  
		free(s);  
    }  
    return 0;  
}  
```

**5.3．key类型为字符指针的完整的例子**

```cpp
#include <string.h>  /* strcpy */  
#include <stdlib.h>  /* malloc */  
#include <stdio.h>   /* printf */  
#include "uthash.h"  

struct my_struct {  
    const char *name;          /* key */  
    int id;  
    UT_hash_handle hh;         /* makes this structure hashable */  
};  

int main(int argc, char *argv[]) {  
    const char **n, *names[] = { "joe", "bob", "betty", NULL };  
    struct my_struct *s, *tmp, *users = NULL;  
    int i=0;  

    for (n = names; *n != NULL; n++) {  
        s = (struct my_struct*)malloc(sizeof(struct my_struct));  
        s->name = *n;  
        s->id = i++;  
        HASH_ADD_KEYPTR( hh, users, s->name, strlen(s->name), s );  
    }  

    HASH_FIND_STR( users, "betty", s);  

    if (s) printf("betty's id is %d\n", s->id);  

    /* free the hash table contents */  

    HASH_ITER(hh, users, s, tmp) {  
		HASH_DEL(users, s);  
		free(s);  
    }  
    return 0;  
}  
```

 