# Runtime function call resoultion
&nbsp;  
&nbsp;  
&nbsp;

Runtime function call resolutoin based on the inheritance hierarchies
of the types of the virtual arguments is supported.  
Invalid function calls generate [errors at runtime](runtime_errors.md).

&nbsp;  
&nbsp;


```c++
//file: call_resolution-poly.cc
#include "vane.h"
#include <stdio.h>
#define ____    printf("\n----------------------------------------");
using std::tuple;


//A-B-C-D
struct A      { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : B  { C(char c='c') : B(c) {}  };
struct D : C  { D(char c='d') : C(c) {}  };

//W-X-Y-Z
struct W      { W(char c='w') : n(c) {}  char n;  virtual ~W(){}  };
struct X : W  { X(char c='x') : W(c) {}  };
struct Y : X  { Y(char c='y') : X(c) {}  };
struct Z : Y  { Z(char c='z') : Y(c) {}  };


struct Fx
{
    using type = void (int &i,A*,W*);    //type of the virtual function

    using domains = tuple<     //type domains of the virtual parameters
                    tuple<A,C>,
                    tuple<W,Y>
                    >;
    void operator() (int& i, A *a, W *b) { printf("\n%3d|  %c %c --> fAW", i++, a->n, b->n); }
    void operator() (int& i, A *a, Y *b) { printf("\n%3d|  %c %c --> fAY", i++, a->n, b->n); }
    void operator() (int& i, C *a, W *b) { printf("\n%3d|  %c %c --> fCW", i++, a->n, b->n); }
    void operator() (int& i, C *a, Y *b) { printf("\n%3d|  %c %c --> fCY", i++, a->n, b->n); }
};


////////////////////////////////////////////////////////////////////////////////
int main() try 
{
    vane::multi_func <Fx>                      mfunc;
    vane::virtual_func <void(int &i,A*,W*)>   &vfunc = mfunc;

    Fx  fx;   //ordinary function object

    B b;
    D d;

    X x;
    Z z;


    printf("%s%14s","real args","Fx called");
    int i;
    i=0;    mfunc (i, &b, &x);      fx (i, &b, &x);
    i=10;   mfunc (i, &b, &z);      fx (i, &b, &z);
    i=20;   mfunc (i, &d, &x);      fx (i, &d, &x);
    i=30;   mfunc (i, &d, &z);      fx (i, &d, &z);

____
    struct M : B,Z  { M(char c='M') : B(c),Z(c) {}  };  //--> AY
    struct P : D,X  { P(char c='P') : D(c),X(c) {}  };  //--> CW

    M m;
    P p;

    i=100;  vfunc (i, &m, &m);      fx (i, &m, &m);   // (AY,AY) --> f(A,Y)
    i=110;  vfunc (i, &m, &p);      fx (i, &m, &p);   // (AY,CW) --> f(A,W)
    i=120;  vfunc (i, &p, &m);      fx (i, &p, &m);   // (CW,AY) --> f(C,Y)
    i=130;  vfunc (i, &p, &p);      fx (i, &p, &p);   // (CW,CW) --> f(C,W)
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args     Fx called
  0|  b x --> fAW
  1|  b x --> fAW
 10|  b z --> fAY
 11|  b z --> fAY
 20|  d x --> fCW
 21|  d x --> fCW
 30|  d z --> fCY
 31|  d z --> fCY
----------------------------------------
100|  M M --> fAY
101|  M M --> fAY
110|  M P --> fAW
111|  M P --> fAW
120|  P M --> fCY
121|  P M --> fCY
130|  P P --> fCW
131|  P P --> fCW
*/
```




































```c++
//file: call_resolution-virt.cc
#include "vane.h"
#include <stdio.h>
#define ____    printf("\n----------------------------------------");
using std::tuple;


//A-B-C-D
struct A      { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : B  { C(char c='c') : B(c) {}  };
struct D : C  { D(char c='d') : C(c) {}  };

//W-X-Y-Z
struct W      { W(char c='w') : n(c) {}  char n;  virtual ~W(){}  };
struct X : W  { X(char c='x') : W(c) {}  };
struct Y : X  { Y(char c='y') : X(c) {}  };
struct Z : Y  { Z(char c='z') : Y(c) {}  };

using VA = vane::_virtual <A>;
using VW = vane::_virtual <W>;

struct Fx
{
    using type = void (int &i,VA*,VW*);  //type of the virtual function

    using domains = tuple<     //type domains of the virtual parameters
                    tuple<A,C>,
                    tuple<W,Y>
                    >;
    void operator() (int& i, A *a, W *b) { printf("\n%3d|  %c %c --> fAW", i++, a->n, b->n); }
    void operator() (int& i, A *a, Y *b) { printf("\n%3d|  %c %c --> fAY", i++, a->n, b->n); }
    void operator() (int& i, C *a, W *b) { printf("\n%3d|  %c %c --> fCW", i++, a->n, b->n); }
    void operator() (int& i, C *a, Y *b) { printf("\n%3d|  %c %c --> fCY", i++, a->n, b->n); }
};


////////////////////////////////////////////////////////////////////////////////
int main() try 
{
    vane::multi_func <Fx>                        mfunc;
    vane::virtual_func <void(int &i,VA*,VW*)>   &vfunc = mfunc;

    Fx  fx;   //ordinary function object

    VA::of<B> b;
    VA::of<D> d;

    VW::of<X>  x;
    VW::of<Z>  z;


    printf("%s%14s","real args","Fx called");
    int i;
    i=0;    mfunc (i, &b, &x);      fx (i, &b, &x);
    i=10;   mfunc (i, &b, &z);      fx (i, &b, &z);
    i=20;   mfunc (i, &d, &x);      fx (i, &d, &x);
    i=30;   mfunc (i, &d, &z);      fx (i, &d, &z);

____
    struct M : B,Z  { M(char c='M') : B(c),Z(c) {}  };  //--> AY
    struct P : D,X  { P(char c='P') : D(c),X(c) {}  };  //--> CW

    VA::of<M> ma;
    VW::of<M> mw;
    VA::of<P> pa;
    VW::of<P> pw;

    i=100;  vfunc (i, &ma, &mw);      fx (i, &ma, &mw);   // (AY,AY) --> f(A,Y)
    i=110;  vfunc (i, &ma, &pw);      fx (i, &ma, &pw);   // (AY,CW) --> f(A,W)
    i=120;  vfunc (i, &pa, &mw);      fx (i, &pa, &mw);   // (CW,AY) --> f(C,Y)
    i=130;  vfunc (i, &pa, &pw);      fx (i, &pa, &pw);   // (CW,CW) --> f(C,W)
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args     Fx called
  0|  b x --> fAW
  1|  b x --> fAW
 10|  b z --> fAY
 11|  b z --> fAY
 20|  d x --> fCW
 21|  d x --> fCW
 30|  d z --> fCY
 31|  d z --> fCY
----------------------------------------
100|  M M --> fAY
101|  M M --> fAY
110|  M P --> fAW
111|  M P --> fAW
120|  P M --> fCY
121|  P M --> fCY
130|  P P --> fCW
131|  P P --> fCW
*/
```



































