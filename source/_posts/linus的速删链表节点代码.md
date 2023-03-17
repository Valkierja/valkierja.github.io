---
title: linus的速删链表节点代码
poc: true
categories:
  - - '%e8%af%be%e7%a8%8b%e7%ac%94%e8%ae%b0'
    - 数据结构
tags: []
id: '1171'
date: 2021-09-29 14:50:33
---

```
void RemoveByValue(int value ,LinkNode ** head) {

for (LinkNode** curr = head;*curr ; )
{
LinkNode* entry = *curr;
if (entry->m_Value)
{
*curr = entry->m_Next;  通过改变一级指针间接改变二级指针的指向
free(entry);
}
else curr = &entry->m_Next;//点睛之笔二级指针的一级指针的地址,链表的next指针也属于一级指针
}

}
```