# Covariant return types
&nbsp;  
&nbsp;

Specializations having covariant return types are supported.
&nbsp;  
&nbsp;  
&nbsp;


```c++
//file: covariant_return-poly.cc
#include "vane.h"
#define ____    printf("\n------------------------------");


struct     A  { A(char c='a') : n(c) {}  virtual ~A(){}  char n;  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };


////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx
    {
        using type = const A* (const A&);

        using domains = std::tuple< std::tuple<A,B,C> >;

        const A *operator() (const A &a) { printf("\n%7c --> fA", a.n); return &a; }
        const B *operator() (const B &a) { printf("\n%7c --> fB", a.n); return &a; }
        const C *operator() (const C &a) { printf("\n%7c --> fC", a.n); return &a; }
    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;



int main() try 
{
    vane::mf_init();

    multi_func  mfunc;
    vane::virtual_func <const A*(const A&)>  &vfunc = mfunc;


    printf("real arg    Fx-called  return");
    const A *ret;
____
    A a;
    B b;
    C c;
    ret = mfunc (a);     printf("%10c", ret->n);
    ret = mfunc (b);     printf("%10c", ret->n);
    ret = vfunc (c);     printf("%10c", ret->n);
____
    struct X : A  { X(char c='X') : A(c) {}  };
    struct Y : B  { Y(char c='Y') : B(c) {}  };
    struct Z : C  { Z(char c='Z') : C(c) {}  };

    X x;
    Y y;
    Z z;
    ret = vfunc (x);     printf("%10c", ret->n);
    ret = vfunc (y);     printf("%10c", ret->n);
    ret = vfunc (z);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real arg    Fx-called  return
------------------------------
      a --> fA         a
      b --> fB         b
      c --> fC         c
------------------------------
      X --> fA         X
      Y --> fB         Y
      Z --> fC         Z
*/
```





































```c++
//file: covariant_return-virt.cc
#include "vane.h"
#define ____    printf("\n------------------------------");


struct     A  { A(char c='a') : n(c) {}  virtual ~A(){}  char n;  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };

using Virtual = vane::virtual_<A>;

////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx
    {
        using type = const A* (const Virtual*);

        using domains = std::tuple< std::tuple<A,B,C> >;

        const A *operator() (const A *a) { printf("\n%7c --> fA", a->n); return a; }
        const B *operator() (const B *a) { printf("\n%7c --> fB", a->n); return a; }
        const C *operator() (const C *a) { printf("\n%7c --> fC", a->n); return a; }

    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;



int main() try 
{
    vane::mf_init();

    multi_func   mfunc;
    vane::virtual_func <const A*(const Virtual*)>   &vfunc = mfunc;


    printf("real arg    Fx-called  return");
    const A *ret;
____
    Virtual::of<A> a;
    Virtual::of<B> b;
    Virtual::of<C> c;
    ret = mfunc (&a);     printf("%10c", ret->n);
    ret = mfunc (&b);     printf("%10c", ret->n);
    ret = vfunc (&c);     printf("%10c", ret->n);
____
    struct X : A  { X(char c='X') : A(c) {}  };
    struct Y : B  { Y(char c='Y') : B(c) {}  };
    struct Z : C  { Z(char c='Z') : C(c) {}  };

    Virtual::of<X> x;
    Virtual::of<Y> y;
    Virtual::of<Z> z;
    ret = mfunc (&x);     printf("%10c", ret->n);
    ret = mfunc (&y);     printf("%10c", ret->n);
    ret = vfunc (&z);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real arg    Fx-called  return
------------------------------
      a --> fA         a
      b --> fB         b
      c --> fC         c
------------------------------
      X --> fA         X
      Y --> fB         Y
      Z --> fC         Z
*/
```





































```c++
//file: covariant_return-var.cc
#include "vane.h"
#define ____    printf("\n------------------------------");


struct     A  { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };

using Virtual = vane::var<A>;

////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx
    {
        using type = const A* (const Virtual*);

        using domains = std::tuple< std::tuple<A,B,C> >;

        A *operator() (A *a) { printf("\n%7c --> fA", a->n); return a; }
        B *operator() (B *a) { printf("\n%7c --> fB", a->n); return a; }
        C *operator() (C *a) { printf("\n%7c --> fC", a->n); return a; }
    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;



int main() try 
{
    vane::mf_init();

    multi_func   mfunc;
    vane::virtual_func <const A*(const Virtual*)>   &vfunc = mfunc;


    printf("real arg    Fx-called  return");
    const A *ret;
____
    Virtual::of<A> a;
    Virtual::of<B> b;
    Virtual::of<C> c;
    ret = mfunc (&a);     printf("%10c", ret->n);
    ret = mfunc (&b);     printf("%10c", ret->n);
    ret = vfunc (&c);     printf("%10c", ret->n);
____
    struct X : A  { X(char c='X') : A(c) {}  };
    struct Y : B  { Y(char c='Y') : B(c) {}  };
    struct Z : C  { Z(char c='Z') : C(c) {}  };

    Virtual::of<X> x;
    Virtual::of<Y> y;
    Virtual::of<Z> z;
    ret = mfunc (&x);     printf("%10c", ret->n);
    ret = mfunc (&y);     printf("%10c", ret->n);
    ret = vfunc (&z);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real arg    Fx-called  return
------------------------------
      a --> fA         a
      b --> fB         b
      c --> fC         c
------------------------------
      X --> fA         X
      Y --> fB         Y
      Z --> fC         Z
*/
```
