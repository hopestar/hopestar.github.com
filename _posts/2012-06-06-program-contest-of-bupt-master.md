---
layout: post
title: 北邮计算机考研复试上级题目
---

现在是等录取通知书的时候。离研究生入学复试结束已经过去一个半月了，我能有幸以307的低分被录取，觉得关键还是要重视复试，尤其是计算机上机考试。

下面分享2012年北邮计算机学院的上机题目，我做出了其中的A、B、D三题，RankList排名在33（总人数281，四道题全部做出来的有11人）。

## A 二进制数 ##

**描述：**

- 大家都知道，数据在计算机里中存储是以二进制的形式存储的。
- 有一天，小明学了C语言之后，他想知道一个类型为`unsigned int`类型的数字，存储在计算机中的二进制串是什么样子的。
- 你能帮帮小明吗？并且，小明不想要二进制串中前面的没有意义的0串，即要去掉前导0。

**输入：**

- 第一行，一个数字T（T<=1000），表示下面要求的数字的个数。
- 接下来有T行，每行有一个数字n，表示要求的二进制串。

**输出:**

- 输出共T行。每行输出求得的二进制串。

**参考代码（C++）：**

    #include <iostream>
    #include <stack>
    using namespace std;

    int main()
    {
        unsigned int x;
        stack<int> bin;
        while (cin >> x) {
            while (x != 0) {
               int b = x % 2;
                bin.push(b);
                x = x / 2;
            }
            while (!bin.empty()) {
                cout << bin.top();
                bin.pop();
            }
            cout << endl;
        }
        return 0;
    }

## B 矩阵幂 ##

## C 二叉排序树 ##

## D IP数据包解析 ##

**描述：**

- 我们都学习过计算机网络，知道网络层IP协议数据包的头部格式如下：
- 其中IHL表示IP头的长度，单位是4字节；总长表示整个数据包的长度，单位是1字节。
- 你传输层的TCP协议数据段的头部格式如下：
- 你的任务是，简要分析输入数据中的若干个TCP数据段的头部。 详细要求请见输入输出部分的说明。

**输入：**

- 第一行为一个整数T，代表测试数据的组数。
- 以下有T行，每行都是一个TCP数据包的头部分，字节用16进制表示，以空格隔开。数据保证字节之间仅有一个空格，且行首行尾没有多余的空白字符。
- 保证输入数据都是合法的。

**输出：**

- 对于每个TCP数据包，输出如下信息：
- `Case #x`，x是当前测试数据的序号，从1开始。
- `Total length = L bytes`，L是整个IP数据包的长度，单位是1字节。
- `Source = xxx.xxx.xxx.xxx`，用点分十进制输出源IP地址。输入数据中不存在IPV6数据分组。
- `Destination = xxx.xxx.xxx.xxx`，用点分十进制输出源IP地址。输入数据中不存在IPV6数据分组。
- `Source Port = sp`，sp是源端口号。
- `Destination Port = dp`，dp是目标端口号。
- 请注意，输出的信息中，所有的空格、大小写、点符号、换行均要与样例格式保持一致，并且不要在任何数字前输出多余的前导0，也不要输出任何不必要的空白字符。

**参考代码：**

    #include <iostream>
    #include <string>
    using namespace std;

    int htod_2(char ba, char bb) {

        int da, db;
        if (ba >= '0' && ba <= '9')
            da = ba - '0';
        else if (ba >= 'a' && ba <= 'f')
            da = ba - 'a' + 10;
        if (bb >= '0' && bb <= '9')
            db = bb - '0';
        else if (bb >= 'a' && bb <= 'f')
            db = bb - 'a' + 10;
        return da * 16 + db;
    }

    int htod_4(char ba, char bb, char bc, char bd) {
        int da, db, dc, dd;
        if (ba >= '0' && ba <= '9')
            da = ba - '0';
        else if (ba >= 'a' && ba <= 'f')
            da = ba - 'a' + 10;
        if (bb >= '0' && bb <= '9')
            db = bb - '0';
        else if (bb >= 'a' && bb <= 'f')
            db = bb - 'a' + 10;
        if (bc >= '0' && bc <= '9')
            dc = bc - '0';
        else if (bc >= 'a' && bc <= 'f')
            dc = bc - 'a' + 10;
        if (bd >= '0' && bd <= '9')
            dd = bd - '0';
        else if (bd >= 'a' && bd <= 'f')
            dd = bd - 'a' + 10;
        return da * 16 * 16 * 16 + db * 16 * 16 + dc * 16 + dd;
    }

    int main() {

        int n;
        cin >> n;
        cin.ignore();

        for (int i = 0; i != n; ++i) {
            string ipd;
            getline(cin, ipd);

            cout << "Case #" << i + 1 << endl;
            cout << "Total length = "
                 << htod_4(ipd[6], ipd[7], ipd[9], ipd[10])
                 << " bytes" << endl;
            cout << "Source = "
                 << htod_2(ipd[36], ipd[37]) << "."
                 << htod_2(ipd[39], ipd[40]) << "."
                 << htod_2(ipd[42], ipd[43]) << "."
                 << htod_2(ipd[45], ipd[46]) << endl;
            cout << "Destination = "
                 << htod_2(ipd[48], ipd[49]) << "."
                 << htod_2(ipd[51], ipd[52]) << "."
                 << htod_2(ipd[54], ipd[55]) << "."
                 << htod_2(ipd[57], ipd[58]) << endl;
            cout << "Source Port = "
                 << htod_4(ipd[60], ipd[61], ipd[63], ipd[64]) << endl;
            cout << "Destination Port = "
                 << htod_4(ipd[66], ipd[67], ipd[69], ipd[70]) << endl;
            cout << endl;
        }
        return 0;
    }


