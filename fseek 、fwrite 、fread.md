# fseek
```
 int fseek(FILE *stream, long offset, int fromwhere);
```
功能：把与fp有关的文件位置指针放到一个指定位置。
```
fseek(fp, 0L, SEEK_END);
范例一：fseek(fp, 0L, SEEK_END);
解释：文件指针定位到文件末尾，偏移0个字节
范例二：  fseek(fp,50L,0)；或fseek(fp,50L,SEEK_SET);
解释：其作用是将位置指针移到离文件头50个字节处。
```

起始点  | 对应的数字 | 代表的文件位置
---|--|---
SEEK_SET| 0 | 文件开头
SEEK_CUR | 1| 文件当前位置
SEEK_END | 2| 文件末尾
返回值
如果成功，则该函数返回零，否则返回非零值
# fwrite

```
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream)
```
参数
- ptr -- 这是指向要被写入的元素数组的指针。
- size -- 这是要被写入的每个元素的大小，以字节为单位。
- nmemb -- 这是元素的个数，每个元素的大小为 size 字节。
- stream -- 这是指向 FILE 对象的指针，该 FILE 对象指定了一个输出流。

返回值
如果成功，该函数返回一个 size_t 对象，表示元素的总数，该对象是一个整型数据类型。如果该数字与 nmemb 参数不同，则会显示一个错误。

```
#include <stdio.h>
#include <string.h>
 
int main()
{
   FILE *fp;
   char c[] = "This is runoob";
   char buffer[20];
 
   /* 打开文件用于读写 */
   fp = fopen("file.txt", "w+");
 
   /* 写入数据到文件 */
   fwrite(c, strlen(c) + 1, 1, fp);
 
   /* 查找文件的开头 */
   fseek(fp, 0, SEEK_SET);
 
   /* 读取并显示数据 */
   fread(buffer, strlen(c)+1, 1, fp);
   printf("%s\n", buffer);
   fclose(fp);
   
   return(0);
}
```

# fread
C 库函数 size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream) 从给定流 stream 读取数据到 ptr 所指向的数组中。

```
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream)
```
参数
- ptr -- 这是指向带有最小尺寸 size*nmemb 字节的内存块的指针。
- size -- 这是要读取的每个元素的大小，以字节为单位。
- nmemb -- 这是元素的个数，每个元素的大小为 size 字节。
- stream -- 这是指向 FILE 对象的指针，该 FILE 对象指定了一个输入流。

返回值
成功读取的元素总数会以 size_t 对象返回，size_t 对象是一个整型数据类型。如果总数与 nmemb 参数不同，则可能发生了一个错误或者到达了文件末尾。

```
#include <stdio.h>
#include <string.h>
 
int main()
{
   FILE *fp;
   char c[] = "This is runoob";
   char buffer[20];
 
   /* 打开文件用于读写 */
   fp = fopen("file.txt", "w+");
 
   /* 写入数据到文件 */
   fwrite(c, strlen(c) + 1, 1, fp);
 
   /* 查找文件的开头 */
   fseek(fp, 0, SEEK_SET);
 
   /* 读取并显示数据 */
   fread(buffer, strlen(c)+1, 1, fp);
   printf("%s\n", buffer);
   fclose(fp);
   
   return(0);
}
```
让我们编译并运行上面的程序，这将创建一个文件 file.txt，然后写入内容 This is runoob。接下来我们使用 fseek() 函数来重置写指针到文件的开头，文件内容如下所示：

```
This is runoob
```
