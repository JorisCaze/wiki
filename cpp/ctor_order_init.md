What's the point with order initalization in ctor
=================================================

Last time, I had the following warning with *gcc* (version 7.5):

```console
src/Class.hpp:25:10: warning: class::m_b’ will be initialized after [-Wreorder]
```

It seemed that the order of member attributes initialization was not perfect, yet at first glance the following code looked fine to me:

```cpp
#include <iostream>

class Class
{
public:
    int m_a;
    int m_b;
    Class() : m_b(2), m_a(m_b) {}
};

int main()
{
    Class myClass;

    std::cout << "m_a: " << myClass.m_a << std::endl;
    std::cout << "m_b: " << myClass.m_b << std::endl;
}
```

Output:

```console
m_a: 32767
m_b: 2
```

Here we can clearly see that there is an issue to initialize `m_a`. According to our constructor we expect to have this order of operations:

```console
m_b = 2
m_a = m_b = 2
```

Yet, in reality since member attribute `m_a` is declared before `m_b` in class definition, it will be the first to be initialize. Hence, here `m_a` is set to `m_b` but `m_b` has no value yet, thus some random (old value in memory) value is set. Then, `m_b` is correctly set to `2`. To sum up, what is happening in reality:

```console
m_a = m_b = ? (here I had 32767 on this run)
m_b = 2
```

Keep in mind that initialization of member attributes in ctor is done in the order of declaration.

Sources:
- https://stackoverflow.com/questions/1828037/whats-the-point-of-g-wreorder
- https://openclassrooms.com/forum/sujet/will-be-initialized-after-95658
