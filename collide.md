# Shape Collision
&nbsp;  
&nbsp;  
&nbsp;  
```c++
//file: collide-poly.cc
#include "vane.h"   //required
#define ____    printf("\n----------------------------------------");


//one common base class
struct Shape {    const char *n;
                  virtual ~Shape() {}   //polymorphic base is required
                           Shape    (const char *c = "shape"    ) : n(c)     {} };
//derivatives
struct Rectangle : Shape { Rectangle(const char *c = "rectangle") : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c = "ellipse"  ) : Shape(c) {} };
struct Polygon   : Shape { Polygon  (const char *c = "polygon"  ) : Shape(c) {} };


using Collide_Func = vane::virtual_func <void (const Shape&, const Shape&)>;


////////////////////////////////////////////////////////////////////////////////
void big_bang (Collide_Func &collide)
{
    Rectangle  r;
    Ellipse    e;
    Polygon    p;

    printf("%15s%20s","real args","fx called");

    collide (r, r);
    collide (r, e);
    collide (r, p);
    collide (e, e);
    collide (e, p);
    collide (p, p);
____
    struct Square : Rectangle { Square() : Rectangle("~SQUARE~") {}; };
    Square square;

    collide (square, e);   //fRE !! -- Rectangle-Ellipse
}


////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx  //co-class that defines the traits & function set for the multi_func
    {
        using type    = void (const Shape&, const Shape&);   //signature of the multi_func
//   or using type    = Collide_Func::type;

        using domains = std::tuple<  //specialization confiner
                        std::tuple <Rectangle, Ellipse, Polygon>,   //for the 1st Shape*
                        std::tuple <Rectangle, Ellipse, Polygon>    //for the 2nd Shape*
                        >;

        //specializations
        void operator() (const Rectangle &p, const Rectangle &q) { printf("\n(%-9s %9s) --> fRR",    p.n, q.n);}
        void operator() (const Rectangle &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fRE !!", p.n, q.n);}
        void operator() (const Rectangle &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fRP",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fEE",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fEP",    p.n, q.n);}
        void operator() (const Polygon   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fPP",    p.n, q.n);}
    };
}
using My_Collide_Func  = vane::multi_func <detail::Fx>;


int main()
{
    vane::mf_init();

    My_Collide_Func  collide;

    big_bang(collide);
}


/* output **********************************************************************
      real args           fx called
(rectangle rectangle) --> fRR
(rectangle   ellipse) --> fRE !!
(rectangle   polygon) --> fRP
(ellipse     ellipse) --> fEE
(ellipse     polygon) --> fEP
(polygon     polygon) --> fPP
----------------------------------------
(~SQUARE~    ellipse) --> fRE !!
*/
```
















```c++
//file: collide-virt.cc
#include "vane.h"   //required
#define ____    printf("\n----------------------------------------");


//one common base class
struct Shape {    const char *n;
                  virtual ~Shape() {}   //polymorphic base is required
                           Shape    (const char *c = "shape"    ) : n(c)     {} };
//derivatives
struct Rectangle : Shape { Rectangle(const char *c = "rectangle") : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c = "ellipse"  ) : Shape(c) {} };
struct Polygon   : Shape { Polygon  (const char *c = "polygon"  ) : Shape(c) {} };


using VShape       = vane::virtual_<Shape>;
using Collide_Func = vane::virtual_func <void (const VShape&, const VShape&)>;


////////////////////////////////////////////////////////////////////////////////
void big_bang (Collide_Func &collide)
{
    VShape::of<Rectangle>  r;
    VShape::of<Ellipse>    e;
    VShape::of<Polygon>    p;

    printf("%15s%20s","real args","fx called");

    collide (r, r);
    collide (r, e);
    collide (r, p);
    collide (e, e);
    collide (e, p);
    collide (p, p);
____
    struct Square : Rectangle { Square() : Rectangle("~SQUARE~") {}; };
    VShape::of<Square> square;

    collide (square, e);   //fRE !! -- Rectangle-Ellipse
}


////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx  //co-class that defines the traits & function set for the multi_func
    {
        using type    = void (const VShape&, const VShape&);   //signature of the virtual function
//   or using type    = Collide_Func::type;

        using domains = std::tuple<  //specialization confiner
                        std::tuple <Rectangle, Ellipse, Polygon>,   //for the 1st Shape*
                        std::tuple <Rectangle, Ellipse, Polygon>    //for the 2nd Shape*
                        >;

        //specializations
        void operator() (const Rectangle &p, const Rectangle &q) { printf("\n(%-9s %9s) --> fRR",    p.n, q.n);}
        void operator() (const Rectangle &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fRE !!", p.n, q.n);}
        void operator() (const Rectangle &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fRP",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fEE",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fEP",    p.n, q.n);}
        void operator() (const Polygon   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fPP",    p.n, q.n);}
    };
}
using My_Collide_Func  = vane::multi_func <detail::Fx>;


int main()
{
    vane::mf_init();

    My_Collide_Func  collide;

    big_bang(collide);
}


/* output **********************************************************************
      real args           fx called
(rectangle rectangle) --> fRR
(rectangle   ellipse) --> fRE !!
(rectangle   polygon) --> fRP
(ellipse     ellipse) --> fEE
(ellipse     polygon) --> fEP
(polygon     polygon) --> fPP
----------------------------------------
(~SQUARE~    ellipse) --> fRE !!
*/
```
















