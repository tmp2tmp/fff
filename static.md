# \_static<>
&nbsp;  
&nbsp;  
&nbsp;  
When declaring the **type signature of a virtual_func**, &nbsp; 
any polymorphic classe/struct type is considered virtual. &nbsp; 
To treat a polymorphic type as a non-virtual,  
wrap it with **```_static<>```** in the declaration as in:
<pre>using type = void (<strong>_static&lt;Base&&gt;</strong>, Base*);</pre>
: only 'Base*' is considered virtual.
&nbsp;  
&nbsp;

**Consecutive &nbsp;\_static<>**'s &nbsp;can be combined into one:
<div>
<pre style=''>void (<strong>_static&lt;Base&&gt;</strong>, <strong>_static&lt;Base*&gt;</strong>, <strong>_static&lt;Base&&&gt;</strong>, Base*);</pre>
is equivalent to
<pre>void (<strong>_static&lt;Base&, Base*, Base&&&gt;</strong>, Base*);</pre>
</div>
&nbsp;  

<div>
For non-polymorphic or primitive types that are considered non-virtual, &nbsp; <strong>_static&lt;&gt;</strong> has no effects.<br>
<code>void (<strong>_static&lt;int&gt;</strong>)</code> &nbsp; is equivalent to &nbsp; <code>void (int)</code><br>
and also
<pre>void (<strong>_static&lt;Base&&gt;</strong>, int, <strong>_static&lt;Base&&&gt;</strong>, Base*);
<i>//is equivalent to</i>
void (<strong>_static&lt;Base&, int, Base&&&gt;</strong>, Base*);</pre>
</div>
