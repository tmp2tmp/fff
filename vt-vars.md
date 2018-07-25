# vt-vars.cc

```c++
//vt-vargs.cc
#include "vane.h"   //required
#include <stdio.h>

using vane::_virtual;   //for _virtual <Shape>


struct Shape {
    virtual ~Shape() {} //polymorphic base is required
    void print() { printf("\n%-9s %-16s", name, "@Shape::print"); }
    const char *name;
                           Shape    (const char *c = "shape"    ) : name(c)  {} };
struct Rectangle : Shape { Rectangle(const char *c = "rectangle") : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c = "ellipse"  ) : Shape(c) {} };
struct Polygon   : Shape { Polygon  (const char *c = "polygon"  ) : Shape(c) {} };



/////////////////////////////////////////////////////////////////////////////
void print_Rectangle (Rectangle *rectangle) {
    printf("\n%-9s %-16s", rectangle->name, "@print_Rectangle");
}


void print_VShape_like_variants (_virtual<Shape> *vshape)
{
    auto id = vshape->virt_id();

    if( id == _virtual<Shape>::of<Rectangle>::virt_id() )
        print_Rectangle( vane::get<Rectangle>(vshape) );
    else if( id == _virtual<Shape>::of<Ellipse>::virt_id() )
        vane::get<Ellipse>(vshape)->print();
    else if( id == _virtual<Shape>::of<Polygon>::virt_id() )
        vane::get<Polygon>(vshape)->print();
    else
        vane::get<Shape>(vshape)->print();

    printf(" @like_variants");
}

int main()
{
    _virtual<Shape>::of<Rectangle>  vR{"vR"};

    //as a Rectangle;  a _virtual<Shape>::of<Rectangle> is-A Rectangle
    Rectangle r = vR;
    printf("%s", vR.name );
    print_Rectangle( &vR );
    vR.print();

____//like variants
    _virtual<Shape>::of<Ellipse>    vE;
    _virtual<Shape>::of<Polygon>    vP;

    print_VShape_like_variants( &vR );
    print_VShape_like_variants( &vE );
    print_VShape_like_variants( &vP );

____//new subclass
    struct Square : Shape { Square(const char *c = "~SQUARE~"  ) : Shape(c) {} };
    _virtual<Shape>::of<Square>  vQ;
    vQ.print();
    print_VShape_like_variants( &vQ );
}

/* output **********************************************************************
vR
vR        @print_Rectangle
vR        @Shape::print   
-----------------------------------------
vR        @print_Rectangle @like_variants
ellipse   @Shape::print    @like_variants
polygon   @Shape::print    @like_variants
-----------------------------------------
~SQUARE~  @Shape::print   
~SQUARE~  @Shape::print    @like_variants
*/


```
