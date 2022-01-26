# cJson文档

## 数据结构

> cJSON使用`cJSON`结构体来存储JSON数据

```c
/* cJSON结构 */
typedef struct cJSON
{
    struct cJSON *next;
    struct cJSON *prev;
    struct cJSON *child;
    int type;
    char *valuestring;
    //不应该直接向valueint写数据，而是使用cJSON_SetNumberValue函数进行赋值
    int valueint;
    double valuedouble;
    char *string;
} cJSON;
```

*   一个`cJSON`结构体存储一个JSON的值。`cJSON`结构体中的`type`是指向JSON值的类型，同时是以`bit-flag`的形式存储，这意味着不能仅仅通过比较`type`的值来判断JSON值的类型。
*   可以使用`cJSON_Is...`函数来检查`cJSON`结构体存储JSON的值的类型，它会对空指针进行检查，同时返回一个布尔值来判断否是该值。
*   `cJSON`可能的值有：
    *   `cJSON_Invalid`：无效值，没有存储任何的值。当成员全部清零时便是该值
    *   `cJSON_False`：为假的布尔值
    *   `cJSON_True`：为真的布尔值
    *   `cJSON_NULL`：空值
    *   `cJOSN_Number`：数值，既作为双精度浮点数存储于`valuedouble`,又作为整型存储于`valueint`。如果数值超过了整型的范围，`valueint`将被赋值为`INT_MAX`或者`INT_MIN`
    *   `cJSON_String`：字符串，以’\\0’的形式结尾，存储于`valuestring`
    *   `cJSON_Array`：数组，通过`cJSON`节点链表来存储array值，每一个元素使用`next`和`prev`进行相互连接，第一个成员的`prev`值为NULL，最后一个成员的`next`值为NULL。同时使用成员`child`指针来指向该链表
    *   `cJSON_Object`：对象，与数组有相似的存储方式，唯一不同的是对象会将键值放入成员`string`中
    *   `cJSON_Raw`：JSON格式的字符串，以’\\0’结尾，存储于`valuestring`。在一次又一次的打印相同的JSON场景下，它能够节省内存。cJSON不会在解析json格式的字符串时产生这种类型。值的注意的是cJSON库不会检查其值是否是合法的
    *   `cJSON_IsReference`：成员`child`指针或者`valuestring`指向的节点并不属于自己，自己仅仅是一个引用。因此，`cJSON_Delete`和其他相关的函数只会释放这个引用本身，而不会去释放`child`或者`valuestring`
    *   `cJSON_StringIsConst`：成员`string`指向的是一个字面量，因此，`cJSON_Delete`和其他相关的函数不会试图去释放`string`的内存

### 使用这些数据结构

> 对于每一种类型的值都有一个对应的函数`cJSON_Create...`来创建。这类函数会动态分配一个`cJSON`的结构，所以需要在使用完后用`cJSON_Delete`释放掉内存，以避免内存泄漏。
>
> 注意：当你已经把一个节点加入了一个数组或者对象，你就不能在用`cJSON_Delete`去释放这个节点的内存了，当该数组或者对象被删除时，这个节点也会被删除

* 创建基本类型的函数

  *   `null`：`cJSON_CreateNull`
  *   `booleans`：`cJSON_CreateTrue`，`cJSON_CreateFalse`或者`cJSON_CreateBool`
  *   `numbers`：`cJSON_CreateNumber`
  *   `strings`：`cJSON_CreateString`

* 数组

  > 你可以使用`cJSON_CreateArray`函数来创建一个新的空数组。`cJSON_CreateArrayReference`函数可以创建一个数组的引用，因为它没有属于自己的内容，所以它的子元素不会被`cJSON_Delete`给删除

  > 使用`cJSON_AddItemToArray`函数可以在数组的最后增加元素。使用`cJSON_AddItemReferenceToArray`函数将会增加一个元素去引用其他的节点，这就意味着`cJSON_Delete`不会去删除这个元素的`child`或者`valuestring`属性，因此当这些属性在其他地方使用的时候，不用担心重复释放内存的事情发生。使用`cJSON_InsertItemInArray`函数可以将一个新元素插入数组中的0索引的位置，旧的元素的索引依次加1

  > 如果你想根据索引去移除数组中的一个元素并且继续去使用它，可以使用`cJSON_DetachItemFromArray`函数，它将会返回被分离的数组。为避免内存泄漏，确保将返回值赋值给一个指针

  > 当你需要替换数组中的某一个元素时，`cJSON_ReplaceItemInArray`函数使用索引的方式来进行替换。`cJSON_ReplaceItemViaPointer`函数使用指向该元素的指针，同时如果失败则会返回0。这两个函数会分离出旧的元素并删除它，同时在这个位置加入新的元素

  > 使用`cJSON_GetArraySize`函数得到数组的大小。使用`cJSON_GetArrayItem`得到一个元素的索引

  > 因为数组是以链表的方式进行存储的，所以通过索引的方式进行遍历效率是很低的( O(n^2) )。建议使用宏`cJSON_ArrayForEach`来遍历数组，它具有时间复杂度为( O(n) )