```c++
//file: collide-var.cc
#include "vane.h"   //required
#define ____    printf("\n----------------------------------------");



struct Rectangle { const char *n = "rectangle"; };
struct Ellipse   { const char *n = "ellipse";   };
struct Polygon   { const char *n = "polygon";   };


struct Shape;
using VShape       = vane::var<Shape>;   //`Shape' is just a tag like a hash-tag on social media
using Collide_Func = vane::virtual_func <void (const VShape&, const VShape&)>;


////////////////////////////////////////////////////////////////////////////////
void big_bang (Collide_Func &collide)
{
    VShape::of<Rectangle>  r;
    VShape::of<Ellipse>    e;
    VShape::of<Polygon>    p;

    printf("%15s%20s","real args","fx called");

    collide (r, r);
    collide (r, e);
    collide (r, p);
    collide (e, e);
    collide (e, p);
    collide (p, p);
____
    struct Square : Rectangle { Square() { n = "~SQUARE~"; } };
    VShape::of<Square> square;

    collide (square, e);   //fRE !! -- Rectangle-Ellipse
}


////////////////////////////////////////////////////////////////////////////////
namespace detail {
    struct Fx  //co-class that defines the traits & function set for the multi_func
    {
        using type    = void (const VShape&, const VShape&);   //signature of the virtual function
//   or using type    = Collide_Func::type;

        using domains = std::tuple<  //specialization confiner
                        std::tuple <Rectangle, Ellipse, Polygon>,   //for the 1st Shape*
                        std::tuple <Rectangle, Ellipse, Polygon>    //for the 2nd Shape*
                        >;

        //specializations
        void operator() (const Rectangle &p, const Rectangle &q) { printf("\n(%-9s %9s) --> fRR",    p.n, q.n);}
        void operator() (const Rectangle &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fRE !!", p.n, q.n);}
        void operator() (const Rectangle &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fRP",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Ellipse   &q) { printf("\n(%-9s %9s) --> fEE",    p.n, q.n);}
        void operator() (const Ellipse   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fEP",    p.n, q.n);}
        void operator() (const Polygon   &p, const Polygon   &q) { printf("\n(%-9s %9s) --> fPP",    p.n, q.n);}
    };
}
using My_Collide_Func  = vane::multi_func <detail::Fx>;


int main()
{
    vane::mf_init();

    My_Collide_Func  collide;

    big_bang(collide);
}


/* output **********************************************************************
      real args           fx called
(rectangle rectangle) --> fRR
(rectangle   ellipse) --> fRE !!
(rectangle   polygon) --> fRP
(ellipse     ellipse) --> fEE
(ellipse     polygon) --> fEP
(polygon     polygon) --> fPP
----------------------------------------
(~SQUARE~    ellipse) --> fRE !!
*/
```


