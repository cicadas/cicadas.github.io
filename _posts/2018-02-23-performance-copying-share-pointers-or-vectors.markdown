---
layout: post
title:  "拷贝shared_ptr和拷贝vector的性能比较"
date:   2018-02-23 09:46:16 +0800
categories: 文章
---

C++里的共享指针带来很多编程上的方便之处，但是共享指针本身也有部分开销。这里写了一个简单的测试代码，来测试一个大小为100的float数组，拷贝数组的开销和拷贝共享指针的开销有多大差别。

测试代码如下：

```
#include <iostream>
#include <chrono>
#include <string>
#include <memory>
#include <vector>

using namespace std;
#define MAXLOOP 100000
int main(void)
{
    auto sharedVec = std::make_shared<vector<float>>();
    vector<float>& vec = *(sharedVec.get());
    for (int i=0; i<100; ++i) {
        vec.push_back(i);
    }

    auto startTime = std::chrono::high_resolution_clock::now();
    auto endTime = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double, std::milli> elapsed;

    startTime = std::chrono::high_resolution_clock::now();
    for (int i=0; i<MAXLOOP; ++i) {
        std::shared_ptr<vector<float>> other = sharedVec;
        (*other)[10] = 0.2;
    }
    endTime = std::chrono::high_resolution_clock::now();
    elapsed = endTime - startTime;
    cout << "copy share ptr, time: " << elapsed.count() << " ms" << endl;

    startTime = std::chrono::high_resolution_clock::now();
    for (int i=0; i<MAXLOOP; ++i) {
        vector<float> other = *sharedVec;
        other[10] = 0.2;
    }
    endTime = std::chrono::high_resolution_clock::now();
    elapsed = endTime - startTime;
    cout << "copy vector, time: " << elapsed.count() << " ms" << endl;

    startTime = std::chrono::high_resolution_clock::now();
    for (int i=0; i<MAXLOOP; ++i) {
        vector<float>* other = sharedVec.get();
        (*other)[10] = 0.2;
    }
    endTime = std::chrono::high_resolution_clock::now();
    elapsed = endTime - startTime;
    cout << "copy pointer, time: " << elapsed.count() << " ms" << endl;
}
```

编译命令：

```
g++ -Wall -std=c++14 -g -O2 test2.cpp
```

执行结果：

```
=> ./a.out
copy share ptr, time: 0.193916 ms
copy vector, time: 10.4002 ms
copy pointer, time: 6.4e-05 ms
```

如果把测试代码里的数组大小改成12，执行结果如下：

```
=> ./a.out
copy share ptr, time: 0.266917 ms
copy vector, time: 3.65854 ms
copy pointer, time: 5.1e-05 ms
```

可以看出，拷贝vector的开销还是明显大于拷贝shared_ptr的开销。因此在日常代码里，能够用shared_ptr代码vector拷贝的情况下还是尽量用shared_ptr更好。另外，虽然shared_ptr的拷贝开销比拷贝指针大很多倍，但是10万次拷贝才会造成小于1ms的延时，这个在真正代码执行里一般可以忽略掉。除非非常在意性能的情况下，可以考虑替换shared_ptr。