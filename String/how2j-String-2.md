# how2j-Java-String-练习2
## 题目
创建一个长度是8的字符串数组
使用8个长度是5的随机字符串初始化这个数组
对这个数组进行排序，按照每个字符串的首字母排序(无视大小写)
注1： 不能使用Arrays.sort() 要自己写
注2： 无视大小写，即 Axxxx 和 axxxxx 没有先后顺序
## 审题
要做以下几件事:
1. 生成长度为5的随机字符串，首字母可以是大写/小写
2. 对字符串进行排序，从小到大
## 题解
### 解决随机字符串生成
随机字符串生成实际上是要创建一个char[]数组，把每个单元赋予随机字符，然后转为String就好了。那么如何去生成这个字符串？
使用random()是肯定的，虽然只能生成0-1范围内的随机数，但是可以用来划定范围，这里用到ASCII码表，把0-z的范围框定出来就好了
```java
public static void main(String[] args) {
         
      	char[] a = new char[5];
		// 划定首尾，分别是0和z字符的ASCII码值
        int start = '0';
        int end = 'z';
        char temp;
        for(int i=0;i<a.length;i++)
        {
			// 一直循环，知道随机的是一个字符就退出
            while(true) {
				// 在范围内随机生成数，不是字符就继续生成
                temp = (char)(Math.random()*(end - start)+start);
                if(Character.isLetter(temp))
                {
                    a[i] = temp;
                    break;
                }                    
            }
        }
		// char[]转为String类型
       	String s = String.copyValueOf(a);
        System.out.println(s);
}
```
### 解决字符串排序
```
		for(int i = 1;i<8;i++)
        {
            for(int j = 0;j<8-i;j++)
            {
				// 把首字符全部转换为小写进行ASCII码值的比较即可
                int k =Character.toLowerCase(c_sort[j].charAt(0));
                int s =Character.toLowerCase(c_sort[j+1].charAt(0));
                if(k>s)
                {
                    String temp = new String();
                    temp = c_sort[j];
                    c_sort[j] = c_sort[j+1];
                    c_sort[j+1] = temp;
                }
            }
        }
		// 输出排序后的内容
        for(String s :c_sort)
            System.out.println(s);
```