智能指针的设计理念：帮助我们解决 C/C++ 中指针带来的内存问题

1. 指针没有初始化

2. new 了之后没有及时 delete

3. delete 了之后没有及时至空

   野指针：void *ptr = new A，A.delete() 这个时候没有将 ptr 置空，造成野指针

<br/>

**`RefBase / LightRefBase` 底层支持**：基类，提供 `引用计数实现` 和 `销毁逻辑`

- 二者是对象需要继承的基类，提供引用计数的核心实现
- RefBase 支持 sp 和 wp，而 LightRefBase 只支持 sp

**`sp / wp` 上层工具**：智能指针，提供 `强引用` 和 `弱引用` 2 个概念，以智能化的方式管理指针的生命周期

- sp 和 wp 是智能指针，底层依赖 RefBase 或 LightRefBase 的实现来管理对象
- sp 调用 incStrong/decStrong，wp 调用 incWeak/decWeak

```c
+-----------------+
|    对象基类     |
|  (RefBase 或    |
|  LightRefBase)  |
+-----------------+
        |         \
        |          \ (只支持 sp)
        |           \
   (支持 sp 和 wp)   \
        |             \
+----------------+   +----------------+
|   sp<T>        |   |   wp<T>        |
| (强指针)       |   | (弱指针)       |
| incStrong      |   | incWeak        |
| decStrong      |   | decWeak        |
+----------------+   +----------------+
```

<br/>

# 引用计数基类

## LightRefBase

**`LightRefBase`**：通过引用计数来自动管理对象的生命周期

- **强引用计数**：

  LightRefBase 内部维护一个 `mCount` 变量，表示对象的强引用计数

  内部由 `incStrong() / decStrong()` 控制引用计数

  当引用计数变为 0 时，对象会被自动删除

- **仅支持 sp 使用**：

  LightRefBase 通常配合 sp（强指针模板类）使用，sp 负责增加或减少引用计数

  当 sp 被赋值或销毁时，会自动调用 LightRefBase 的 incStrong 或 decStrong 方法

- **简单高效**：

  <u>仅支持强引用 sp</u>

  它不像 RefBase 那样支持弱指针（sp/wp），因此更加轻量，适用于一些简单的对象管理需求

  由于不支持弱指针，LightRefBase 无法解决对象间的 "循环引用" 问题

  它牺牲了弱指针功能，换取了更低的开销和更高的性能，适合简单的对象管理场景

> 存在于 libutils 库中：system/core/include/utils/RefBase.h

```c++
template <typename T>
class LightRefBase {
public:
    inline LightRefBase() : mCount(0) { }

    inline void incStrong(const void* id) const {
        __sync_fetch_and_add(&mCount, 1); 							// 原子操作增加引用计数
    }

    inline void decStrong(const void* id) const {
        if (__sync_fetch_and_sub(&mCount, 1) == 1) { 		// 原子操作减少引用计数
            delete static_cast<const T*>(this); 				// 计数为 0 时删除对象
        }
    }

    inline int32_t getStrongCount() const {
        return mCount;
    }

protected:
    inline ~LightRefBase() { } 													// 受保护的析构函数，避免直接删除
																												// 防止直接通过基类指针删除对象，确保删除逻辑由 decStrong 控制
private:
    mutable volatile int32_t mCount; 										// 引用计数变量
};
```

<br/>

### 案例

以下是一个简单的使用 LightRefBase 和 sp 的例子

```c
#include <utils/RefBase.h>
#include <utils/StrongPointer.h>
#include <stdio.h>

class MyClass : public LightRefBase<MyClass> {
public:
    MyClass() { printf("MyClass created\n"); }
    ~MyClass() { printf("MyClass destroyed\n"); }
    void doSomething() { printf("Doing something\n"); }
};

int main() {
    sp<MyClass> ptr = new MyClass(); // 创建对象，引用计数为 1
    ptr->doSomething();              // 使用对象
    // 当 ptr 超出作用域时，引用计数减为 0，对象被自动删除
    return 0;
}
```

<br/>

## RefBase

**`RefBase`** ：通过引用计数来自动管理对象的生命周期

- **定义**：

  RefBase 相比 LightRefBase 增加了一个 `weakref_type` 类型的弱引用计数器，增加了弱引用控制

  可以配合指针指针 sp/wp 一起使用

- **强引用计数**：

  通过 `sp` 管理，表示对对象的实际拥有权（拥有对象）

  当强引用计数降为 0 时，对象会被销毁

- **弱引用计数**：

  通过 `wp` 管理，允许在不拥有对象的情况下引用它（不拥有对象）

  <u>当强引用计数为 0 时，即使弱引用仍然存在，对象也会被销毁，弱指针会失效</u>

