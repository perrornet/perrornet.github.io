---
title: Understanding Redis SDS
date: 2023-02-18 17:52:22
tags:
    - redis
---
What is SDS?

SDS is an abstract type built by Redis, mainly used to store Redis' default string representation, the AOF buffer in the AOF module, and the client state input buffer.

Advantages of SDS

Why doesn't Redis use C strings? First, we need to understand the shortcomings of C strings:

1. C strings do not record their own length information. Obtaining the length of a string requires traversing the entire string, with a time complexity of O(n).
2. C strings do not record their own length, and a slight mistake can cause a buffer overflow.
3. For cache-type databases like Redis, the value of the cache often changes frequently. However, each growth or reduction of a C string requires a memory reallocation operation.
4. The content cached in the Redis database is not specific, and it may be binary data such as images or audio files. However, the characters in C strings must comply with a certain encoding, and the strings cannot contain spaces. These restrictions also mean that Redis cannot use C strings as its own string implementation.

SDS eliminates all these shortcomings one by one:

1. SDS records its own length information, making the time complexity of obtaining the string length O(1).
2. SDS uses pre-allocated space and lazy space release algorithms to solve the problem of frequent memory allocation operations.
3. Since SDS saves its own length, SDS will not determine the end of the string like C does according to '\0'.

Implementation of SDS

Redis uses the sdshdr structure to represent SDS values:

```
struct sdshdr{
   int len; // Records the number of bytes used in the buf array
   int free; // Records the number of unused bytes in buf
   char buf[]; // Save the array of strings
}

```

SDS follows the convention of C strings ending with a null character so that functions in C strings can be reused. This null character at the end of the string does not increase the value of the len field. For example, if you need to use SDS to save the string "golang," the SDS corresponding to the structure is:

![https://s1.ax1x.com/2022/03/25/qNgzqI.png](https://s1.ax1x.com/2022/03/25/qNgzqI.png)

Since SDS records its own length, getting the length of the string in Redis only needs to return the len field, and its time complexity is O(1). The space allocation strategy of SDS completely eliminates the possibility of buffer overflow, which frequently occurs in C strings. When Redis needs to modify an SDS string, it first checks whether the space of the SDS satisfies the space required for the modification. If it does not meet the requirements, the space of the SDS will be expanded to meet the capacity required for the modification. The SDS space expansion algorithm is as follows:

1. If the length of the SDS after modification is less than 1MB, the program will allocate unused space of the same size as the len field value. For the example above, the total capacity of the SDS is 6 (len) + 6 (free) + 1 ('\0') = 13 bytes.
2. If the length of the SDS after modification is greater than 1MB, the SDS will allocate 1MB of unused space. If the capacity of an SDS after the len value is modified is 2MB, Redis will allocate 1MB of unused space for the SDS. At this time, the total capacity of the SDS is 2MB (len) + 1MB (len) + 1byte ('\0') = 3073 byte capacity.

This expansion algorithm reduces the frequency of requesting memory from the system. The SDS space release is not real-time but lazy: Redis believes that if the capacity of an SDS reaches N, it is likely to reach N after it is reduced. And lazy release also reduces the possibility of the next memory allocation. If there is a "golang" string stored in Redis now, and we need to modify "golang" to "go," then the SDS structure of Redis will be like this:

![https://s1.ax1x.com/2022/03/25/qN2RTP.png](https://s1.ax1x.com/2022/03/25/qN2RTP.png)

![https://s1.ax1x.com/2022/03/25/qN24fS.png](https://s1.ax1x.com/2022/03/25/qN24fS.png)

SDS provides an API to clean up memory. We can call this API when needed to release memory.

Summary

1. Redis only uses C strings as literals, and SDS is used to represent most strings.
2. The length of the string can be obtained with constant time complexity.
3. Eliminates buffer overflow.
4. Reduces the number of times memory needs to be reallocated for strings. And for binary data security, it is compatible with some C string functions.
