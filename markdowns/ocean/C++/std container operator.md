# std 操作算法

## 查找算法

```C++
using namespace std;
int main() {
  int num = 0;
  vector<int> nums;
  nums.reserve(5);
  cout << "please input the numbers, 0 means end \n";
  while (true) {
    cin >> num;
    if (num == 0) {
      break;
    }
    cout << "added=" << num << endl;
    nums.push_back(num);
  }
  cout << "please input the number you are looking for:\n";
  cin >> num;
  auto ei = nums.cend();
  auto i = find(nums.cbegin(), ei, num);
  if (i == ei) // the result is end iterator, not found it
  {
    cout << "could not found the number in inputs\n";
  } else {
    cout << *i << " is founded in the inputs\n";
  }
}
```
如果是用find_if可以跟一个返回值为bool的lambda表达式：  
```C++
  std::cin >> num;
  auto ei = nums.cend();
  auto i = find_if(nums.cbegin(), ei, [num](int i) { return i == num;});
```

`find_all()` 函数会返回所有符合谓词的所有item。  
`count_if()` 判断并求数量
`generate()` 通过方法的返回值，替换vector中的值

## 生成一个2的对数的队列，并将其打印（generate()）

```C++
using namespace std;
int main() {
  vector<int> ints(10, 0);
  int value = 1;
  generate(begin(ints), end(ints), [&value] {
    value *= 2;
    return value;
  });
  for_each(cbegin(ints), cend(ints), [](int val) { cout << val << endl; });
}
```

`find_if_not`  
`minmax_element` find a pair with min and max value  
`find_first_of`  find first element  
`find_end` latest element  
`search`  `search_n` subsequence  

## 工具算法
`all_of()` `any_of()` `none_of()` `count()` `count_if()`


### mismatch
std::mismatch will return a pair, it contains the first position of the two compared container.  
so you can easily get the value that first un-equal element:  

### lexicographical_compare
the lexicographical_compare decide if the second container contain the first container from the first element, like a:`123`, b:`12345`, it will return `true`, if a:`23`, it will return `false`

```C++
  std::vector<int> a{1, 2}, b{2, 3, 4}, c{1, 2, 3, 4, 5};
  // v1 begin, v1 end, v2 begin
  std::cout << "compare equal = "
            << std::equal(std::cbegin(a), std::cend(a), std::cbegin(b))
            << std::endl;

  auto print_match = [](const std::pair<std::vector<int>::const_iterator, std::vector<int>::const_iterator> &pr,
                        const std::vector<int>::const_iterator &end_iter) {
    if (pr.first != end_iter) // the first unmatched position == end pos, match the vector
      std::cout << "\nFirst pair of words that differ are " << *pr.first
                << " and " << *pr.second << std::endl;
    else
      std::cout << "\nRanges are identical." << std::endl;
  };
  print_match(std::mismatch(std::cbegin(a), std::cend(a), std::cbegin(b)), std::cend(a));

  if (std::lexicographical_compare(std::cbegin(a), std::cend(a), std::cbegin(c), std::cend(c)))
  {
    std::cout << "first contain \n";
  }

  if (std::lexicographical_compare(std::cbegin(b), std::cend(b), std::cbegin(c), std::cend(c)))
  {
    std::cout << "first contain \n";
  } else {
    std::cout << "not first contain \n";
  }
  
```
## transform

1. 当作for_each循环修改
实例代码：
将A vector中的数自乘并形成新数组：
```C++
  std::vector<int> a{1, 2}, b{2, 3, 4}, c{1, 2, 3, 4, 5};
  std::transform(std::begin(a), std::end(a), std::begin(a),
                 [](int a){return a * a;});
  std::for_each(std::cbegin(a), std::cend(a), [](int a) { std::cout << a << std::endl; });
```

2. 将a vector + b vector放入c vector中：

```C++
int main() {
  std::vector<int> a{1, 2}, b{2, 3, 4}, c(2, 0); // transform will not push_back element to c automatically, initial it according to its need
  c.reserve(4);
  std::transform(std::begin(a), std::end(a), std::begin(b), std::begin(c),
                 [](int a, int b) { return a * b; });
  std::for_each(std::cbegin(c), std::cend(c),
                [](int c) { std::cout << c << std::endl; });
}
```

