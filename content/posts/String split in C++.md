---
publish: true
title: tp-posts
aliases: []
date: 2024-03-28T21:47:56Z
lastmod: 2024-03-29T22:52:23Z
tags: []
category: posts
summary: C++ has no built-in string split function. This post demostrate routines like python str.split()
---
## Split string

1. use `getline` and `stringstream`

```cpp
std::vector<std::string> split(const std::string& in, char delimiter){
	std::stringstream ss(in);
	vector<std::string> tokens;
	std::string token;
	while(std::getline(ss,token,delimiter)){
	    tokens.push_back(token);
	}
	return tokens;
}
```

2. use the [`std::string::find()`](http://en.cppreference.com/w/cpp/string/basic_string/find) function to find the position of your string delimiter, then use [`std::string::substr()`](http://en.cppreference.com/w/cpp/string/basic_string/substr) to get a token.
in this case, the delimiter can be a string that would be more useful when handling delimiters that are longer than one character.

```cpp
std::vector<std::string> split(const std::string& s, std::string delimiter){
	std::vector<std::string> tokens;
	size_t pos = 0;
	size_t start = 0;
	while( (pos = s.find(delimiter, start)) != std::string::npos){
	    tokens.push_back(s.substr(start,pos););
	    start = pos + delimiter.length();
	}
	// last token
	tokens.push_back(s.substr(start));

	return tokens;
}
```

refer to:  https://stackoverflow.com/questions/14265581/parse-split-a-string-in-c-using-string-delimiter-standard-c

