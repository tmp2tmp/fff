# Virtual functions as function objects
&nbsp;  
&nbsp;
### Open methods as function objects

Vane implements open multi-methods as function objects.  

Thanks to C++, &nbsp;C++ allows overloading function call operators,
and this feature alllows implementing free-standing functions as member functions - i.e.
	member functions that have the interfaces of non-member funtions and can be used 
	in the contexts where ordinay functions of the same signatures can be used.

We don't have to necessarily be reluctant to implement open methods as objects.
Implementing multi-methods as function objects rather has advantages over implementing as ordinary functions:
- Specializations, being implemented as member functions, can be confined and managed more conveniently 
  than being associated to ordinary functions of global/static symbols.
- Instance specific data can be associated to each function object
  while ordinary functions can have only common global/static data.
- Specialized function sets can be defined [reusing existing code easily by inheritance](replacing_virtual_functions.md).
- Forcing static dispatch can easily be done using the inherited static dispatch interface of the base class: &nbsp;
  this also can be used, for [example](runtime_errors.md),
  to check at compile time whether or not a specific combination of argument types
  has an available specialization.

&nbsp;  

### Double virtual
<p>
C++ directly supports runtime dispatch on a single argument via C++ <code><b>virtual</b></code> functions.
&nbsp; Being that the functions - whose calls are dispatched at runtime
	based on the dynamic types of single one of the arguments - are virtual,
&nbsp; it's consistent with it that the functions - whose calls are dispatched at runtime based on the dynamic types
	of two or more of the arguments - are called virtual
: &nbsp; vane::multi_func is virtual on multiple arguments, defining a set of specializations for the arguments.
</p>

And a vane::multi\_func itself - as a parameter passed as `this' pointer - is polymorphic of specialization set:
&nbsp; the virtual function object of Vane [can be replaced at runtime](replacing_virtual_functions.md) 
switching the whole set of specializations of it.


