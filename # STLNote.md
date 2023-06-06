# array

就是数组, 包装后让其可以享受算法等部件的交互```array```没有ctor、dtor。

```array```的```iterator```就是一个指针编译器会对其进行优化，将其视为普通的C++数组。

大小固定：一旦创建了```array```，它的**大小就不能更改**。

连续存储：```array```中的元素在内存中是连续存储的，可以通过指针算术运算快速访问元素。

支持STL算法：```array```可以像其他容器一样使用STL算法。

不支持插入和删除操作：由于```array```的大小固定，因此无法在其中插入或删除元素。

 ```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// 访问元素
std::cout << "arr[0] = " << arr[0] << std::endl;
std::cout << "arr.at(1) = " << arr.at(1) << std::endl;

// 迭代器遍历
for (auto it = arr.begin(); it != arr.end(); ++it) {
	std::cout << *it << " ";
}
 ```

# vector

增长为二倍增长, ```capacity()```可以看到目前的容量大小

```vector```内部只有三个```protected```的指针: ```start```, ```finish```, ```end_of_storage```

```sizeof```一个```vector```大小为3个指针的大小

finish-start为```size```, end_of_storage-start为```capacity```

使用```vector```时会大量调用构造函数, 复制构造函数, 析构函数, 是很大的开销

```vector```是一个动态数组，其底层实现基于连续的内存空间。当向```vector```中添加元素时，如果当前```vector```的内存空间已满，就需要重新分配一块更大的内存空间，并将原有元素复制到新的内存空间中，然后再将新元素添加到```vector```中。

```vector```的底层实现主要涉及以下几个部分：

数据存储：```vector```中的元素是按照一定的顺序存储在一块连续的内存空间中。```vector```的底层实现使用指针来维护该内存空间，指针指向该内存空间的首地址。

容量管理：```vector```内部维护了当前容量和已使用的空间大小，当需要添加元素时，如果当前已使用的空间大小等于容量，就需要重新分配一块更大的内存空间，并将原有元素复制到新的内存空间中，然后再将新元素添加到```vector```中。一般情况下，每次重新分配内存空间时，```vector```的容量会成倍增加，以减少频繁重新分配内存空间的开销。

元素访问：```vector```的元素可以使用下标操作符[]访问，也可以使用迭代器进行访问。

内存分配器：```vector```使用内存分配器来管理内存空间的分配和释放，内存分配器会根据具体的场景选择不同的策略，以提高内存分配效率和减少内存碎片。

内存管理：```vector```是动态数组，其底层实现基于连续的内存空间。当需要添加元素时，如果当前已分配的内存空间不足，```vector```会重新分配一块更大的内存空间，并将原有元素复制到新的内存空间中，然后再将新元素添加到```vector```中。因此，在大量添加元素的情况下，频繁的内存分配和拷贝会影响性能，可以通过预先分配足够的空间或使用```reserve()```来提高性能。

迭代器失效：当```vector```中的元素发生增删操作时，迭代器可能会失效，因为这些操作会导致```vector```的内存地址和元素顺序发生变化。因此，在使用迭代器时，需要避免在修改```vector```时使用过期的迭代器，可以在每次修改```vector```之后重新获取迭代器。

值类型限制：```vector```中存储的元素必须是具有复制构造函数和赋值运算符的类型，因为在```vector```的添加、删除、扩容等操作中会对元素进行拷贝或赋值操作。

插入和删除：```vector```提供了多种方法进行元素的插入和删除操作，如```push_back()```、```pop_back()```、```insert()```和```erase()```等。在使用这些操作时，需要注意其时间复杂度和效率，避免不必要的拷贝和移动操作。

