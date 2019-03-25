####  数字转化为字符串
```c++
#include <sstream>
void i2s(long long x,string& str)
{
    stringstream ss;
    ss<<x;
    ss>>str;
}
```

#### 字符串转化为数字
```c++
void s2i(string &str,int &num)
{
    stringstream ss;
    ss<<str;
    ss>>num;
}
```

#### 分割字符串
```c++
string s;
getline(cin,s);
string temp;    // 切分后的每一段
istringstream iss(s);
while(getline(iss,temp,' '))
{
    s2i(temp,data[index++]);
}
```