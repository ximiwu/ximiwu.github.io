---
layout:     post
title:      "数据结构：Stack实现及其应用：infix数学表达式转postfix并求值"
subtitle:   ""
date:       2024-07-24 06:00:00
author:     "西米屋花火"
catalog: true
tags:
    - 栈
    - 数据结构
---

## Stack.h

```cpp
#pragma once
#include <iostream>
using namespace std;
template <typename T>
struct Node
{
	T value = 0;
	Node<T>* next = nullptr;
};
 
template <typename T>
class Stack
{
private:
	Node<T>* top = nullptr;
 
public:
	void push(T value);
	T pop_out();
	T get_top();
	bool is_empty();
	void print_content();
	~Stack();
};
 
template<typename T>
inline void Stack<T>::push(T value)
{
	if (top == nullptr) top = new Node<T>;
	else
	{
 
		Node<T>* p = new Node<T>;
		p->next = top;
		top = p;
	}
	top->value = value;
}
 
template<typename T>
inline T Stack<T>::pop_out()
{
	if (top != nullptr) {
		Node<T>* p = top;
		T value = top->value;
		top = top->next;
		delete p;
		return value;
	}
	return 0;
}
 
template<typename T>
inline T Stack<T>::get_top()
{
	return top->value;
}
 
template<typename T>
inline bool Stack<T>::is_empty()
{
	if (top == nullptr) return true;
	else return false;
}
 
template<typename T>
inline void Stack<T>::print_content()
{
	Node<T>* p = top;
	cout << "----" << endl;
	while (p != nullptr) {
		cout << p->value << endl;
		p = p->next;
	}
	cout << "====" << endl;
}
 
template<typename T>
inline Stack<T>::~Stack()
{
	while (top != nullptr) {
		Node<T>* p = top->next;
		delete top;
		top = p;
	}
}
```

## Expression.h

```cpp
#pragma once
#include <string>
#include "stack.h"
using namespace std;
class Expression
{
private:
	int in_stack_precedence(char value);
	int out_stack_precedence(char value);
	bool is_digit(char value);
 
public:
	double evaluate(string infix);
};
 
```

## Expression.cpp

```cpp
#include "Expression.h"
#include <iostream>
#include <cmath>
using namespace std;
 
int Expression::in_stack_precedence(char value)
{
	switch (value)
	{
	case '+': case '-':
		return 2;
	case '*': case '/':
		return 4;
	case '^':
		return 5;
	case '(':
		return 0;
	}
}
 
int Expression::out_stack_precedence(char value)
{
	switch (value)
	{
	case '+': case '-':
		return 1;
	case '*': case '/':
		return 3;
	case '^':
		return 6;
	case '(':
		return 7;
	case ')':
		return 0;
	}
}
 
bool Expression::is_digit(char value)
{
	if (value >= 48 and value <= 57) return true;
	else return false;
}
 
 
 
double Expression::evaluate(string infix)
{
	//infix to postfix
	string postfix;
 
	Stack<char> st_char;
	auto i = infix.begin();
	while (i != infix.end()) {
 
		if (is_digit(*i)) {
			postfix.push_back(*i);
			++i;
			while (i != infix.end() and is_digit(*i)) {
				postfix.push_back(*i);
				++i;
			}
			postfix.push_back('#');
			continue;
		}
		if (st_char.is_empty()) {
			st_char.push(*i);
			++i;
			continue;
		}
		if (st_char.get_top() == '(' and *i == ')') {
			st_char.pop_out();
			++i;
			continue;
		}
 
		if (out_stack_precedence(*i) > in_stack_precedence(st_char.get_top())) {
			st_char.push(*i);
			++i;
		}
		else postfix.push_back(st_char.pop_out());
	}
	while (!st_char.is_empty()) postfix.push_back(st_char.pop_out());
 
 
	//postfix evaluation
	Stack<double> st;
	int size = postfix.size();
	int j = 0;
 
	while (j != size) {
		if (is_digit(postfix[j])) {
			int end = postfix.find('#', j);
			st.push(stod(postfix.substr(j, end)));
			j = end + 1;
			continue;
		}
 
		double b = st.pop_out();
		double a = st.pop_out();
		switch (postfix[j])
		{
		case '+':
			st.push(a + b);
			break;
		case '-':
			st.push(a - b);
			break;
		case '*':
			st.push(a * b);
			break;
		case '/':
			st.push(a / b);
			break;
		case '^':
			st.push(pow(a, b));
			break;
		}
		++j;
		
	}
	return st.pop_out();
}
```

## Main function

```cpp
#include <iostream>
#include "Expression.h"
using namespace std;
 
int main()
{
    Expression exp;
    cout << exp.evaluate("((13+2)*16)-2^2^3") << endl;
    cout << exp.evaluate("1+2") << endl;
}
```

## 输出结果

    -16
    3

