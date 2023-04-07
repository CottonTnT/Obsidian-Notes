### 简介
**Trie树**，又叫**字典树**、**前缀树（Prefix Tree）**、**单词查找树** 或 **键树**，是一种多叉树结构。如下图：

![[Pasted image 20230406221947.png]]


上图是一棵 Trie 树，表示了关键字集合{“a”, “to”, “tea”, “ted”, “ten”, “i”, “in”, “inn”} 。从上图可以归纳出 Trie 树的基本性质：
``` summary
根节点不包含字符，除根节点外的每一个子节点都包含一个字符。
从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串。
每个节点的所有子节点包含的字符互不相同。
```

通常在实现的时候，会在节点结构中设置一个标志，用来标记该结点处是否构成一个单词（关键字）。

可以看出，Trie 树的关键字一般都是字符串，而且 Trie 树把每个关键字保存在一条路径上，而不是一个结点中。另外，两个有公共前缀的关键字，在 Trie 树中前缀部分的路径相同，所以 Trie 树又叫做前缀树（Prefix Tree）。


### 适用范围

1. 字符串检索
2. 词频统计
3. 字符串排序
4. 前缀匹配




###  原理及操作

```                                   c
#include <iostream>
#include <string>
using namespace std;

#define ALPHABET_SIZE 26

typedef struct trie_node
{
    int count;   // 记录该节点代表的单词的个数
    trie_node *children[ALPHABET_SIZE]; // 各个子节点 
}*trie;

trie_node* create_trie_node()
{
    trie_node* pNode = new trie_node();
    pNode->count = 0;
    for(int i=0; i<ALPHABET_SIZE; ++i)
        pNode->children[i] = NULL;
    return pNode;
}

void trie_insert(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p)
    {
        if(node->children[*p-'a'] == NULL)
        {
            node->children[*p-'a'] = create_trie_node();
        }
        node = node->children[*p-'a'];
        ++p;
    }
    node->count += 1;
}

void trie_delete(trie root, char* key){

}

/**
 * 查询：不存在返回0，存在返回出现的次数
 */ 
int trie_search(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p && node!=NULL)
    {
        node = node->children[*p-'a'];
        ++p;
    }

    if(node == NULL)
        return 0;
    else
        return node->count;
}

int main()
{
    // 关键字集合
    char keys[][8] = {"the", "a", "there", "answer", "any", "by", "bye", "their"};
    trie root = create_trie_node();

    // 创建trie树
    for(int i = 0; i < 8; i++)
        trie_insert(root, keys[i]);

    // 检索字符串
    char s[][32] = {"Present in trie", "Not present in trie"};
    printf("%s --- %s\n", "the", trie_search(root, "the")>0?s[0]:s[1]);
    printf("%s --- %s\n", "these", trie_search(root, "these")>0?s[0]:s[1]);
    printf("%s --- %s\n", "their", trie_search(root, "their")>0?s[0]:s[1]);
    printf("%s --- %s\n", "thaw", trie_search(root, "thaw")>0?s[0]:s[1]);

    return 0;
}

```

