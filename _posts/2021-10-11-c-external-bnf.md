---
layout: post
title: c99 external ebnf
tags: c语言
excerpt: 深入学习C
---

#### external bnf

```c
translation-unit:
	external-declaration
  translation-unit external-declaration
    
external-declaration:
	function-definition
  declaration
    
/**
*		static void a(void) {}
		static void -> declaration-specifiers
		a(void) -> declarator
*/
function-definition:
	declaration-specifiers declarator [declaration-list]opt compound-statement
    
declaration-list:
	declaration
  declaration-list declaration
    
/**
 *	static const int a1=3, *a2, a3[5], a4(void);
 		static const int -> declaration-specifiers
 		a1=3 -> init-declarator
 			a1 -> declarator
 			 3 -> initializer
 		*a2 -> init-declarator point
 			*a2 -> declarator
 		a3[5] -> init-declarator array
 		a4(void) -> init-declarator function
 */
declaration:
	declaration-specifiers [init-declarator-list]opt;

declaration-specifiers:
	storage-class-specifier [declaration-specifiers]opt
	type-specifier [declaration-specifiers]opt
  type-qualifier [declaration-specifiers]opt
  function-specifier [declaration-specifiers]opt
 
init-declarator-list:
	init-declarator
  init-declarator-list, init-declarator

init-declarator:
	declarator
  declarator = initializer
    
storage-class-specifier:
	typedef
  extern
  static
  auto
  register
 
type-specifier:
	void
  char
  short
  int
  long
  float
  double
  signed
  unsigned
  _Bool
  _Complex
  struct-or-union-specifier
  enum-specifier
  typedef-name
    
 type-qualifier:
		const
  	restrict
  	volatile
    
 function-specifier:
		inline
 
 typedef-name:
		identifier
    
 struct-or-union-specifier:
		struct-or-union identifier-opt {struct-declaration-list}
		struct-or-union identifier

 struct-or-union:
		struct
    union
      
 struct-declaration-list:
		struct-declaration
    struct-declaration-list struct-declaration
 
 struct-declaration:
		specifier-qualifier-list struct-declarator-list;

 specifier-qualifier-list:
		type-specifier specifier-qualifier-list-opt
    type-qualifier specifier-qualifier-list-opt
  
 struct-declarator-list:
		struct-declarator
    struct-declarator-list, struct-declarator
 
 struct-declarator:
		declarator
    declarator-opt : constant-expression
      
      
 enum-specifier:
		enum identifier-opt {enumerator-list}
		enum identifier-opt {enumerator-list,}
		enum identifier
      
 enumerator-list:
		enumerator
    enumerator-list, enumerator
      
 enumerator:
		enumeration-constant
    enumeration-constant = constant-expression
      
 /**
 		static const int *a[2];
		static const int *a[];
		static const int *a[const 2];
		static const int *a[static const 2];
		static const int *a[const static 2];
		static const int *a[const *];
 **/
 declarator:
		pointer-opt direct-declarator
 direct-declarator:
		identifer
    (declarator)
    direct-declarator[type-qualifier-list-opt assignment-expression-opt]
    direct-declarator[static type-qualifier-list-opt assignment-expression]
    direct-declarator[type-qualifier-list static assignment-expression]
    direct-declarator[type-qualifier-list-opt *]
    direct-declarator(parameter-type-list)
    direct-declarator(identifier-list-opt) /// old function style
      
  pointer:
		* type-qualifier-list-opt
    * type-qualifier-list-opt pointer
  
  type-qualifier-list:
		type-qualifier
    type-qualifier-list type-qualifier
   
  parameter-type-list:
		parameter-list
    parameter-list, ...
  
  parameter-list:
		parameter-declaration
    parameter-list, parameter-declaration
  
  parameter-declaration:
		declaration-specifiers declarator
    declaration-specifiers abstract-declarator-opt
      
  identifier-list:
		identifier
    identifier-list, identifier
   
  /// type-name
  type-name:
		specifier-qualifier-list abstract-declarator-opt
  
  abstract-declarator:
		pointer
    pointer-opt direct-abstract-declarator
    
  direct-abstract-declarator:
		(abstract-declarator)
		direct-abstract-declarator-opt [assignment-expression-opt]
    direct-abstract-declarator-opt [*]
    direct-abstract-declarator-opt (parameter-type-list-opt)
  /////////////  
   
  initializer:
		assignment-expression
    {initializer-list}
		{initializer-list,}
	
	initializer-list:
		[designation]opt initializer
    initializer-list , [designation]opt initializer
      
  designation:
		designator-list = 
  
  designator-list:
		designator
    designator-list designator
   
  designator:
		[constant-expression]
		. identifer
  ///////
```

​	

对声明符和抽象声明符的改造

```c
 /// 原始的文法

 /// 声明符
 declarator:
		[pointer]-opt direct-declarator
      
 direct-declarator:
		identifer
    (declarator)
    direct-declarator[type-qualifier-list-opt assignment-expression-opt]
    direct-declarator[static type-qualifier-list-opt assignment-expression]
    direct-declarator[type-qualifier-list static assignment-expression]
    direct-declarator[type-qualifier-list-opt *]
    direct-declarator(parameter-type-list)
    direct-declarator(identifier-list-opt)
      
  pointer:
		* [type-qualifier-list]-opt
    * [type-qualifier-list]-opt pointer
      
  
  /// 抽象声明符
  abstract-declarator:
		pointer
    [pointer]-opt direct-abstract-declarator
    
  direct-abstract-declarator:
		(abstract-declarator)
		direct-abstract-declarator-opt [assignment-expression-opt]
    direct-abstract-declarator-opt [*]
    direct-abstract-declarator-opt (parameter-type-list-opt)
      
  /// 改造后
  
  abstract-declarator:
		* [type-qualifer-list]opt abstract-declarator
    postfix-abstract-declarator
  
  postfix-abstract-declarator:
		direct-abstract-declarator
		postfix-abstract-declarator [ [constant-expression]opt ]
		postfix-abstrace-declarator( [parameter-type-list]opt )
  
  direct-abstract-declarator:
		(abstract-declarator)
		e
      
  declarator:
		* [type-qualifier-list]opt declarator
    postfix-declarator
  
  postfix-declarator:
		direct-declarator
		postfix-declarator[[constant-expression]opt]
    postfix-declarator(parameter-type-list)
    postfix-declarator([identifier-list]opt)
  
  direct-declarator:
		ID
    (declarator)
      
  
 /// kind
 /// function declaration
      DEC_CONCRETE | DEC_ABSTRACT
 /// function definition
      DEC_CONCRETE
 /// type-name, sizeof(type-name), (type-name)(expr)
      DEC_ABSTRACT
```