内存泄漏：如果```vector```中存储的元素是指针类型，需要注意在删除```vector```时，要手动释放指针指向的内存空间，避免内存泄漏。

 ```Cpp{.line-numbers}
template < class T , class Alloc = alloc>
class vector {
public:
    typedef T value_type; 
    typedef value_type* iterator;
    typedef value_type& reference;
    typedef size_t size_type;
protected:
    iterator start;
    iterator finish;
    iterator end_of_storage;
public:
    iterator begin() { return start; }
    iterator end() { return finish; }
    size_type size() const {
        return size_type( end() - begin() );
            //使用了两次函数调用, 开销微不足道, 以后若vector内部
            //发生改变也无需重写size()函数
    }
    size_type capacity() const {
        return size_type( end_of_storage - begin() );
    }
    bool empty() const { return begin() == end(); }
    reference operator[] ( size_type n ) {
        return *( begin() + n );
    }
    reference front() { return *begin(); }
    reference back() { return *( end() - 1 ); }
};
 ```

 ```cpp
vector两倍增长后:
try{
	1.将原来的内容拷贝到新的vector中
	2.为新元素设置初值
	3.调整finish
	4.拷贝安插点后的原内容
}catch(.....){
	把新分配的删除, 重新分配再赋值等操作
}
删除原来的vector
调整迭代器指向新vector
 ```
## resize

```resize```是用于更改向量的大小。如果向量的当前大小比```resize```指定的大小小，则将在向量的末尾添加新元素，以使其大小达到指定大小。如果当前大小大于resize指定的大小，则会删除多余的元素，使其大小达到指定大小。如果```resize```指定的大小等于向量的当前大小，则没有任何操作。

当```resize```需要增加元素时，使用值初始化的默认值取决于元素类型。例如，如果vector中存储的是int类型，则新增的元素将被初始化为0；如果存储的是```bool```类型，则新增的元素将被初始化为```false```；如果存储的是```std::string```类型，则新增的元素将被初始化为空字符串。

 ```cpp
vector<int> v;
v.resize(5, 1); // 将向量大小设置为5，并用1初始化新增的元素
 ```
## reserve

```reserve(n)``` 函数用于为 ```vector``` 预留至少 n 个元素的内存空间，但不会改变 ```vector``` 的实际大小，也不会对 ```vector``` 中已有的元素进行初始化。在插入元素时，如果 ```vector``` 的实际大小超过了预留的内存空间，则仍然需要分配额外的内存空间。```reserve``` 函数并不保证预留的空间足够容纳所有的元素，因为内存分配可能失败。可以搭配```capacity```得到容量大小，从而确定要预留的空间。

## resize 和 reserve 对比

1. ```reserve``` 仅仅改变容器的容量，而不改变容器的大小；
	```resize``` 则改变容器的大小。
	 
2. ```reserve``` 为容器预留空间，但不初始化新元素，也不改变容器的大小；
	```resize``` 则不仅预留空间，还可以初始化新元素，或者截去多余的元素。

3. 在 ```reserve``` 调用之后，容器的大小仍然为 0，但是可以添加元素；
	在 ```resize``` 调用之后，容器的大小已经改变，可以访问新元素。

4. ```reserve``` 只改变容器的容量，对于新增元素操作，如果容量不足，仍需要重新分配内存；
	```resize``` 则根据需要增加或者缩小容器的容量，避免频繁的内存分配和释放。

5. 当向 ```std::vector``` 插入元素时，如果事先知道元素数量的范围，可以使用 ```reserve``` 提前分配足够的内存空间，从而避免插入元素时频繁的内存分配操作，提高性能；当需要动态地调整 ```std::vector``` 的大小时，可以使用 ```resize``` 改变容器的大小。

# bitset

避免使⽤```vector< bool >```,⽤```deque< bool >```和```bitset```代替,因为```vector```并不容纳```bool```类型
因为 ```vector<bool>``` 保存的是```bits```而不是```bool```,从而无法返回.

```bool```&```std::bitset``` 的所有成员函数都是 ```constexpr```：可以在常量表达式的计算中创建和使用```std::bitset``` 对象。