- **循环引用解决**：

  弱指针机制可以打破强指针之间的循环引用，避免内存泄漏

- **`weakref_type`**

  由 weakref_type 提供引用计数（`mStrong/mWeak`）

  由 `incStrong() / decStrong()` 控制强引用计数（同时增加 mStrong/mWeak）

  由 `weakref_type.incWeak() / decWeak()` 控制弱引用计数（只增加 mWeak）

> 存在于 libutils 库中：system/core/include/utils/RefBase.h

```c
class RefBase
{
public:
  // 强引用
  void            incStrong(const void* id) const;
  void            decStrong(const void* id) const;

  void            forceIncStrong(const void* id) const;
  int32_t         getStrongCount() const;

  // 弱引用
  class weakref_type
  {
    public:
        RefBase* refBase() const;
        void incWeak(const void* id);
        void decWeak(const void* id);
        bool attemptIncStrong(const void* id);
        // ...
  };
  weakref_type*   createWeak(const void* id) const;
  weakref_type*   getWeakRefs() const;

  inline  void            printRefs() const { getWeakRefs()->printRefs(); }
  inline  void            trackMe(bool enable, bool retain)
  {
    getWeakRefs()->trackMe(enable, retain);
  }
  typedef RefBase basetype;
  
protected:
  RefBase();
  virtual                 ~RefBase();
  
  // LIFETIME标志位
  enum {
    OBJECT_LIFETIME_STRONG  = 0x0000,			// 默认值
    OBJECT_LIFETIME_WEAK    = 0x0001,
    OBJECT_LIFETIME_MASK    = 0x0001
  };
  void            extendObjectLifetime(int32_t mode);
  enum {
    FIRST_INC_STRONG = 0x0001
  };

  virtual void            onFirstRef();																						// 第一次强引用时的回调
  virtual void            onLastStrongRef(const void* id);												// 最后一个强引用移除时的回调
  virtual bool            onIncStrongAttempted(uint32_t flags, const void* id);		// 尝试恢复强引用时的回调
  virtual void            onLastWeakRef(const void* id);													// 最后一个弱引用移除时的回调

private:
  friend class ReferenceMover;
  static void moveReferences(void* d, void const* s, size_t n,
                             const ReferenceConverterBase& caster);

private:
  friend class weakref_type;
  class weakref_impl;
  RefBase(const RefBase& o);
  RefBase&        operator=(const RefBase& o);

  // 弱引用实现对象
  weakref_impl* const mRefs;															// 弱引用计数器: 提供 mStrong、mWeak 计数
};
```

<br/>

### 主要函数

**`createWeak()`**：返回弱引用计数器，并对弱引用 +1

```c
RefBase::weakref_type* RefBase::createWeak(const void* id) const
{
    mRefs->incWeak(id);					// 增加弱引用
    return mRefs;
}
```

**`incStrong()`**：同时增加强引用、强引用的计数值

- 当第一次 incStrong 时，对 mStrong 的值进行调整

```c
void RefBase::incStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    refs->incWeak(id);																														// mWeak++
    
    refs->addStrongRef(id);							// 调试相关debug
    const int32_t c = refs->mStrong.fetch_add(1, std::memory_order_relaxed);			// mStrong++
  
	  // 判断是不是第一次，对mStrong对值进行调整
  	if (c != INITIAL_STRONG_VALUE)  {		
        return;
    }
  	// 如果是第一次，需要减去 INITIAL_STRONG_VALUE（第一次对值是 INITIAL_STRONG_VALUE+1）
    int32_t old __unused = refs->mStrong.fetch_sub(INITIAL_STRONG_VALUE, std::memory_order_relaxed);
    refs->mBase->onFirstRef();
}
```

**`decStrong()`**：同时增加强引用、强引用的计数值

- 当强引用数值=0时，进行 delete

```c
void RefBase::decStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    refs->removeStrongRef(id);																									// 调试用
    const int32_t c = refs->mStrong.fetch_sub(1, std::memory_order_release);		// mStrong--
    if (c == 1) {
        std::atomic_thread_fence(std::memory_order_acquire);
        refs->mBase->onLastStrongRef(id);																				// 如果是最后一个，回调onLastStrongRef
        int32_t flags = refs->mFlags.load(std::memory_order_relaxed);
        if ((flags&OBJECT_LIFETIME_MASK) == OBJECT_LIFETIME_STRONG) {						// OBJECT_LIFETIME_STRONG 标志位
            delete this;																												// 删除对象
        }
    }
    refs->decWeak(id);																													// mWeak--
}
```

