# Replacing virtual functions at runtime
&nbsp;  
&nbsp;

A vane::multi\_func is implemented as a function object and is polymorphic of the specialization sets.
The whole set of specializations can be replaced at runtime by switching the vane::multi\_func instances
pointed by vane::virtual\_func pointers.

&nbsp;  
&nbsp;


```c++
//file: runtime-switch-collide-poly.cc
#include "vane.h"   //required


struct Shape {
	virtual ~Shape() {}   //polymorphic base is required
	Shape(const char *c = "shape") : name(c) {}
	const char *name;
};

using Collide_func = vane::virtual_func <void(Shape&, Shape&)>;



////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
/* inheritance hierarchy

		   / Rectangle
	Shape +- Ellipse
		   \ Polygon

	_ShapeCollision +- _Fx_Big --- _Fx_Bigger
					 \ _Fx_Bong
*/
////////////////////////////////////////////////////////////////////////////////
struct _ShapeCollision
{
	using type = Collide_func::type;	//function's type
//	or
//	using type = void(Shape&, Shape&);

	_ShapeCollision(const char *version) : api_version(version) {}
	const char *api_version;

	void _collide(Shape &p, Shape &q, const char *tag) {  //util function
		printf("(%-9s %9s) --> %14s::%s\n", p.name, q.name, api_version, tag);
	}
};




////////////////////////////////////////////////////////////////////////////////
//Big Collision
struct Rectangle : Shape { Rectangle(const char *c = "rectangle") : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c = "ellipse"  ) : Shape(c) {} };

namespace detail {
	struct _Fx_Big : protected _ShapeCollision
	{
		_Fx_Big (const char *version="collide big") : _ShapeCollision(version) {}

		using _ShapeCollision::type;	//function's type

		using domains = std::tuple <  	//argument type domains
						std::tuple <Rectangle, Ellipse>,
						std::tuple <Rectangle, Ellipse>
						>;

		//specializations: Fx's
		void operator() (Rectangle &p, Rectangle &q) { _collide(p,q, "fRR"); }
		void operator() (Ellipse   &p, Ellipse   &q) { _collide(p,q, "fEE"); }
		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE"); }
	};
}
using Collide_Big = vane::multi_func <detail::_Fx_Big>;



////////////////////////////////////////////////////////////////////////////////
//Bigger Collision
namespace detail {
	struct _Fx_Bigger : protected _Fx_Big
	{
		_Fx_Bigger (const char *ver="collide bigger") : _Fx_Big(ver) {}

		using _Fx_Big::type;
		using _Fx_Big::domains;  //use the same domains

		//specializations:
		using _Fx_Big::operator();  //reuses the Fx's of the base class

		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE -modified"); }
		void operator() (Ellipse   &p, Rectangle &q) { _collide(p,q, "fER -new"); }
	};
}
using Collide_Bigger = vane::multi_func <detail::_Fx_Bigger>;




////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
//Bong Collision
struct Polygon : Shape { Polygon (const char *c = "polygon") : Shape(c) {} };

namespace detail {
	struct _Fx_Bong : protected _ShapeCollision
	{
		_Fx_Bong (const char *version="collide bong") : _ShapeCollision(version) {}

		using _ShapeCollision::type;

		using domains = std::tuple <	//different domains than the formers
						std::tuple <Rectangle, Ellipse, Polygon>,
						std::tuple <Rectangle, Ellipse, Polygon>
						>;

		//specializations; a whole new set of Fx's
		void operator() (Rectangle &p, Rectangle &q) { _collide(p,q, "fRR -compatible behavior"); }
		void operator() (Ellipse   &p, Ellipse   &q) { _collide(p,q, "fEE -compatible behavior"); }
		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE -redefined  behavior"); }
		void operator() (Ellipse   &p, Rectangle &q) { _collide(p,q, "fER -redefined  behavior"); }

		void operator() (Rectangle &p, Polygon   &q) { _collide(p,q, "fRP -new");   }
		void operator() (Ellipse   &p, Polygon   &q) { _collide(p,q, "fEP -new");   }
	};
}
vane::multi_func <detail::_Fx_Bong>  collide_bong;	//as a global function




////////////////////////////////////////////////////////////////////////////////
void big_bang(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bang");
	Rectangle  r;
	Ellipse	   e;

	collide (r, r);
	collide (e, e);
	collide (r, e);
}
 
void bigger_bang (Collide_func  &collide) {
	puts("---------------------------------------------------------------bigger bang");
	Rectangle  r;
	Ellipse	   e;
	Polygon	   p;

	collide (r, r);
	collide (e, e);
	collide (r, e);
	collide (e, r);
}

void big_bong(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bong");
	Rectangle  r;
	Ellipse	   e;
	Polygon	   p;

	collide (r, r);
	collide (e, e);
	collide (r, e);
	collide (e, r);

	collide (r, p);
	collide (e, p);
}


#define	____  puts("\n~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
int main() try
{
	vane::mf_init();

	Collide_Big     collide_big;
	Collide_Bigger  collide_bigger;

	printf("%15s%36s","real args","fx called");
____
	big_bang    (collide_big);
____
	bigger_bang (collide_bigger);
	big_bang    (collide_bigger);
____
	big_bong    (collide_bong);
	bigger_bang (collide_bong);
	big_bang    (collide_bong);
}
catch(const std::exception &e) { printf("\nexception: %s", e.what());  }


/* output **********************************************************************
      real args                           fx called
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bang
(rectangle rectangle) -->    collide big::fRR
(ellipse     ellipse) -->    collide big::fEE
(rectangle   ellipse) -->    collide big::fRE

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------bigger bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified
(ellipse   rectangle) --> collide bigger::fER -new
---------------------------------------------------------------big bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bong
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
(rectangle   polygon) -->   collide bong::fRP -new
(ellipse     polygon) -->   collide bong::fEP -new
---------------------------------------------------------------bigger bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
---------------------------------------------------------------big bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
*/
```


























