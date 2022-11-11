## golang-hashmap的具体原理和相关实现
### map底层结构体
```
// Map contains Type fields specific to maps.
type Map struct {
	Key  *Type // Key type
	Elem *Type // Val (elem) type

	Bucket *Type // internal struct type representing a hash bucket
	Hmap   *Type // internal struct type representing the Hmap (map header object)
	Hiter  *Type // internal struct type representing hash iterator state
}
```
![](https://raw.githubusercontent.com/DYX1112/LeetcodeTest/main/QQ%E5%9B%BE%E7%89%8720221026233140.png)    
**bucket**:在我们学习算法的时候我们会学习一种算法叫做散列法，其中散列法分为开散列法(又称链地址法以及开链法)和闭散列法，而我们的hash桶所采取的是开散列法，，而开散列法计算散列地址为（数组下标%（数组的长度-1）），如果我们在后面计算出了相同的下标，我们会在后面建立链表继续记录。例子：
![](https://raw.githubusercontent.com/DYX1112/LeetcodeTest/main/QQ%E5%9B%BE%E7%89%8720221026230144.png)   
而其中关键码相同的成为同一个子集合；每一个子集合即表示一个桶；集合采取的是链表进行相连接，而头结点则存储在哈希表中。   
 
 **hmap**
 ```
 // A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
 ```
![](./%5DJEAWUJ%6006H8NST9LS_0ICQ.png)   


