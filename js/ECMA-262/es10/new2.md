# JSON.stringify

## 一、历史
如果输入Unicode格式但是超出范围的字符，返回格式错误的Unicode字符串。

## 二、改进
实现了一个改变JSON.stringify的第三阶段提案，因此它为其输出转义序列，使其成为有效Unicode(并以UTF-8表示)