```cpp
//因为不是保存bool类型，test传入vector<bool>出错
template <class T>
void process(T& t) {
    // do something with t
}
template <class T, class A>
void test(std::vector<T, A>& v) {
    for (auto& t : v)
		//error:error: invalid initialization of non-const reference of 
		//type 'std::_Bit_reference&' from an rvalue of type 
		//'std::_Bit_iterator::reference {aka std::_Bit_reference}'
        process(t);
}
```
```cpp
template< std::size_t N> class bitset;
bitset<size> bt;
bt.all();	//检查是否所有位都是true
bt.any();	//检查是否有true的位
bt.none();	//检查是否没有为true的位
bt.count();	//返回bt中为true的位数
```
# list

```list``` 为双向链表
```cpp
 forward_list 为单向链表   //C++11
 #include<ext\slist> 下的 slist 跟 forward_list 一模一样
 ```
```slist``` 为 ```GNU C``` 中的，每放一个数据就开辟一个元素大小的空间

```forward_list``` 没有 ```push_pack()``` ,只有 ```push_front()```

刻意在环状 list 尾端加一个空白节点, 以符合STL“前闭后开”区间

在使用list时需要注意以下几点：

1. ```list```是一个双向链表，因此插入、删除元素的时间复杂度为O(1)，但是访问元素的时间复杂度为O(n)。

2. ```list```中的迭代器不支持随机访问，因此不能使用下标操作符[]，也不能使用算术运算符进行迭代器偏移。只能使用++、--等操作。

3. 在插入、删除元素时，要注意修改前后节点的指针，否则可能会出现指针错误。

4. ```list```的迭代器是不稳定的，如果在遍历```list```时对其中的元素进行删除或插入操作，可能会导致迭代器失效，因此应该避免在遍历时对```list```进行修改操作。

5. ```list```不支持随机访问，因此不能使用STL中的```sort```算法，需要使用自带的```sort```成员函数或者手动实现排序算法。

6. 如果需要在```list```中存储自定义类型的元素，需要重载元素类型的比较运算符。

7. ```list```提供了```merge```、```splice```、```remove```、```unique```等常用操作，可以大大简化对```list```的操作。

## merge()
 ```cpp
void merge (list& x);
void merge (list&& x);
template <class Compare>
void merge (list& x, Compare comp);
template <class Compare>
void merge (list&& x, Compare comp);
 ```
# deque

由一个个buffer构成，分段连续, 每次增长一个buffer大小的空间,buffer内连续, buffer分段不连续。

内部有一个```vector```, ```vector```内的元素为指向buffer的指针, 当```vector```需要扩充时,是扩充为两倍, 把原来的放到值copy到中段, 可以向前和向后使用```vector```空间

如果```push_back( a)```把所在buffer空间用完, 则新分配一个缓冲区, 把指向其的指针放到```vector```中, 往前扩充同理

