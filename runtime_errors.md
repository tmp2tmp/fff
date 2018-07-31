# runtime errors
&nbsp;  
&nbsp;  
&nbsp;

Invalid function calls (ambiguous calls or calls that have no matching functions)  
generate runtime errors as exceptions:  
&nbsp; &nbsp; <code>vane::multifunction_error::<b>invalid_call</b></code> &nbsp; 
    derived from <code><b>std::runtime_error</b></code>.  

Checking ambiguity is not 100% consistent with the C++ language call-resolution behaviors.  
&nbsp;

For debugging:  
Though not 100% consistent, calling the function call operators of the co-class(FX) of a mult\_func
can be useful a little sometimes &nbsp; to test at compile time 
whether calls on some expected combinations of argument types
will generate runtime errors of call-resolution or not 
&nbsp; when calls on those argument types require no call-resolution errors at runtime,  
again though not 100% guaranteed.  

<!--
calling the function call operator of the co-class(FX) of a mult\_func is 100% of C++ language call-resolution semantics.


Runtime errors occurr at runtime only. This makes debugging inconvenient.  
Whether a vane::multi\_func call generates a runtime error or not can be tested
  by getting the code-path acquire execution at runtime.  
But it can be test at compile time if we know the possible combinations of the argument types at the call.  
For the combinations of the arguments

The co-class of a multi\_func
-->

&nbsp;  
&nbsp;



```c++
//file: runtime_errors-poly.cc
#include "vane.h"


/* inheritance hierarchy:
      A-B
    O-X-Y
*/

struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){}  };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  virtual ~O(){}  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = void (int&, const A&, const O&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for A&
                    std::tuple <X,Y>    //for O&
                    >;
    //specializatins:
    void operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); }
    void operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const A &a, const O &x);


int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, A(), Y());
                     fx( i, A(), Y());
    i=10;   call (mfunc, i, B(), X());
                     fx( i, B(), X());

    i=100;  call (mfunc, i, B(), Y());  //runtime error: no matching function or ambiguous call
                 //  fx( i, B(), Y());  //compile error: call is ambiguous
    i=200;  call (mfunc, i, A(), X());  //runtime error: no matching function or ambiguous call
                 //  fx( i, A(), X());  //compile error: no matching function
    i=300;  call (mfunc, i, A(), O());  //runtime error: argument type out of domain
                 //  fx( i, A(), O());  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const A &a, const O &x) try
{
    mfunc (i, a, x);

    //trivial check at compile time
    //  of the call interface a multi_func<Fx> should have
    if(0) {
        (*(Fx*)nullptr) (i, A(), Y());
        (*(Fx*)nullptr) (i, B(), X());
    }
}
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY
  1| a y --> fAY
 10| b x --> fBX
 11| b x --> fBX
100| bad_call: multi-func error: invalid call: no matching function or ambiguous call
200| bad_call: multi-func error: invalid call: no matching function or ambiguous call
300| bad_type: multi-func error: invalid argument type: out of the type-domain
*/
```











































```c++
//file: runtime_errors-virt.cc
#include "vane.h"


/* inheritance hierarchy:
      A-B
    O-X-Y
*/

struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){}  };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  virtual ~O(){}  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


using VA = vane::virtual_<A>;
using VO = vane::virtual_<O>;

////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = void (int&, const VA&, const VO&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for VA &
                    std::tuple <X,Y>    //for VO &
                    >;
    //specializatins:
    void operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); }
    void operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const VA&, const VO&);


int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    VA::of<A> a;    VO::of<X> x;
    VA::of<B> b;    VO::of<Y> y;
                    VO::of<O> o;

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, a, y);
                     fx( i, a, y);
    i=10;   call (mfunc, i, b, x);
                     fx( i, b, x);

    i=100;  call (mfunc, i, b, y);  //runtime error: no matching function or ambiguous call
                 //  fx( i, b, y);  //compile error: call is ambiguous
    i=200;  call (mfunc, i, a, x);  //runtime error: no matching function or ambiguous call
                 //  fx( i, a, x);  //compile error: no matching function
    i=300;  call (mfunc, i, a, o);  //runtime error: argument type out of domain
                 //  fx( i, a, o);  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const VA &a, const VO &x) try
{
    mfunc (i, a, x);

    //trivial check at compile time
    //  of the call interface a multi_func<Fx> should have
    if(0) {
        (*(Fx*)nullptr) (i, A(), Y());
        (*(Fx*)nullptr) (i, B(), X());
    }
}
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY
  1| a y --> fAY
 10| b x --> fBX
 11| b x --> fBX
100| bad_call: multi-func error: invalid call: no matching function or ambiguous call
200| bad_call: multi-func error: invalid call: no matching function or ambiguous call
300| bad_type: multi-func error: invalid argument type: out of the type-domain
*/
```





































