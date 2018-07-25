# hello_world
&nbsp;  
&nbsp;  
&nbsp;  

<p class='_on_ul'>
Given <b>a set of functions</b>,
determining at runtime which one in the set to call - based on the types of the multiple arguments -
is <strong>multiple dispatch</strong>.
It corresponds to mapping from the possible lists of the argument types to the functions to be called.
Each argument can be assigned the set of the possible types that it can be of. &nbsp; 
In Vane this set of types is named the <strong>type domain of the virtual argument</strong>.
Vane searches the argument type list space confined by the user-given argument type domains,
for the possible functions in the user-given function set, and makes the mapping table at compile time.<br>
Specifying this is through a <strong>co-class</strong> defining three parts:
</p>
<ul>
<li>declaring the type signature of the <b>virtual function</b> as in:   
   <pre class='_code'>using <strong>type</strong> = int(char*, Base1*, Base2&, Base3&&);</pre>
</li>
<li>defining what types each <b>virtual argument</b> can be of, like:   
<pre class='_code'>using <strong>domains</strong> = tuple &lt;domain1, domain2, domain3&gt;;
<i>//where domain1 = tuple &lt;Base1,Drived1,Drived2...&gt;</i>
<i>//      domain2....</i></pre>
</li>
<li>specifing the <b>function set</b> as member operators of the co-class like: &nbsp; &nbsp; &nbsp; <i>//<b>specializations</b></i>
<pre class='_code'>int <strong>operator()</strong> (char*,Base1*,Deived1*,Deive2*){...} 
<i>//and more....</i></pre>
</li>
</ul>
&nbsp;  
&nbsp;  


<p class='_on_ul'>
   Vane has <b>three ways</b> of multi-dispathcing based on the <strong>types of the virtual arguments</strong>.
</p>

- multi-dispaching by **polymorphic** class arguments (by-poly in short)  
  Any argument type of ordinary classes is considered virtual if only it's polymorphic.  
  (to treat it as a non-virtual, wrap it with [**\_static<>**](static.md))  
  slowest
- by **\_virtual<>**-wrapped typed arguments (by-virt)  
  Any argument type of ordinary classes that is wrapped with \_virtual<> is considered virtual if only it's polymorphic.  
  much faster than by-poly.
- by **varg<>**-wrapped typed arguments (by-varg)  
  Any arbitrary (including non-polymorphic or primitive) type of arguments is considered virtual.  
  fastest:  
  - compared with by-virt:
  &nbsp; slightly faster in general (about 15~30%: varies according to the number of virtual arguments),
  	or quite faster when <b>virtual bases</b> are involved (about 40% ~ in most cases - two times).  
  - compared with by-poly:
	  &nbsp; much faster (in general about 5~7 times; &nbsp; and when <b>virtual bases</b> are involved, 10~20 times, mostly more than 15 times).

  But the type domains of the parameters cannot be altered/replaced once established.
&nbsp;  
&nbsp;  
&nbsp;  


#### hello_world's
```c++
//file: hello_world-poly.cc
#include "vane.h"   //required
using std::tuple;


struct Base         { virtual ~Base(){} };	//required: polymorphic base
struct Hello : Base { };
struct World : Base { };


////////////////////////////////////////////////////////////////////////////////
//co-class: 
//     defines the traits & function set for the multi_func
struct Fx
{
	//declares the type signature of the multi_func
	using type = void (const char*, Base&&);
				     // polymorphic Base&& is the virtual parameter
					 //    *, &, && types are supported

	//argument type selectors:  eventually confines the specialized function set
	using domains = tuple<
		tuple <Base, Hello, World> //types that the virtual parameter Base&& can be of
		>;

	//argument-type-specialized functions:
	void operator() (const char *p, Base  &&) { printf("%12s --> Base?\n", p);  } //f0
	void operator() (const char *p, Hello &&) { printf("%12s --> Hello\n", p);  } //f1
	void operator() (const char *p, World &&) { printf("%12s --> World\n", p);  } //f2
};


////////////////////////////////////////////////////////////////////////////////
template<typename Func>
void call_test_baseTyped(Func *func, const char *p, Base &&base) {
	(*func) (p, std::move(base));
}

int main() try 
{
	vane::mf_init();

	vane::multi_func <Fx>    multi_func;
	vane::virtual_func <void (const char*, Base&&)>
                            *virtual_func = &multi_func;

	Fx  fx;  //ordinary function object


	call_test_baseTyped (	      &fx,		     "fx", Hello());
	call_test_baseTyped (	      &fx,		     "fx", World());
	call_test_baseTyped ( &multi_func,   "multi_func", Hello());
	call_test_baseTyped ( &multi_func,   "multi_func", World());
	call_test_baseTyped (virtual_func, "virtual_func", Hello());
	call_test_baseTyped (virtual_func, "virtual_func", World());
}
catch(const std::exception &e) { printf("\nexception : %s", e.what()); }


/* output **********************************************************************
        func --> Base?
        func --> Base?
  multi_func --> Hello 
  multi_func --> World 
virtual_func --> Hello 
virtual_func --> World 
*/
```

