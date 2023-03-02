# array
 ```cpp
就是数组, 包装后让其可以享受算法等部件的交互array没有ctor、dtor
array的iterator就是一个指针
 ```

# vector
 ```cpp
增长为二倍增长, capacity函数可以看到目前的容量大小

vector内部只有三个protected的指针: start, finish, end_of_storage

sizeof一个vector大小为3个指针的大小

finish-start为size, end_of_storage-start为capacity

使用vector时会大量调用构造函数, 复制构造函数, 析构函数, 是很大的开销
 ```
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

# bitset
```cpp
避免使⽤vector< bool >,⽤deque< bool >和bitset代替,因为vector并不容纳bool类型
因为 vector<bool> 保存的是bits而不是bool,从而无法返回bool&
std::bitset 的所有成员函数都是 constexpr：可以在常量表达式的计算中创建和使用 
std::bitset 对象。
```
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
 ```cpp
list 为双向链表
forward_list 为单向链表   //C++11

 #include<ext\slist> 下的 slist 跟 forward_list 一模一样

slist 为 GNU C 中的每放一个数据就开辟一个元素大小的空间

forward_list 没有 push_pack() ,只有 push_front()

刻意在环状 list 尾端加一个空白节点, 以符合STL“前闭后开”区间
 ```
# deque
 ```cpp
由一个个buffer构成，分段连续, 每次增长一个buffer大小的空间,
buffer内连续, buffer分段不连续

内部有一个vector, vector内的元素为指向buffer的指针, 当vector需要扩充时,
是扩充为两倍, 把原来的放到值copy到中段, 可以向前和向后使用vector空间

如果push_back( a)把所在buffer空间用完, 则新分配一个缓冲区, 把指向其的
指针放到vector中, 往前扩充同理

deque的iterator是一个class, 里面四个元素:cur, first, last, node
first指向buffer头( 不是buffer内放的第一个元素)
last指向buffer尾( 不是buffer内放的最后一个元素)
cur指向buffer内的当前元素
node指向vector中当前buffer的地址
 ```

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
```cpp
内部使用deque实现, 不提供iterator, 不能遍历
queue不能用vector做底层结构, stack可以用vector做底层结构, 因为vector不能return front
```
# rb_tree
```cpp
rb_tree的遍历是中序遍历, 不应该使用iterator改变元素的值(但没有阻止更改),
rb_tree为map和set服务, map可以改变data, key不可改

rb_tree提供两种insertion操作: insert_unique()和insert_equal()
前者表示key独一无二, 否则插入失败，后者表示key可以重复
 ```
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
 ```cpp
rb-tree为底层, 元素有自动排序的特性, 排序的依据是key, value和key合而为一, 
key就是value, 无法使用其iterators改变元素值( 因为key有其遵循的排列规则)
其iterator是底部rb-tree的const iterator, 就是为了禁止user对元素的赋值

set元素的key必须独一无二, 因此其insert()用的是rb_tree的insert_unique()
multiset元素的key可以重复, 因此其insert()用的是rb_tree的insert_equal()
 ```
## map/multimap
 ```cpp
把key变成const key, 和set差不多, 只不过value中有data
map重载了[]操作符, 可以通过[key]来更改data的值( 使用lower_bound(key)来查
找key的位置)

lower_bound(key): 在已排序的[first, last)中找到key, 如果有key则返回找到的
第一个值为key的元素位置, 如果没有则返回第一个不小于key的元素, 如果key大于所
有元素, 则返回last
lower_bound(key)返回key的位置或者最适合安插key的位置

使用[]做插入比直接使用insert()插入更慢
 ```
## unordered容器
 ```cpp
unordered_map、unordered_set、unordered_multimap、unordered_multiset底层用hashtable实现
 ```
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
 ```cpp
可以使用hashtable iterators改变元素的data而不能改变key, hashtable使用key排序
hashtable使用bucket vector, 如果元素个数比bucket多时, 把bucket数量增加两倍后
附近的素数作为新的bucket数量
 ```
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
 ```cpp
只用operator new和operator delete完成allocate()和deallocate(), 
没有特殊设计 

GCC2.9的alloc使用了较少的cookie, 使用方法:
vector< Type, __gnu_cxx::__pool_alloc<Type>>
 ```	
# Algorithm
 ```cpp
Algorithm看不见Containers, 对其一无所知; 它所需要的一切信息都要通过Iterator
获得,而Iterator(由Containers提供)必须能够回答Algorithm的所有提问, 才能搭配该
Algorithm的所有操作。 
 ```
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
 ```cpp
Iterator Traits必须有能力分辨class iterators和non-class iterators
利用partial specialization可达到目标(不同的type有不同的traits)
 ```
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
    指的是Iterator的移动性质, 有的只能++, 有的只能--, 等等
#### 2.difference_type(距离):
    指的是两个iterator之间距离的类型
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
 ```cpp
除了array和vector以外, 所有容器的iterator都是class  
所有容器内的元素都是前闭后开区间内
 ``` 
```cpp
不同容器要用不同的删除方法：
1.删除容器中有特定值的所有对象
 ```