```c++
//file: runtime_errors-var.cc
#include "vane.h"


/* inheritance hierarchy:
      A-B
    O-X-Y
*/
//note: for var<>, parameters being polymorphic or not doesn't matter
struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){} };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


using var = vane::var<>;    //anonymous var<>

////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = void (int&, const var&, const var&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for the 1st parameter var&
                    std::tuple <X,Y>    //for the 2nd parameter var&
                    >;
    //specializatins:
    void operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); }
    void operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const var &a, const var &x);


int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    var::of<A> a;   var::of<X> x;
    var::of<B> b;   var::of<Y> y;
                    var::of<O> o;

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, a, y);
                     fx( i, a, y);
    i=10;   call (mfunc, i, b, x);
                     fx( i, b, x);
    i=100;  call (mfunc, i, b, y);  //runtime error: no matching function or ambiguous call
                 //  fx( i, b, y);  //compile error: call is ambiguous
    i=200;  call (mfunc, i, a, x);  //runtime error: no matching function or ambiguous call
                 //  fx( i, a, x);  //compile error: no matching function
    i=300;  call (mfunc, i, a, o);  //runtime error: argument type out of domain
                 //  fx( i, a, o);  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const var &a, const var &x) try
{
    mfunc (i, a, x);

    //trivial check at compile time
    //  of the call interface a multi_func<Fx> should have
    if(0) {
        (*(Fx*)nullptr) (i, A(), Y());
        (*(Fx*)nullptr) (i, B(), X());
    }
}
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY
  1| a y --> fAY
 10| b x --> fBX
 11| b x --> fBX
100| bad_call: multi-func error: invalid call: no matching function or ambiguous call
200| bad_call: multi-func error: invalid call: no matching function or ambiguous call
300| bad_type: multi-func error: invalid argument type: out of the type-domain
*/
```
&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  





#### custom error handlers

```c++
//file: runtime_errors-custom_handler.cc
#include "vane.h"
#include <iostream>


/* inheritance hierarchy:
      A-B
    O-X-Y
*/

struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){}  };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  virtual ~O(){}  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = int (int&, const A&, const O&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for A &
                    std::tuple <X,Y>    //for O &
                    >;
    //specializatins:
    int operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); return i;}
    int operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); return i;}


//custom error handlers
    int error_NO_MATCH(int &i, const A &a, const O &o) {
        printf("\n%3d|(%c %c)--> custom_error_handler for bad_call: no matching function or ambiguous call",
                i++, a.n, o.n);
        return -i;
    }

    int error_OOD(int &i, const A &a, const O &o) {
        printf("\n%3d|(%c %c)--> custom_error_handler for bad_type: out of the type-domain",
                i++, a.n, o.n);
        throw "my error for bad_type";
        return 0;
    }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const A&, const O&);


#define ____    printf("\n-------------------------------------------------------------");
int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, A(), Y());
                     fx( i, A(), Y());
    i=10;   call (mfunc, i, B(), X());
                     fx( i, B(), X());
____
    i=100;  call (mfunc, i, B(), Y());  //runtime error: no matching function or ambiguous call
                 //  fx( i, B(), Y());  //compile error: call is ambiguous
    i=200;  call (mfunc, i, A(), X());  //runtime error: no matching function or ambiguous call
                 //  fx( i, A(), X());  //compile error: no matching function
    i=300;  call (mfunc, i, A(), O());  //runtime error: argument type out of domain
                 //  fx( i, A(), O());  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const A &a, const O &x) try
{
    auto r = mfunc (i, a, x);
    std::cout << " = " << r;
}
catch(const char *e)                    { printf("\n%3d| custom exception: %s", i, e); }
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY = 1
  1| a y --> fAY
 10| b x --> fBX = 11
 11| b x --> fBX
-------------------------------------------------------------
100|(b y)--> custom_error_handler for bad_call: no matching function or ambiguous call = -101
200|(a x)--> custom_error_handler for bad_call: no matching function or ambiguous call = -201
300|(a o)--> custom_error_handler for bad_type: out of the type-domain
301| custom exception: my error for bad_type
*/
```




