* 对象

  > 你可以使用`cJSON_CreateObject`函数来创建一个新的空对象。`cJSON_CreateObjectReference`函数可以创建一个对象的引用，因为它没有属于自己的内容，所以它的子元素不会被`cJSON_Delete`给删除

  > 使用`cJSON_AddItemToObject`函数来增加一个元素到对象里。使用`cJSON_AddItemToObjectCS`函数来增加对象里的元素时，使用的键值（结构体`cJSON`中`string`成员）是一个引用或者是字面量，因此它会被`cJSON_Delete`给忽略。使用`cJSON_AddItemReferenceToArray`函数将会增加一个元素去引用其他的节点，这就意味着`cJSON_Delete`不会去删除这个元素的`child`或者`valuestring`属性，因此当这些属性在其他地方使用的时候，不用担心重复释放内存的事情发生

  > 使用`cJSON_DetachItemFromObjectCaseSensitive`函数来从对象中分离出一个元素，从函数命名可以看出对于指定的键值是大小写敏感的，它将会返回被分离的数组。为避免内存泄漏，确保将返回值赋值给一个指针

  > 使用`cJSON_DeleteItemFromObjectCaseSensitive`函数来从一个对象中删除一个元素，可以把它看成先从对象中分离出该元素然后在删除

  > 当你需要替换对象中的某一个元素时，`cJSON_ReplaceItemInObjectCaseSensitive`函数使用j键值查找的方式来进行替换。`cJSON_ReplaceItemViaPointer`函数使用指向该元素的指针来查找并替换，同时如果失败则会返回0。这两个函数会分离出旧的元素并删除它，同时在这个位置加入新的元素

  > 因为对象的存储方式和数组很像，所以同样可以通过`cJSON_GetArraySize`来得到对象里元素的个数

  > 使用`cJSON_GetObjectItemCaseSensitive`来访问对象中的某一个元素

  > 使用宏`cJSON_ArrayForEach`来遍历一个对象

  > `cJSON`同样也提供便利的工具函数来快速的在对象内部创建一个新的元素，比如说`cJSON_AddNullToObject`函数将会返回新加的元素指针，如果失败则返回`NULL`

### 解析JSON字符串

> 可以使用`cJSON_Parse`函数来一些以’\\0’结尾的字符串进行解析

```c
cJSON *json = cJSON_Parse(string);
```

> 解析后的结果是`cJSON`的树状的数据结构，一旦解析成功，就有责任在使用完后使用`cJSON_Delete`释放内存

> 默认分配内存使用的是`malloc`函数，释放内存使用的是`free`函数。但是可以使用`cJSON_InitHooks`函数来全局性改变

> 当一个错误发生时，`cJSON_GetErrorPtr`函数可以得到指向输入字符串中错误的位置的指针。值的注意的是在多线程的情况下，该函数会产生竞争条件，更好的方法是使用带有`return_parse_end`参数的`cJSON_ParseWithOpts`函数。

> 如果你想有更多的选项，使用`cJSON_ParseWithOpts(const char *value, const char **return_parse_end, cJSON_bool require_null_terminated)`函数，`return_parse_end`返回输入的JSON字符串的结尾或者一个错误发生的地方（从而在保障线程安全的情况下替换`cJSON_GetErrorPtr`函数）。`require_null_terminated`如果该值设为1，那么当输入的字符串在有效的以’\\0’结尾的json字符串后还包含其他的数据，就会报错

打印JSON

> 使用`cJSON_Print`函数将一个`cJSON`数据结构打印为字符串

