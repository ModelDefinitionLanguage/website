The Task Properties Object
==========================

The Task Properties Object is intended to convey information to specific
target software related to algorithms, settings, options for performing
a given task. ~~In the current version of MDL, the implementation of
this is very limited.~~ Some Task Properties Object settings are generic
(apply across different target software) but most are specific to the
intended target software.

Task properties are specific to the task e.g. **ESTIMATE**,
**SIMULATE**, **OPTIMISE**, **EVALUATE**. **TARGET\_SETTINGS** are
specific to a given target software tool. If the user intends to
estimate with a given tool but no **ESTIMATE** block is given in the
Task Properties Object and/or if the task properties are not specified
for the intended tool, then default settings are used.

**~~In the current version of MDL, only the estimation algorithm
property can be set.~~**

Intended use of Task Properties
-------------------------------

The user should provide information about settings and options for each
target software that they wish to use for a given (estimation) task.
These settings and options might be specified to ensure reproducibility
of results regardless of target software i.e. to ensure that the results
from a given target software are comparable with results from different
target software.

Alternatively, the user may wish to provide settings and options that
they will frequently use with a given target software – so that every
time they estimate with a given target they use the same settings.

The modularity of MDL allows the user to preset or reuse Task Properties
objects between models. The user may then have preferred Task Properties
for estimation that can be called upon during the relevant points in
their M & S workflow.

As with other MDL blocks, it is possible to specify multiple Task
Properties blocks within a single .mdl file and then reference only the
one required for use with a specific task in either the MOG Object or
via R. This allows the user to specify preferred settings for multiple
target software tools – facilitating reproducibility and reuse. Even
though many Task Properties Objects can be given in an .mdl file, the
MOG Object should contain only ONE Task Properties Object.

Why use Task Properties for settings and options rather than arguments of functions in the ddmore R package?
------------------------------------------------------------------------------------------------------------

Task Properties are distinct from arguments to the ddmore R package
functions for executing tasks – the former passes information to the
appropriate target software about the particular settings and options
required for a given task. The ddmore function arguments are equivalent
to the command line settings or options which are employed when invoking
target software.

For example, we may use Task Properties to define the estimation
algorithm and associated settings with NONMEM, but define command line
options for PsN which govern how NONMEM should be called by PsN.

How are Task Properties used by MDL and PharmML?
------------------------------------------------

The Task Properties generic items are parsed and understood by the MDL
editor within the MDL-IDE. However target software specific settings and
options are not parsed by the MDL-IDE – these are passed as is via the
PharmML to the target software converter where they are interpreted and
converted where they may appear in target software code, external
settings or options files as appropriate.

**ESTIMATE** Block
------------------

~~In the current MDL version, the only block supported is the
**ESTIMATE** block.~~ The syntax for this block is:

set **algo** is \<**foce | focei |** **saem | mcmc**\>

~~As stated previously, this block will be extended in future versions
of MDL to capture target software specific settings and options.~~

### TARGET\_SETTINGS

The TARGET\_SETTINGS block holds specific settings for the given target
software. These settings may be given in name – value pairs within the
TARGET\_SETTINGS block, or they may be specified in an external file.

MDL does not check whether the specified options within the
TARGET\_SETTINGS block are appropriate for a given tool. This is
performed by the converter for the target tool. See the MDL Reference
section for a list of which options are available for each target
software.

The syntax for the TARGET\_SETTINGS block is:

TARGET\_SETTINGS(target=”NONMEM” | “MONOLIX” | “BUGS” | “PFIM”,
settingsFile = “\<filename\>”){

set \<target specific setting name\> = \<target specific setting
value\>,

\<target specific setting name\> = \<target specific setting value\>,

…

}

Note the comma separation between elements of the set statement.

For example (UseCase1\_1):

SAEM\_task = **taskObj** {

**ESTIMATE**{

**set** **algo** **is** **saem**

**TARGET\_SETTINGS**(**target**="NONMEM"){

\# \$EST METHOD=SAEM INTERACTION NBURN=3000 NITER=500 PRINT=100
SEED=1556678 ISAMPLE=2

\# \$COV MATRIX=R PRINT=E UNCONDITIONAL SIGL=12

**set** **INTER**=**true**, **NBURN**=3000, **NITER**=500,
**PRINT**=100, **SEED**=1556678, **ISAMPLE**=2,

COV\_=true, **COV\_MATRIX**="R", **COV\_PRINT**="E",
**COV\_UNCONDITIONAL**=**true**,

**COV\_SIGL**=12

}

**TARGET\_SETTINGS**(**target**="MONOLIX",
**settingsFile**=["tables.xmlx"]){

**set** **graphicsSettings**="tables.xmlx"

}

}

} \# end of task object

BUGS\_task = **taskObj**{

**ESTIMATE**{

**set** **algo** **is** **mcmc**

**TARGET\_SETTINGS**(**target**="BUGS"){

**set** **nchains** = 1, \# Number of MCMC chains

**burnin** = 1000, \# Number of MCMC Burn-in iterations

**niter** = 5000, \# Number of iterations

**parameters** = "V,CL,KA,TLAG", \# Parameters to monitor

**odesolver**="LSODA", \# or "RK45"

**winbugsgui** ="false" \# or "true"

}

}

}

FOCEI\_task= **taskObj** {

**ESTIMATE**{

**set** **algo** **is** **focei**

**TARGET\_SETTINGS**(**target**="NONMEM"){

\# \$EST METHOD=COND INTERACTION MAXEVAL=9999 NSIG=3 SIGL=10 PRINT=5
NOABORT NOPRIOR=1 \# FILE=example1.*ext*

\# \$COV MATRIX=R PRINT=E UNCONDITIONAL SIGL=12

**set** **INTER**=**true**, **MAXEVAL**=9999, **NSIG**=3, **SIGL**=10,
**PRINT**=5, **NOABORT**=**true**,

**NOPRIOR**=1, **FILE**="example1.*ext*",

**COV\_MATRIX**="R", **COV\_PRINT**="E",
**COV\_UNCONDITIONAL**=**true**, **COV\_SIGL**=12,

**MSFO\_FILE**="MSFO.msf"

}

}

} \# end of task object

For BUGS, the following properties are valid (as shown above):

nchains = \<integer\>

niter = \<integer\>

parameters = \<comma-separated character string of parameter names\>

odesolver = \<”LSODA” | “RK45”\>

winbugsgui = \<”true” | “false”\>

For NONMEM, the following rules are used in the **TARGET\_SETTINGS**
block:

a\) Individual properties influence only \$EST statement or the \$COV

b\) Properties starting with the string ‘COV\_’ will be added to the
\$COV statement using the following rules

a\. The string ‘COV\_’ will be removed from the property

b\. If the remainder of the property is a string/integer assignment, it
will be added as a parameter on the \$COV statement verbatim (with any
unnecessary quotes removed)

c\. If the remainder of the property is a boolean assignment resolving to
true, just the parameter name will be added on the \$COV statement
verbatim

c\) If there are any ‘COV\_’ properties present, then a \$COV statement
will be generated. Otherwise it will not be generated

d\) Any property starting with a string that is **not** ‘COV\_’ will be
added to the \$EST statement using the same string/integer/boolean rules
as for \$COV