```deque```的```iterator```是一个class, 里面四个元素:cur, first, last, node
1. first指向buffer头( 不是buffer内放的第一个元素)
2. last指向buffer尾( 不是buffer内放的最后一个元素)
3. cur指向buffer内的当前元素
4. node指向```vector```中当前buffer的地址


 ```cpp{.line-numbers}
tenmplate < class T, class Alloc = alloc, size_t BufSiz = 0>
			//BufSize是指每个buffer容纳的元素个数, 
			//如果传的不是0, 则表示buffer size由使用者自定
			//如果为0，则表示buffer size使用预设值
			//新版本已经不能指定了
	/*计算要使用的buffer size预设值
		inline size_t __deque_buf_size( size_t n, size_t sz){
				//sz为sizeof(value_type)
			return n!= 0 ? n : ( sz < 512 ? size_t( 512/sz): size_t(1));
		}
	*/
class deque{
public:
	typedef T value_type;
	typedef __deque_iterator < T, T&, BufSize> iterator;
		
protected:
	typedef pointer* map_pointer;	//T**
protected:
	iterator start;
	iterator finish;
	map_pointer map;
	size_type map_size;
	//deque大小为2个iterator大小加上2个指针大小
public:
	iterator begin() {	return start;	}
	iterator end() {	return finish;	}
	size_type size() const {	return finish - start;	}
.....
};
 ```
 ```cpp{highlight=3-7, .line-numbers}
template< class T, class Ref, class Ptr, size_t BufSize>
struct __deque_iterator {
	typedef bidirectional_iterator_tag iterator_category;	
	typedef _Tp value_type;	
	typedef _Tp* pointer;	
	typedef _Tp& reference;	
	typedef ptrdiff_t difference_type;	
	typedef size_t size_type;
	typedef T** map_pointer;
	typedef __deque_iterator self;

	T* cur;
	T* first;
	T* last;
	map_pointer node;
	//故一个iterator大小为4个指针的大小
	.....
}
 ```
 ```cpp
//在position处安插一个元素, 值为x
iterator insert( iterator position, const value_type& x){
	if( position.cur == start.cur ){	//如果插入点是deque的最前端, 
						//则直接交给push_front()做
		push_front(x);
		return start;
	} else if( position.cur == finish.cur){//如果插入点是deque的最末端, 
						//则直接交给push_back()做
		push_back(x);
		iterator tmp = finish;
		--tmp;
		return tmp;
	} else {	//在中间
		return insert_aux( position, x);
	}
}
template< class T, class Alloc, size_t BufSize>
typename deque< T, Alloc, BufSize>::iterator
deque< T, Alloc, BufSize>::iterator_aux( iterator pos, const value_type& x){
	difference_type index = pos - start;	//安插点前的元素个数
	value_type x_copy = x;
	if( index < size() / 2){	//如果安插点之前的元素个数较少
		push_front( front());		//如果在最前端加入与第一元素同值得元素
		.....
		copy( front2, pos1, front1);	//元素搬移得以空出一个位置给新值
	} else {		//安插点之后得元素个数较少
		push_back( back());		//在尾端加入与最后一个元素同值的元素
		.....
		copy_backward( pos, back2, back1);	//元素搬移得以空出一个位置给新值
	}
	*pos = x_copy;		//在安插点上设定新值
	return pos;
}
 ```
 ```cpp
reference operator[](size_type n){
	return start[ difference_type(n)];
}
reference front(){
	return *start;
}
regerence back(){
	iterator temp = finish;
	--tmp;	//因为finish指向最后一个元素的下一个位置
	return *tmp;
}
size_type size() const{
	return finish-start;	//-做了重载
}
bool empty() const{
	return finish == start;
}
 ```

# stack / queue

内部使用```deque```实现, 不提供```iterator```, 不能遍历```queue```不能用```vector```做底层结构, ```stack```可以用```vector```做底层结构, 因为```vector```不能return front

# rb_tree

rb_tree的遍历是中序遍历, 不应该使用```iterator```改变元素的值(但没有阻止更改),rb_tree为```map```和```set```服务, ```map```可以改变data, key不可改

rb_tree提供两种insertion操作: ```insert_unique()```和```insert_equal()```前者表示key独一无二, 否则插入失败，后者表示key可以重复

 ```cpp
template< class Key,	//关键字
	class Value,	//关键字和data的组合, 而非data
	class KeyOfValue,	//value中的关键字怎么拿出来
	class Compare,	//关键字的比较方式
	class Alloc = alloc>	//分配器
class rb_tree {
protected:
	typedef __rb_tree_node<Value> rb_tree_node;
	.....
public:
	typedef rb_tree_node* link_type;
	.....
protected:
	//红黑树只用以下三个参数表现他自己
	size_type node_count;	//记录红黑树节点个数
	link_type header;
	Compare key_compare;	//key的大小比较准则, 可能是个function object
	.....
};
 ```
 ```cpp
//使用实例：
rb_tree< int,	//key的类型
	int,	//value的类型, 此时代表只有key没有data
	identity<int>,	//如何取得key, 因为此时value中只有key, 所以直接返回key
	less<int>,	//key的比较方式
	alloc>	
myTree;

template < class Arg, class Result>
struct unary_function{
	typedef Arg argument_type;
	typedef Result result_type;
};
template< class T>
struct identity: public unary_function< T, T> {
	const T& operator() ( const T& x) const {
		return x;
	}
};
template < class Arg1, class Arg2, class Result>
struct binary_function{
	typedef Arg1 first_argument_type;
	typedef Arg2 second_argument_type;
	typedef Result result_type;
};
template <class T>
struct less: public binary_function< T, T, bool>{
	bool operator()( const T&x, const T& y) const{
		return x < y;
	}
};
 ```