<br/>

### weakref_type 弱引用计数器

**`weakref_type`** 是一个引用计数器

- `incWeak()`
- `decWeak()`
- `promote()`：不拥有对象，仅观察，需通过 promote() 提升为 sp 使用

weakref_type

- 对弱引用计数值的计算是通过上面 2 个方法进行
- 对强引用的计算是通过 RefBase 的 incStrong() / decStrong() 实现的（会同时增加 mStrong 和 mWeak 的值）

```c
class weakref_type
{
public:
  RefBase*            refBase() const;

  void                incWeak(const void* id);								// 增加弱引用计数
  void                decWeak(const void* id);								// 减少弱引用计数

  bool                attemptIncStrong(const void* id);
  bool                attemptIncWeak(const void* id);
	void                printRefs() const;
  void                trackMe(bool enable, bool retain);
  int32_t             getWeakCount() const;
};
```

<br/>

### weakref_impl 实现类

**`weakref_impl`** 是 weakref_type 的实现类

- `mStrong`
- `mWeak`

```c
#define INITIAL_STRONG_VALUE (1<<28)

class RefBase::weakref_impl : public RefBase::weakref_type
{
public:
    std::atomic<int32_t>    mStrong;					// 强引用计数
    std::atomic<int32_t>    mWeak;						// 弱引用计数
    RefBase* const          mBase;
    std::atomic<int32_t>    mFlags;

#if !DEBUG_REFS			// 非Debug模式

    explicit weakref_impl(RefBase* base)
        : mStrong(INITIAL_STRONG_VALUE)			// mStrong
        , mWeak(0)													// mWeak
        , mBase(base)
        , mFlags(0)
    {
    }

    void addStrongRef(const void* /*id*/) { }
    void removeStrongRef(const void* /*id*/) { }
    void renameStrongRefId(const void* /*old_id*/, const void* /*new_id*/) { }
    void addWeakRef(const void* /*id*/) { }
    void removeWeakRef(const void* /*id*/) { }
    void renameWeakRefId(const void* /*old_id*/, const void* /*new_id*/) { }
    void printRefs() const { }
    void trackMe(bool, bool) { }

#else
  ...
#endif
}
```

**`incWeak()`**

```c
void RefBase::weakref_type::incWeak(const void* id)
{
    weakref_impl* const impl = static_cast<weakref_impl*>(this);
    impl->addWeakRef(id);																		// 调试debug用的
    const int32_t c __unused = impl->mWeak.fetch_add(1,			// 增加弱引用引用计数 mWeak
            std::memory_order_relaxed);
    ALOG_ASSERT(c >= 0, "incWeak called on %p after last weak ref", this);
}
```

**`decWeak()`**

- 当弱引用计数值减小到 0 的时候，还需要 `根据 LIFETIME 标志位和强引用计数值大小` 进行不同的处理
- 当目标对象是 OBJECT_LIFETIME_WEAK 类型时，直接进行 delete
- 当目标对象是 OBJECT_LIFETIME_STRONG 类型时，根据 mStrong 的值判断：如果 mStrong 是初始值（未被引用），直接 delete；否则不处理，而是交给 decStrong 进行处理（被引用）

```c
enum {
  OBJECT_LIFETIME_STRONG  = 0x0000,				// 默认值
  OBJECT_LIFETIME_WEAK    = 0x0001,
  OBJECT_LIFETIME_MASK    = 0x0001
};

void RefBase::weakref_type::decWeak(const void* id)
{
  weakref_impl* const impl = static_cast<weakref_impl*>(this);
  impl->removeWeakRef(id);			// 调试
  const int32_t c = impl->mWeak.fetch_sub(1, std::memory_order_release);			// 减小弱引用计数值mWeak

  // 弱引用计数值为0，根据LIFETIME标志位分别进行不同的处理
  if (c != 1) return;
  atomic_thread_fence(std::memory_order_acquire);

  int32_t flags = impl->mFlags.load(std::memory_order_relaxed);
  if ((flags&OBJECT_LIFETIME_MASK) == OBJECT_LIFETIME_STRONG) {					// OBJECT_LIFETIME_STRONG
    if (impl->mStrong.load(std::memory_order_relaxed)
        == INITIAL_STRONG_VALUE) {
      ALOGW("RefBase: Object at %p lost last weak reference "								
            "before it had a strong reference", impl->mBase);									// 强引用不为初始值，不处理（交由decStrong处理）
    } else {
      delete impl;																														// 强引用为初始值，进行delete
    }
  } else {																															// OBJECT_LIFETIME_WEAK
    impl->mBase->onLastWeakRef(id);
    delete impl->mBase;																												// 弱引用计数值为0，进行delete
  }
}
```