## copy copy_backward copy_if copy_n
## move move_backward (need element provide `operator= &&`)
## replace replace_if
## unique()
## reverse()
## shuffle() 乱序
## delete methods
1. erase() 
2. remove()  
先调用remove,在调用erase,就是删除-擦处法。  
下例讲的是如何删除空元素  
```C++
int main() {
  std::vector<std::string> strings {"", "jesse", "", "is", "good"};
  std::for_each(std::cbegin(strings), std::cend(strings), [](const std::string& s) {
    std::cout << s << std::endl;
  });
  std::cout<< "---------------after process------------------\n";
  auto it = std::remove_if(std::begin(strings), std::end(strings), [](const std::string &s) {
    return s.empty();
  });
  strings.erase(it, std::end(strings));
  std::for_each(std::cbegin(strings), std::cend(strings), [](const std::string& s) {
    std::cout << s << std::endl;
  });
}
```

## partition copy, 部分拷贝，将容器一分为二
实例，将一组数区分为奇数和偶数：  
```C++
int main() { 
  std::vector<int> original{1, 2, 3, 4, 5, 6, 7}; 
  std::vector<int> even(original.size(), -1), odd(original.size(), -1);
  std::partition_copy(std::cbegin(original), std::cend(original), std::begin(even), std::begin(odd), [](int i) {
    return i%2==0;
  });
  std::cout << "even:\n";
  for(auto i : even) {
    if (i >0)
    {
      std::cout << i << std::endl;
    }
  }
  std::cout << "odd:\n";
  for(auto i : odd) {
    if (i >0)
    {
      std::cout << i << std::endl;
    }
  }
}
```

## 排序
1. 普通排序

```C++
  std::vector<int> v1{2, 1, 3, 6, 4, 3, 7};
  std::sort(std::begin(v1), std:: end(v1));
  for (int i : v1) {
    std::cout << i << std::endl;
  }
```

## 集合算法
set_union() 并集  
set_intersection() 交集  
set_difference()  A-B  
set_symmetric_difference 对称差  
实例，取两个集合的并集：  
```C++
int main() {
  std::vector<int> v1{1, 2, 3}, v2{2, 3, 4};
  std::vector<int> result(3);
  std::set_intersection(std::cbegin(v1), std::cend(v1), std::cbegin(v2),
                      std::cend(v2), std::begin(result));
  for (auto i : result) {
    std::cout << i << std::endl;
  }
}
```

## 包含算法 include()

```C++
int main() {
  std::vector<int> v1{2, 3}, v2{1, 2, 2, 3, 4, 5}; // must be sorted
  if (std::includes(std::cbegin(v2), std::cend(v2), std::cbegin(v1),
                    std::cend(v1))) {
    std::cout << "v2 includes v1\n";
  }
}
```

## merge() 将两个已经sorted容器合并成一个sorted容器

## 最值 min max minmax min_element max_element minmax_element
_element返回的是迭代器位置  
minmax返回的是一个std::pair对象。

## 数值处理算法
accumulate() 累计运算
使用std::multiplies来计算容器元素的累乘

```C++
int main() {
  std::vector<int> v1{2, 3 , 4, 5};
  auto result = std::accumulate(std::cbegin(v1), std::cend(v1), 1, std::multiplies<int>());
  std::cout << result << std::endl;
}
```
inner_product() 向量内积  
iota() 顺序初始化 2,3,4,5...

## 通过std::copy 和ostream_iterator遍历容器
```C++
  std::vector<int> v(10);
  std::iota(std::begin(v), std::end(v), 1); // 1,2,3,4,5...10
  std::copy(std::cbegin(v), std::cend(v), std::ostream_iterator<int>(std::cout, "\n"));
```

## 小技巧
### unordered_map count()代替find()
`if(map.count(key)){}`
### 将两个vector进行排序，并将值付给第一个vector
排序要用到< operator，确保结构体或者类重写了这个operator
```C++
    std::vector<BranchMapVector> result;
    std::sort(basicBlockBranch.begin(), basicBlockBranch.end());
    std::sort(processedDomTreeBranch.begin(), processedDomTreeBranch.end());
    std::set_intersection(basicBlockBranch.begin(), basicBlockBranch.end(),
                          processedDomTreeBranch.begin(), processedDomTreeBranch.end(),
                          std::back_inserter(result));
    
    basicBlockBranch.swap(result);
```