---
updated: '2023-11-01 08:00:00'
categories: code
excerpt: 'C++`fs::directory_iterator` 异常的处理方法'
date: '2023-11-01 08:00:00'
tags:
  - C++
urlname: cxx_directory_iterator_exception
title: C++ directory_iterator 异常处理
---

C++ 标准库 遍历文件夹时偶尔出现异常


```c++
The system cannot find the path specified
```


异常可能是访问错误或者权限方面的原因，要避免异常可以这样写


```c++
std::error_code ec;
for ( auto it = fs::directory_iterator( dir, ec ); !ec && it != fs::directory_iterator(); it.increment( ec ) )
{
	if ( !it->is_regular_file( ec ) ) continue;
}
```


或者封装一层 采用 for ... auto 语法


```c++
/*
std::error_code ec;
safe_directory sdir{ std::filesystem::current_path(), ec };

for ( auto dirEntry : sdir )
{
    if ( dirEntry.is_regular_file( ec ) )
        std::cout << dirEntry.path() << std::endl;
}
*/

//iterator of directory items that will save any errors in (ec) instead of throwing exceptions
struct safe_directory_iterator
{
    fs::directory_iterator it;
    std::error_code & ec;
    safe_directory_iterator & operator ++() { it.increment(ec); return *this; }
    auto operator *() const { return *it; }
};

// object of this struct can be passed to range based for
struct safe_directory
{
    fs::path dir;
    std::error_code & ec;

    base::safe_directory_iterator begin()
    {
        return base::safe_directory_iterator{ fs::directory_iterator(dir, ec), ec };
    }

    fs::directory_iterator end()
    {
        return {};
    }
};

inline bool operator !=(const safe_directory_iterator & a, const fs::directory_iterator & b)
{
    return !a.ec && a.it != b;
}

```


使用方法


```c
std::error_code ec;
safe_directory sdir{ std::filesystem::current_path(), ec };

for ( auto dirEntry : sdir )
{
    if ( dirEntry.is_regular_file( ec ) )
        std::cout << dirEntry.path() << std::endl;
}
```

