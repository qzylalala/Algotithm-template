## 算法模板

* 一般哈希

  ```c++
  (1) 拉链法
      int h[N], e[N], ne[N], idx;
  
      // 向哈希表中插入一个数
      void insert(int x)
      {
          int k = (x % N + N) % N;
          e[idx] = x;
          ne[idx] = h[k];
          h[k] = idx ++ ;
      }
  
      // 在哈希表中查询某个数是否存在
      bool find(int x)
      {
          int k = (x % N + N) % N;
          for (int i = h[k]; i != -1; i = ne[i])
              if (e[i] == x)
                  return true;
  
          return false;
      }
  
  (2) 开放寻址法
      int h[N];
  
      // 如果x在哈希表中，返回x的下标；如果x不在哈希表中，返回x应该插入的位置
      int find(int x)
      {
          int t = (x % N + N) % N;
          while (h[t] != null && h[t] != x)
          {
              t ++ ;
              if (t == N) t = 0;
          }
          return t;
      }
  ```

  [模拟散列表](https://www.acwing.com/activity/content/problem/content/890/1/)



* 字符串哈希

  ```c++
  核心思想：将字符串看成P进制数，P的经验值是131或13331，取这两个值的冲突概率低
  小技巧：取模的数用2^64，这样直接用unsigned long long存储，溢出的结果就是取模的结果
  
  typedef unsigned long long ULL;
  ULL h[N], p[N]; // h[k]存储字符串前k个字母的哈希值, p[k]存储 P^k mod 2^64
  
  // 初始化
  p[0] = 1;
  for (int i = 1; i <= n; i ++ )
  {
      h[i] = h[i - 1] * P + str[i];
      p[i] = p[i - 1] * P;
  }
  
  // 计算子串 str[l ~ r] 的哈希值
  ULL get(int l, int r)
  {
      return h[r] - h[l - 1] * p[r - l + 1];
  }
  ```

  [字符串哈希](https://www.acwing.com/activity/content/problem/content/891/1/)

  

* C++  STL常用函数

  ```c++
  vector, 变长数组，倍增的思想
      size()  返回元素个数
      empty()  返回是否为空
      clear()  清空
      front()/back()
      push_back()/pop_back()
      begin()/end()
      []
      支持比较运算，按字典序
  
  pair<int, int>
      first, 第一个元素
      second, 第二个元素
      支持比较运算，以first为第一关键字，以second为第二关键字（字典序）
  
  string，字符串
      size()/length()  返回字符串长度
      empty()
      clear()
      substr(起始下标，(子串长度))  返回子串
      c_str()  返回字符串所在字符数组的起始地址
  
  queue, 队列
      size()
      empty()
      push()  向队尾插入一个元素
      front()  返回队头元素
      back()  返回队尾元素
      pop()  弹出队头元素
  
  priority_queue, 优先队列，默认是大根堆
      size()
      empty()
      push()  插入一个元素
      top()  返回堆顶元素
      pop()  弹出堆顶元素
      定义成小根堆的方式：priority_queue<int, vector<int>, greater<int>> q;
  
  stack, 栈
      size()
      empty()
      push()  向栈顶插入一个元素
      top()  返回栈顶元素
      pop()  弹出栈顶元素
  
  deque, 双端队列
      size()
      empty()
      clear()
      front()/back()
      push_back()/pop_back()
      push_front()/pop_front()
      begin()/end()
      []
  
  set, map, multiset, multimap, 基于平衡二叉树（红黑树），动态维护有序序列
      size()
      empty()
      clear()
      begin()/end()
      ++, -- 返回前驱和后继，时间复杂度 O(logn)
  
      set/multiset
          insert()  插入一个数
          find()  查找一个数
          count()  返回某一个数的个数
          erase()
              (1) 输入是一个数x，删除所有x   O(k + logn)
              (2) 输入一个迭代器，删除这个迭代器
          lower_bound()/upper_bound()
              lower_bound(x)  返回大于等于x的最小的数的迭代器
              upper_bound(x)  返回大于x的最小的数的迭代器
      map/multimap
          insert()  插入的数是一个pair
          erase()  输入的参数是pair或者迭代器
          find()
          []  注意multimap不支持此操作。 时间复杂度是 O(logn)
          lower_bound()/upper_bound()
  
  unordered_set, unordered_map, unordered_multiset, unordered_multimap, 哈希表
      和上面类似，增删改查的时间复杂度是 O(1)
      不支持 lower_bound()/upper_bound()， 迭代器的++，--
  
  bitset, 圧位
      bitset<10000> s;
      ~, &, |, ^
      >>, <<
      ==, !=
      []
  
      count()  返回有多少个1
  
      any()  判断是否至少有一个1
      none()  判断是否全为0
  
      set()  把所有位置成1
      set(k, v)  将第k位变成v
      reset()  把所有位变成0
      flip()  等价于~
      flip(k) 把第k位取反
  ```

  