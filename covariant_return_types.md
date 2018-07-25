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
#include <stdio.h>
using std::tuple;


struct     A  { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };


namespace detail {
    struct Fx
    {
        using type = A* (A*);   //note: the virtual function must return a A*

        using domains = tuple< tuple<A,B,C> >;

        A *operator() (A *a) { printf("\n%8c --> fA", a->n); return a; }
        B *operator() (B *a) { printf("\n%8c --> fB", a->n); return a; }
        C *operator() (C *a) { printf("\n%8c --> fC", a->n); return a; }
    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;


////////////////////////////////////////////////////////////////////////////////

int main() try 
{
    multi_func   mfunc;
    vane::virtual_func <A*(A*)>   &vfunc = mfunc;

    A a;
    B b;
    C c;


    printf("%s%13s  %s","real args","Fx-called","return");
    A *ret;
    ret = mfunc (&a);     printf("%10c", ret->n);
    ret = mfunc (&b);     printf("%10c", ret->n);
    ret = vfunc (&c);     printf("%10c", ret->n);


    struct D : C  { D(char c='d') : C(c) {}  };

    D d;
    ret = vfunc (&d);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args    Fx-called  return
       a --> fA         a
       b --> fB         b
       c --> fC         c
       d --> fC         d
*/
```





































```c++
//file: covariant_return-virt.cc
#include "vane.h"
#include <stdio.h>
using std::tuple;


struct     A  { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };

using Virtual = vane::_virtual <A>;

namespace detail {
    struct Fx
    {
        using type = A* (Virtual*); //note: the virtual function must return a A*

        using domains = tuple< tuple<A,B,C> >;

        A *operator() (A *a) { printf("\n%8c --> fA", a->n); return a; }
        B *operator() (B *a) { printf("\n%8c --> fB", a->n); return a; }
        C *operator() (C *a) { printf("\n%8c --> fC", a->n); return a; }
    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;


////////////////////////////////////////////////////////////////////////////////

int main() try 
{
    multi_func   mfunc;
    vane::virtual_func <A*(Virtual*)>   &vfunc = mfunc;

    Virtual::of<A> a;
    Virtual::of<B> b;
    Virtual::of<C> c;


    printf("%s%13s  %s","real args","Fx-called","return");
    A *ret;
    ret = mfunc (&a);     printf("%10c", ret->n);
    ret = mfunc (&b);     printf("%10c", ret->n);
    ret = vfunc (&c);     printf("%10c", ret->n);


    struct D : C  { D(char c='d') : C(c) {}  };

    Virtual::of<D> d;
    ret = vfunc (&d);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args    Fx-called  return
       a --> fA         a
       b --> fB         b
       c --> fC         c
       d --> fC         d
*/
```





































```c++
//file: covariant_return-varg.cc
#include "vane.h"
#include <stdio.h>
using std::tuple;


struct     A  { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : A  { C(char c='c') : A(c) {}  };

using Virtual = vane::varg <A,B,C>;

namespace detail {
    struct Fx
    {
        using type = A* (Virtual*); //note: the virtual function must return a A*

        using domains = tuple< tuple<A,B,C> >;

        A *operator() (A *a) { printf("\n%8c --> fA", a->n); return a; }
        B *operator() (B *a) { printf("\n%8c --> fB", a->n); return a; }
        C *operator() (C *a) { printf("\n%8c --> fC", a->n); return a; }
    };
    //That's it!
}
using multi_func = vane::multi_func <detail::Fx>;


////////////////////////////////////////////////////////////////////////////////

int main() try 
{
    multi_func   mfunc;
    vane::virtual_func <A*(Virtual*)>   &vfunc = mfunc;

    Virtual::of<A> a;
    Virtual::of<B> b;
    Virtual::of<C> c;


    printf("%s%13s  %s","real args","Fx-called","return");
    A *ret;
    ret = mfunc (&a);     printf("%10c", ret->n);
    ret = mfunc (&b);     printf("%10c", ret->n);
    ret = vfunc (&c);     printf("%10c", ret->n);


    struct D : C  { D(char c='d') : C(c) {}  };

    Virtual::of<D> d;
    ret = vfunc (&d);     printf("%10c", ret->n);
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args    Fx-called  return
       a --> fA         a
       b --> fB         b
       c --> fC         c
       d --> fC         d
*/
```
