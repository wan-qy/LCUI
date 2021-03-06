# LCUI 代码风格

本文档只是描述一些附加的代码风格，主要代码风格可以参考Linux内核的CodingStyle文档
：https://github.com/torvalds/linux/blob/master/Documentation/zh_CN/CodingStyle

## 缩进 & 每行字符数限制

代码缩进宽度应设置为8个字符，每一行的代码长度也应限制在80列以内，8个字符的缩进可
以让代码更容易阅读，再加上每行字符数的限制，当你的代码缩进层次太深的时候可以给你
警告，以提醒你去考虑是否需要调整代码。

限制代码的列数的主要目的不是为了适应那种显示空间有限的终端屏幕，现在的计算机显示屏
的分辨率都比较高（但这并不是说你可以把代码写到和屏幕一样宽），有的代码编辑器支持多
窗口编辑，通常是两个，左右各一个代码编辑窗口，为了能让代码能够在代码编辑框里完整呈
现，减少多余的横向滚动条滚动操作，有必要限制每行代码的列数，因此，决定限制为80列。

由于LCUI项目是一个函数库，其代码处于用户代码的底层，作为底层代码，并且用的是C，
不会出现其他编程语言中那种冗长的函数名和很深的代码缩进，因此，LCUI中的代码不应该写
得那么冗长、拥挤和臃肿，上述要求对实际编码不会有很大负面影响。

## 标识符的定义

如果你定义的全局变量和函数仅在当前源文件中使用，那么在定义时需在前面加上static修
饰符，以表示其只用在当前源文件内。

对于宏(#define)、类型定义(typedef)、结构体定义(struct)、枚举(enum)等，如果仅在当
前源文件中使用，那么就不应当写在头文件中。

## 函数

一个函数的最大长度是和该函数的复杂度和缩进级数成反比的，如果你的函数中的代码很简
单、而且缩进层次较少，那么，这样的函数尽管很长，也是可以的。否则，你应该先重新考
虑你的函数是否干了太多的事，然后把它拆分成若干个小函数。

函数中的局部变量的数量不应超过5－10个，否则你的函数就有问题了，你需要做的和上述
的一样，对函数代码进行拆分。

在源文件里，使用空行隔开不同的函数。

## 集中的函数退出途径

虽然被某些人声称已经过时，但是goto语句的等价物还是经常被编译器所使用，具体形式是
无条件跳转指令。

当一个函数从多个位置退出并且需要做一些通用的清理工作的时候，goto的好处就显现出来
了。

理由是：

- 无条件语句容易理解和跟踪
- 嵌套程度减小
- 可以避免由于修改时忘记更新某个单独的退出点而导致的错误
- 减轻了编译器的工作，无需删除冗余代码

```c
int fun(int a)
{
	int result = 0;
	char *buffer;

	buffer = malloc(SIZE);
	...
	if( ... ) {
		result = -1;
		goto error_out;
	}
	...
	if( ... ) {
		result = -2;
		goto error_out;
	}
	...
	while (1) {
		for( ... ) {
			for( ... ) {
				if( ... ) {
					result = -3;
					goto error_out;
				}
			}
		}
	}
	...
	return 0;
	
error_out:
	...
	free(buffer);
	return result;
}
```

当想减少缩进，又不想添加新函数，可以这样：

```c
int fun(int a) {
	for( ... ) {
		switch( ... ) {
		case 0:
			switch( ... ) {
			case 'a': 
				...
				goto func_a;
			case 'b': 
				...
				goto func_b;
			}
		case 1:
			if( ... ) {
				goto_func_1;
			} else if ( ... ) {
				goto_func_2;
			} else {
				goto_func_3;
			}
		}
		continue;
func_a:
		...
		continue;
func_b:
		...
		continue;
func_1:
		...
		continue;
func_2:
		...
		continue;
func_3:
	}
}
```


## 函数命名规范

如果你的函数是向外提供的公用函数，那么，函数的命名方式应该类似于：

函数类/对象/模块名_操作+对象属性

每个单词首字母应该大写，这样更容易区分出单词。

函数命名示例：

```c
    Widget_AddClass();
```

该函数的操作对象是部件（Widget），实现的操作是添加（Add），被操作的对象属性是样式类（Class），即：为部件添加新样式类。

以下有三种命名风格可参考：

```
             A                        B                         C
新建实例     w = LCUIWidget_New()     s = Selector()            Graph_Init(&g)
相关操作     Widget_AddClass(w, ...)  Selector_Compare(s, ...)  Graph_Mix(&g, ...)
销毁实例     Widget_Destroy(w)        Selector_Delete(s)        Graph_Free(&g)
```


如果你的函数仅在当前源文件中使用，那么可以不必遵循以上规则，但还是建议你的函数命名便于阅读和理解。

如果函数名较长，超出了80列字符的限制，那么应该进行分行，将参数列表分成多行，例如：

```c
    static MyType* ObjectName_OperateAttribute( XXXX *object, XXXX arg1, XXXX arg2, XXXX arg3 )
```

应该改成：

```c
    static MyType* ObjectName_OperateAttribute( XXXX *object, XXXX arg1,
                                                XXXX arg2, XXXX arg3 )
```

如果函数名前的修饰符较多，可以将它们放到单独行里，例如：

```c
    static struct MyObjectStruct* 
    ObjectName_OperateAttribute( XXXX *object, XXXX arg1, XXXX arg2, XXXX arg3 )
```

如果还是比较长的话，可以将函数名中的单词改成缩写，或者使用其它意思相同但比较短
的单词，像这样：

```c
    static struct MyObjectStruct* 
    ObjName_OptAttr( XXXX *object, XXXX arg1, XXXX arg2, XXXX arg3 )
```

由于作者使用的开发工具是 VisualStudio，如果函数的返回值类型和修饰符不与函数名同行，编辑
器的代码提示功能会无法显示位于函数上方的注释内容，例如：

```c
    /** this is function. */
    static struct mydata*
    test_function(void);
```

因此，建议最好将返回值类型和修饰符与函数名写在同一行。

调用函数时，每个参数的逗号后面需要加个空格。

```c
    func( arg1, arg2, arg3, arg4 );
```

## 注释
**函数注释：** 如果是在头文件中有声明的公用函数，那么函数注释只需要在头文件的函数声明中写上，而
源文件中可以不用重复写注释，这样可以省去同时维护两个注释的麻烦。

**代码中的注释：** 注释不应该太多，注释内容简单明了即可，没必要每行代码都加注释，毕竟不是给刚学
习编程的人看的。

**数据定义时的注释：** 在定义结构体时，应该在每行的成员声明后面加上注释。
