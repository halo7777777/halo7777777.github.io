# LZ77压缩算法(C++实现)

### 算法原理
> LZ77压缩算法采用字典的方式进行压缩，是一个简单但十分高效的数据压缩算法。其方式就是把数据中一些可以组织成短语(最长字符)的字符加入字典，然后再有相同字符出现采用标记来代替字典中的短语，如此通过标记代替多数重复出现的方式以进行压缩。

---
### 关键词
{{< admonition note "向前缓冲区">}}
 每次读取数据的时候，先把一部分数据预载入前向缓冲区。为移入滑动窗口做准备
{{< /admonition >}}
</br>
{{< admonition note "滑动窗口">}}
 一旦数据通过缓冲区，那么它将移动到滑动窗口中，并变成字典的一部分。
{{< /admonition >}}
</br>
{{< admonition note "短语字典">}}
 从字符序列S1...Sn，组成n个短语。比如字符(A,B,D) ,可以组合的短语为{(A),(A,B),(A,B,D),(B),(B,D),(D)},如果这些字符在滑动窗口里面，就可以记为当前的短语字典，因为滑动窗口不断的向前滑动，所以短语字典也是不断的变化。
{{< /admonition >}}

---
### 实例图解

##### 编码


![编码过程](https://pic.downk.cc/item/5ece38b7c2a9a83be59f2b62.png "编码格式图")
> 假设滑动窗口大小为10，向前缓冲区大小为6

</br>

![1](https://pic.downk.cc/item/5ece38c4c2a9a83be59f39d2.png "步骤1")
> (1) 
> </br>
> 假设滑动窗口中已经编码了“abcdbbccaa”, 在之后的向前缓冲区“abaeaa”中，匹配滑动窗口字典，得到最大匹配串“ab”,则滑动窗口和编码位置移动 2+1 = 3个位置

</br>

![2](https://pic.downk.cc/item/5ece38c4c2a9a83be59f39d6.png "步骤2")
> (2) 
> </br>
> 在之后的向前缓冲区“eaaaba”中，匹配滑动窗口字典，无法找到匹配串,则滑动窗口和编码位置移动 0+1 = 1个位置

</br>

![3](https://pic.downk.cc/item/5ece38c4c2a9a83be59f39d9.png "步骤3")
> (3) 
> </br>
> 在之后的向前缓冲区“aaabae”中，匹配滑动窗口字典，得到最大匹配串“aaabae”,则滑动窗口和编码位置移动 6+1 = 7个位置,此时编码位置超过了要编码的最大长度，结束编码

</br>

![结果](https://pic.downk.cc/item/5ece38c4c2a9a83be59f39dc.png "编码输出结果")
> 如上图，应输出02a00e46e,编码完成
---
##### 解码
> 编码过程逆向

---
### 代码实现
##### 编码
```cpp
#include<cstdio>
#include<iostream>
#include<iomanip>
#include<fstream>
#include<vector>
#include<ctime>
using namespace std;

/* 用于求KMP算法的next数组 */
void getNext(char s[], int len, char next[])
{
    int j = -1;
    next[0] = -1;
    for (int i = 1; i < len; ++i)
    {
        while (j != -1 && s[i] != s[j + 1])
        {
            j = next[j];
        }
        if (s[i] == s[j + 1])
        {
            j++;
        }
        next[i] = j;
    }
}

int main()
{
    ifstream in_file;
    in_file.open("asciiart.txt", ios::in);

    vector<char> inbuf;                // 文件输入的数据缓存
    vector<char> outbuf;               // 输出到文件的数据缓存
    char c;
    while (in_file >> c)
    {
        inbuf.push_back(c);
    }

    clock_t start, end;
    start = clock();

    int pos = 0;     // 当前指针所指inbuf的位置

    const int SlideWindowSize = 10;            // 滑动窗口的大小
    const int BufferSize = 6;                  // 前向缓冲区大小
    int SlideWindowIndex = 0;            // 当前滑动窗口的开始索引

    while (pos < inbuf.size())
    {
        char text[SlideWindowSize + 1];
        char pattern[BufferSize + 1];
        char next[BufferSize + 1];

        memset(text, 0, sizeof(text));
        memset(pattern, 0, sizeof(pattern));
        memset(next, 0, sizeof(next));

        memcpy(pattern, &inbuf[pos], (pos + BufferSize > inbuf.size() ? inbuf.size() - pos : BufferSize) * sizeof(char));
        SlideWindowIndex = pos > SlideWindowSize ? pos - SlideWindowSize : 0;
        memcpy(text, &inbuf[SlideWindowIndex], (pos > SlideWindowSize ? SlideWindowSize : pos) * sizeof(char));

        getNext(pattern, BufferSize, next);

        int j = -1;
        int off = 0;
        int len = 0;
        for (int i = 0; i < SlideWindowSize; ++i)
        {
            while (j != -1 && text[i] != pattern[j + 1])
            {
                j = next[j];
            }
            if (text[i] == pattern[j + 1])
            {
                j++;
            }
            if (j + 1 > len)
            {
                len = j + 1;
                if (pos < SlideWindowSize)
                    off = SlideWindowSize - pos + i - j;
                else
                    off = i - j;
            }
        }
        outbuf.push_back(off + '0');
        outbuf.push_back(len + '0');
        if(pos + len < inbuf.size())          // 若最后匹配的字符串刚好结束，之后无单个字符，则不输出编码
            outbuf.push_back(inbuf[pos + len]);
        pos += len + 1;
    }
    in_file.close();

    end = clock();
    cout << "编码时间：" << setprecision(10) << 1000.0 * (double)(end - start) / CLOCKS_PER_SEC << "ms";

    /* 写入到文件 */
    ofstream out_file;
    out_file.open("asciiart.bin", ios::out | ios::binary);
    out_file.write((char*)&outbuf[0], sizeof(char) * outbuf.size());
    out_file.close();

    return 0;
}
```
{{< admonition tip>}}
使用了KMP算法来匹配最长字符串
{{< /admonition >}}


---
##### 解码
```cpp
#include<cstdio>
#include<iostream>
#include<iomanip>
#include<fstream>
#include<vector>
#include<queue>
#include<ctime>
using namespace std;

int main()
{
    ifstream in_file;
    in_file.open("asciiart.bin", ios::in | ios::binary);

    queue<char> SlideWindow;                   // 滑动窗口

    vector<char> inbuf;            // 文件输入的数据缓存
    vector<char> outbuf;           // 输出到文件的数据缓存
    
    char c;
    while (in_file.read((char*) &c, sizeof(char)))
    {
        inbuf.push_back(c);
    }


    clock_t start, end;
    start = clock();

    int pos = 0;     // 当前指针所指inbuf的位置

    const int SlideWindowSize = 10;            // 滑动窗口的大小

    while (pos < inbuf.size())
    {
        if (inbuf[pos + 1] != '0')
        {
            queue<char> temp = SlideWindow;
            int num = (inbuf[pos] - '0') - (SlideWindowSize - SlideWindow.size());

            for (int i = 0; i < num; ++i)
            {
                temp.pop();
            }
            for (int i = 0; i < inbuf[pos + 1] - '0'; ++i)
            {
                SlideWindow.push(temp.front());
                temp.pop();
            }
        }
        if(pos + 2 < inbuf.size())      // 若最后匹配的字符串刚好结束，之后无单个字符，则不输入解码
            SlideWindow.push(inbuf[pos + 2]);
        for (int i = 10; i < SlideWindow.size(); ++i)
        {
            outbuf.push_back(SlideWindow.front());
            SlideWindow.pop();
        }

        pos += 3;
    }

    in_file.close();

    end = clock();
    cout << "解码时间：" << setprecision(10) << 1000.0 * (double)(end - start) / CLOCKS_PER_SEC << "ms";

    /* 写入到文件 */
    ofstream out_file;
    out_file.open("asciiart.txt", ios::out);
    for (int i = 0; i < outbuf.size(); i++)
    {
        out_file << outbuf[i];
    }
    while (!SlideWindow.empty())
    {
        out_file << SlideWindow.front();
        SlideWindow.pop();
    }
    out_file.close();

    return 0;
}
```

{{< admonition tip>}}
在最长匹配字符串比较多时，由于代码使用过多容器，会使解码速度下降，需要改进
{{< /admonition >}}

---
{{< admonition note "参考文章">}}
 [图解LZ77压缩算法](https://mp.weixin.qq.com/s/XGzuSF-L-qKriyRGcGs-KA)
{{< /admonition >}}