```c++
//file: call_resolution-varg.cc
#include "vane.h"
#include <stdio.h>
#define ____    printf("\n----------------------------------------");
using std::tuple;


//A-B-C-D
struct A      { A(char c='a') : n(c) {}  char n;  virtual ~A(){}  };
struct B : A  { B(char c='b') : A(c) {}  };
struct C : B  { C(char c='c') : B(c) {}  };
struct D : C  { D(char c='d') : C(c) {}  };

//W-X-Y-Z
struct W      { W(char c='w') : n(c) {}  char n;  virtual ~W(){}  };
struct X : W  { X(char c='x') : W(c) {}  };
struct Y : X  { Y(char c='y') : X(c) {}  };
struct Z : Y  { Z(char c='z') : Y(c) {}  };

using VA = vane::varg <A,B,C,D>;
using VW = vane::varg <W,X,Y,Z>;

struct Fx
{
    using type = void (int &i,VA*,VW*);  //type of the virtual function

    using domains = tuple<     //type domains of the virtual parameters
                    tuple<A,C>,
                    tuple<W,Y>
                    >;
    void operator() (int& i, A *a, W *b) { printf("\n%3d|  %c %c --> fAW", i++, a->n, b->n); }
    void operator() (int& i, A *a, Y *b) { printf("\n%3d|  %c %c --> fAY", i++, a->n, b->n); }
    void operator() (int& i, C *a, W *b) { printf("\n%3d|  %c %c --> fCW", i++, a->n, b->n); }
    void operator() (int& i, C *a, Y *b) { printf("\n%3d|  %c %c --> fCY", i++, a->n, b->n); }
};


////////////////////////////////////////////////////////////////////////////////
int main() try 
{
    vane::multi_func <Fx>                        mfunc;
    vane::virtual_func <void(int &i,VA*,VW*)>   &vfunc = mfunc;

    Fx  fx;   //ordinary function object

    VA::of<B> b;
    VA::of<D> d;

    VW::of<X>  x;
    VW::of<Z>  z;


    printf("%s%14s","real args","Fx called");
    int i;
    i=0;    mfunc (i, &b, &x);      fx (i, &b, &x);
    i=10;   mfunc (i, &b, &z);      fx (i, &b, &z);
    i=20;   mfunc (i, &d, &x);      fx (i, &d, &x);
    i=30;   mfunc (i, &d, &z);      fx (i, &d, &z);

____
    struct M : B,Z  { M(char c='M') : B(c),Z(c) {}  };  //--> AY
    struct P : D,X  { P(char c='P') : D(c),X(c) {}  };  //--> CW

    VA::of<M> ma;
    VW::of<M> mw;
    VA::of<P> pa;
    VW::of<P> pw;

    i=100;  vfunc (i, &ma, &mw);      fx (i, &ma, &mw);   // (AY,AY) --> f(A,Y)
    i=110;  vfunc (i, &ma, &pw);      fx (i, &ma, &pw);   // (AY,CW) --> f(A,W)
    i=120;  vfunc (i, &pa, &mw);      fx (i, &pa, &mw);   // (CW,AY) --> f(C,Y)
    i=130;  vfunc (i, &pa, &pw);      fx (i, &pa, &pw);   // (CW,CW) --> f(C,W)
}
catch( const std::exception &ex ) { printf("\nexception: %s", ex.what() ); }


/* output **********************************************************************
real args     Fx called
  0|  b x --> fAW
  1|  b x --> fAW
 10|  b z --> fAY
 11|  b z --> fAY
 20|  d x --> fCW
 21|  d x --> fCW
 30|  d z --> fCY
 31|  d z --> fCY
----------------------------------------
100|  M M --> fAY
101|  M M --> fAY
110|  M P --> fAW
111|  M P --> fAW
120|  P M --> fCY
121|  P M --> fCY
130|  P P --> fCW
131|  P P --> fCW
*/
```