# map/set	
```cpp
unordered_set<string> wordSet( wordDict.begin(), wordDict.end());
	//wordDict是容器，上述语句把wordDict中所有元素放入wordSet中
 ```
## set/multiset

rb-tree为底层, 元素有自动排序的特性, 排序的依据是key, value和key合而为一, key就是value, 无法使用其```iterators```改变元素值( 因为key有其遵循的排列规则)，其iterator是底部rb-tree的```const iterator```, 就是为了禁止user对元素的赋值

```set```元素的key必须独一无二, 因此其```insert()```用的是rb_tree的```insert_unique()```
```multiset```元素的key可以重复, 因此其```insert()```用的是rb_tree的```insert_equal()```

## map/multimap

把key变成const key, 和```set```差不多, 只不过value中有data

```map```重载了[]操作符, 可以通过[key]来更改data的值( 使用```lower_bound(key)```来查找key的位置)

```lower_bound(key)```: 在已排序的[first, last)中找到key, 如果有key则返回找到的第一个值为key的元素位置, 如果没有则返回第一个不小于key的元素, 如果key大于所有元素, 则返回last 
```lower_bound(key)```返回key的位置或者最适合安插key的位置

使用[]做插入比直接使用insert()插入更慢

## unordered容器

```unordered_map```、```unordered_set```、```unordered_multimap```、```unordered_multiset```底层用```hashtable```实现

 ```cpp
template <typename T, typename Hash = hash<T>,
	typename EqPred = equal_to<T>, typename Allocator = allocator<T>>
class unordered_set;
 ```
 ```cpp
template <typename T, typename Hash = hash<T>,
	typename EqPred = equal_to<T>, typename Allocator = allocator<T>>
class unordered_multiset;
 ```
 ```cpp
template <typename Key, typename T, typename Hash = hash<T>,
	typename EqPred = equal_to<T>,
	typename Allocator = allocator<pair<const Key, T> > >
class unordered_map;
 ```
 ```cpp
template <typename Key, typename T, typename Hash = hash<T>,
	typename EqPred = equal_to<T>,
	typename Allocator = allocator<pair<const Key, T> > >
class unordered_multimap;
 ```
# hashtable

可以使用```hashtable iterators```改变元素的data而不能改变key, ```hashtable```使用key排序```hashtable```使用```bucket vector```, 如果元素个数比bucket多时, 把bucket数量增加两倍后附近的素数作为新的bucket数量

 ```cpp
template< class Value, class Key, class HashFcn, class ExtractKey,
	class EqualKey, class Alloc = alloc>
	//EqualKey为比较方式(可为仿函数)
class hashtable{
public:
	typedef HashFcn hasher;
	typedef EqualKey key_equal;
	typedef size_t size_type;
private:
	hasher hash;
	key_equal equals;
	ExtractKey get_key;
	typedef __hashtable_node<Value> node;
	vector< node*, Alloc> buckets;
	size_type num_elements;
public:
	size_type bucket_count() const {
		return buckets.size();
	}
.....
};
 ```
 ```cpp
template< class Value, class Key, class HashFcn, class ExtractKey, class EqualKey,
	class Alloc>
struct __hashtable_iterator{
	.....
	node* cur;
	hashtable* ht;
};
 ```
 ```cpp
//实例
struct eqstr{
	bool operator()( const char* s1, const char* s2) const{
		return strcmp( s1, s2) == 0;	
	}
};
//比较两个c-string是否相等可以用strcmp(), 但是它传回-1,0,1,不是传回bool，
//所以必须要加一个外壳
hashtable<const char*, const char*, hash<const char*>,
	identity<const char*>,
	eqstr, akkic> ht( 50, hash<const char*>(), eqstr());

 ```
