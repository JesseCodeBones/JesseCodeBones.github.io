# user and group
## getpwnam & getpwuid
```C++
auto result = getpwnam("jesse"); // return NULL of cannot find user
auto result2 = getpwuid(result->pw_uid);
std::cout << result2->pw_dir << std::endl;
```
## 其他方法
`getgrnam` `getgrgid` 和group相关  

## crypt
crypt()算法会接受一个最长可达 8 字符的密钥（即密码），并施之以数据加密算法（DES）的一种变体。salt 参数指向一个两字符的字符串，用来扰动（改变）DES 算法  
```C++
//http://www.itl.nist.gov/fipspubs/fip46-2.htm
auto result = crypt("12345678", "21");
std::cout << result << std::endl;
auto result2 = crypt("12345678", "21");
std::cout << result2 << std::endl;
// target_link_libraries(cmaketest -lcrypt)
```