```c++
//file: runtime_errors-custom_handler-virt.cc
#include "vane.h"
#include <iostream>


/* inheritance hierarchy:
      A-B
    O-X-Y
*/

struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){}  };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  virtual ~O(){}  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


using VA = vane::virtual_<A>;
using VO = vane::virtual_<O>;

////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = int (int&, const VA&, const VO&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for VA &
                    std::tuple <X,Y>    //for VO &
                    >;
    //specializatins:
    int operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); return i;}
    int operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); return i;}


//custom error handlers
    int error_NO_MATCH(int &i, const VA &va, const VO &vo) {
        const A &a = va;
        printf("\n%3d|(%c %c)--> custom_error_handler for bad_call: no matching function or ambiguous call",
                i++, a.n, static_cast<const O&>(vo).n);
        return -i;
    }

    int error_OOD(int &i, const VA &va, const VO &vo) {
        const O &o = vo;
        printf("\n%3d|(%c %c)--> custom_error_handler for bad_type: out of the type-domain",
                i++, static_cast<const A&>(va).n, o.n);
        throw "my error for bad_type";
        return 0;
    }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const VA&, const VO&);


#define ____    printf("\n-------------------------------------------------------------");
int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    VA::of<A> a;    VO::of<X> x;
    VA::of<B> b;    VO::of<Y> y;
                    VO::of<O> o;

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, a, y);
                     fx( i, a, y);
    i=10;   call (mfunc, i, b, x);
                     fx( i, b, x);
____
    i=100;  call (mfunc, i, b, y);  //runtime error: no matching function or ambiguous call
                 //  fx( i, b, y);  //compile error: call is ambiguous
    i=200;  call (mfunc, i, a, x);  //runtime error: no matching function or ambiguous call
                 //  fx( i, a, x);  //compile error: no matching function
    i=300;  call (mfunc, i, a, o);  //runtime error: argument type out of domain
                 //  fx( i, a, o);  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const VA &a, const VO &x) try
{
    auto r = mfunc (i, a, x);
    std::cout << " = " << r;
}
catch(const char *e)                    { printf("\n%3d| custom exception: %s", i, e); }
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY = 1
  1| a y --> fAY
 10| b x --> fBX = 11
 11| b x --> fBX
-------------------------------------------------------------
100|(b y)--> custom_error_handler for bad_call: no matching function or ambiguous call = -101
200|(a x)--> custom_error_handler for bad_call: no matching function or ambiguous call = -201
300|(a o)--> custom_error_handler for bad_type: out of the type-domain
301| custom exception: my error for bad_type
*/
```




















