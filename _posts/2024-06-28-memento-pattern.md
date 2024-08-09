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

