# diamond &nbsp; -- &nbsp; virtual/multiple inheritance
&nbsp;  
&nbsp;  
&nbsp;

Virtual and multiple inheritance are supported  
but at the cost of ```dynamic_cast``` when multi-dispatching `by-poly'. &nbsp; 
Dispatching `by-varg' has no such disadvantage of slowdown and is much faster than the other two.
&nbsp;  
&nbsp;  

Virtual and multiple inheritance are supported  
In case of multi-dispatching by-poly, 




```c++
//file: diamond-poly.cc
#include "vane.h"   //required
#include <stdio.h>
#define ____    printf("\n---------------------------------");


/* inheritance hierachy:
         O = virtual base
        / \
       A   X
      / \ / \
     B   D   Y      D,E = diamonds
      \     /
       \   /
        \ /
         E
*/

struct O             { virtual ~O(){}  //polymorphic base is required
                       char n;
                       O(char c='o') : n(c) {}  };

struct A : virtual O { A(char c='a') : O(c) {}  };  //A-O
struct B : A         { B(char c='b') : O(c) {}  };  //B-A-O

struct X : virtual O { X(char c='x') : O(c) {}  };  //A-O
struct Y : X         { Y(char c='y') : O(c) {}  };  //B-A-O

struct D : A, X      { D(char c='D') : O(c) {}  };  //D-{A,X}-O   //Note: diamond


////////////////////////////////////////////////////////////////////////////////
//co-class that defines the traits & function set for the multi_func
struct Fx
{
    using type = void (int&, O&, O*);   //type of the virtual function

    using domains = std::tuple<    //type domains of the virtual arguments
            std::tuple <A,B, D>,
            std::tuple <X,Y, D>
        >;
    //specializations:
    void operator() (int& i, A& a, X* b) { printf("\n%3d| %c %c --> fAX", i++, a.n, b->n); }
    void operator() (int& i, A& a, Y* b) { printf("\n%3d| %c %c --> fAY", i++, a.n, b->n); }
    void operator() (int& i, B& a, X* b) { printf("\n%3d| %c %c --> fBX", i++, a.n, b->n); }
    void operator() (int& i, B& a, Y* b) { printf("\n%3d| %c %c --> fBY", i++, a.n, b->n); }
    void operator() (int& i, D& a, D* b) { printf("\n%3d| %c %c --> fDD", i++, a.n, b->n); }
    void operator() (int& i, B& a, D* b) { printf("\n%3d| %c %c --> fBD", i++, a.n, b->n); }
};


////////////////////////////////////////////////////////////////////////////////
void call_uniformed (vane::multi_func<Fx> *vfunc, int &i, O &a, O *b) {
    (*vfunc)(i, a,b);
}

int main() try 
{
    vane::multi_func<Fx>    mfunc;
    Fx  fx;   //ordinary function object


    A a;    X x;    D D;   
    B b;    Y y;


    printf("%-13s%s","real args","Fx called");
    int i;
    i=0;    call_uniformed (&mfunc, i, a, &x);
                                fx( i, a, &x);
    i=10;   call_uniformed (&mfunc, i, a, &y);
                                fx( i, a, &y);
    i=20;   call_uniformed (&mfunc, i, b, &x);
                                fx( i, b, &x);
    i=30;   call_uniformed (&mfunc, i, b, &y);
                                fx( i, b, &y);
    i=40;   call_uniformed (&mfunc, i, D, &D);
                                fx( i, D, &D);
____
    struct E : B, Y { E(char c='E') : O(c) {} };    //diamond;   E-{B,Y}-O
    E E;

    i=100;  call_uniformed (&mfunc, i, E, &E);
                                fx( i, E, &E);
    i=110;  call_uniformed (&mfunc, i, D, &E);
                                fx( i, D, &E);
    i=120;  call_uniformed (&mfunc, i, E, &D);
                                fx( i, E, &D);
}
catch(const std::exception &ex) { printf("\nexception : %s", ex.what()); }


/* output **********************************************************************
real args    Fx called
  0| a x --> fAX
  1| a x --> fAX
 10| a y --> fAY
 11| a y --> fAY
 20| b x --> fBX
 21| b x --> fBX
 30| b y --> fBY
 31| b y --> fBY
 40| D D --> fDD
 41| D D --> fDD
---------------------------------
100| E E --> fBY
101| E E --> fBY
110| D E --> fAY
111| D E --> fAY
120| E D --> fBD
121| E D --> fBD
*/
```
























 
```c++
//file: diamond-virt.cc
#include "vane.h"   //required
#include <stdio.h>
#define ____    printf("\n---------------------------------");


/* inheritance hierachy:
         O = virtual base
        / \
       A   X
      / \ / \
     B   D   Y      D,E = diamonds
      \     /
       \   /
        \ /
         E
*/

struct O             { virtual ~O(){}  //polymorphic base is required
                       char n;
                       O(char c='o') : n(c) {}  };

struct A : virtual O { A(char c='a') : O(c) {}  };  //A-O
struct B : A         { B(char c='b') : O(c) {}  };  //B-A-O

struct X : virtual O { X(char c='x') : O(c) {}  };  //X-O
struct Y : X         { Y(char c='y') : O(c) {}  };  //Y-X-O

struct D : A, X      { D(char c='D') : O(c) {}  };  //D-{A,X}-O   //Note: diamond


using VO = vane::_virtual <O>;

#if 0
template<typename T>
struct VO_of : VO::of<T> {
    VO_of(char c) : O(c) {}
};
#else //or more generalizedly
template<typename T>
struct VO_of : VO::of<T> {
    template<typename...Args>
	VO_of(char c, Args&&...args) : O(c), VO::of<T>(std::forward<Args>(args)...) {}
};
#endif


////////////////////////////////////////////////////////////////////////////////
//co-class that defines the traits & function set for the multi_func
struct Fx
{
    using type = void (int&, VO*, VO*);   //type of the virtual function

    using domains = std::tuple<    //type domains of the virtual arguments
            std::tuple <A,B, D>,
            std::tuple <X,Y, D>
        >;
    //specializations:
    void operator() (int& i, A* a, X* b) { printf("\n%3d| %c %c --> fAX", i++, a->n, b->n); }
    void operator() (int& i, A* a, Y* b) { printf("\n%3d| %c %c --> fAY", i++, a->n, b->n); }
    void operator() (int& i, B* a, X* b) { printf("\n%3d| %c %c --> fBX", i++, a->n, b->n); }
    void operator() (int& i, B* a, Y* b) { printf("\n%3d| %c %c --> fBY", i++, a->n, b->n); }
    void operator() (int& i, D* a, D* b) { printf("\n%3d| %c %c --> fDD", i++, a->n, b->n); }
    void operator() (int& i, B* a, D* b) { printf("\n%3d| %c %c --> fBD", i++, a->n, b->n); }
};


////////////////////////////////////////////////////////////////////////////////
void call_uniformed (vane::multi_func<Fx> *vfunc, int& i, VO* a, VO* b) {
    (*vfunc)(i, a,b);
}

int main() try 
{
    vane::multi_func<Fx>  mfunc;
    Fx  fx;   //ordinary function object


    VO_of<A> a{'a'};    VO_of<X> x{'x'};    VO_of<D> D{'D'};   
    VO_of<B> b{'b'};    VO_of<Y> y{'y'};

    
    printf("%-13s%s","real args","Fx called");
    int i;
    i=0;    call_uniformed (&mfunc, i, &a, &x);
                                fx( i, &a, &x);
    i=10;   call_uniformed (&mfunc, i, &a, &y);
                                fx( i, &a, &y);
    i=20;   call_uniformed (&mfunc, i, &b, &x);
                                fx( i, &b, &x);
    i=30;   call_uniformed (&mfunc, i, &b, &y);
                                fx( i, &b, &y);
    i=40;   call_uniformed (&mfunc, i, &D, &D);
                                fx( i, &D, &D);
____
    struct E : B, Y { E(char c='E') : O(c) {} };    //diamond;   E-{B,Y}-O 
    VO_of<E>  E{'E'};

    i=100;  call_uniformed (&mfunc, i, &E, &E);
                                fx( i, &E, &E);
    i=110;  call_uniformed (&mfunc, i, &D, &E);
                                fx( i, &D, &E);
    i=120;  call_uniformed (&mfunc, i, &E, &D);
                                fx( i, &E, &D);
}
catch(const std::exception &ex) { printf("\nexception : %s", ex.what()); }


/* output **********************************************************************
real args    Fx called
  0| a x --> fAX
  1| a x --> fAX
 10| a y --> fAY
 11| a y --> fAY
 20| b x --> fBX
 21| b x --> fBX
 30| b y --> fBY
 31| b y --> fBY
 40| D D --> fDD
 41| D D --> fDD
---------------------------------
100| E E --> fBY
101| E E --> fBY
110| D E --> fAY
111| D E --> fAY
120| E D --> fBD
121| E D --> fBD
*/
```




















 
```c++
//file: diamond-varg.cc
#include "vane.h"   //required
#include <stdio.h>
#define ____    printf("\n---------------------------------");


/* inheritance hierachy:
         O = virtual base
        / \
       A   X
      / \ / \
     B   D   Y      D,E = diamonds
      \     /
       \   /
        \ /
         E
*/

struct O             { virtual ~O(){}  //being polymorphic or not doesn't matter
                       char n;
                       O(char c='o') : n(c) {}  };

struct A : virtual O { A(char c='a') : O(c) {}  };  //A-O
struct B : A         { B(char c='b') : O(c) {}  };  //B-A-O

struct X : virtual O { X(char c='x') : O(c) {}  };  //X-O
struct Y : X         { Y(char c='y') : O(c) {}  };  //Y-X-O

struct D : A, X      { D(char c='D') : O(c) {}  };  //D-{A,X}-O   //Note: diamond


using Varg = vane::varg <A,B,X,Y, D, int, std::string>;

#if 0
template<typename T>
struct Varg_of : Varg::of<T> {
    Varg_of(char c) : O(c) {}
};
#else //or more generalizedly
template<typename T>
struct Varg_of : Varg::of<T> {
    template<typename...Args>
	Varg_of(char c, Args&&...args) : O(c), Varg::of<T>(std::forward<Args>(args)...) {}
};
#endif


////////////////////////////////////////////////////////////////////////////////
//co-class that defines the traits & function set for the multi_func
struct Fx
{
    using type = void (int&, Varg*, Varg*);   //type of the virtual function

    using domains = std::tuple<    //type domains of the virtual arguments
            std::tuple <A,B,X,D>,
            std::tuple <X,Y,D, int, std::string>
        >;
    //specializations:
    void operator() (int& i, A* a, X* b) { printf("\n%3d| %c %c --> fAX", i++, a->n, b->n);  }
    void operator() (int& i, A* a, Y* b) { printf("\n%3d| %c %c --> fAY", i++, a->n, b->n);  }
    void operator() (int& i, B* a, X* b) { printf("\n%3d| %c %c --> fBX", i++, a->n, b->n);  }
    void operator() (int& i, B* a, Y* b) { printf("\n%3d| %c %c --> fBY", i++, a->n, b->n);  }
    void operator() (int& i, D* a, D* b) { printf("\n%3d| %c %c --> fDD", i++, a->n, b->n);  }
    void operator() (int& i, B* a, D* b) { printf("\n%3d| %c %c --> fBD", i++, a->n, b->n);  }

    void operator() (int& i, A* a, int* b) { printf("\n%3d| %c %d --> fA.int",i++, a->n, *b);  }
    void operator() (int& i, X* a, std::string* b) { printf("\n%3d| %c %s --> fX.std::string", i++,a->n,b->c_str()); }
};


////////////////////////////////////////////////////////////////////////////////
void call_uniformed (vane::multi_func<Fx> *vfunc, int &i, Varg *a, Varg *b) {
    (*vfunc)(i, a,b);
}

int main() try 
{
    vane::multi_func<Fx>  mfunc;
    Fx  fx;   //ordinary function object


    Varg_of<A> a{'a'};    Varg_of<X> x{'x'};     Varg_of<D> D('D');
    Varg_of<B> b{'b'};    Varg_of<Y> y{'y'};


    printf("%-13s%s","real args","Fx called");
    int i;
    i=0;    call_uniformed (&mfunc, i, &a, &x);
                                fx( i, &a, &x);
    i=10;   call_uniformed (&mfunc, i, &a, &y);
                                fx( i, &a, &y);
    i=20;   call_uniformed (&mfunc, i, &b, &x);
                                fx( i, &b, &x);
    i=30;   call_uniformed (&mfunc, i, &b, &y);
                                fx( i, &b, &y);
    i=40;   call_uniformed (&mfunc, i, &D, &D);
                                fx( i, &D, &D);
____
    struct E : B, Y { E(char c='E') : O(c) {} };    //diamond;   E-{B,Y}-O
    Varg_of<E> E{'E'};

    i=100;  call_uniformed (&mfunc, i, &E, &E);
                                fx( i, &E, &E);
    i=110;  call_uniformed (&mfunc, i, &D, &E);
                                fx( i, &D, &E);
    i=120;  call_uniformed (&mfunc, i, &E, &D);
                                fx( i, &E, &D);
____
    Varg::of<int>         number{7};
    Varg::of<std::string> string{"vane"};

    i=70;   call_uniformed (&mfunc, i, &D, &number);
                                fx( i, &D, &(int&)number);  //needs to be cast; 'int' is not a class/struct
    i=80;   call_uniformed (&mfunc, i, &E, &string);
                                fx( i, &E, &string);
}
catch(const std::exception &ex) { printf("\nexception : %s", ex.what()); }


/* output **********************************************************************
real args    Fx called
  0| a x --> fAX
  1| a x --> fAX
 10| a y --> fAY
 11| a y --> fAY
 20| b x --> fBX
 21| b x --> fBX
 30| b y --> fBY
 31| b y --> fBY
 40| D D --> fDD
 41| D D --> fDD
---------------------------------
100| E E --> fBY
101| E E --> fBY
110| D E --> fAY
111| D E --> fAY
120| E D --> fBD
121| E D --> fBD
---------------------------------
 70| D 7 --> fA.int
 71| D 7 --> fA.int
 80| E vane --> fX.std::string
 81| E vane --> fX.std::string
*/
```