<br/>

### 案例

以下是一个使用 RefBase、sp 和 wp 的例子

```c
#include <utils/RefBase.h>
#include <utils/StrongPointer.h>
#include <stdio.h>

// RefBase
class MyClass : public RefBase {
public:
    MyClass() { printf("MyClass created\n"); }
    ~MyClass() { printf("MyClass destroyed\n"); }
    void doSomething() { printf("Doing something\n"); }
};

int main() {
    sp<MyClass> strongPtr = new MyClass(); // sp：强引用计数为 1
    wp<MyClass> weakPtr = strongPtr;       // wp：弱引用指向同一对象

    strongPtr->doSomething();              // 输出 "Doing something"

    // 检查弱指针是否有效
    sp<MyClass> temp = weakPtr.promote();  // 尝试提升为强指针
    if (temp != nullptr) {
        printf("Weak pointer is valid\n");
    }

    strongPtr = nullptr;                   // sp 销毁，强引用计数为 0，对象销毁
    temp = weakPtr.promote();              // 弱指针失效
    if (temp == nullptr) {
        printf("Weak pointer is invalid\n");
    }

    return 0;
}
```

输出

```c
MyClass created
Doing something
Weak pointer is valid
MyClass destroyed
Weak pointer is invalid
```

<br/>

---

<br/>

# 智能指针

## StrongPointer 强指针 sp

**`sp`**：强指针类型的智能指针，表示对对象的拥有权

- **引用计数基类**：

  必须搭配引用计数基类使用，sp 只对继承自 RefBase 或 LightRefBase 的类有效

  `m_ptr` 指向实际的目标对象 `LightRefBase / RefBase`

- **避免裸指针混用**：

  不要将同一裸指针多次交给 sp，否则会导致重复释放问题

- **循环引用风险**：

  若两个对象通过 sp 相互引用，会导致内存泄漏，需用 wp 解决

- **性能**：

  比裸指针多引用计数开销，但远小于手动管理的复杂性

引用规则

- 通过 `构造函数、=操作符`，增加强引用计数值
- 通过 `析构函数`，减小强引用计数值

> 存在于 libutils 包中：system/core/include/utils/StrongPointer.h

```c
template<typename T>
class sp {
public:
  	// 构造函数
    inline sp() : m_ptr(0) { }

    sp(T* other);												// 从裸指针构造
    sp(const sp<T>& other);							// 拷贝构造函数
    sp(sp<T>&& other);
    template<typename U> sp(U* other); 
    template<typename U> sp(const sp<U>& other);
    template<typename U> sp(sp<U>&& other); 

    ~sp();

    // 操作符重载

    sp& operator = (T* other);
    sp& operator = (const sp<T>& other);
    sp& operator = (sp<T>&& other);
  
  	// 支持 * 和 ->，方便像裸指针一样使用
  	inline T&       operator* () const     { return *m_ptr; }						// *
    inline T*       operator-> () const    { return m_ptr;  }						// ->
    inline T*       get() const            { return m_ptr; }
    inline explicit operator bool () const { return m_ptr != nullptr; }

    template<typename U> sp& operator = (const sp<U>& other);
    template<typename U> sp& operator = (sp<U>&& other);
    template<typename U> sp& operator = (U* other);

    void force_set(T* other);
    void clear();

    // Operators

    COMPARE(==)
    COMPARE(!=)
    COMPARE(>)
    COMPARE(<)
    COMPARE(<=)
    COMPARE(>=)

private:    
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;
    void set_pointer(T* ptr);
    T* m_ptr;																				// 目标对象：实际存储的指针
};
```

<br/>

### 主要函数

**`构造函数和析构函数`**：对 m_ptr 引用计数的目标对象进行了处理

```c
template<typename T>
sp<T>::sp(T* other) : m_ptr(other) {
    if (other)
        other->incStrong(this);				// 增加引用计数
}

template<typename T>
sp<T>::~sp() {
    if (m_ptr)
        m_ptr->decStrong(this);				// 减少引用计数
}
```

**`重写 = 操作符`**：这样当我们调用 sp\<Object> sp = new Object 的时候，就会将该 Object 赋值给 m_ptr

```c
template<typename T>
sp<T>& sp<T>::operator =(T* other) {
    T* oldPtr(*const_cast<T* volatile*>(&m_ptr));
    if (other) other->incStrong(this);								// 增加引用计数（新）
    if (oldPtr) oldPtr->decStrong(this);							// 减少引用计数（旧）
    if (oldPtr != *const_cast<T* volatile*>(&m_ptr)) sp_report_race();
    m_ptr = other;																		// 重新赋值
    return *this;
}
```

