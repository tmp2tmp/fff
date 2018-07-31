# make_shared &nbsp; utility
&nbsp;  
&nbsp;  
&nbsp;

<div style='font: 12pt consolas; -white-space:pre'>
when&nbsp;&nbsp;using VirtualShape = vane::varg&lt;Rectangle,...&gt;;<br>
or&nbsp;&nbsp;&nbsp;&nbsp;using VirtualShape = vane::_virtual&lt;Shape&gt;;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<i>//where struct Rectangle : Shape {...};</i><br>
<br>
</div>

<pre><code>std::<b>make_shared</b> &lt;VirtualShape::of&lt;Rectangle&gt;&gt; {...};
<i>//is equivalent to:</i>
vane::<b>make_shared</b> &lt;Rectangle, VirtualShape&gt; {...};
</code></pre>

<pre><code>std::<b>make_unique</b> &lt;VirtualShape::of&lt;Rectangle&gt;&gt; {...};
<i>//is equivalent to:</i>
vane::<b>make_unique</b> &lt;Rectangle, VirtualShape&gt; {...};
</code></pre>



&nbsp;  
&nbsp;  
&nbsp;
#### examples

```c++
//file: make_shared-virt.cc
#include "vane.h"   //required
using vane::virtual_;


struct Shape             { const char *name;
                           virtual ~Shape() {}    //polymorphic
                           Shape    (const char *c) : name(c)  {} };
struct Rectangle : Shape { Rectangle(const char *c) : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c) : Shape(c) {} };
struct Polygon   : Shape { Polygon  (const char *c) : Shape(c) {} };


struct PrintFx {
    using type = void (const virtual_<Shape>&);
    using domains = std::tuple <
                    std::tuple <Rectangle, Ellipse, Polygon>
                    >;
    void operator() (const Rectangle &s) { printf("\n%s  @fx=fR", s.name); }
    void operator() (const Ellipse   &s) { printf("\n%s  @fx=fE", s.name); }
    void operator() (const Polygon   &s) { printf("\n%s  @fx=fP", s.name); }
};


#define ____    printf("\n-----------------------------------------");
int main() try
{
    vane::mf_init();

    vane::multi_func <PrintFx>                          mprint;
    vane::virtual_func <void(const virtual_<Shape>&)>  &vprint = mprint;

    PrintFx  printfx;   //ordinary function object


    //std::make_shared equiv.
    auto shared_Rp =  std::make_shared <virtual_<Shape>::of<Rectangle>> ("shared_R");
    auto shared_Ep = vane::make_shared <Ellipse, virtual_<Shape>>       ("shared_E");
    auto shared_Pp = vane::make_shared <Polygon, virtual_<Shape>>       ("shared_P");   

    mprint  (*shared_Rp);
    vprint  (*shared_Ep);
    printfx (*shared_Pp);
    printf  ("\n%s  //printf", shared_Pp->name);


____//std::make_unique equiv.
    auto unique_Rp =  std::make_unique <virtual_<Shape>::of<Rectangle>> ("unique_R");
    auto unique_Ep = vane::make_unique <Ellipse, virtual_<Shape>>       ("unique_E");
    auto unique_Pp = vane::make_unique <Polygon, virtual_<Shape>>       ("unique_P");

    mprint  (*unique_Rp);
    vprint  (*unique_Ep);
    printfx (*unique_Pp);
    printf  ("\n%s  //printf", unique_Pp->name);
}
catch( std::exception &x ) { printf("\nexception: %s\n", x.what()); }


/* output **********************************************************************
shared_R  @fx=fR
shared_E  @fx=fE
shared_P  @fx=fP
shared_P  //printf
-----------------------------------------
unique_R  @fx=fR
unique_E  @fx=fE
unique_P  @fx=fP
unique_P  //printf
*/
```




































```c++
//file: make_shared-var.cc
#include "vane.h"


struct Rectangle { Rectangle(const char *c): name(c) {}  const char *name; };
struct Ellipse   { Ellipse  (const char *c): name(c) {}  const char *name; };
struct Polygon   { Polygon  (const char *c): name(c) {}  const char *name; };

using Shape = vane::var<>;  //anonymous var<>


struct PrintFx {
    using type = void (const Shape&);
    using domains = std::tuple <
                    std::tuple <Rectangle, Ellipse, Polygon>
                    >;
    void operator() (const Rectangle &s) { printf("\n%s  @fx=fR", s.name); }
    void operator() (const Ellipse   &s) { printf("\n%s  @fx=fE", s.name); }
    void operator() (const Polygon   &s) { printf("\n%s  @fx=fP", s.name); }
};


#define ____    printf("\n-----------------------------------------");
int main() try
{
    vane::mf_init();

    vane::multi_func <PrintFx>                mprint;
    vane::virtual_func <void(const Shape&)>  &vprint = mprint;

    PrintFx  printfx;   //ordinary function object


    //std::make_shared equiv.
    auto shared_Rp =  std::make_shared <Shape::of<Rectangle>> ("shared_R");
    auto shared_Ep = vane::make_shared <Ellipse, Shape>       ("shared_E");
    auto shared_Pp = vane::make_shared <Polygon, Shape>       ("shared_P");

    mprint  (*shared_Rp);
    vprint  (*shared_Ep);
    printfx (*shared_Pp);
    printf  ("\n%s  //printf", shared_Pp->name);


____//std::make_unique equiv.
    auto unique_Rp =  std::make_unique <Shape::of<Rectangle>> ("shared_R");
    auto unique_Ep = vane::make_unique <Ellipse, Shape>       ("shared_E");
    auto unique_Pp = vane::make_unique <Polygon, Shape>       ("shared_P");

    mprint  (*unique_Rp);
    vprint  (*unique_Ep);
    printfx (*unique_Pp);
    printf  ("\n%s  //printf", unique_Pp->name);
}
catch( std::exception &x ) { printf("\nexception: %s\n", x.what()); }


/* output **********************************************************************
shared_R  @fx=fR
shared_E  @fx=fE
shared_P  @fx=fP
shared_P  //printf
-----------------------------------------
shared_R  @fx=fR
shared_E  @fx=fE
shared_P  @fx=fP
shared_P  //printf
*/
```