# BC++、VC++和GCC的allocator

只用```operator new```和```operator delete```完成```allocate()```和```deallocate()```, 没有特殊设计 

```GCC2.9```的```alloc```使用了较少的cookie, 使用方法:
```cpp
vector< Type, __gnu_cxx::__pool_alloc<Type>>
 ```

# Algorithm

Algorithm看不见Containers, 对其一无所知; 它所需要的一切信息都要通过```iterator```获得,而```iterator```(由Containers提供)必须能够回答Algorithm的所有提问, 才能搭配该Algorithm的所有操作。 

## find_if
用于在给定范围内查找第一个满足指定条件的元素，返回迭代器指向该元素。
```cpp
template<class InputIt, class UnaryPredicate>
InputIt find_if(InputIt first, InputIt last, UnaryPredicate p);
```
InputIt 是输入迭代器的类型，表示要查找的范围，UnaryPredicate 是一元函数对象类型，描述要满足的条件。函数对象的返回值为 bool 类型。
每次调用 p 函数对象，它会被传入当前迭代器指向的元素，返回值为 true 表示该元素满足条件，返回值为 false 表示该元素不满足条件。find_if 将从 first 开始逐个迭代器调用 p，直到找到第一个满足条件的元素，或者到达 last 端点。

```cpp
//使用 find_if 在 vector 中查找第一个偶数，并打印其值
std::vector<int> nums = {1, 3, 2, 5, 4, 7, 6};
auto it = std::find_if(nums.begin(), nums.end(), [](int x){ 
	return x % 2 == 0; 
});
if (it != nums.end()) {
	std::cout << "First even number found is: " << *it << std::endl;
		//First even number found is: 2
} else {
	std::cout << "No even number found!" << std::endl;
}
```
## lower_bound
用于在有序序列中查找第一个**大于等于**给定值的元素的迭代器。 如果找到了该元素，它将返回该元素的迭代器；如果未找到，它将返回序列的末尾迭代器。
```cpp
template <class ForwardIterator, class T>
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last, const T& val);
```
first 和 last 分别是指向有序序列起始位置和终止位置的迭代器，val 是要查找的值。
时间复杂度为 O(log n)。

```cpp
//有序数组 arr，并且想要查找第一个大于等于给定值 x 的元素在数组中的下标
int idx = lower_bound(arr.begin(), arr.end(), x) - arr.begin();
```

## upper_bound
用于在有序序列中查找第一个**大于**给定值的元素的迭代器。如果找到了该元素，它将返回该元素的迭代器；如果未找到，它将返回序列的末尾迭代器。
```cpp
template <class ForwardIterator, class T>
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last, const T& val);
```
first 和 last 分别是指向有序序列起始位置和终止位置的迭代器，val 是要查找的值。
时间复杂度为 O(log n)。
```cpp
//有序数组 arr，并且想要查找第一个大于给定值 x 的元素在数组中的下标
int idx = upper_bound(arr.begin(), arr.end(), x) - arr.begin();
```
# less<T>
```less<T>``` 是 STL 中的一个函数对象，通常用于比较两个元素是否满足某种排序关系。
要确保```less<T>```和```operator<```具有相同的语义
 ```cpp
#include <iostream>
#include <algorithm>
#include <functional>	//less<T>
using namespace std;
int main() {
    int arr[5] = {3, 1, 4, 2, 5};
    sort(arr, arr + 5, less<int>());
    for (int i = 0; i < 5; i++) {
        cout << arr[i] << " ";	
    }	//1 2 3 4 5
    return 0;
}
```
```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <functional>

using namespace std;

struct Person {
	string name;
	int age;
};
bool compareByAge(const Person& p1, const Person& p2) {
	return p1.age < p2.age;
}
int main() {
	Person p1 = {"Bob", 30};
	Person p2 = {"Alice", 20};
	Person p3 = {"John", 25};
	Person people[] = {p1, p2, p3};

	sort(people, people + 3, less<>()(compareByAge));

	for (int i = 0; i < 3; i++) {
		cout << people[i].name << " " << people[i].age << endl;
	}
	//输出：
	//Alice 20
	//John 25
	//Bob 30

	return 0;
}
```
使用 ```less<>()(compareByAge)``` 调用 ```sort()``` 时，```less<>()``` 的作用是表示按照第一个参数小于第二个参数的方式来排序，而实际的比较方式是根据我们自定义的 ```compareByAge函数``` 来进行比较的。