```c
char *string = cJSON_Print(json);
```

> 该函数将会动态分配内存给一个字符串，将JSON表达式放入其中。一旦该函数返回，就有责任释放该内存（默认是`free`，取决于设置的`cJSON_InitHooks`）

> `cJSON_Print`将会用空白符来格式化JSON字符串。可以使用`cJSON_PrintUnformatted`来无格式化的打印

> 如果你关于返回的结果的字符串的大小有一个想法，你可以使用`cJSON_PrintBuffered(const cJSON *item, int prebuffer, cJSON_bool fmt)`函数。`fmt`是一个决定是否用空白字符格式化JSON字符串，`prebuffer`指出了所用的第一个缓冲区大小。`cJOSN_Print`当前使用256字节的缓冲区大小。一旦打印超过了大小，新的缓冲区会被动态分配，在继续打印之前旧的缓冲区里的内容复制到新的缓冲区里。

> 使用`cJSON_PrintPreallocated(cJSON *item, char *buffer, const int length, const cJSON_bool format)`函数可以完全避免动态的内存分配，该函数需要指向缓冲区的指针和该缓冲区的大小，如果缓冲区过小，打印将会失败，函数返回0。一旦成功，函数返回1。值得注意的是需要准备超过实际需要的字节还要多5个字节，因为cJSON并不是100%的精确估计提供的内存是否足够

## API

