---
title: hello world #zzzzz
---

# zzzzzzzzzzzzzzzzzzzzzzzzzzzz

---

---

title: zzzz #aaaaa  
code-width: 1000px

```c++
//file: 1--hello_world.cc
#include "vane.h"   //required
#include <stdio.h>


/* inheritance hierachy:
      Speak        What 
      /  \         /  \
  Hello  Open  World  Sesame
*/
struct Speak		{ virtual ~Speak(){} };	//polymorphic base is required
struct Hello : Speak	{ };
struct Open  : Speak	{ };


struct What		{ virtual ~What(){}  };	//polymorphic base is required
struct World : What	{ };
struct Sesame: What	{ };


////////////////////////////////////////////////////////////////////////////////
struct Fx_required_parts
{
    using type    = void (const char*, Speak*, What&);	/*  declares the virtual function signatue
							    virtual params : {Speak*,What&} ;  two polymorphic params
							*/
    using domains = std::tuple<			/* declares the valid types of arguments */
	    vane::_domain<Speak, Hello, Open>,	//1st virtual argument is valid  when: it is a Speak or a Hello or a Open
	    vane::_domain<What,  World, Sesame>	//2nd virtual one      is valid  when: it is a What or a World or a Sesame
	  >;
};

//Fx: defines a set of functions
struct Fx : Fx_required_parts
{
    void operator()(const char *p, Speak*, What  &) { printf("%10s --> speak_what??\n", p);  } //f0
    void operator()(const char *p, Hello*, World &) { printf("%10s --> hello_world \n", p);  } //f1
    void operator()(const char *p, Open *, Sesame&) { printf("%10s --> open_sesame \n", p);  } //f2
};


////////////////////////////////////////////////////////////////////////////////
template<typename Func>
void process(Func *func, const char *p, Speak *s, What &w) //polymorphism tester
{
    (*func) (p, s, w);
}

int main() try 
{
    Fx                     func;        //ordinary function object
    vane::multi_func<Fx>   multi_func;

    vane::virtual_func< void (const char*, Speak*, What&) >  *vfunc
			= &multi_func;	//a multi_func is a virtual_func

    Hello  hello; 
    World  world;
    Open   open;
    Sesame sesame;

    process (&func,       "func",       &hello, world);
    process (&multi_func, "multi_func", &hello, world);
    process (vfunc,       "vfunc",      &open,  sesame);

}
catch(const std::exception &e) {
    printf("\n\nexception : %s", e.what());
}

/* output **********************************************************************
      func --> speak_what??
multi_func --> hello_world 
     vfunc --> open_sesame 
*/
```