```c++
//file: runtime_errors-custom_handler-var.cc
#include "vane.h"
#include <iostream>


/* inheritance hierarchy:
      A-B
    O-X-Y
*/

struct A        { A(char c='a') : n(c) {}   char n;  virtual ~A(){}  };
struct B : A    { B(char c='b') : A(c) {}   };

struct O        { O(char c='o') : n(c) {}   char n;  virtual ~O(){}  };
struct X : O    { X(char c='x') : O(c) {}   };
struct Y : X    { Y(char c='y') : X(c) {}   };


struct tag1;
struct tag2;

using Var1 = vane::var<tag1>;
using Var2 = vane::var<tag2, int, float /*just tags... like hash tags on social media*/>;
//(or just)= vane::var<>;   //anonymous one

////////////////////////////////////////////////////////////////////////////////

struct Fx
{
    using type = int (int&, const Var1&, const Var2&);   //type of the virtual function

    using domains = std::tuple <        //type domains of the virtual parameters
                    std::tuple <A,B>,   //for Var1 &
                    std::tuple <X,Y>    //for Var2 &
                    >;
    //specializatins:
    int operator()(int &i, const A &a, const Y &x) { printf("\n%3d| %c %c --> fAY", i++, a.n, x.n); return i;}
    int operator()(int &i, const B &a, const X &x) { printf("\n%3d| %c %c --> fBX", i++, a.n, x.n); return i;}


//custom error handlers
    int error_NO_MATCH(int &i, const Var1 &v1, const Var2 &v2) {
        printf("\n%3d|(%p %p)--> custom_error_handler for bad_call: no matching function or ambiguous call",
                i++, &typeid(v1), &typeid(v2) );
        return -i;
    }

    int error_OOD(int &i, const Var1 &v1, const Var2 &v2) {
        printf("\n%3d|(%s %s)--> custom_error_handler for bad_type: out of the type-domain",
                i++, typeid(v1).name(), typeid(v2).name() );
        throw "my error for bad_type";
        return 0;
    }
};


////////////////////////////////////////////////////////////////////////////////
template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const Var1&, const Var2&);


#define ____    printf("\n-------------------------------------------------------------");
int main()
{
    vane::mf_init();

    vane::multi_func<Fx>  mfunc;
    Fx                    fx;   //ordinary function object

    Var1::of<A> a;  Var2::of<X> x;
    Var1::of<B> b;    Var2::of<Y> y;
                    Var2::of<O> o;

    printf("\nreal args    Fx called actually");

    int i;
    i=0;    call (mfunc, i, a, y);
                     fx( i, a, y);
    i=10;   call (mfunc, i, b, x);
                     fx( i, b, x);
____
    i=100;  call (mfunc, i, b, y);  //runtime error: no matching function or ambiguous call
                 //  fx( i, b, y);  //compile error: call is ambiguous
    i=200;  call (mfunc, i, a, x);  //runtime error: no matching function or ambiguous call
                 //  fx( i, a, x);  //compile error: no matching function
    i=300;  call (mfunc, i, a, o);  //runtime error: argument type out of domain
                 //  fx( i, a, o);  //compile error: no matching function
}


template <typename Fx>
void call (vane::multi_func<Fx> &mfunc, int &i, const Var1 &a, const Var2 &x) try
{
    auto r = mfunc (i, a, x);
    std::cout << " = " << r;
}
catch(const char *e)                    { printf("\n%3d| custom exception: %s", i, e); }
catch(const vane::bad_type &e)          { printf("\n%3d| bad_type: %s",         i, e.what()); }
catch(const vane::bad_call &e)          { printf("\n%3d| bad_call: %s",         i, e.what()); }
catch(const vane::multifunc_error &e)   { printf("\n%3d| multifunc_error: %s",  i, e.what()); }
catch(const std::runtime_error &e)      { printf("\n%3d| runtime_error: %s",    i, e.what()); }
catch(const std::exception &e)          { printf("\n%3d| exception : %s",       i, e.what()); }


/* output **********************************************************************
real args    Fx called actually
  0| a y --> fAY = 1
  1| a y --> fAY
 10| b x --> fBX = 11
 11| b x --> fBX
-------------------------------------------------------------
100|(0x10040b520 0x10040b600)--> custom_error_handler for bad_call: no matching function or ambiguous call = -101
200|(0x10040b4e0 0x10040b5c0)--> custom_error_handler for bad_call: no matching function or ambiguous call = -201
300|(N4vane3varIJ4tag1EE2ofI1AS2_EE N4vane3varIJ4tag2ifEE2ofI1OS2_EE)--> custom_error_handler for bad_type: out of the type-domain
301| custom exception: my error for bad_type
*/
```



















