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
#include "vane.h"   //required
#define ____    printf("\n---------------------------------");


/* inheritance hierarchy:
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
void call_uniformed (vane::multi_func<Fx,true> *vfunc, int &i, O &a, O *b) {
    (*vfunc)(i, a,b);
}

int main() try 
{
    vane::mf_init();

    vane::multi_func<Fx,true>   mfunc;
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
#define ____    printf("\n---------------------------------");


/* inheritance hierarchy:
         O = virtual base
        / \
       A   X
      / \ / \
     B   D   Y      D,E = making diamonds
      \     /
       \   /
        \ /
         E
*/

struct O             { virtual ~O(){}  //polymorphic base is required
                       char n;
                     };
struct A : virtual O { A() { n='a'; }   };  //A-O
struct B : A         { B() { n='b'; }   };  //B-A-O

struct X : virtual O { X() { n='x'; }   };  //X-O
struct Y : X         { Y() { n='y'; }   };  //Y-X-O

struct D : A, X      { D() { n='D'; }   };  //D-{A,X}-O   //Note: diamond


using VO = vane::virtual_<O>;



////////////////////////////////////////////////////////////////////////////////
//co-class that defines the traits & function set for the multi_func
struct Fx
{
    using type = void (int&, VO&, VO&);   //type of the virtual function

    using domains = std::tuple<    //type domains of the virtual arguments
                    std::tuple <A,B, D>,
                    std::tuple <X,Y, D>
                    >;

    //specializations:
    void operator() (int& i, A &a, X &b) { printf("\n%3d| %c %c --> fAX", i++, a.n, b.n); }
    void operator() (int& i, A &a, Y &b) { printf("\n%3d| %c %c --> fAY", i++, a.n, b.n); }
    void operator() (int& i, B &a, X &b) { printf("\n%3d| %c %c --> fBX", i++, a.n, b.n); }
    void operator() (int& i, B &a, Y &b) { printf("\n%3d| %c %c --> fBY", i++, a.n, b.n); }
    void operator() (int& i, D &a, D &b) { printf("\n%3d| %c %c --> fDD", i++, a.n, b.n); }
    void operator() (int& i, B &a, D &b) { printf("\n%3d| %c %c --> fBD", i++, a.n, b.n); }
};


////////////////////////////////////////////////////////////////////////////////
void call_uniformed (vane::multi_func<Fx,true> &vfunc, int& i, VO &a, VO &b) {
    vfunc(i, a,b);
}

int main() try 
{
    vane::mf_init();

    vane::multi_func<Fx,true>  mfunc;
    Fx  fx;   //ordinary function object


    VO::of<A> a;    VO::of<X> x;    VO::of<D> D;   
    VO::of<B> b;    VO::of<Y> y;
    

    printf("%-13s%s","real args","Fx called");
    int i;
    i=0;    call_uniformed (mfunc, i, a, x);
                               fx( i, a, x);
    i=10;   call_uniformed (mfunc, i, a, y);
                               fx( i, a, y);
    i=20;   call_uniformed (mfunc, i, b, x);
                               fx( i, b, x);
    i=30;   call_uniformed (mfunc, i, b, y);
                               fx( i, b, y);
    i=40;   call_uniformed (mfunc, i, D, D);
                               fx( i, D, D);
____
    struct E : B, Y { E() { n='E'; } };     //diamond;   E-{B,Y}-O 
    VO::of<E>  E;

    i=100;  call_uniformed (mfunc, i, E, E);
                               fx( i, E, E);
    i=110;  call_uniformed (mfunc, i, D, E);
                               fx( i, D, E);
    i=120;  call_uniformed (mfunc, i, E, D);
                               fx( i, E, D);
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
#define ____    printf("\n---------------------------------");


/* inheritance hierarchy:
         O = virtual base
        / \
       A   X
      / \ / \
     B   D   Y      D,E = making diamonds
      \     /
       \   /
        \ /
         E
*/

struct O             { virtual ~O(){}  //being polymorphic or not doesn't matter
                       char n;
                     };
struct A : virtual O { A() { n='a'; }   };  //A-O
struct B : A         { B() { n='b'; }   };  //B-A-O

struct X : virtual O { X() { n='x'; }   };  //X-O
struct Y : X         { Y() { n='y'; }   };  //Y-X-O

struct D : A, X      { D() { n='D'; }   };  //D-{A,X}-O   //Note: diamond


using Var = vane::var<>;


////////////////////////////////////////////////////////////////////////////////
//co-class that defines the traits & function set for the multi_func
struct Fx
{
    using type = void (int&, Var&, Var&);   //type of the virtual function

    using domains = std::tuple<    //type domains of the virtual arguments
                    std::tuple <A,B,X,D>,
                    std::tuple <X,Y,D, int, std::string>
                    >;

    //specializations:
    void operator() (int& i, A &a, X &b) { printf("\n%3d| %c %c --> fAX", i++, a.n, b.n);  }
    void operator() (int& i, A &a, Y &b) { printf("\n%3d| %c %c --> fAY", i++, a.n, b.n);  }
    void operator() (int& i, B &a, X &b) { printf("\n%3d| %c %c --> fBX", i++, a.n, b.n);  }
    void operator() (int& i, B &a, Y &b) { printf("\n%3d| %c %c --> fBY", i++, a.n, b.n);  }
    void operator() (int& i, D &a, D &b) { printf("\n%3d| %c %c --> fDD", i++, a.n, b.n);  }
    void operator() (int& i, B &a, D &b) { printf("\n%3d| %c %c --> fBD", i++, a.n, b.n);  }

    void operator() (int& i, A &a, int &b) { printf("\n%3d| %c %d --> fA.int",i++, a.n, b);  }
    void operator() (int& i, X &a, std::string &b) { printf("\n%3d| %c %s --> fX.std::string", i++, a.n, b.c_str()); }
};


////////////////////////////////////////////////////////////////////////////////
void call_uniformed (vane::multi_func<Fx,true> &mfunc, int &i, Var &a, Var &b) {
    mfunc(i, a,b);
}

int main() try 
{
    vane::mf_init();

    vane::multi_func<Fx,true>  mfunc;
    Fx  fx;   //ordinary function object


    Var::of<A> a;    Var::of<X> x;     Var::of<D> D;
    Var::of<B> b;    Var::of<Y> y;


    printf("%-13s%s","real args","Fx called");
    int i;
    i=0;    call_uniformed (mfunc, i, a, x);
                               fx( i, a, x);
    i=10;   call_uniformed (mfunc, i, a, y);
                               fx( i, a, y);
    i=20;   call_uniformed (mfunc, i, b, x);
                               fx( i, b, x);
    i=30;   call_uniformed (mfunc, i, b, y);
                               fx( i, b, y);
    i=40;   call_uniformed (mfunc, i, D, D);
                               fx( i, D, D);
____
    struct E : B, Y { E() { n='E'; } };     //diamond;   E-{B,Y}-O 
    Var::of<E>  E;

    i=100;  call_uniformed (mfunc, i, E, E);
                               fx( i, E, E);
    i=110;  call_uniformed (mfunc, i, D, E);
                               fx( i, D, E);
    i=120;  call_uniformed (mfunc, i, E, D);
                               fx( i, E, D);
____
    Var::of<int>         number{7};
    Var::of<std::string> string{"vane"};

    i=70;   call_uniformed (mfunc, i, D, number);
                               fx( i, D, number);
    i=80;   call_uniformed (mfunc, i, E, string);
                               fx( i, E, string);
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

