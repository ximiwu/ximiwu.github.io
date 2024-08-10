---
layout:     post
title:      "设计模式：备忘录模式-实现编辑器多级撤销"
subtitle:   ""
date:       2024-06-28 21:30:00
author:     "西米屋花火"
catalog: true
tags:
    - 设计模式
---
# [NOI1995] 石子合并

## 题目描述

在一个圆形操场的四周摆放 $N$ 堆石子，现要将石子有次序地合并成一堆，规定每次只能选相邻的 $2$ 堆合并成新的一堆，并将新的一堆的石子数，记为该次合并的得分。

试设计出一个算法,计算出将 $N$ 堆石子合并成 $1$ 堆的最小得分和最大得分。

## 输入格式

数据的第 $1$ 行是正整数 $N$，表示有 $N$ 堆石子。

第 $2$ 行有 $N$ 个整数，第 $i$ 个整数 $a_i$ 表示第 $i$ 堆石子的个数。

## 输出格式

输出共 $2$ 行，第 $1$ 行为最小得分，第 $2$ 行为最大得分。

## 样例 #1

### 样例输入 #1

```
4
4 5 9 4
```

### 样例输出 #1

```
43
54
```

## 提示

$1\leq N\leq 100$，$0\leq a_i\leq 20$。

## Editor:存储内容、生成EditorState

```cpp
 
#pragma once
#include <string>
#include "EditorState.h"
using namespace std;
class Editor
{
public:
	EditorState* createState();
	void restore(EditorState& state);
	void setContent(string str);
	string getContent();
private:
	string content;
};
 
 
#include "Editor.h"
#include <iostream>
using namespace std;
EditorState* Editor::createState()
{
	return new EditorState(content);
}
 
void Editor::restore(EditorState& state)
{
	content = state.getContent();
	delete &state;
}
 
void Editor::setContent(string str)
{
	
	content = str;
}
 
string Editor::getContent()
{
 
	return content;
}
```

## EditorState:存储内容

```cpp
#pragma once
#include <string>
using namespace std;
class EditorState
{
public:
	EditorState(string& content);
	string& getContent();
private:
	string content;
};
 
 
#include "EditorState.h"
 
EditorState::EditorState(string& content) : content{content}
{
}
 
string& EditorState::getContent()
{
	return content;
}
```

## History:保存、操作历史状态

```cpp
#pragma once
#include <string>
#include <vector>
#include "EditorState.h"
using namespace std;
class History
{
private:
	vector<EditorState*> states;
public:
	void push(EditorState* state);
	EditorState* pop();
};
 
 
#include "History.h"
 
void History::push(EditorState* state)
{
	states.push_back(state);
}
 
EditorState* History::pop()
{
	EditorState* state = states.back();
	states.pop_back();
	return state;
	
}
```

## Main function:

```cpp
#include <iostream>
#include "Editor.h"
#include "History.h"
using namespace std;
 
void setContent(string str, Editor& editor, History& history) {
    history.push(editor.createState());
    editor.setContent(str);
    cout << "writeContent:" << editor.getContent() << endl;
    
}
 
void undo(Editor& editor, History& history) {
    EditorState* state = history.pop();
    editor.restore(*state);
    cout << "undoContent:" << editor.getContent() << endl;
}
 
int main()
{
    Editor editor;
    History history;
    setContent("1", editor, history);
    setContent("2", editor, history);
    setContent("3", editor, history);
    undo(editor, history);
    undo(editor, history);
    undo(editor, history);
}
```

## 输出结果：

```cpp
writeContent:1
writeContent:2
writeContent:3
undoContent:2
undoContent:1
undoContent:
```

