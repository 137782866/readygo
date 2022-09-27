## std::vector 介绍

使用标准库 vector 时，需要包含头文件 `<vecotr>`。

vector 模板声明如下：

```cpp
template < class T, class Alloc = allocator<T> >
```

vector 表示大小可变的数组序列容器，和数组类似，vector 对其元素使用连续的存储位置，这就意味着可以使用指向其元素的常规指针的偏移量来访问元素，且与数组中的元素一样有效。但是 vector 的大小是可动态变化的，它们的存储有容器自动处理。

在 vector 内部，使用了动态分配的内存来存储其元素。当插入新的元素时，可能需要重新分配其管理的数组以增加大小，这意味着，当分配了新的数组时，需要将原数组的元素移动到该数组。从效率方面来讲，这是不小的开销，因此，vector 不会在每次将元素添加到容器时重新分配内存。相反，vector 容器会提前分配一些额外的空间来适应可能的增长，所以容器的实际容量也会大于等于其包含的元素需要的存储大小。可以实现不同的增长策略，以使内存使用和重新分配之间取得平衡。

与数组相比，vector 会消耗更多的内存，以换取存储管理和有效动态增长。

与其他序列容器相比，vector 访问元素像数组一样高效，从末端添加或删除元素也很高效，对于涉及到末尾以外的位置插入或删除元素的操作，性能较差，且会出现迭代器失效情况。

## 构造 std::vector

vector 有多种构造方法，C++11 中共 6 种，分别是：

1. 默认构造函数。构造一个空的容器，没有元素。

   ```cpp
   explicit vector (const allocator_type& alloc = allocator_type());
   ```

2. 填充构造。构造一个包含 n 个值为 val 的元素。

   ```cpp
   explicit vector (size_type n);
   vector (size_type n, const value_type& val, const allocator_type& alloc = allocator_type());
   ```

3. 范围构造。构造一个包含与范围 `[first，last)` 一样多的元素的容器，且顺序相同。

   ```cpp
   template <class InputIterator>  vector (InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
   ```

4. 拷贝构造。以相同的顺序构造一个容器，其中包含 x 中每个元素的副本。

   ```cpp
   vector (const vector& x);vector (const vector& x, const allocator_type& alloc);
   ```

5. 移动构造。构造一个获取 *x* 元素的容器。如果指定了 alloc 并且与 x 的分配器不同，则移动元素。否则，不会构造任何元素（所有权直接转移）。

   ```cpp
   vector (vector&& x);
   vector (vector&& x, const allocator_type& alloc);
   ```

6. 初始化列表构造。以相同的顺序构造一个容器，其中包含 il 中每个元素的副本。

   ```cpp
   vector (initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
   ```

## 成员函数

1. 容量和大小相关

   - **size()**

     返回 vector 中的元素的个数

   - **max_size()**

     返回 vector 可容纳最大元素数，由于系统或库的限制，这是容易可以达到的最大潜在大小，但容器绝不能保证能够达到该大小，因为在达到该大小之前，就很可能无法分配存储，一般操作系统容纳最大数为 2305843009213693951。

     ```cpp
     std::vector<int> vec;
     std::cout << "max_size:" << vec.max_size() << std::endl; //max_size:2305843009213693951
     ```

   - **void resize(size_type n)**

   - **void resize (size_type n, const value_type& val)**

     调整容器 size 使其可以包容纳 n 个元素。如果 n 小于当前容器元素个数，将减少到前 n 个元素，删除超出的元素并销毁。

     如果 n 大于当前容器 size，则通过在末尾插入所需数量元素以达到 n 的大小来扩展，如果指定了 val，则每个新元素初始化为 val。

     如果 n 也大于当前容器容量 capacity，则会自动分配存储空间为 n 大小，且下插入元素后 capacity 扩展为 n 的 2 倍，但不一定是 2 的次方。

     总结：调用 resize 方法后，vector 的 size  会变成 n，如果 old_capacity >= n，capacity = old_capacity，如果old_capacity < n，capacity = n。

   - **capacity()**

     返回 vector 分配的存储空间大小，capacity 不一定等于 size，但必须 >= size。因为不用在每次插入时重新分配内存。此容量不会对 vector 的大小限制，当此容量耗尽时会自动拓展它，vector 的大小的限制由成员 max_size() 给出。

   - **empty()**

     返回向量是否为空，也就是成员 size 方法是否为零。不会以任何方式修改容器，要清楚矢量内容，需要调用 clear 方法。此方法与 capacity 没有关系。

   - **reserve (size_type n)**

     将 vector 的容量改变为至少足以包含 n 个元素，如果 n 大于当前 vector 的 capacity，则会重新分配存储，并将其容量增加到 n （或更大），在其他情况下，不会导致重新分配，并且当前 vector 不受影响，此函数对 vector 的大小没有影响，也不能改变其元素。注意：如果请求大小大于最大大小 max_size，将会引发 length_error 异常。
     
     ```cpp
     std::vector<int> vec;
     for (int i = 0; i < 10; ++i)
     { 
         vec.push_back(i);
     } 
     vec.reserve(12);
     cout << vec.size() << "-" << vec.capacity() << endl;	//10-16
     
     vec.reserve(8);
     cout << vec.size() << "-" << vec.capacity() << endl;	//10-16
     ```
     
   - **shrink_to_fit (size_type n)**

     将 vector 的 capacity 减小到 size 大小，请求是非强制的，容器的实现可以自由的进行优化，并使 vector 的 capacity 具有大于其 size 的容量。可能会导致重新分配，但是对 vector 的size 没有影响，也无法改变其元素。需要注意迭代器失效的情况。

     ```cpp
     std::vector<int> vec(100);
     cout << vec.capacity() << endl;		//100
     vec.resize(10);
     cout << vec.capacity() << endl;		//100
     vec.shrink_to_fit();
     cout << vec.capacity() << endl;		//10
     ```

2. 访问元素相关

   - **ref operator[] (size_type n)**

     **const_ref operator[] (size_type n) const**

     返回 vector 容器中下标 n 处元素的引用，成员函数 at 与此运算符有相同的行为，但是 vector::at 有经过边界检查，如果请求位置超出范围，会通过引发 out_of_range 异常发出信号。永远不要用超出范围的参数 n 调用此函数，因为这会导致未定义的行为。在容器中，第一个元素的位置为 0（而不是 1）。

   - **ref at (size_type n)**
   
   - **const_ref at (size_type n) const**
   
     返回 vector 中位于下下标 n 的元素的引用。上边已经说过 operator[] 与 at 的区别。













