> cJSON版本[Version](https://so.csdn.net/so/search?q=Version): 1.7.14

#### cJSON\_Version

```
CJSON_PUBLIC(const char*) cJSON_Version(void);  打印当前cJSON的版本 
```

使用：

```
printf("Version: %s\n", cJSON_Version());
```

#### cJSON\_InitHooks

```
CJSON_PUBLIC(void) cJSON_InitHooks(cJSON_Hooks* hooks);
初始化hooks, 不同系统内存分配函数稍有差别，通过cJSON_InitHooks实现兼容。
```

使用：

```
cJSON_Hooks* hooks=NULL;
cJSON_InitHooks(hooks);
```

#### cJSON\_Parse

```
CJSON_PUBLIC(cJSON *) cJSON_Parse(const char *value);  
序列化json字符串
```

使用：

```
char * string = "{\"name\":\"xxx\", \"name2\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
```

#### cJSON\_ParseWithLength

```
CJSON_PUBLIC(cJSON *) cJSON_ParseWithLength(const char *value, size_t buffer_length);  
//和 cJSON_Parse没有太大区别，其内部也要计算json字符串的长度
```

使用：

```
char * string = "{\"name\":\"xxx\", \"name2\":\"xxx2\"}";
cJSON * root = cJSON_ParseWithLength(string, strlen(string)+sizeof(""));//考虑到字符串末尾的长度
```

#### cJSON\_ParseWithOpts

```
CJSON_PUBLIC(cJSON*) cJSON_ParseWithOpts(const char *value, const char **return_parse_end, cJSON_bool require_null_terminated);
也是格式化json字符串return_parse_end， require_null_terminated默认传0
```

使用：

```
char * string = "{\"name\":\"xxx\",\"name2\":\"xxx2\"\"}";
cJSON * root = cJSON_ParseWithOpts(string, 0, 0);
if(root==NULL){
	   printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else printf("%s\n", cJSON_Print(root));
```

#### cJSON\_ParseWithLengthOpts

```
CJSON_PUBLIC(cJSON *) cJSON_ParseWithLengthOpts(const char *value, size_t buffer_length, const char **return_parse_end, cJSON_bool require_null_terminated);
参数要求同cJSON_ParseWithOpts一致
```

使用：

```
char * string = "{\"name\":\"xxx\",\"name2\":\"xxx2\"}";
cJSON * root = cJSON_ParseWithLengthOpts(string, strlen(string)+sizeof(""), 0, 0);
if(root==NULL){
	printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else printf("%s\n", cJSON_Print(root));
```

#### cJSON\_Print

```
CJSON_PUBLIC(char *) cJSON_Print(const cJSON *item);
格式化打印json对象
```

使用：
同cJSON\_ParseWithLengthOpts函数用法一致

#### cJSON\_PrintUnformatted

```
CJSON_PUBLIC(char *) cJSON_PrintUnformatted(const cJSON *item);
非格式化打印json对象
```

使用：同cJSON\_Print使用方法一致

#### cJSON\_PrintBuffered

```
CJSON_PUBLIC(char *) cJSON_PrintBuffered(const cJSON *item, int prebuffer, cJSON_bool fmt);
使用缓冲策略将cJSON实体渲染为文本。prebuffer是对最终大小的猜测。猜得好会减少再分配（意思就是最好渲染成文本的长度刚好就是prebuffer的长度，如果不够，会一直触发再分配内存函数直到满足为止，这就消耗太多时间）。
fmt=0表示未格式化，=1表示格式化
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxxxxxxxx\",\"name2\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
	printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
	//为了只需要一次性分配内存，prebuffer大一点比较好，但也不要太大，这个看情况估计,下面的都是刚刚好
	printf("%s\n", cJSON_PrintBuffered(root, strlen(string), 0));//非格式化输出
	printf("%s\n", cJSON_PrintBuffered(root, strlen(string), 1)); //格式化输出
}
```

#### cJSON\_PrintPreallocated

```
CJSON_PUBLIC(cJSON_bool) cJSON_PrintPreallocated(cJSON *item, char *buffer, const int length, const cJSON_bool format);
注意：cJSON并不总是100%准确地估计它将使用多少内存，所以为了安全起见，要比实际需要多分配5个字节。而且使用cJSON_PrintPreallocated的话，缓冲区就全部由调用者控制，内存的分配与回收调用者需要十分清楚
```

使用：暂无

#### cJSON\_Delete

```
CJSON_PUBLIC(void) cJSON_Delete(cJSON *item);
删除json对象
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"name2\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
	printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
	printf("%s\n", cJSON_PrintUnformatted(root));
	cJSON_Delete(root);//root指向的地址没有改变，但是无法使用那块内存，所以用给root赋值NULL好判断
	root = NULL;
	if(root==NULL)printf("删除成功\n");
	else printf("%s\n", cJSON_PrintUnformatted(root));
}
```

#### cJSON\_GetArraySize

```
CJSON_PUBLIC(int) cJSON_GetArraySize(const cJSON *array);
返回数组（或对象）中的项数，就是获取数组大小/对象/项。所谓的项数，就是同一层下，有多少对键值对
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"name2\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
	printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
	printf("%d\n", cJSON_GetArraySize(root));//输出2，因为root下面有两对键值对
}
```

#### cJSON\_GetArrayItem

```
CJSON_PUBLIC(cJSON *) cJSON_GetArrayItem(const cJSON *array, int index);
从数组“array”中检索项目编号“index”。如果不成功，则返回NULL。
就是在json对象当前层下，可以通过索引遍历键名和键值。注意，cJSON_GetArrayItem返回的是cJSON *类型，通过查看该类型的数据结构从而正确访问键名和键值
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"name2\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
	printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
	for(int i=0; i<cJSON_GetArraySize(root); i++){
		cJSON * temp = cJSON_GetArrayItem(root, i);
		if(temp != NULL)printf("root[%s] = %s\n", temp->string, temp->valuestring);
	}
}
```

#### cJSON\_GetObjectItem

```
CJSON_PUBLIC(cJSON *) cJSON_GetObjectItem(const cJSON * const object, const char * const string);
如果想直接通过键名的方式获得键值，可以通过此方法。定位到想要的键名的层次之后，调用此函数即可。（注意，输入的键名是不区分大小写的，也就是说cJSON_GetObjectItem(root, "name")和cJSON_GetObjectItem(root, "NAME")）是一样的。要是想要区分大小写，请使用cJSON_GetObjectItemCaseSensitive函数，使用方法跟cJSON_GetObjectItem一致
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"name2\":\"xxx2\"}";
  cJSON * root = cJSON_Parse(string);
   if(root==NULL){
       printf("Error_content : %s\n", cJSON_GetErrorPtr());
   }else{
       cJSON * temp = cJSON_GetObjectItem(root, "name");//通过键名访问键值
       printf("root[%s] = %s\n", temp->string, temp->valuestring);
       temp = cJSON_GetObjectItem(root, "name2");//通过键名访问键值
       printf("root[%s] = %s\n", temp->string, temp->valuestring);
   }
```

#### cJSON\_HasObjectItem

```
CJSON_PUBLIC(cJSON_bool) cJSON_HasObjectItem(const cJSON *object, const char *string);
用来判断查找的键名是否存在
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"test\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
   printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
   if(!cJSON_HasObjectItem(root, "name2"))printf("该键名不存在\n");
   if(cJSON_HasObjectItem(root, "name")){
       cJSON * temp = cJSON_GetObjectItem(root, "name");//通过键名访问键值
       printf("root[%s] = %s\n", temp->string, temp->valuestring);
   }
}
```

#### cJSON\_IsInvalid等

```
这些函数检查项目的类型
CJSON_PUBLIC(cJSON_bool) cJSON_IsInvalid(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsFalse(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsTrue(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsBool(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNull(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNumber(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsString(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsArray(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsObject(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsRaw(const cJSON * const item);
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"test\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    cJSON *temp = cJSON_GetObjectItem(root, "name");
    //简单例子
    if(cJSON_IsInvalid(temp))printf("改对象是无效的\n");
    if(cJSON_IsString(temp))printf("是字符串\n");
    if(cJSON_IsFalse(temp))printf("是假\n");
    if(cJSON_IsTrue(temp))printf("是真\n");
    if(cJSON_IsBool(temp))printf("是布尔类型\n");
    if(cJSON_IsNull(temp))printf("是空\n");
    if(cJSON_IsNumber(temp))printf("是数字\n");
    if(cJSON_IsArray(temp))printf("是数组\n");
}
```

#### cJSON\_GetStringValue和cJSON\_GetNumberValue

```
CJSON_PUBLIC(char *) cJSON_GetStringValue(const cJSON * const item);
CJSON_PUBLIC(double) cJSON_GetNumberValue(const cJSON * const item);
检查项目类型并返回其值
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\",\"test\":\"xxx2\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    if(cJSON_IsString(root))printf("%s\n", cJSON_GetStringValue(root));
    else printf("不是字符串类型\n");

    cJSON * temp = cJSON_GetObjectItem(root, "name");
    //判断该对象的值是不是字符串类型，并且通过特定函数直接获取
    if(cJSON_IsString(temp))printf("%s\n", cJSON_GetStringValue(temp));
    else printf("不是字符串类型\n");

    // GetNumberValue函数用法也是一样的

}
```

#### cJSON\_AddItemToArray等

```
以下是给json对象添加数组或对象或字符串等所需要的API函数
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToObject(cJSON *object, const char *string, cJSON *item);
CJSON_PUBLIC(cJSON*) cJSON_AddNullToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddTrueToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddFalseToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddBoolToObject(cJSON * const object, const char * const name, const cJSON_bool boolean);
CJSON_PUBLIC(cJSON*) cJSON_AddNumberToObject(cJSON * const object, const char * const name, const double number);
CJSON_PUBLIC(cJSON*) cJSON_AddStringToObject(cJSON * const object, const char * const name, const char * const string);
CJSON_PUBLIC(cJSON*) cJSON_AddRawToObject(cJSON * const object, const char * const name, const char * const raw);
CJSON_PUBLIC(cJSON*) cJSON_AddObjectToObject(cJSON * const object, const char * const name);
CJSON_PUBLIC(cJSON*) cJSON_AddArrayToObject(cJSON * const object, const char * const name);
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("添加键值对\n");
    cJSON_AddNullToObject(root, "isnull");
    // cJSON_AddTrueToObject(root, "istrue");//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知
    // cJSON_AddFalseToObject(root, "isfalse");//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知
    // cJSON_AddBoolToObject(root, "bool", 0);//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知

    cJSON_AddStringToObject(root, "string_name", "test");
    cJSON_AddNumberToObject(root, "num", 123456);
    cJSON_AddObjectToObject(root, "new_obj");
    cJSON_AddRawToObject(root, "raw_obj", "{\"age\":\"110\"}");//所谓原生json就是还是字符串形式，需要格式化后才可使用
    cJSON_AddArrayToObject(root, "arr");
    
    printf("%s\n", cJSON_Print(root));
}
```

#### cJSON\_CreateNull等

```
这些调用将创建相应类型的cJSON项
CJSON_PUBLIC(cJSON *) cJSON_CreateNull(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateTrue(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateFalse(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateBool(cJSON_bool boolean);
CJSON_PUBLIC(cJSON *) cJSON_CreateNumber(double num);
CJSON_PUBLIC(cJSON *) cJSON_CreateString(const char *string);
/* raw json */
CJSON_PUBLIC(cJSON *) cJSON_CreateRaw(const char *raw);
CJSON_PUBLIC(cJSON *) cJSON_CreateArray(void);
CJSON_PUBLIC(cJSON *) cJSON_CreateObject(void);
```

使用：

```
char * string = "{\"name\":\"xxxxxxxxx\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("添加键值对\n");
    cJSON_AddItemToObject(root, "num", cJSON_CreateNumber(100));
    // cJSON_AddItemToObject(root, "is_null", cJSON_CreateNull());//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知
    // cJSON_AddItemToObject(root, "is_true", cJSON_CreateTrue());//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知
    // cJSON_AddItemToObject(root, "is_false", cJSON_CreateFalse());//可能会莫名报错（执行后cJSON_Print(root)打印为空），原因未知
    cJSON_AddItemToObject(root, "str", cJSON_CreateString("string"));
    cJSON_AddItemToObject(root, "raw_json", cJSON_CreateRaw("{\"age\":\"111\"}"));
    cJSON_AddItemToObject(root, "arr", cJSON_CreateArray());
    cJSON_AddItemToObject(root, "new_obj", cJSON_CreateObject());

    printf("%s\n", cJSON_Print(root));
}
```

#### cJSON\_CreateStringReference

```
CJSON_PUBLIC(cJSON *) cJSON_CreateStringReference(const char *string);
创建一个字符串的引用（说白了就是常量字符串变量），但不可通过cJSON_Delete释放该引用( 源代码里面对引用类型的数据直接跳过，不做free处理 )。
```

使用：

```
char * string = "{\"key\":\"value\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    cJSON * temp = cJSON_CreateStringReference("name");//只能获得键值，不能获得键名
    printf("%s : %s\n", temp->string, temp->valuestring);
    cJSON_Delete(temp);//无法通过引用删除，所以无效
    printf("%s : %s\n", temp->string, temp->valuestring);//还可以输出name
}
```

同样的

```
CJSON_PUBLIC(cJSON *) cJSON_CreateObjectReference(const cJSON *child);
CJSON_PUBLIC(cJSON *) cJSON_CreateArrayReference(const cJSON *child);
这几个引用的用法跟cJSON_CreateStringReference是一样的
```

#### cJSON\_CreateIntArray等

```
CJSON_PUBLIC(cJSON *) cJSON_CreateIntArray(const int *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateFloatArray(const float *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateDoubleArray(const double *numbers, int count);
CJSON_PUBLIC(cJSON *) cJSON_CreateStringArray(const char *const *strings, int count);
这四个都是创建数组的方式，区别是数组的值类型不同
```

使用：

```
char * string = "{\"key\":\"value\"}";
int arr[] = {1,2,3,4};
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    cJSON * temp = cJSON_CreateIntArray(arr, 4);
    temp->string = "arr_key";//设置键名
    cJSON_AddItemToArray(root,  temp);//将该键值对加入到root对象
    printf("%s\n", cJSON_Print(root));
    /*
        类似的，cJSON_CreateFloatArray、cJSON_CreateDoubleArray、cJSON_CreateStringArray函数的用法与上面的一致
    */
}
```

#### cJSON\_AddItemToArray等

```
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToObject(cJSON *object, const char *string, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemToObjectCS(cJSON *object, const char *string, cJSON *item);
将cJSON *类型的变量添加到json对象中

对于cJSON_AddItemToObjectCS函数，当字符串肯定是const（即一个等同于literal），并且肯定会在cJSON对象中保留下来时，使用此选项。注意：使用此函数时，请确保在使用前始终检查（item->type&cJSON_StringIsConst）是否为零
```

使用：

```
char * string = "{\"key\":\"value\"}";
int arr[] = {1,2,3,4};
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    cJSON * temp = cJSON_CreateIntArray(arr, 4);
    // temp->string = "arr_key";//设置键名
    // cJSON_AddItemToArray(root,  temp);//将该键值对加入到root对象

    //也可以通过别的方式设置键名
    cJSON_AddItemToObject(root, "arr_key", temp);
    printf("%s\n", cJSON_Print(root));
}
```

#### cJSON\_CreateObjectReference和cJSON\_CreateArrayReference

```
CJSON_PUBLIC(cJSON *) cJSON_CreateObjectReference(const cJSON *child);
CJSON_PUBLIC(cJSON *) cJSON_CreateArrayReference(const cJSON *child);
创建不同类型对象的引用，如数组等
```

使用：暂无

#### cJSON\_AddItemReferenceToArray和cJSON\_AddItemReferenceToObject

```
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemReferenceToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(cJSON_bool) cJSON_AddItemReferenceToObject(cJSON *object, const char *string, cJSON *item);
如果想要将引用的cJSON*变量追加到json对象里面，上面两个函数可以解决该问题
```

使用：

```
char * string = "{\"key\":\"value\"}";
int arr[] = {1,2,3};
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    // cJSON * temp = cJSON_CreateArrayReference(arr);
    cJSON * temp = cJSON_CreateArrayReference(NULL);//相当于创建一个空数组，由于是引用，无法用cJSON_Delete删除
    // printf("%s\n", cJSON_Print(temp));
    cJSON_AddItemToObject(root, "arr_key", temp);//追加到root对象里面
    printf("%s\n", cJSON_Print(root));
}
```

#### cJSON\_DetachItemViaPointer等

```
从数组/对象中移除/分离项*/
CJSON_PUBLIC(cJSON *) cJSON_DetachItemViaPointer(cJSON *parent, cJSON * const item);
//cJSON_DetachItemViaPointer 通过指针分离键值对
```

```
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromArray(cJSON *array, int which);
CJSON_PUBLIC(void) cJSON_DeleteItemFromArray(cJSON *array, int which);
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromObject(cJSON *object, const char *string);
CJSON_PUBLIC(cJSON *) cJSON_DetachItemFromObjectCaseSensitive(cJSON *object, const char *string);
CJSON_PUBLIC(void) cJSON_DeleteItemFromObject(cJSON *object, const char *string);
CJSON_PUBLIC(void) cJSON_DeleteItemFromObjectCaseSensitive(cJSON *object, const char *string);
```

使用：

```
//cJSON_DetachItemViaPointer：
char * string = "{\"key\":\"value\", \"key2\":{\"key3\":\"value3\"}}";
// int arr[] = {1,2,3};
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n", cJSON_Print(root));

    //cJSON_DetachItemViaPointer的使用
    cJSON * temp = cJSON_GetObjectItem(root, "key2");//获得该键名的json对象
    cJSON * temp2 = cJSON_GetObjectItem(temp, "key3");
    //root是要准备分离的总json对象，temp则指向root里面其中一个键值对
    //
    // cJSON_DetachItemViaPointer(root, temp2); //无效   开始分离temp2。 root和temp2必须处于连续的层次中，中间隔一层是无法分离的（故本次无效）
    cJSON_DetachItemViaPointer(root, temp);//可以分离
    printf("%s\n", cJSON_Print(root));

}
```

```
//cJSON_DetachItemFromArray：
char * string = "{\"key\":\"value\", \"key2\":[{\"index\":\"value\"},{\"index2\":\"value2\"}]}";
// int arr[] = {1,2,3};
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n", cJSON_Print(root));

    // cJSON_DetachItemFromArray 的使用
    cJSON * temp = cJSON_GetObjectItem(root, "key2");//获得该键名的json对象
    cJSON_DetachItemFromArray(temp, 0);//分离数组第一个

    // cJSON_DetachItemFromObject(root, "key");//获取当前层下的key键名，通过键名删除key对象
    //cJSON_DetachItemFromObjectCaseSensitive和cJSON_DetachItemFromObject用法一致，前者区别大小写

    printf("%s\n", cJSON_Print(root));
}
```

cJSON\_DeleteItemFromArray与cJSON\_DetachItemFromArray一致

#### cJSON\_InsertItemInArray等

```
更新JSON对象
CJSON_PUBLIC(cJSON_bool) cJSON_InsertItemInArray(cJSON *array, int which, cJSON *newitem); /* Shifts pre-existing items to the right. 将预先存在的项向右移动*/
CJSON_PUBLIC(cJSON_bool) cJSON_ReplaceItemViaPointer(cJSON * const parent, cJSON * const item, cJSON * replacement);
CJSON_PUBLIC(cJSON_bool) cJSON_ReplaceItemInArray(cJSON *array, int which, cJSON *newitem);
CJSON_PUBLIC(cJSON_bool) cJSON_ReplaceItemInObject(cJSON *object,const char *string,cJSON *newitem);
CJSON_PUBLIC(cJSON_bool) cJSON_ReplaceItemInObjectCaseSensitive(cJSON *object,const char *string,cJSON *newitem);
```

使用：

```
char * string = "{\"key\":\"value\", \"key2\":[{\"index\":\"value\"},{\"index2\":\"value2\"}]}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n\n", cJSON_Print(root));

    cJSON * temp = cJSON_GetObjectItem(root, "key2");//获得该键名的json对象
    cJSON * obj = cJSON_CreateObject();
    cJSON_AddStringToObject(obj, "name", "xxx");
    cJSON_InsertItemInArray(temp, 0, obj);//在数组key2第一个位置插入json对象obj
    printf("%s\n\n", cJSON_Print(root));
```

```
    cJSON * obj2 = cJSON_CreateObject();
    cJSON_AddStringToObject(obj2, "name2", "xxx2");
    cJSON_ReplaceItemViaPointer(temp, obj, obj2);//在temp对象中，将obj替换成obj2(只要可以找到该对象，就可以替换)
    // 同理，cJSON_ReplaceItemInArray、cJSON_ReplaceItemInObjectCaseSensitive、cJSON_ReplaceItemInObject用法同上，注意参数要求

    printf("%s\n", cJSON_Print(root));

}
```

#### cJSON\_Duplicate

```
CJSON_PUBLIC(cJSON *) cJSON_Duplicate(const cJSON *item, cJSON_bool recurse);
该函数复制相同的cJSON对象，其中recurse若为1，则表示全部复制（包括深层次的json对象）
```

使用：

```
char * string = "{\"key\":\"value\", \"key2\":[{\"index\":\"value\"},{\"index2\":\"value2\"}]}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n\n", cJSON_Print(root));

    // cJSON *temp = cJSON_Duplicate(root, 0);
    cJSON *temp = cJSON_Duplicate(root, 1);
    printf("%s\n", cJSON_Print(temp));

}
```

```
递归地比较两个cJSON项是否相等。如果a或b为空或无效，则它们将被视为不相等。
区分大小写确定对象键是区分大小写（1）还是不区分大小写（0
CJSON_PUBLIC(cJSON_bool) cJSON_Compare(const cJSON * const a, const cJSON * const b, const cJSON_bool case_sensitive);
```

使用：

```
char * string = "{\"key\":\"value\", \"key2\":[{\"index\":\"value\"},{\"index2\":\"value2\"}]}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n\n", cJSON_Print(root));

    cJSON *temp = cJSON_Duplicate(root, 1);
    int bools = cJSON_Compare(root, temp, 1);//区分大小写
    printf("%d\n", bools);

}
```

#### cJSON\_Minify

```
CJSON_PUBLIC(void) cJSON_Minify(char *json);
压缩字符串，删除字符串中的空白符
```

使用：

```
char * string = "{\"key\":\"v a l u e\"}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    printf("%s\n\n", cJSON_Print(root));
    cJSON* temp = cJSON_GetObjectItem(root, "key");
    cJSON_Minify(temp->valuestring);//压缩字符串
    printf("%s\n", cJSON_Print(root));

}
```

#### cJSON\_ArrayForEach

```
用于迭代数组或对象的宏
#define cJSON_ArrayForEach(element, array) for(element = (array != NULL) ? (array)->child : NULL; element != NULL; element = element->next)
```

使用：

```
char * string = "{\"key\":\"value\", \"key2\":[{\"index\":\"value\"},{\"index2\":\"value2\"}]}";
cJSON * root = cJSON_Parse(string);
if(root==NULL){
    printf("Error_content : %s\n", cJSON_GetErrorPtr());
}else{
    cJSON * temp = cJSON_GetObjectItem(root, "key2");
    cJSON * element; 
    cJSON_ArrayForEach(element, root){
        printf("%s\n", cJSON_Print(element));//迭代遍历json对象（只对当前层有效）
    }

    cJSON_ArrayForEach(element, temp){
        printf("%s\n", cJSON_Print(element));//迭代遍历数组
    }

    printf("%s\n", cJSON_Print(temp));

}
```