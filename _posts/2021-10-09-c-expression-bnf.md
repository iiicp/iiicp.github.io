---
layout: post
title: c99 expression ebnf
tags: c语言
excerpt: 深入学习C
---

#### C99 expression ebnf

```bash
primary-expression:
	identifer
	constant
	string-literal
	(expression)

// E15 ([] () . -> ++ --)
postfix-expression:
	primary-expression
	postfix-expression [expression]
	postfix-expression (argument-expression-list(opt))
	postfix-expression . identifier
	postfix-expression -> identifier
	postfix-expression ++
	postfix-expression --
	(type-name) {initializer-list}  // int *a = &(int){42};
	(type-name) {initializer-list,} // int *p = (int []){2, 4};
	
argument-expression-list:
	assignment-expression
	argument-expression-list , assignment-expression
	
// E14 (++ -- sizeof & * + - ~ ! cast)
unary-expression:
	postfix-expression
	++ unary-expression				
	-- unary-expression								=>  unary-operator unary-expression
	unary-operator cast-expression		=> 	(type-name) unary-expression
	sizeof unary-expression
	sizeof (type-name)

unary-operator: one of
	& * + - ~ ! ++ --

cast-expression:
	unary-expression
	(type-name) cast-expression

// E13
multiplicative-expression:
	cast-expression
	multiplicative-expression * cast-expression
	multiplicative-expression / cast-expression
	multiplicative-expression % cast-expression

// E12
additive-expression:
	multiplicative-expression
	additive-expression + multiplicative-expression
	additive-expression - multiplicative-expression

// E11
shift-expression:
	additive-expression
	shift-expression << additive-expression
	shift-expression >> additive-expression

// E10
relational-expression:
	shift-expression
	relational-expression < shift-expression
	relational-expression > shift-expression
	relational-expression <= shift-expression
	relational-expression >= shift-expression

// E9 
equality-expression:
	relational-expression
	equality-expression == relational-expression
	equality-expression != relational-expression
	
// E8
AND-expression:
	equality-expression
	AND-expression & equality-expression

// E7
exclusive-OR-expression:
	AND-expression
	exclusive-OR-expression ^ AND-expression

// E6
inclusive-OR-expression:
	exclusive-OR-expression
	inclusive-OR-expression | exclusive-OR-expression
	
// E5
logical-AND-expression:
	inclusive-OR-expression
	logical-AND-expression && inclusive-OR-expression
	
// E4
logical-OR-expression:
	logical-AND-expression
	logical-OR-expression || logical-AND-expression

// E3
conditional-expression:
	logical-OR-expression
	logical-OR-expression ? expression : conditional-expression

assignment-expression:
	conditional-expression
	unary-expression assignment-operator assignment-expression

// E2
assignment-operator: one of
	= *= /= %= += -= <<= >>= &= ^= |=

expression:
	assignment-expression
	expression, assignment-expression
	
constant-expression:
	conditional-expression
```

#### 表达式parse

二元表达式

​	二元表达式是由若干个一元表达式通过二元操作符连接起来

​	unary1 BOP4 unary2 BOP7 unary3 BOP3 unary4

```c
						BOP3
				 /         \
		BOP4					unary4
	  /     \
unary1		BOP7
				 /     \
			unary2 	unary3
```

   二元表达式是按照优先级来进行parse，优先级更高的先parse.

一元表达式

后缀表达式

基本表达式



#### 关于typedef

typedef 定义的变量在当前作用域及其子作用域内都能使用。还是具备变量作用域。

typedef定义的变量在子作用域中能够被重载。（在当前作用域中不能被重载）

```c
// level 0
typedef int AINT
// error 当前作用域不能重载
int AINT; 

{
  /// 可以使用typedef变量
  AINT a;
  // 子作用可以重载
  int AINT; 
  /// error 重载之后，不能再用了
  AINT b;
  /// level 1
  typedef float FLT;
  FLT f;
}
// error，使用的作用域必须要大于等于typedef定义的作用域
// level 0 (0 不大于 1)，所有error
FLT f; 
```



ucc作者的实现思路是，在每个compiler unit中使用typedefArr和overloadArr来记录typedef的定义变量，和typedef定义的变量是否被重载。（重载也就是说在当前作用域中，typedef定义的变量不可见了）

typedefArr/overloadArr中装的数据结构是

```c
struct tyname {
	char *name;
	int level;
	bool isOverload;		/// 是否被重载
	int overloadLevel;	/// 重载的level
}
```