```c++
//file: runtime-switch-collide-virt.cc
#include "vane.h"   //required


struct Shape {
	virtual ~Shape() {}   //polymorphic base is required
	Shape(const char *c = "shape") : name(c) {}
	const char *name;
};

using VShape       = vane::virtual_<Shape>;
using Collide_func = vane::virtual_func <void(VShape*, VShape*)>;



////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
/* inheritance hierarchy

		   / Rectangle
	Shape +- Ellipse
		   \ Polygon

	_ShapeCollision +- _Fx_Big --- _Fx_Bigger
					 \ _Fx_Bong
*/
////////////////////////////////////////////////////////////////////////////////
struct _ShapeCollision
{
	using type = Collide_func::type;	//function's type

	_ShapeCollision(const char *version) : api_version(version) {}
	const char *api_version;

	void _collide(Shape *p, Shape *q, const char *tag) {  //util function
		printf("(%-9s %9s) --> %14s::%s\n", p->name, q->name, api_version, tag);
	}
};




////////////////////////////////////////////////////////////////////////////////
//Big Collision
struct Rectangle : Shape { Rectangle(const char *c = "rectangle") : Shape(c) {} };
struct Ellipse   : Shape { Ellipse  (const char *c = "ellipse"  ) : Shape(c) {} };

namespace detail {
	struct _Fx_Big : protected _ShapeCollision
	{
		_Fx_Big (const char *version="collide big") : _ShapeCollision(version) {}

		using _ShapeCollision::type;	//function's type

		using domains = std::tuple <  	//argument type domains
						std::tuple <Rectangle, Ellipse>,
						std::tuple <Rectangle, Ellipse>
						>;

		//specializations: Fx's
		void operator() (Rectangle *p, Rectangle *q) { _collide(p,q, "fRR"); }
		void operator() (Ellipse   *p, Ellipse   *q) { _collide(p,q, "fEE"); }
		void operator() (Rectangle *p, Ellipse   *q) { _collide(p,q, "fRE"); }
	};
}
using Collide_Big = vane::multi_func <detail::_Fx_Big>;



////////////////////////////////////////////////////////////////////////////////
//Bigger Collision
namespace detail {
	struct _Fx_Bigger : protected _Fx_Big
	{
		_Fx_Bigger (const char *ver="collide bigger") : _Fx_Big(ver) {}

		using _Fx_Big::type;
		using _Fx_Big::domains;  //use the same domains

		//specializations:
		using _Fx_Big::operator();  //reuses the Fx's of the base class

		void operator() (Rectangle *p, Ellipse   *q) { _collide(p,q, "fRE -modified"); }
		void operator() (Ellipse   *p, Rectangle *q) { _collide(p,q, "fER -added"); }
	};
}
using Collide_Bigger = vane::multi_func <detail::_Fx_Bigger>;




////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
//Bong Collision
struct Polygon : Shape { Polygon (const char *c = "polygon") : Shape(c) {} };

namespace detail {
	struct _Fx_Bong : protected _ShapeCollision
	{
		_Fx_Bong (const char *version="collide bong") : _ShapeCollision(version) {}

		using _ShapeCollision::type;

		using domains = std::tuple <	//different domains than the formers
						std::tuple <Rectangle, Ellipse, Polygon>,
						std::tuple <Rectangle, Ellipse, Polygon>
						>;

		//specializations; a whole new set of Fx's
		void operator() (Rectangle *p, Rectangle *q) { _collide(p,q, "fRR -compatible behavior"); }
		void operator() (Ellipse   *p, Ellipse   *q) { _collide(p,q, "fEE -compatible behavior"); }
		void operator() (Rectangle *p, Ellipse   *q) { _collide(p,q, "fRE -redefined  behavior"); }
		void operator() (Ellipse   *p, Rectangle *q) { _collide(p,q, "fER -redefined  behavior"); }

		void operator() (Rectangle *p, Polygon   *q) { _collide(p,q, "fRP -new");   }
		void operator() (Ellipse   *p, Polygon   *q) { _collide(p,q, "fEP -new");   }
	};
}
vane::multi_func <detail::_Fx_Bong>  collide_bong;	//as a global function



////////////////////////////////////////////////////////////////////////////////
void big_bang(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bang");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>    e;

	collide (&r, &r);
	collide (&e, &e);
	collide (&r, &e);
}
 
void bigger_bang (Collide_func  &collide) {
	puts("---------------------------------------------------------------bigger bang");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>    e;
	VShape::of<Polygon>    p;

	collide (&r, &r);
	collide (&e, &e);
	collide (&r, &e);
	collide (&e, &r);
}

void big_bong(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bong");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>	   e;
	VShape::of<Polygon>	   p;

	collide (&r, &r);
	collide (&e, &e);
	collide (&r, &e);
	collide (&e, &r);

	collide (&r, &p);
	collide (&e, &p);
}


#define	____  puts("\n~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
int main() try
{
	vane::mf_init();

	Collide_Big     collide_big;
	Collide_Bigger  collide_bigger;

	printf("%15s%36s","real args","fx called");
____
	big_bang    (collide_big);
____
	bigger_bang (collide_bigger);
	big_bang    (collide_bigger);
____
	big_bong    (collide_bong);
	bigger_bang (collide_bong);
	big_bang    (collide_bong);
}
catch(const std::exception &e) { printf("\nexception: %s", e.what());  }


/* output **********************************************************************
      real args                           fx called
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bang
(rectangle rectangle) -->    collide big::fRR
(ellipse     ellipse) -->    collide big::fEE
(rectangle   ellipse) -->    collide big::fRE

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------bigger bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified
(ellipse   rectangle) --> collide bigger::fER -added
---------------------------------------------------------------big bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bong
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
(rectangle   polygon) -->   collide bong::fRP -new
(ellipse     polygon) -->   collide bong::fEP -new
---------------------------------------------------------------bigger bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
---------------------------------------------------------------big bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
*/
```


