# Iterator
 ```cpp
vector<int> iterator :: vi;
 ```
 ```Cpp{.line-numbers,highlight=21-25}		
	struct _List_node_base{
		_List_node_base* _M_next;
		_List_node_base* _M_prev;
	};
	template< typename _Tp>
	struct _List_node : public _List_node_base{
		_Tp _M_data;
	};
	template< typename _Tp, typename _Alloc = std::allocator<_Tp>>
	class list : protected _List_base< _Tp, _Alloc>{
	.....
	public:
		typedef __list_iterator< _Tp> iterator;
	.....
	};
	//所有容器的iterator都要有下面5个高亮typedef
	template< typename _Tp>
	struct __list_iterator {
		typedef __list_iterator< typename _Tp> self;
		typedef __list_node<_Tp>* link_type;
		typedef bidirectional_iterator_tag iterator_category;	
		typedef _Tp value_type;	
		typedef _Tp* pointer;	
		typedef _Tp& reference;	
		typedef ptrdiff_t difference_type;	
	.....
	}; 
 ```	
## Iterator需要遵循的原则:
### iterator是泛化的指针	

```iterator Traits```必须有能力分辨```class iterators```和```non-class iterators```
利用partial specialization可达到目标(不同的type有不同的traits)

 ```Cpp{.line-numbers}
template<class I>
struct iterator_traits {	//I是class iterator时
    typedef typename I::value_type value_type;	
};

//两个partial specialization:
template<class T>
struct iterator_traits<T*> {	//I是pointer to T时
    typedef T value_type;
};
template <class T>
struct iterator_traits<const T*>{	//I是pointer to const T时
    typedef T value_type;		//注意是T而不是const T
    //value_type的主要目的是用来声明变量, 声明一个无法被赋值的
    //变量没有什么用, 所以iterator的value type不加上const
};

template<typename I, ...>
void algorithm(...){
    typename iterator_traits<I>::value_type v1;
}
 ```
### 算法提出问题, iterator需要回答问题, 标准库有5种问题(4,5没出现过):
	1.iterator_category		2.difference_type 	3.value_type
	4.reference_type		5.pointer_type
#### 1.iterator_category(分类):
指的是```iterator```的移动性质, 有的只能++, 有的只能--, 等等
#### 2.difference_type(距离):
指的是两个```iterator```之间距离的类型
#### 3.value_type
指的是变量的类型

### 各种容器的iterator的iterator_category
 ```cpp
//5种iterator category

struct input_iterator_tag{};
//基于顺序操作的iterator，iterator指向的每一个值都只读一次，然后iterator递增
//只能向前递增,此iterator只在我们想要访问元素时使用，不能给元素赋值，不能递减
//只能用==不能用<,>等关系运算，也不能用+-运算
struct output_iterator_tag{};
//不能访问值, 只能赋值
struct forward_iterator_tag:public input_iterator_tag{};
//只能走一个方向，单向的
struct bidirectional_iterator_tag:public forward_iterator_tag{};
//双向的，但是不能跳跃
struct random_access_iterator_tag: public bidirectional_iterator_tag{};
//随机可跳跃
 ```

# 杂记

除了```array```和```vector```以外, 所有容器的```iterator```都是class  
所有容器内的元素都是前闭后开区间内
不同容器要用不同的删除方法：
1.删除容器中有特定值的所有对象
