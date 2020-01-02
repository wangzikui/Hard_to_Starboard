## 位运算方法计算a+b

```java
/**
a + b 的问题拆分为 (a 和 b 的无进位结果) + (a 和 b 的进位结果)
	无进位加法:a ^ b
	进位:(a & b) << 1
	循环此过程，直到进位为 0
*/
public int getSum(int a, int b) {
    while(b!=0)
    {
        int res = (a&b)<<1;
        a = a^b;
        b = res;              
    }
    return a;
}
```

------

## 