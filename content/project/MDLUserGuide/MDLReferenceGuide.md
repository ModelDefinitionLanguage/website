MDL Language Reference
======================

*The aim of this section is to provide the technical aspects behind the
MDL language: documenting its syntax and semantics in detail. New users
may wish to read the sections on Data Object, Parameter Object, Model
Object, Task Properties Object and Model Object Group, and explore the
MDL implemented in the Use Case examples first in order to familiarise
themselves with how MDL is used to define models. The information in
this section may be of more interest to users who are writing their own
models using MDL and wish to know the detail of syntax and grammar
implemented in the MDL-IDE. *

Like many computational languages MDL has two layers. First is the
syntactic layer, or core, that defines how the words and symbols of the
language are combined together in meaningful ways. This is like the
building blocks of the language, it’s vocabulary, punctuation and
grammar. Building on this foundation then is the second, semantic layer
of MDL. This is where the meaning of the language is defined and how the
building blocks of the core are used to create a language that describes
pharmacometric models.

This organisation is reflected in this section, which starts with the
description of the language core, followed by an explanation of MDL’s
type system before moving on to the semantics layer of MDL.

1.  <span id="_Toc459299566" class="anchor"></span>**Core syntactic
    elements**

The core units of the language are described here from the bottom up.
Starting with the language keywords, through the definitions of
expressions and statements until we reach the highest level of
organisation in MDL, the object.

1.  <span id="_Toc459299567" class="anchor"></span>**Keywords**

The keyword names are reserved and cannot be reused elsewhere, for
example as attribute or variable names. The keywords in MDL have
deliberately been kept to a minimum and at present there are 16. They
are:

> as, if, else, elseif, ln, exponentiale, false, in, inf, is, otherwise,
> pi, piecewise, set, then, when, withCategories, true

include a keyword that are not currently used, but which is reserved for
future versions of MDL:

> ordered

1.  <span id="_Toc459299568" class="anchor"></span>**Variable names**

Variables names in MDL must conform to the following rules:

-   There are no reserved variable names in MDL

-   Variable names may only contain letters or numbers and ‘\_’ and must
    start with a letter or ‘\_’ character.

-   As MDL is a case sensitive language the case of letters matters in
    variable names so ‘t’ is a different variable to ‘T’.

In technical terms a variable name must comply with the following
regular expression:

('a'..'z'|'A'..'Z'|'\_')('a'..'z'|'A'..'Z'|'\_'| '0'..'9')\*

An addition constraint not reflected in the above regular expression is
that the variable name also cannot begin with ‘MDL\_\_’. This prefix is
reserved for internal used by code generators that may be used to
convert MDL to other languages and may need to create synthetic variable
names.

1.  <span id="_Toc459299569" class="anchor"></span>**Literals**

Values such as numbers and strings etc can be written explicitly in MDL.
Technically such values are referred to as literals. MDL supports the
following types:

-   Vector: [0, 1, 2, 25.0]

-   Matrix: [[0, 20, 2; 23.5, 20, 1]]

-   Strings: “a string“

-   Integers: 99, 22, 0, -1, -477

-   Real: 99.9, -0.473, 9e-2, -0.3424e5

-   Boolean: true, false

Note that in the current MDL a vector containing only integer values
will be regarded as a vector of type integer (not of type real). To
ensure that the vector is of type real, the user may need to specify one
of the numbers as a real value e.g. 25.0.

1.  <span id="_Toc459299570" class="anchor"></span>**Expressions**

An expression in MDL is primarily used to express mathematical concepts
and evaluate mathematics. Expression are divided into two types, Boolean
and numerical. The former is an expression that evaluates to a Boolean
(True or False), while the latter evaluates to a Real number. Examples
are:

x \> 5 && y \<= 0

x – 5 \* 23

x\^(2\*x/z)

Expressions can contain mathematical functions too:

sin(x)

ln(x + y) + ln(22)

Some functions can use named arguments:

x + func(arg1=1, arg2=3)

In MDL conditional statements are also expressions:

x \* if(y \> 22) then 300 \* a else 1

And they can use categorical variables:

x \* if(sex == sex.female) then 1 else 0

1.  **Numerical and Boolean Expression**

Expressions are built up using operators that either take one or two
operands:

binary\_op := \<operand\> \<operator\> \<operand\>

unary\_op := \<operator\> \<operand\>

Logical expressions are formed using combinations of Boolean operators
(&&, ||, !) or comparison operators (\<, \>, \<=, \>=, !=, ==).
Numerical expressions use the standard mathematical operators (+, -,
/, \*, \^, %). These are shown below.

  ----------------------- ------------ --------------- ---------------- ------------
  **Operator**            **Symbol**   **Left Type**   **Right Type**   **Result**
  Logical AND             &&           Boolean         Boolean          Boolean
  Logical OR              ||           Boolean         Boolean          Boolean
  Less than               \<           Real            Real             Boolean
  Greater than            \>           Real            Real             Boolean
  Less than or equal      \<=          Real            Real             Boolean
  Greater than or equal   \>=          Real            Real             Boolean
  Equal                   ==           Real            Real             Boolean
  Not equal               !=           Real            Real             Boolean
  Power                   \^           Real            Real             Real
  Multiplied by           \*           Real            Real             Real
  Divided by              /            Real            Real             Real
  Modulo (remainder)      %            Real            Real             Real
  Add                     +            Real            Real             Real
  Subtract                -            Real            Real             Real
  Negation (unary)        -                            Real             Real
  Positive (unary)                                     Real             Real
  ----------------------- ------------ --------------- ---------------- ------------

The operators have the same operator precedence you would expect in a
standard mathematical equation. In the table below operator precedence
is shown, ordered from highest to lowest.

  Operators        Precedence
  ---------------- ---------------
  Unary            + - !
  Power            \^
  Multiplicative   \* / %
  Additive         + -
  Relational       \< \> \<= \>=
  Equality         == !=
  Logical AND      &&
  Logical OR       ||

1.  **Variable references**

A symbol name used in an expression is treated as a reference to a
symbol defined elsewhere in the same object.

> a = 10
>
> b = 10 + a

The above code snippet shows how the variable reference ‘a’ in the
expression refers to the definition of variable “a” which is initialised
to 10. This is intuitive as is the fact the expression evaluates to 20.
However, references to categorical variables behave slightly
differently. A reference to it takes two forms:

1.  A reference to the variable itself, e.g. sex

2.  Or a reference to a category value, e.g. sex.female

Note that the second form uses a qualified name based on a combination
of the categorical variable and its value:

> \<categorical variable\>.\<category value\>

The meaning of a reference to a category variable is self-evident,
however, a reference to the categorical variable itself is less so.
Consider the following:

sex withCategories {male, female}

sex == sex.female

In essence, the reference to the categorical variable ‘sex’ is referring
to the category value held by ‘sex’. If ‘sex’ holds the value
‘sex.female’ then this expression evaluates to true. This enables us to
write conditions expressions (see below) like this:

if(sex == sex.female) then 1 else 0

1.  **Type Specifications**

As is described below MDL (see Type System) infers the type of parameter
or variable where it can. However, when a variable is not initialised
the author must tell MDL what type to expect. The type specification
syntax enables this. Simple examples are below:

A::real

B::string

C:: int

D::pdf

E::Boolean

The basic syntax is ‘::’ followed by the type name. Vectors, and
matrices require more complex handling:

F::vector[::int] \# vector of ints

G::matrix[[::string]] \# matrix of strings

H::matrix[[ ::vector[boolean] ]] \# matrix of vectors of booleans

As you can see vector types need a type specification to define its
element type as does the matrix. In order to reduce typing some type
specifications can be omitted in which case the type is assumed to be
Real:

A \# type real

F::vector \# vector of reals

G::matrix \# matrix of real

H::matrix[[ ::vector ]] \# matrix of vectors of reals

1.  <span id="_Ref459282240" class="anchor"></span>**Conditional
    > expressions**

Sometimes it is useful for an expression to evaluate to different values
depending on some arbitrary criteria. In a mathematical expression this
is handled by a piecewise function:

$$f\left( x \right) = \left\{ \begin{matrix}
 - 1,\ \ \& x < 0 \\
1,\ \ \&\mathrm{\text{otherwise}} \\
\end{matrix} \right.\ $$

In MDL we have a directly equivalent peicewise construct:

piecewise {{ -1 when x \< 0; otherwise 1 }}

This can take more than 1 condition as you would expect:

piecewise {{ -1 when x \< 0; 1 when x \> 0; otherwise 0 }}

$$f\left( x \right) = \left\{ \begin{matrix}
 - 1,\ \ \& x < 0 \\
1,\ \ \& x > 0 \\
0,\ \ \&\mathrm{\text{otherwise}} \\
\end{matrix} \right.\ $$

Or just condition and no otherwise clase:

Expressioens with more than one condition are possible:

piecewise {{ -1 when x \< 0; 1 when x \>= 0 }}

which is equivalent to:

$$f\left( x \right) = \left\{ \begin{matrix}
 - 1,\ \ \& x < 0 \\
1,\ \ \& x \geq 0 \\
\end{matrix} \right.\ $$

<span id="_Ref435810248" class="anchor"></span>**Equation 1**

A conditional expression should cover all possible conditions in order
to prevent the generation of an undefined value, which will result in a
runtime error. MDL does not enforce this, but to help ensure this it
requires the writer provide at least two clauses. Ideally the last
clause will be an ‘otherwise’ as this guarantees all conditions are
covered, but in cases such as **Equation 1** this clearly is not
necessary so this is not mandatory.

A related rule is that the conditions should also not define overlapping
domains. In other-words only one condition can be true for any given set
of values. This is illustrated by the code snippet below which breaks
this rule:

piecewise {{ -1 when x \< 0; 1 when x \< 2; otherwise 0 }} \# bad
conditions!

The first two conditions can be true if x is -1 for example, which makes
the correct evaluation of this expression impossible (remember that the
written order of the conditions is not meaningful). This last rule is
very important, because the order that the conditions are evaluated
cannot be guaranteed and so the result of the above expression may not
be as expected. At the moment MDL does not check this so the author must
ensure that all conditions are independent.

Complementary to the piecewise expression is the if/else expression.
Note that this is an *expression* not a statement so it is evaluated as
part of a mathematical expression and is not used to control which
statements are evaluated, as it would be in an imperative language such
as R. The syntax is as you would expect:

if(x \< 0) then -1 elseif(x \> 0) then 1 else 0

Here the order of evaluation is significant. So ‘x \< 0’ is evaluated
before ‘x \> 0’. This means that the above example in an if/else
expression is unambiguous:

if(x \< 0) then -1 elseif(x \< 2) then 1 else 0

as the condition ‘x \< 2’ will never be evaluated. If/else expressions
can be nested too:

if(x \< 0) then -1 else if(x \> 0) then 1 else 0

which is functionally equivalent to the ‘elseif’ non-nested equivalent.

1.  <span id="_Ref459282277" class="anchor"><span id="_Ref437350273"
    > class="anchor"></span></span>**Conditional Lists**

A varant of the condition expression is the conditional list. This
allows the conditional assignment to a variable of a list. For example:

A : if(B \> A) then { type is linear, pop=POP\_A, ranEff = ETA\_A }\
 else { type is general, grp=GRP\_A, ranEff = ETA\_A }

This assign one or other of the lists to A depending on the condition.
This follows the same rules as for conditional expression, but the list
being assigned must have the same List Super-type (see Type System,
below). In the above example both lists have a super-type of
‘IndivAbstractList’ so the list assignment ‘:’ is valid.

A piecewise variant is also permitted:

A : piecewise {{

{ type is linear, pop=POP\_A, ranEff = ETA\_A } when B \> A;\
 otherwise { type is general, grp=GRP\_A, ranEff = ETA\_A }\
 }}

1.  <span id="_Ref459230084" class="anchor"></span>**Functions**

Functions take two forms in MDL. Simple and named argument functions.
The latter is equivalent to a standard mathematical function such as
sin(a). The argument order is significant and all arguments are
required. So for example:

logx(y, b)

defines a logarithm of y to base b. Swapping the arguments around would
change the meaning accordingly.

The second form as one might expect takes named arguments and
consequently the order of arguments is not important and some arguments
are optional. For example:

foo(arg1 = val1, arg2 = val2, arg3 = val3)

foo(arg1 = val1, arg2 = val2)

call the same function, but ‘arg3’ can be omitted. Typically the
function will use a default value, but the exact behaviour is determined
by the definition of the function. Arguments can be constrained so that
only specific combinations are permitted as below:

foo(arg1 = val1, arg2 = val2, arg3 = val3)

foo(arg1 = val1, arg4 = val4, arg5 = val5)

foo(arg1 = val1, arg4 = val4, arg2 = val2) \# invalid

foo(arg1 = val1, arg5 = val5) \# invalid

Here the combinations of arg2, arg3 and arg4 and arg5 are permitted, but
combination such as arg2 and arg4 are not. Likewise, arg5 cannot be used
unless arg4 is also provided. This gives a lot of flexibility in the
parameterisation of functions.

1.  <span id="_Ref452280144" class="anchor"></span>**Function
    > references**

When a function is invoked a reference to that function’s definition is
invoked and the appropriate value returned. For example, in the
expression:

> a = ln(10)

the symbol ‘ln’ is a reference to the function defining the natural log.

However, in some circumstances it is desirable to refer to a function,
but not to evaluate it immediately, for example when referring to a
function to be used for interpolation of data. In these circumstances a
‘&’ precedes the function name and the function arguments are omitted.
For example:

Obs : { use is dv, variable = Y, interp=&linear }

Here the ‘interp’ attribute is assigned a reference to the ‘linear’
function definition. This function can then be invoked whenever
interpolation between two data-points is required.

1.  **Vectors**

Vectors are defined within in square brackets and can take numbers,
strings, Boolean, variable references, logical and numerical
expressions. Vectors must contain items of consistent type. The
exception is when the vector contains numbers and variable references of
numerical types (see type system below for more information). Some
examples are below:

[ a, b-1, 1.0 \* 3\^5, 2.0\^3.5, inf ]

[ “str”, “a str too”, “a”, “str” ]

[ true, false, true == false ]

Note that all elements must be populated. So the following is *not*
permitted:

[ 3, , 3 ] \# forbidden

Vectors can also be nested, but again the vectors must have the same
type:

[ [ 22, 45 ], [ 67, 89 ] ]

Vectors may be assigned to a variable or parameter and then used in
other expressions:

A = [ 1.1, 2.2, 3.3, 4.4 ]

B = A \# B is assigned the contents of variable A

B = A[1] \# B is a scalar and assigned the first value in A: 1.1

B = A[2:3] \# B is vector and assigned elements 2 and 3 in A: [2.2, 3.3]

This illustrates how vector indexing works. The first element is at
position 1 and the last index is equal to the length of the vector. If a
single index value is used in square brackets, then a single scalar
value is returned. As can be seen, MDL permits ranges of values to be
specified, using a ‘:’ operator. This range operator is only permitted
for vector and matrix indexing and ensures that the indexed vector
returns a vector. This has a useful side effect when a vector containing
a single vector is required:

B = A[3:3]

this returns a vector containing just element 3 of vector A, i.e. [3.3].

Vector index ranges can be omitted too. This indicates that indexing
should start at the beginning or the end of the vector. For example:

A[:3] \# first element to element 3

A[2:] \# element 2 to the last element

A[:] \# first to last elements, i.e., the complete vector

Note that the last index is equivalent to the vector reference:

B = A[:]

B = A \# equivalent to above

1.  **Matrices**

Matrices are very similar in their semantics to vectors and are
indicated syntactically by the double square bracket with each row
terminated by a ‘;’:

M = [[ 1, 2, 3; 3, 4, 6 ]]

Like vectors all cells in a matrix literal must be populated and in
addition all rows must have the same number of columns:

[[ a, b, c;\
 d, e; \# forbidden - inconsistent num columns\
 g, h, i ]]

[[ a, b, c;\
 d, , f; \# forbidden - missing value\
 g, h, i ]]

Matrix indexing is similar to vector indexing and uses the same square
bracket operator. Instead, an index for the row and column are
specified:

M = [[ 1, 2, 3; 4, 5, 6 ]]

N = M[2, 3] \# a scalar value of 6

The first index indicates the row number and the second the column. As
with vectors indexing starts at 1. Range operators behave in the same
way as vectors, but in this case they always return a matrix:

N = M[2, 2:3] \# returns matrix: [[5, 6]]

N = M[1:2, 2] \# returns matrix: [[2; 5]]

N = M[1:2, 1:2] \# returns matrix: [[1, 2; 4, 5]]

N = M[1:1, 2:2] \# returns matrix: [[2]]

Empty ranges behave as with vectors, indicating the start or end of a
row or column. Row or column indexes can also be empty to indicate all
rows or columns:

N = M[, 1] \# returns matrix: [[1; 4]]

N = M[, 1,2] \# returns matrix: [[1, 2; 4, 5]]

N = M[2,] \# returns matrix: [[4, 5, 6]]

N = M[,] \# equivalent to N = M

1.  **Sublist Expressions**

The sublist is a convenient way of grouping together related pieces of
information together in an expression. In its basic form the sublist is
a set of attributes combined together as below:

> { att1 = val1, att2 = val2, … }

Sublists which have different attribute sets are regarded as different.
In fact sublists are fully fledged types in MDL (see section ) so
sublists with different attributes sets are distinct types. This is
illustrated in the example below where th sublists correspond to sublist
types a and b:

> { att1 = val1, att3 = val3 } \# sublist a
>
> { att2 = val2, att4 = val4 } \# sublist b

This means that an argument in a function (see section for more detail
on argument typing) can require a particular sublist type. In this way
type system can ensure that only the valid type system is used. For
example if an attribute ‘foo’ expects a sublist of type a, then the
following validity is enforced:

> foo = { att1 = val1, att3 = val3 } \# sublist a - valid
>
> foo = { att2 = val2, att4 = val4 } \# sublist b - invalid

How is the type of a sublist determined? Very simply all sublists must
have a unique set of attributes. The MDL processor takes the combination
of attributes written and matches them to a sublist in its dictionary of
sublist types.

Sublist can also restrict the set of permitted attributes, just like you
can with named function arguments (see section 9.1.6.3). The details of
how this is carried out is described in detail below (section 0). The
sublist can be contained in a vector as well. For example:

> [
>
> { att1 = val1, att2 = val2},
>
> { att1 = val3, att2 = val4},
>
> ]

A Sublist can be used by any attribute, function argument or property.
Below is an example of its used in a function. It shows how the sublist
provides a convenient way to group together sets of covariate and fixed
effect parameters in a linear individual parameter definition:

> ln(CL) = linear( trans is ln, pop = POP\_CL, fixEff = [\
>  {coeff = BETA\_CL\_WT, cov = logtWT},\
>  {coeff = POP\_FCL\_FEM, catCov = SEX.female },\
>  {coeff = BETA\_CL\_AGE, cov = tAGE}\
> ],\
> ranEff = [ETA\_CL] )

1.  **Variable selection expression**

MDL provides support for conditionally assigning values in a variable to
another variable using a special syntactic structure called a *variable
selection expression*. This construct is often used in a list (described
in section ) and is best illustrated by an example:

CMT: { use is cmt }

AMT: { use is amt,\
 define={1 in CMT as GUT, 2 in CMT as CENTRAL} }

The value mapping syntax is used with the ‘define’ attribute. It can be
read as “if value is ‘1’ **in** variable ‘CMT’ then select **as**
variable ‘GUT’, if value is 2 **in** ‘CMT’ then select **as** variable
‘CENTRAL’”. The semantics of what selection means can vary depending on
context. In the above example, which is valid MDL, the lists AMT and CMT
each define a column in a dataset with the AMT column values being
assigned to the variable selected by the corresponding value in the CMT
column.

The syntax for the variable selection expression is as follows:

> { \<test value\> in \<qry var ref\> as \<select var ref\>,\
>  \<test value\> in \<qry var ref\> as \<select var ref\>, … }

The query (qry) and selection (select) variable references cannot be the
same, the same query variable must be used throughout the expression and
the test value must be a numerical value.

1.  **Category value selection expression**

Related to the variable selection expression is the category value
selection. This carries out a similar function, but for category values,
in this case a particular category value is selected when a given
expression is matched. For example:

> { sex.female when 0, sex.male when 1 }

shows how sex.female is selected when the value is 0, and sex.male when
it is 1. Typically this expression is used in conjunction with a
variable selection expression where the value in a list is to be mapped
to a categorical variable. This is illustrated below where the values in
a DV column definition are mapped to either a variable or a set of
category values:

> DVID: { use is dvid }
>
> DV: { use is dv,\
>  define={1 in CMT as GUT,\
>  2 in CMT as { Outcome.dead when 0, Outcome.alive when 1}\
>  }\
> }

The basic syntax is:

> { \<category.value\> when \<test value\>,\
>  \<category.value\> when \<test value\>, … }

The test value should be a numerical value and the category values
belong to the same category.

1.  <span id="_Ref437348606" class="anchor"><span id="_Toc459299571"
    class="anchor"></span></span>**Attributes, Arguments, Properties and
    Values**

Attributes in lists, arguments in functions and properties all behave in
the same way in MDL. For the sake of simplicity this description will
refer to attributes, but this should be understood as a synonym for
argument.

An attribute is simply an identifier that is associated with a value.
That value can be of any valid type and is usually assigned to the with
the ‘=’ operator. For example:

> att1 = 0
>
> att2 = true
>
> att3 = “string val”
>
> att4 = { a = 0, b = 3}\
>  \# sublist
>
> att5 = [0, 2, 3]
>
> att6 = [1.5, inf, x]
>
> att7 = { 1 in CMT as foo }\
>  \# a mapping
>
> att8 = varRef

In addition an attribute can be assign a value from a controlled
vocabulary of options, called a built-in enumeration. Because the MDL
parser needs help to distinguish these option names from a variable
reference it is necessary to use a different assignment operator.
Therefore, we use the keyword ‘is’ to indicate that an attribute has
been assigned an option. For example:

> att8 is anOption

A typical usage of a built-in enumeration is to define the key value of
a list, for example:

c1 : { use is id }

c2 : { use is adm, variable = D }

c3 : { use is idv }

Any name can be used as an attribute name as long as it is not a
language keyword and it is a valid variable name.

In the previous version of MDL an attribute expecting a vector type
would have to be written thus:

A : { foo = [ z ] } \# foo expectes a vector

even when there was only one element in the vector. To avoid the
additional typing and to improve readability it can now be written as:

A : { foo = z }

The ‘foo’ attribute still expects a vector, so behind the scenes MDL
converts this value to [ z ].

1.  <span id="_Toc459299572" class="anchor"></span>**Statements**

The statement is the core of MDL and comes in several forms. However,
they all have the following characteristics:

1.  A statement can be split over any number of lines. The parser
    detects the start and end of the statement based on its context.

2.  Sometimes it is helpful to the user to indicate where a statement
    starts and ends in which case an optional ‘;’ character can be used.
    Note this is completely optional.

The different statement types are below.

1.  **Equation definition**

This defines a variable using a notation equivalent to a mathematical
equation. It can have three forms:

1.  x = \<expression\>\
    x = 2 + 5 / ln(22)

2.  fn(x) = \<expression\>, where x is transformed by a function\
    ln(x) = 2 + r

3.  x, where the variable x is declared but not initialised.\
    x

The symbol (parameter or variable) it defines always has a type of Real.

In example 2 above, a transformation was used on the variable x. It is
important to note that the variable x can be used later (without the
transformation) and it is implied that a back-transformation will have
been applied. The user need not explicitly back-transform the parameter
in the MDL code.

1.  **Category definition**

A category definition creates a variable that has two roles. First it
groups together a set of categories that belong to this variable and
second it holds a value that is one of these categories. Exactly what
this means is explained below, but here is the syntax of the category
definition.

> X withCategories { cat1, cat2, … catN }

The definition can create any number of categories but must have at
least one. A simple example is:

> sex withCategories { male, female }

The category values are specific for each category variable so the
following is permitted:

> sex1 withCategories { male, female }
>
> sex2 withCategories { male, female }

The categorical variable always has a type of Enum.

1.  <span id="_Ref436338259" class="anchor"></span>**List Definition**

The list is a way of associating attributes and values with a variable.
In many ways it is similar to a class seen in an Object Oriented
programming language. The list has the following syntax:

lst : { keyAtt (=|is) \<value\>, att (=|is) \<value\>, … }

Note that a list holds a specific set of attributes. The exact set is
determined by the key attribute used, it’s value and the block
containing the list. This is illustrated below.

> BLK1{
>
> lst1 : { key is val1, att2 = val2, att3 = val3 }
>
> lst2 : { key is val2, att20 = val20, att3 = val3 }
>
> }

In the above example the attribute named ‘key’ is the key attribute in
block BLK1. Note that a block can have only one key attribute. That
means, as in this example, the value may be used to distinguish between
lists. So when the key attribute has a value of val1 the list uses a
different set of attributes compared to when the value is val2.

> BLK2{
>
> lst3 : { key = keyVal, att2 = val2, att3 = val3 }
>
> lst4 : { key = keyVal, att2 = val2, att3 = val3 }
>
> }

BLK2 by contrast is configured not to use the value of the key attribute
so each list must use the same set of attributes. Note that the
attribute names can be the same across lists and the same key attribute
name can be used in different blocks. Attribute names cannot be repeated
*within* a list however and the key attribute is always mandatory.

1.  **Anonymous lists**

The anonymous list is a legitimate version of a list, but it does not
define a named list variable. This is used where one wishes to group
together a set of attributes that are related to each other, but when we
do not want the list to be referred to elsewhere. Its rules and
behaviour are identical to this of the list in every other respect. Its
syntax is as follows:

:: { keyAtt (=|is) \<value\>, att (=|is) \<value\>, … }

Note the ‘::’ symbol which designates this as an anonymous list.

1.  **Random variable definitions**

Random variables are defined using the ‘\~’ assignment operator which is
the common mathematical convention when relating a random variable to a
probability distribution. In most respects the random variable
definition behaves like a standard equation definition except that the
expression on the right hand side of the ‘\~’ must have a type of Pdf.
Note that the variable can have a transformation function on the left
hand side. This is illustrated in the examples are below:

> ETA\_CL \~ Normal(mean = 0, sd = PPV\_CL) \# valid
>
> ln(ETA\_CL) \~ Normal(mean = 0, sd = PPV\_CL) \# valid
>
> ETA\_CL \~ PPV\_CL \# invalid

The generic syntax description is as follows:

\<ID\> \~ \<PDF expression\>

1.  **Category Lists**

It is possible to define a list that also defines a set of a categories.
This type of definition allows the writer to associate a category
definition with other attributes and information. It also supports the
ability to select a category based on an expression. This is illustrated
in the example below:

> SEX: { use is catCov withCategories { M when 0, F when 1 } }

where the list definition, SEX, can be treated as a categorical variable
with categories ‘M’ and ‘F’. This list also defines a data column so the
‘when’ syntax indicates that the ‘M’ category value is assign to SEX if
the data value is 0 and the ‘F’ category value if 1. This provides a
shorthand that allows us to both define the categories and their
mapping. The syntax of a category list is:

> \<ID\>: { \<attName\> is \<builtinEnum\>\
> withCategories { \<cat value\> when \<selection expr\>,\
>  \<cat value\> when \<selection expr\> } [,\
> \<attName\> [is|=] \<cat value\>, …] }

1.  **Property Definitions**

Sometimes it is desirable to define properties that we wish to associate
with a whole object. Sometimes this is because we want to define default
attribute values for statements with the block or to set properties that
only need to be defined once and which do not need to be referred to in
an expression. To support this MDL provides the property syntax as
follows:

> set \<prop name\> [=|is] \<prop value\> [,\
>  \<prop name\> [=|is] \<prop value\>, …]

An example of this can be found in the task object where the properties
of the task, such as the estimation algorithm are set in this way:

> ESTIMATION{\
>  set algo is saem\
> }

The property name is specific to the block and is unique to the block,
so repeating property definition is forbidden. So:

> ESTIMATION{\
>  set algo is saem, ~~algo is *focei*~~ *\# invalid*\
> }

or

> ESTIMATION{\
>  set algo is saem\
>  ~~set algo is focei~~ \# invalid~~\
> ~~}

will result in an error. In all other respects property names behave
just like attributes within a list, they can have the same types, they
can be optional and mandatory and can be constraint to only permit
certain combinations of property.

1.  <span id="_Toc459299573" class="anchor"></span>**Blocks**

The block is used to organise similar concepts and it can be configured
to only contain certain types of statement. As we have seen above the
block also provides the context for what list and property attributes
are available for use. The generic syntax is

> blkName [(name=value)] { \<statement\> [;] \<statement\> [;] … }

In the above syntax description a statement may also be another block
and so in this way sub blocks may be nested within each other. In MDL
such nesting is limited to one level as can be seen in the example
below:

> MODEL\_PREDICTION {\
>  DEQ{
>
> RATEIN = if(T \>= TLAG) then GUT \* KA else 0\
>  GUT : { deriv =(- RATEIN), init = 0, x0 = 0 }\
>  CENTRAL : { deriv =(RATEIN - CL \* CENTRAL / V) }\
>  }\
>  CC = CENTRAL / V\
> }

where the DEQ block is used to contain the definition of differential
equations.

Blocks constraint the statements they contain in the number of ways:

1.  by type. Some blocks may only permit lists or equation definitions
    or combinations of statement types.

2.  by count. Each block defines the minimum and maximum number of
    statements it can contain.

3.  by sub-block. The block may permit no sub-blocks or sub-blocks with
    specific names.

    1.  <span id="_Toc459299574" class="anchor"></span>**Objects**

The object is the highest level of syntactic organisation in MDL. It
defines a container for a set of blocks and the variables defined inside
them. In MDL the object has a specific purpose and its semantic are a
combination of that purpose and the semantics of the blocks and
statements it contains. Its generic syntax is below:

> \<ID\> = \<objName\> { \<block\> [, \<block\>, … ] }

<span id="_Ref436337940" class="anchor"></span>where the objName is an
internal MDL identifier for the type of the object. The object’s type is
related to semantic purpose. Note that the object type controls what
types of block it contains. For example in MDL an obj of type ‘dataObj’
cannot contain an ‘IDV’ block. A short example of a dataObj is given
below:

> warfarin\_PK\_ODE\_dat = dataObj {
>
> DECLARED\_VARIABLES{GUT Y}
>
> DATA\_INPUT\_VARIABLES {
>
> ID : { use is id }
>
> TIME : { use is idv }
>
> WT : { use is covariate }
>
> AMT : { use is amt, variable = GUT }
>
> DV : { use is dv, variable = Y }
>
> } \# end DATA\_INPUT\_VARIABLES
>
> SOURCE {
>
> srcfile : {file = "warfarin\_conc.csv",
>
> inputFormat is nonmemFormat }
>
> }
>
> }

1.  <span id="_Toc459299575" class="anchor"></span>**The Type System**

One of the core mechanisms for ensuring correctness in MDL is its type
system. In this section we explain what the different types are, and any
rules associated with their correct usage.

### <span id="_Ref437348324" class="anchor"><span id="_Toc459299576" class="anchor"></span></span>The types

  **Name**          **Description**
  ----------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Int               Integer
  Real              Real number
  Boolean           Boolean
  Enum              An enumeration type. A namespace for categories. Enumeration types do not require quotation marks (compared to strings).
  Enum value        A specific enumeration defined by an enumeration type and the value held by a variable of enumeration type.
  Builtin Enum      Commonly abbreviated to BE, this is a predefined set of enumerated values that are part of the MDL definition. These are usually prefixed by the ‘is’ keyword and are often (but not exclusively) used in lists to indicate the key attribute.
  List              A data structure that associates a set of attributes with a variable. See below for details.
  List Super-Type   An abstract type that a list can inherit from.
  Pdf               A Probability density function. Usually returned by a statistical distribution function.
  Random Variable   A value that is a random variable and was obtained from a Probability Distribution. It can have subsidiary types of Real (continuous probability distributions), Int and Enum (discrete probability distributions).
  String            A character string
  Pmf               A probability mass function type. Typically returned by discrete probability distribution functions.
  Mapping           This is the type of the data mapping syntax structure.
  Sublist           This is a sublist, essentially an attribute and value pair.
  Vector            A one dimensional array of values of a single type.
  Matrix            A two dimensional matrix.
  Reference         A reference to a value. The value can be of any type
  Undefined         No type. This is used internally to indicate a validation error during type checking. This is not a valid type.

### The default type

In MDL the default type is Real. In a standard equation or random
variable statement the symbol defined on the LHS of the definition is
always of Real type. Examples:

> A = \<expression\>
>
> ln(B) = \<expression\>
>
> C \~ \<expression\>
>
> D

### Type promotion

MDL allows an integer type to be used in mathematical expression. It
does this using type promotion, where the integer value is automatically
converted to a real value. This gives the kind of behaviour that the
reader would expect. For example:

> A = 22.55 + 1
>
> A = 22.55 + 1.0

are equivalent. Note that mathematical expressions always evaluate to a
value with a Real type so:

> A = 2 \* 55

is effectively evaluated as:

> A = 2.0 \* 55.0

### Typing of more complex types

#### Vector type

A vector can potentially have elements of any type and in general all
its elements must be of the same type. We refer to a vector type as
“vector of type X” or an “X vector”. For example, vector of type String
or String vector.

The type promotion rules above also apply to vectors, which means that a
Real vector can contain a mixture of integer and real values.

Note that when writing a vector literal (see above) the type is inferred
from its content. This means that for a vector to be of type Real it
must contain at least one Real value as can be seen here:

> [ 0, 2, 3, 4 ] \# Int vector
>
> [ 0, 2.0, 3, 4 ] \# Real vector
>
> [ -1.0, 2.0, 3.0, 4.0 ] \# Real vector
>
> [ true, false ] \# Boolean vector
>
> [ “A”, “b”, “C” ] \# String vector
>
> [ { att1 = val1 }, { att1 = val2 } ] \# Sublist vector

#### Matrix type

This behaves identically to the vector type. The type pf the matrix is
inferred from its contents in exactly the same way.

#### List type

The type of the list is defined by a combination of its

1.  owning block

2.  key attribute

3.  key value

This is best illustrated in the example below. Here the ‘c1’ variable is
of type ‘List:Idv’ and ‘c2’ of type ‘List:Amt’. They both belong to the
same block, the key attribute is ‘use’ so the discriminating factor in
determining their type is the values ‘idv’ and ‘amt’.

> DATA\_INPUT\_VARIABLES {
>
> c1 : { use is idv , variable= GUT }
>
> c2 : { use is amt , variable= GUT }
>
> }
>
> DATA\_DERIVED\_VARIABLES{
>
> c3 : { use is doseTime, idvColumn=c1, amtColumn=c2 }
>
> }

In the DATA\_DERIVED\_VARIABLES block ‘c3’ is of a different type again,
but contains two attributes, idvColumn’ and ‘amtColumn’ that expect
references to variables of type ‘List:Idv’ and ‘List:Amt’ respectively.
List types are very specific for referencing another column type (as
below) would result in a typing error:

> DATA\_DERIVED\_VARIABLES{
>
> ~~c3 : { use is doseTime, idvColumn=c2, amtColumn=c1 }~~ \# invalid
>
> \# type error
>
> }

In some cases the list value is not required to define the type. In the
example below the type ‘List:deriv’ is specified by just the block,
‘MODEL\_PREDICTION’, and the key attribute ‘deriv’. As a consequence
‘GUT’ and ‘CENTRAL’ both have the same type.

MODEL\_PREDICTION {

DEQ{

RATEIN = if(T \>= TLAG) then GUT \* KA else 0

GUT : { deriv =(- RATEIN) }

CENTRAL : { deriv =(RATEIN - CL \* CENTRAL / V) }

}

CC = CENTRAL / V

}

Each List type can potentially be converted to one another type when
necessary. This is particularly useful if the semantics of the list make
it desirable to use the list variables in an mathematical expression.
The above example shows just such a case. The semantic of List:deriv
lists is to define a difference equation, with the list variable
corresponding to the derivative variable. The List:deriv type has a
conversion type of Real. This means that when used in contexts that
expect a Real type the Type system uses the conversion type. This allows
the example above to be valid despite the fact that list variables are
used as real values. All list type can potentially have **one**
conversion type, but it’s use is optional and if not define then the
list cannot be converted.

In the latest version of MDL there is an alternative method of
identifying a list. This is via the attribute name. This acts as the
list key and must be unique across all list types associated with the
same block. For example, we can differentiate between different
parameters as follows:

A : { value = 0.2 }

A : { vectorValue = [0.2, 0.3] }

A : { matrixValue = [[ 0.2, 0.4; 0.5, 0.9 ]] }

The above lists each have different types identified by the attribute
name used.

#### List super-types

Sometimes it is useful to group related lists and refer to them
interchangeably. For example, observation variables use different list
types to define different continuous error models:

Y1 : { type is combinedError1 … } \# list type combinedError1List

Y2 : { type is combinedError2 … } \# list type combinedError2List

However, they are clearly related and should be compatible with each
other. This is where the list super type is useful. In the above case
both list types share the same super type, ‘observation’. This enables
another attribute to be assigned this super-type and it will be able to
refer to either of these variables:

Z1 : { superTypeAtt = Y1 } \# valid

Z2: { superTypeAtt = Y2 } \# valid

which would not be possible with normal list typing:

X1 : { listTypeAtt = Y1 } \# valid as expects type is combinedError1List

X2: { listTypeAtt = Y2 } \# invalid as expects type combinedError1List

In the same way super-types enable conditional list assignments that
involve different list types:

U : if (x \> 0) then { type is combinedError1, … }\
 else { type is combinedError2, … }

here ‘U’ is allocated the type ‘observation’ because both lists share
the same super-type. In this way MDL knows that these lists are
compatible.

#### Built-in enumeration type

A built-in enumeration is essentially a controlled vocabulary that
constrains the values that can be assigned to an attribute. Each
built-in enumeration is different and consists of a set of strings. The
enumeration is match if the name assigned to the attribute is one of the
names held by its built-in type. As an example if there is a built-in
type ‘eg’ that permits the names ‘foo’ and ‘bar’ then the following
cases are valid and invalid:

> att1 is foo \# valid
>
> att1 is bar \# valid
>
> ~~att1 is ugg \# invalid~~
>
> ~~att1 = foo \# invalid~~

The last case is important to note. MDL only knows to expect a building
enumeration if it is preceded by the ‘is’ keyword. In the last statement
above the assignment symbol was used and in this case MDL would treat
this as a reference to a variable.

#### Sublist type

Sublists are types and they are distinguished from each other by their
attributes. This is best illustrated by an example:

> ln(CL) = linear( trans is ln, pop = POP\_CL, fixEff = [
>
> {coeff = BETA\_CL\_WT, cov = logtWT},
>
> {coeff = POP\_FCL\_FEM, catCov = SEX.female },
>
> {coeff = BETA\_CL\_AGE, cov = tAGE}
>
> ],
>
> ranEff = [ETA\_CL] )

Above the attribute ‘fixEff’ expects a vector type of ‘Sublist:FixEff’.
The type system looks at the available sublist types and identifies the
one that has the same attribute names. Note that the same type can allow
different attribute combinations as can be seen in the example. If no
subtype can be identified then the sublist is given a type of Undefined
which will result in a typing error. Identifying the correct subtype
also then allows the attributes in the sublist itself to be type
checked.

#### Enum and Enum Value types

Enumeration types are unusual in MDL in that the type is to some extent
defined within MDL. Take the following definitions:

> Gender1 withCategories { male, female, other }
>
> Gender2 withCategories { male, female, other }
>
> State withCategories {alive, dead }

The first defines an enumeration type with individual values, ‘male’,
‘female’ and ‘other’. The second definition is distinct from the first
because it is associated with a different symbol. So in a sense the
above definitions have created 3 new types called Gender1, Gender2 and
State. However, the above statements also define 3 variables of the same
name and as variables they can be initialised with one of their
enumeration values. So the variable ‘State’ can hold a value of ‘alive’
or ‘dead’, or more correctly ‘State.alive’ or ‘State.dead’. This means
that when used as a variable reference these variables always have a
type of Enum value. This is illustrated by the example expressions
below:

> Gender1 == Gender1.male \# valid
>
> ~~Gender2 == Gender1.male~~ \# invalid
>
> ~~Gender1 == Gender2~~ \# invalid

### <span id="_Toc459299580" class="anchor"><span id="_Ref437351889" class="anchor"><span id="_Ref437458281" class="anchor"><span id="_Ref437458283" class="anchor"><span id="_Ref437458284" class="anchor"></span></span></span></span></span>Type inference rules for Symbols

When a variable is defined is must be allocated a type and unlike
language such as R, this type cannot change. Where is can MDL works out
the type of the variable, usually from the type of any expression on the
right hand side of an assignment operartor (‘=’, ‘\~’ or ‘:’). Where
there is no assignment the variable type defaults to Real unless the
author explciltly defines its type using a type specification (see
above). This is best describe using examples:

A = [ 1, 2, 3 ] \# vector of ints

B \# real

C::string \# string

D = true \# Boolean

E \~ Poisson1(rate=lambda) \# Random Variable of int

F withCategories {heads, tails} \~ Bernoulli1(probability=p1)\
 \# Random Variable of enum

G = [[ “A”, “B”; “C”, “D”]] \# matrix of strings

Scoping and statement ordering
------------------------------

MDL is declarative. It describes what the model is not how to implement
it. In common with other declarative languages MDL has the following
features:

-   The order of blocks and statements within blocks is ***not***
    significant.

-   This means that symbols can be defined after they are referenced.

-   A variable ***must not*** be assigned to more than once.

-   In general a variable cannot be assigned to an expression that is
    dependent on itself. For example the following is not permitted:

> a = b
>
> b = c
>
> c = a (this makes a cycle back to the first statement)

A list can be defined that is exempt from this rule. For example, a
derivative list can refer to iself:

> x : { deriv = -x }

This is consistent with mathematical definitions such as:

> dx/dt = -x

The scoping unit for MDL is the object. Variables defined inside an
object are visible to expressions in the same object, but not outside
it. This means that each MDL object is a self contained definition that
does not rely on any other object.

<span id="h.rnst7pkmtqe4" class="anchor"><span id="_Toc459299582" class="anchor"></span></span>MDL Reference Guide
------------------------------------------------------------------------------------------------------------------

Appendix X contains the MDL reference guide. This document gives detail
of Object, Block, List, Sublist, Function and Type definitions. As it is
based on and generated from the language definition file it documents
exactly how the language is constructed and is definitive in describing
what is valid MDL as implemented in the MDL-IDE. Elements of the
Reference Guide have been extracted here in a format intended to help
user understanding which may be used for quick reference.

### How to read the MDL Reference Guide.

There is a hierarchy in the MDL Reference Guide. This reflects the way
that users approach the definition of a model in a sequential fashion.
Understanding this hierarchy will help the user identify what statements
and attributes are valid as shown here:.

-   Object definitions describe which Blocks are valid within each
    Object.

-   Block definitions describe

    -   what arguments are associated with the block

    -   what types of statements can be made within the block

    -   what sub-blocks are permitted within the block

    -   what list types are to be used with the block

    -   what properties are permitted within the block.

-   List types define

    -   whether the list can be anonymous i.e. using ::{ **type is** … }

    -   whether the list can define categories

    -   what types are associated with the list i.e. **type is** …

    -   what attributes are permitted withing the list.

    -   what the “signature” of the list looks like, including
        identifying which attributes are optional.

    -   NB: list type names do not necessarily match the names used in
        MDL e.g. in defining individual parameters we type “ CL :
        {**type is** **linear**, … }” but the type of the list in the
        Reference Guide is “IndivParamLinear”. However the list type is
        given in the Block definition List table (left hand column) with
        the associated MDL key value in the right hand column.

-   Sublist definitions define the (small number of) cases where list
    types contain other lists, for example in definition of the fixed
    effect model within a IndivParamLinear list. The sublist definition
    defines:

    -   what attributes are permitted within the sublist

    -   the permitted signature of the sublist and which attributes are
        optional.

-   Function definitions define:

    -   Arguments of the function

    -   Type returned from the function.

-   Type definitions define

    -   the types and associated type classes.

    -   the keyword types and associated enumeration types. e.g. **type
        is** **combinedError1**; **set algo is** **saem**.

    1.  Objects
        -------

Valid types of Objects are:

> dataObj, designObj, parObj, priorObj, mdlObj, taskObj, mogObj

<span id="h.8oaa1qvspuqc" class="anchor"><span id="_Toc459299585" class="anchor"></span></span>Blocks
-----------------------------------------------------------------------------------------------------

The following blocks are defined for the various MDL Objects:

  MDL Object   Block                              Sub-blocks
  ------------ ---------------------------------- -----------------
  dataObj      **DECLARED\_VARIABLES**            
               **DATA\_INPUT\_VARIABLES**         
               **SOURCE**                         
               **DATA\_DERIVED\_VARIABLES**       
  designObj    **DECLARED\_VARIABLES**            
               **INTERVENTION**                   
               **SAMPLING**                       
               **DESIGN\_PARAMETERS**             
               **POPULATION**                     
               **STUDY\_DESIGN**                  
               **DESIGN\_SPACES**                 
  parObj       **STRUCTURAL**                     
               **VARIABILITY**                    
  priorObj     **PRIOR\_PARAMETERS**              
               **NON\_CANONICAL\_DISTRIBUTION**   
               **PRIOR\_VARIABLE\_DEFINITION**    
  mdlObj       **IDV**                            
               **VARIABILITY\_LEVELS**            
               **COVARIATES**                     
               **FUNCTIONS**                      
               **POPULATION\_PARAMETERS**         
               **STRUCTURAL\_PARAMETERS**         
               **VARIABILITY\_PARAMETERS**        
               **GROUP\_VARIABLES**               
               **RANDOM\_VARIABLE\_DEFINITION**   
               **INDIVIDUAL\_VARIABLES**          
               **MODEL\_PREDICTION**              
                                                  **DEQ**
                                                  **COMPARTMENT**
               **OBSERVATION**                    
  taskObj      **ESTIMATE**                       
               **SIMULATE**                       
               **EVALUATE**                       
               **OPTIMISE**                       
  mogObj       **INFO**                           
               **OBJECTS**                        

<span id="h.g4oyzhr5quyi" class="anchor"><span id="h.bhuoae2z5paf"
class="anchor"></span></span>

1.  <span id="h.k390vsgiuruf" class="anchor"><span id="_Toc459299586"
    class="anchor"></span></span>**Functions**

Mathematical functions are provided to support definition of models. The
following mathematical functions are defined:

Statistical functions:

mean, median, probit, logit, invLogit, invProbit

Arithmetic functions:

log, log2, log10, ln, factorial, lnFactorial, floor, ceiling, min, max,
abs, exp, sqrt, sum, toInt, seq, seqby, dseq, rep

Trigonometric functions:

sin, cos, tan, sinh, cosh, tanh, asin, acos, atan, asinh, acosh, atanh

vector and matrix handling:

toMatrixByRow, toMatrixByCol, asVector, inverse, triangle, transpose,
diagonal, gInv, det, eigen, chol, matrix

Interpolation functions:

linearInterp, constInterp, lastValueInterp, nearestInterp, cubicInterp,
pchipInterp, splineInterp

Distributions (see below).

1.  <span id="h.9shm6j8bxfgy" class="anchor"><span id="_Toc459299587"
    class="anchor"></span></span>**Distributions**

Distributions are denoted using the \~ prefix. The following
distributions are defined:

  -----------------------------------------------------------------------------------------------
  Name     Return Type   Argument name   Argument Types   Comment
  -------- ------------- --------------- ---------------- ---------------------------------------
  Normal   Pdf           mean            Real             Maps to ProbOnto distribution Normal1
                                                          
                         sd              Real             

  Normal   Pdf           mean            Real             Maps to ProbOnto distribution Normal2
                                                          
                         var             Real             
  -----------------------------------------------------------------------------------------------

The following ProbOnto distributions are defined:

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  <span id="h.n2ms9yw5a9iv" class="anchor"></span>Name   Return Type   Argument name        Argument Types   Comment
  ------------------------------------------------------ ------------- -------------------- ---------------- ----------------------------------------------------------------------------
  Normal1                                                Pdf           mean                 Real             Note that ProbOnto argument is stdev not sd
                                                                                                             
                                                                       stdev                Real             

  Normal2                                                Pdf           mean                 Real             
                                                                                                             
                                                                       var                  Real             

  Normal3                                                Pdf           mean                 Real             For use with BUGS
                                                                                                             
                                                                       precision            Real             

  LogNormal1                                             Pdf           meanLog              Real             Values given on the ln scale
                                                                                                             
                                                                       stdevLog             Real             

  LogNormal2                                             Pdf           meanLog              Real             Values given on the ln scale
                                                                                                             
                                                                       varLog               Real             

  LogNormal3                                             Pdf           median               Real             median on the natural scale, standard deviation on the ln scale
                                                                                                             
                                                                       stdevLog             Real             

  Bernoulli1                                             Pmf           probability          Real             Probability argument specifies the probability of the **second** category.

  Poisson1                                               Pmf           rate                 Real             

  Binomial1                                              Pmf           probability          Real             Probability of success (second category).
                                                                                                             
                                                                       numberOfTrials       Real             

  Beta1                                                  Pdf           alpha                Real             
                                                                                                             
                                                                       beta                 Real             

  Gamma1                                                 Pdf           shape                Real             
                                                                                                             
                                                                       scale                Real             

  Gamma2                                                 Pdf           shape                Real             
                                                                                                             
                                                                       rate                 Real             

  InverseGamma1                                          Pdf           shape                Real             
                                                                                                             
                                                                       scale                Real             

  NonParametric                                          Pdf           bins                 Vector           
                                                                                                             
                                                                       probability          Vector           

  MultiNonParametric                                     vector        bins                 Matrix           
                                                                                                             
                                                         Pdf           probability          Real             

  Empirical                                              Pdf           data                 Vector           

  MultiEmpirical                                         vector        data                 Matrix           
                                                                                                             
                                                         Pdf                                                 

  MultivariateNormal1                                    vector        mean                 Vector           
                                                                                                             
                                                         Pdf           covarianceMatrix     Matrix           

  MultivariateNormal2                                    vector        mean                 Vector           
                                                                                                             
                                                         Pdf           precisionMatrix      Matrix           

  MultivariateStudentT1                                  vector        mean                 Vector           
                                                                                                             
                                                         Pdf           covarianceMatrix     Matrix           
                                                                                                             
                                                                       degreesOfFreedom     Real             

  MultivariateStudentT2                                  vector Pdf    mean                 Vector           
                                                                                                             
                                                                       precisionMatrix      Matrix           
                                                                                                             
                                                                       degreesOfFreedom     Real             

  NegativeBinomial2                                      Pdf           rate                 Real             
                                                                                                             
                                                                       overdispersion       Real             

  StudentT1                                              Pdf           degreesOfFreedom     Real             

  StudentT2                                              Pdf           mean                 Real             
                                                                                                             
                                                                       scale                Real             
                                                                                                             
                                                                       degreesOfFreedom     Real             

  Uniform1                                               Pdf           minimum              Real             
                                                                                                             
                                                                       maximum              Real             

  Wishart1                                               Matrix        scaleMatrix          Matrix           
                                                                                                             
                                                         Pdf           degreesOfFreedom     Real             

  Wishart2                                               Matrix        inverseScaleMatrix   Matrix           
                                                                                                             
                                                         Pdf           degreesOfFreedom     Real             

  InverseWishart1                                        Matrix        scaleMatrix          Matrix           
                                                                                                             
                                                         Pdf           degreesOfFreedom     Real             

  CategoricalNonordered1                                 Pmf           categoryProb         Vector           

  CategoricalOrdered1                                    Pmf           categoryProb         Vector           

  MixtureDistribution1                                   Pdf           weight               Vector           
                                                                                                             
                                                                       distributions        Vector of PDF    

  ZeroInflatedPoisson1                                   Pdf           rate                 Real             
                                                                                                             
                                                                       probabilityOfZero    Real             

                                                                                                             
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