```c++
//file: hello_world-virt.cc
#include "vane.h"   //required
using vane::virtual_;

struct Base         { virtual ~Base(){} };	//required: polymorphic base
struct Hello : Base { };
struct World : Base { };


////////////////////////////////////////////////////////////////////////////////
//co-class: 
//     defines the traits & function set for the multi_func
struct Fx
{
	//function signature of the multi_func
	using type = void (const char*,  const virtual_<Base>*);
				     // virtual_<Base>* : virtual parameter
					 //    *, &, && types are supported

	//argument type selector:  eventually confines the specialized function set
	using domains = std::tuple<
		std::tuple <Base, Hello, World> //types which the virtual parameter `virtual_<Base>*' can be of
		>;

	//argument-type-specialized functions:
	void operator() (const char *p, const Base  *) { printf("%12s --> Base?\n", p);  } //f0
	void operator() (const char *p, const Hello *) { printf("%12s --> Hello\n", p);  } //f1
	void operator() (const char *p, const World *) { printf("%12s --> World\n", p);  } //f2
};


////////////////////////////////////////////////////////////////////////////////
template<typename Func>
void call_test_baseTyped(Func *func, const char *p, const virtual_<Base> *babe) {
	(*func) (p, babe);
}

void call_test_baseTyped(Fx *func, const char *p, const Base *base) {
	(*func) (p, base);
}

int main() try 
{
	vane::mf_init();

	vane::multi_func <Fx>    multi_func;
	vane::virtual_func <void (const char*, const virtual_<Base>*)>
		                    *virtual_func = &multi_func;

	Fx  fx;  //ordinary function object


	virtual_<Base>::of<Hello>  hello; 
	virtual_<Base>::of<World>  world;

	call_test_baseTyped (		  &fx,		     "fx", &hello);
	call_test_baseTyped (	      &fx,		     "fx", &world);
	call_test_baseTyped ( &multi_func,   "multi_func", &hello);
	call_test_baseTyped ( &multi_func,   "multi_func", &world);
	call_test_baseTyped (virtual_func, "virtual_func", &hello);
	call_test_baseTyped (virtual_func, "virtual_func", &world);
}
catch(const std::exception &e) { printf("\nexception : %s", e.what()); }


/* output **********************************************************************
          fx --> Base?
          fx --> Base?
  multi_func --> Hello
  multi_func --> World
virtual_func --> Hello
virtual_func --> World
*/ 
```

```c++
//file: hello_world-var.cc
#include "vane.h"   //required
using std::tuple;
#define	____	puts("---------------------------------------------------------------");


struct Hello { };
struct World { };

using var = vane::var<>;	//anonymous var<>

////////////////////////////////////////////////////////////////////////////////
//co-class: 
//     defines the traits & function set for the multi_func
struct Fx
{
	//function signature of the multi_func
	using type = void (const char*, const var&);
				     // var& : virtual parameter of type vane::var<>
					 //    *, &, && types are supported

	//argument type selectors:  eventually confines the specialized function set
	using domains = tuple<
		tuple <Hello, World, int, std::string> //types which the virtual parameter `var&' can have
		>;

	//argument-type-specialized functions:
	void operator() (const char *p, const Hello&)         { printf("%21s --> Hello \n", p);  } //f0
	void operator() (const char *p, const World&)         { printf("%21s --> World \n", p);  } //f1
	void operator() (const char *p, const int &i)         { printf("%21s --> %d\n", p, i);   } //f2
	void operator() (const char *p, const std::string &s) { printf("%21s --> %s\n", p, s.c_str());  } //f3
};


////////////////////////////////////////////////////////////////////////////////
void call_mf_uniformTyped(
	vane::multi_func <Fx> *func,   const char *p, const var &v)
{
	(*func) (p, v);
}

void call_vf_uniformTyped(
	vane::virtual_func <void (const char*, const var&)>  *func,
	const char *p,  const var &v)
{
	(*func) (p, v);
}


int main() try 
{
	vane::mf_init();

	vane::multi_func <Fx>    multi_func;
	vane::virtual_func <void (const char*, const var&)>
		                    *virtual_func = &multi_func;

	Fx  fx;  //ordinary function object


	var::of<Hello>	hello; 
	var::of<World>	world;
	var::of<int>			number{3};
	var::of<std::string>	string{"ways of multi-functioning"};

	//fx cannot be called with arguments uniform-typed
											  fx ("fx (hello )", hello);
											  fx ("fx (world )", world);
											  fx ("fx (number)", number);
											  fx ("fx (string)", string);
____
	call_mf_uniformTyped ( &multi_func,   "multi_func (hello )", hello);
	call_mf_uniformTyped ( &multi_func,   "multi_func (world )", world);
	call_mf_uniformTyped ( &multi_func,   "multi_func (number)", number);
	call_mf_uniformTyped ( &multi_func,   "multi_func (string)", string);
____
	call_vf_uniformTyped (virtual_func, "virtual_func (hello )", hello);
	call_vf_uniformTyped (virtual_func, "virtual_func (world )", world);
	call_vf_uniformTyped (virtual_func, "virtual_func (number)", number);
	call_vf_uniformTyped (virtual_func, "virtual_func (string)", string);
}
catch(const std::exception &e) { printf("\nexception : %s", e.what()); }


/* output **********************************************************************
          fx (hello ) --> Hello 
          fx (world ) --> World 
          fx (number) --> 3
          fx (string) --> ways of multi-functioning
---------------------------------------------------------------
  multi_func (hello ) --> Hello 
  multi_func (world ) --> World 
  multi_func (number) --> 3
  multi_func (string) --> ways of multi-functioning
---------------------------------------------------------------
virtual_func (hello ) --> Hello 
virtual_func (world ) --> World 
virtual_func (number) --> 3
virtual_func (string) --> ways of multi-functioning
*/ 
```