<br/>

## WeakPointer 弱指针 wp

**`wp`** 弱指针类型的智能指针

- 强指针在存在 `循环引用` 的时候，就会出现类似死锁的情况

- `promote()`：

  弱指针不拥有对象，仅观察，需通过 promote() 提升为 sp 使用

  没有 ->、* 等操作符，取而代之的是 promote()将 wp 提升为 sp

弱引用规则

- CDad 使用强引用引用 CChild，CChild 使用弱引用引用父类 CDad
- 当强引用计数为 0 的时候，不论弱引用是否为 0，都可以 delete
- 弱引用必须升级为强引用，才能访问它所指向的目标对象

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303071659513.png" alt="image-20210609101525750" style="zoom:50%;" />

wp 对比 sp

- 没有 ->、* 等操作符，取而代之的是 **`promote()`** 将 wp 提升为 sp
- 目标对象是 `RefBase`，而不再是 LightRefBase
- 除了目标对象指针 m_ptr 外，还有另外一个指针 m_refs（weakref_type）

```c
template <typename T>
class wp
{
public:
    typedef typename RefBase::weakref_type weakref_type;

  	// 构造函数
  
    inline wp() : m_ptr(0) { }

    explicit wp(T* other);
    wp(const wp<T>& other);
    explicit wp(const sp<T>& other);
    template<typename U> explicit wp(U* other);
    template<typename U> explicit wp(const sp<U>& other);
    template<typename U> explicit wp(const wp<U>& other);

    ~wp();
  
  	// promote
  	sp<T> promote() const;								// 弱引用转为强引用

    // 操作符重写

    wp& operator = (T* other);
    wp& operator = (const wp<T>& other);
    wp& operator = (const sp<T>& other);

    template<typename U> wp& operator = (U* other);
    template<typename U> wp& operator = (const wp<U>& other);
    template<typename U> wp& operator = (const sp<U>& other);

    void set_object_and_refs(T* other, weakref_type* refs);

    void clear();

    // Accessors

    inline  weakref_type* get_refs() const { return m_refs; }

    inline  T* unsafe_get() const { return m_ptr; }

    // Operators

    COMPARE_WEAK(==)
    COMPARE_WEAK(!=)
    COMPARE_WEAK(>)
    COMPARE_WEAK(<)
    COMPARE_WEAK(<=)
    COMPARE_WEAK(>=)

    inline bool operator == (const wp<T>& o) const {
        return (m_ptr == o.m_ptr) && (m_refs == o.m_refs);
    }
    template<typename U>
    inline bool operator == (const wp<U>& o) const {
        return m_ptr == o.m_ptr;
    }

    inline bool operator > (const wp<T>& o) const {
        return (m_ptr == o.m_ptr) ? (m_refs > o.m_refs) : (m_ptr > o.m_ptr);
    }
    template<typename U>
    inline bool operator > (const wp<U>& o) const {
        return (m_ptr == o.m_ptr) ? (m_refs > o.m_refs) : (m_ptr > o.m_ptr);
    }

    inline bool operator < (const wp<T>& o) const {
        return (m_ptr == o.m_ptr) ? (m_refs < o.m_refs) : (m_ptr < o.m_ptr);
    }
    template<typename U>
    inline bool operator < (const wp<U>& o) const {
        return (m_ptr == o.m_ptr) ? (m_refs < o.m_refs) : (m_ptr < o.m_ptr);
    }
                         inline bool operator != (const wp<T>& o) const { return m_refs != o.m_refs; }
    template<typename U> inline bool operator != (const wp<U>& o) const { return !operator == (o); }
                         inline bool operator <= (const wp<T>& o) const { return !operator > (o); }
    template<typename U> inline bool operator <= (const wp<U>& o) const { return !operator > (o); }
                         inline bool operator >= (const wp<T>& o) const { return !operator < (o); }
    template<typename U> inline bool operator >= (const wp<U>& o) const { return !operator < (o); }

private:
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;

    T*              m_ptr;									// 目标对象RefBase
    weakref_type*   m_refs;									// 弱引用计数器：指向的是m_ptr中的计数器
};
```

构造函数

- 在构造函数中，调用 RefBase.createWeek 获取弱引用计数器 weakref_type，并对弱引用 +1

```c
template<typename T>
wp<T>::wp(T* other)
    : m_ptr(other)
{
    if (other) m_refs = other->createWeak(this);
}
```

