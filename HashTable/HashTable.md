# HashTable
总结碰到的涉及哈希表、哈希集合的相关知识。

## 结构原理

HashMap的实现原理：
1、基于哈希表（数组+链表+二叉树（红黑树）是在1.8jdk才实现的）
2、默认加载因子为0.75（加载因子是指：当存储的数据占据了总容量的75%就不在存储或者需要扩容），默认数组大小是16
3、把对象存储到哈希表中，如何存储？
--- 把key对象通过hash()方法计算hash值，然后用这个hash值对数组长度取余数（默认16），来决定该Key对象
在数组中存储的位置，当这个位置有多个对象时，以链表结构存储，jdk1.8后，当链表长度大于8时，链表将转换为红黑树结构存储。
这样做的目的：是为了取值更快，存储的数据量越大，性能的表现越明显。
4、 扩充原理：当数组的容量超过了75%，那么表示该数组需要扩充，如何扩充？    
扩充的算法是：当前数组容量<<1（相当于是乘2，扩大1倍）   ，扩充次数过多，会影响性能 ，每次扩充表示哈希表重新散列(重新计算每个对象的存储位置)，
我们在开发中尽量要减少扩充次数带来的性能问题
5、线程不安全，适合在单线程中使用。 

## HashMap

```
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

/**
 * Map接口：
 * 1、键-值对存储一组对象
 * 2、Key不能重复(唯一)，Value可以重复
 * 3、具体的重点实现类：HashMap  TreeMap  Hashtable  LinkedHashMap
 */

public class MapDemo {
    public static void main(String[] args) {
        hashMap();
    }
    /**

    
    public static void hashMap() {
        Map<Integer,String> map = new HashMap<>();
        // .put()。Map的存储：先将(1,"Tom")new了一个Entry保存，然后将Entry再存入Map中，如此实现Map中键值的存储
        map.put(1, "Tom");  //存入Map
        map.put(2, "Jerry");
        map.put(3, "Lily");
        map.put(4, "Bob");
        
        // .get()。从Map中取值：通过key取value，不存在则返回null
        System.out.println(map.get(1));
        
		// .getOrDefault。下例预设不存在的返回值为0
		System.out.println(map.getOrDefault(1, 0));
        
        //Map的第1种遍历：遍历key，然后通过key取值
        //遍历key生成一个Set集合，里面的泛型是Integer
        Set<Integer> keys = map.keySet();
        for(Integer k:keys) {
            System.out.println(map.get(k));
        }
        
        //Map的第2种遍历：直接遍历value
        //遍历Value生成一个Set集合，里面的泛型是String
        Collection<String> values = map.values();
        for(String v:values) {
            System.out.println(v);
        }
        
        // .containsKey()。判断是否包含某key；返回boolean
        System.out.println(map.containsKey(5));
        
        
    }
}
```
## HashSet
HashSet关键字常用于归纳不重复元素的问题，set是一个不重复集合。