```c++
//file: runtime-switch-collide-var.cc
#include "vane.h"   //required


struct _Tag_Shape;
using VShape       = vane::var <_Tag_Shape>;
using Collide_func = vane::virtual_func <void(VShape&, VShape&)>;



////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
/* inheritance hierarchy

	_ShapeCollision +- _Fx_Big --- _Fx_Bigger
					 \ _Fx_Bong
*/
////////////////////////////////////////////////////////////////////////////////
struct _ShapeCollision
{
	using type = void (VShape&, VShape&);  //function's type

	template <typename Shape1, typename Shape2>
	void _collide(Shape1 &p, Shape2 &q, const char *tag) {	//util function
		printf("(%-9s %9s) --> %14s::%s\n", p.name, q.name, api_version, tag);
	}

	const char *api_version;
	_ShapeCollision(const char *version) : api_version(version) {}
};




////////////////////////////////////////////////////////////////////////////////
//Big Collision
struct Rectangle  { const char *name = "rectangle"; };
struct Ellipse    { const char *name = "ellipse";   };

namespace detail {
	struct _Fx_Big : protected _ShapeCollision
	{
		_Fx_Big (const char *version="collide big") : _ShapeCollision(version) {}

		using _ShapeCollision::type;	//function's type

		using domains = std::tuple <  	//argument type domains
						std::tuple <Rectangle, Ellipse>,
						std::tuple <Rectangle, Ellipse>
						>;

		//specializations: Fx's
		void operator() (Rectangle &p, Rectangle &q) { _collide(p,q, "fRR"); }
		void operator() (Ellipse   &p, Ellipse   &q) { _collide(p,q, "fEE"); }
		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE"); }
	};
}
using Collide_Big = vane::multi_func <detail::_Fx_Big>;



////////////////////////////////////////////////////////////////////////////////
//Bigger Collision
namespace detail {
	struct _Fx_Bigger : protected _Fx_Big
	{
		_Fx_Bigger (const char *ver="collide bigger") : _Fx_Big(ver) {}

		using _Fx_Big::type;
		using _Fx_Big::domains;  //use the same domains

		//specializations:
		using _Fx_Big::operator();  //reuses the Fx's of the base class

		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE -modified"); }
		void operator() (Ellipse   &p, Rectangle &q) { _collide(p,q, "fER -new");      }
	};
}
using Collide_Bigger = vane::multi_func <detail::_Fx_Bigger>;




////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
//Bong Collision
struct Polygon    { const char *name = "polygon";   };

namespace detail {
	struct _Fx_Bong : protected _ShapeCollision
	{
		_Fx_Bong (const char *version="collide bong") : _ShapeCollision(version) {}

		using _ShapeCollision::type;

		using domains = std::tuple <	//different domains than the formers
						std::tuple <Rectangle, Ellipse, Polygon>,
						std::tuple <Rectangle, Ellipse, Polygon>
						>;

		//specializations; a whole new set of Fx's
		void operator() (Rectangle &p, Rectangle &q) { _collide(p,q, "fRR -compatible behavior"); }
		void operator() (Ellipse   &p, Ellipse   &q) { _collide(p,q, "fEE -compatible behavior"); }
		void operator() (Rectangle &p, Ellipse   &q) { _collide(p,q, "fRE -redefined  behavior"); }
		void operator() (Ellipse   &p, Rectangle &q) { _collide(p,q, "fER -redefined  behavior"); }

		void operator() (Rectangle &p, Polygon   &q) { _collide(p,q, "fRP -new");   }
		void operator() (Ellipse   &p, Polygon   &q) { _collide(p,q, "fEP -new");   }
	};
}
vane::multi_func <detail::_Fx_Bong>  collide_bong;	//as a global function




////////////////////////////////////////////////////////////////////////////////
void big_bang(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bang");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>    e;

	collide (r, r);
	collide (e, e);
	collide (r, e);
}
 
void bigger_bang (Collide_func  &collide) {
	puts("---------------------------------------------------------------bigger bang");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>    e;
	VShape::of<Polygon>    p;

	collide (r, r);
	collide (e, e);
	collide (r, e);
	collide (e, r);
}

void big_bong(Collide_func  &collide) {
	puts("---------------------------------------------------------------big bong");
	VShape::of<Rectangle>  r;
	VShape::of<Ellipse>	   e;
	VShape::of<Polygon>	   p;

	collide (r, r);
	collide (e, e);
	collide (r, e);
	collide (e, r);

	collide (r, p);
	collide (e, p);
}


#define	____  puts("\n~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
int main() try
{
	vane::mf_init();

	Collide_Big     collide_big;
	Collide_Bigger  collide_bigger;

	printf("%15s%36s","real args","fx called");
____
	big_bang    (collide_big);
____
	bigger_bang (collide_bigger);
	big_bang    (collide_bigger);
____
	big_bong    (collide_bong);
	bigger_bang (collide_bong);
	big_bang    (collide_bong);
}
catch(const std::exception &e) { printf("\nexception: %s", e.what());  }


/* output **********************************************************************
      real args                           fx called
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bang
(rectangle rectangle) -->    collide big::fRR
(ellipse     ellipse) -->    collide big::fEE
(rectangle   ellipse) -->    collide big::fRE

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------bigger bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified
(ellipse   rectangle) --> collide bigger::fER -new
---------------------------------------------------------------big bang
(rectangle rectangle) --> collide bigger::fRR
(ellipse     ellipse) --> collide bigger::fEE
(rectangle   ellipse) --> collide bigger::fRE -modified

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
---------------------------------------------------------------big bong
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
(rectangle   polygon) -->   collide bong::fRP -new
(ellipse     polygon) -->   collide bong::fEP -new
---------------------------------------------------------------bigger bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
(ellipse   rectangle) -->   collide bong::fER -redefined  behavior
---------------------------------------------------------------big bang
(rectangle rectangle) -->   collide bong::fRR -compatible behavior
(ellipse     ellipse) -->   collide bong::fEE -compatible behavior
(rectangle   ellipse) -->   collide bong::fRE -redefined  behavior
*/
```


























