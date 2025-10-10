---
title: 桶排序
linkTitle: "桶排序"
weight: 2
draft: false
description: >
    
---

桶排序（英语：Bucket sort），工作的原理是将数组分到有限数量的桶里。每个桶再分别排序，有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序。

算法的自然语言描述：
1. 设置一个定量的数组当作空桶子。
2. 寻访序列，并且把项目一个一个放到对应的桶子去。
3. 对每个不是空的桶子进行排序。
4. 从不是空的桶子里把项目再放回原来的序列中。

伪码描述：

```pseudocode
function bucket-sort(array, n) is
  buckets ← new array of n empty lists
  for i = 0 to (length(array)-1) do
    insert array[i] into buckets[msbits(array[i], k)]
  for i = 0 to n - 1 do
    next-sort(buckets[i])
  return the concatenation of buckets[0], ..., buckets[n-1]
```

CPP 实现

```cpp
#include<iterator>
#include<iostream>
#include<vector>
using namespace std;
const int BUCKET_NUM = 10;

struct ListNode{
	explicit ListNode(int i=0):mData(i),mNext(NULL){}
	ListNode* mNext;
	int mData;
};

ListNode* insert(ListNode* head,int val){
	ListNode dummyNode;
	ListNode *newNode = new ListNode(val);
	ListNode *pre,*curr;
	dummyNode.mNext = head;
	pre = &dummyNode;
	curr = head;
	while(NULL!=curr && curr->mData<=val){
		pre = curr;
		curr = curr->mNext;
	}
	newNode->mNext = curr;
	pre->mNext = newNode;
	return dummyNode.mNext;
}

ListNode* Merge(ListNode *head1,ListNode *head2){
	ListNode dummyNode;
	ListNode *dummy = &dummyNode;
	while(NULL!=head1 && NULL!=head2){
		if(head1->mData <= head2->mData){
			dummy->mNext = head1;
			head1 = head1->mNext;
		}else{
			dummy->mNext = head2;
			head2 = head2->mNext;
		}
		dummy = dummy->mNext;
	}
	if(NULL!=head1) dummy->mNext = head1;
	if(NULL!=head2) dummy->mNext = head2;

	return dummyNode.mNext;
}

void BucketSort(int n,int arr[]){
	vector<ListNode*> buckets(BUCKET_NUM,(ListNode*)(0));
	for(int i=0;i<n;++i){
		int index = arr[i]/BUCKET_NUM;
		ListNode *head = buckets.at(index);
		buckets.at(index) = insert(head,arr[i]);
	}
	ListNode *head = buckets.at(0);
	for(int i=1;i<BUCKET_NUM;++i){
		head = Merge(head,buckets.at(i));
	}
	for(int i=0;i<n;++i){
		arr[i] = head->mData;
		head = head->mNext;
	}
}
```

Python 实现：

```python
def is_sort(arr: list) -> bool:
    for i in range(len(arr) - 1):
        if arr[i] > arr[i + 1]:
            return False
    return True

def bucket_sort(arr: list, is_sub_bucket: bool = False):
    if is_sort(arr):
        return

    bucket_num: int = max(arr) // 10 + 1 if not is_sub_bucket else 10
    buckets: list[list] = [[] for _ in range(bucket_num)]

    for a in arr:
        i: int = a // 10 if not is_sub_bucket else a % 10
        buckets[i].append(a)

    arr.clear()
    for bucket in buckets:
        bucket_sort(bucket, is_sub_bucket=True)
        arr += bucket

if __name__ == '__main__':
    arr = [29, 25, 3, 49, 9, 37, 21, 43]
    bucket_sort(arr)
    print(arr)
```
