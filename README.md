# [Vane](https://tmp2tmp.github.io/fff/)

**---  A multiple-dispatch library for C++14 ---  
     + meta-programming facilities**  
&nbsp;  
&nbsp;  
&nbsp;  
### Abstract
Vane implements multiple-dispatch in three ways  
based on the runtime types of the mutiple arguments of
- intact ordinary polymorphic classes or
- wrapper classes of polymorphic classes that have the same base or
- wrapper classes of arbitrary (including non-polymorphic or primitive) types  

None of them requires the existing class code to be modified.  
Vane is easy to use, requires no chaotic boilerplate devices to be put on the classes.  
**vane::multi\_func** itself is polymorphic &nbsp;  
; &nbsp;  you can change the whole behavior of a 'multi\_func' at runtime  
&nbsp; &nbsp; by simply replacing it with another instance of a different multi\_func class  
&nbsp; &nbsp; just as you can do with ordinary polymorphic class instances.  
Vane also includes meta-programming facilities that make meta-programming much easier.
&nbsp;  
&nbsp;  

****

### Introduction
[Implementing multiple dispatch - basic syntax](hello_world.md)  
[Shape collision](collide.md)  
&nbsp;  

### Features
- Simple & easy syntax : &nbsp; No need to attach complicated distracting unwieldy gears to existing class code. There are no messy macro things.
- [Function call resolutoin is supported](call_resolution.md) : &nbsp; 
  Runtime function call resolutoin based on the inheritance hierarchies of the argument types is supported.
- Arbitrary number of virtual/non-virtual arguments can be arbitrarily mixed.  
  &nbsp; Three sorts of virtual parameters of Vane can also be freely mixed.
- [Virtual & mutiple inheritance (of virtual argument types)](diamond.md) are supported.
- [Covariant return types are supported.](covariant_return_types.md)
- Virtual functions are [implemented as function `objects'](oop_featured.md).
- Virtual function objects are polymorphic:  
  [can be replaced at runtime](replacing_virtual_functions.md) switching the whole set of specializations.

&nbsp;  

### More Usages
<!--
- [utility &nbsp; for std::shared_ptr](make_shared.md)  
- [std::shared_ptr &nbsp; utility](make_shared.md)  
- [```make_shared utility```](make_shared.md)  
- [using with &nbsp; std::shared_ptr](make_shared.md)  
- forcing static dispatch / calling base implementations
-->
- [runtime errors](runtime_errors.md)
- [make_shared &nbsp;utility](make_shared.md)  
- [vt-vars](vt-vars.md)  
&nbsp;  


### [Resources & Referrences](resources.md)



