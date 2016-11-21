The Prior Object
================

For Bayesian estimation, the user must specify a Prior Object, which is
used in place of the Parameter Object. The Prior Object defines prior
distributions or values for all model parameters – both structural and
variability. The prior distributions form an additional level of model
hierarchy above all other levels of variability in the model. This level
of the hierarchy does not need to be explicitly defined in the Model
Object **VARIABILITY\_LEVELS** block. it is implied through use of the
Prior Object.

Prior distributions vs initial values vs fixed values
-----------------------------------------------------

At present we do not support translation of the Prior Object to NONMEM
prior specification for use with the \$BAYES estimation algorithm. The
only supported Bayesian estimation tool currently is WinBUGS. While
NONMEM accepts a mix of prior distribution specification and initial
values for estimation, WinBUGS requires prior distributions for all
parameters, or fixing parameters to a given value. A fixed value can be
thought of as a probability mass function (pmf) on a single value. A
fixed value for a parameter represents a very strong prior on the value
of that parameter. Bounds on parameters should be handled using an
appropriate ProbOnto distribution e.g. Beta, Gamma, Uniform,
Half-Normal, Truncated-Normal.

**PRIOR\_PARAMETERS** Block
---------------------------

The **PRIOR\_PARAMETERS** block holds constants which may be used in the
**PRIOR\_VARIABLE\_DEFINITION** or **NON\_CANONICAL\_DISTRIBUTION**
blocks. This allows the user to specify the general form of the prior
distribution in the **PRIOR\_VARIABLE\_DEFINITION** block and then
examine sensitivity to prior choice by altering the values in the
**PRIOR\_PARAMETERS**.

The **PRIOR\_PARAMETERS** block should contain variable assignment
statements.

\<VARIABLE\> = \<value\>

Note that In the **PRIOR\_PARAMETERS** block, the variable is assigned a
value, not a list with attributes. As mentioned above, if a model
parameter is to be fixed then it takes the value assigned in the
**PRIOR\_PARAMETERS** block. An attribute “fix=true” is not required.

For example (/Priors/UseCase1\_PRIOR):

**PRIOR\_PARAMETERS**{

\# prior on "THETA"

MU\_POP\_CL = 0.2

MU\_POP\_V = 10

MU\_POP\_KA = 0.3

MU\_POP\_TLAG = 0.75

VAR\_POP\_CL = 1

VAR\_POP\_V = 1

VAR\_POP\_KA = 1

VAR\_POP\_TLAG = 0.1

\# prior on "OMEGA"

MU\_R\_CL = 0.2

MU\_R\_V = 0.2

MU\_R\_V\_CL = 0

DF\_OMEGA = 2

MU\_OMEGA\_KA = 1

MU\_OMEGA\_TLAG = 1

\# prior on "SIGMA"

a\_POP\_RUV\_ADD = 1.1

b\_POP\_RUV\_ADD = 3

a\_POP\_RUV\_PROP = 1.1

b\_POP\_RUV\_PROP = 3

} \# end PRIOR\_PARAMETERS

**PRIOR\_VARIABLE\_DEFINITION** – Parametric distributions as priors
--------------------------------------------------------------------

In the **PRIOR\_VARIABLE\_DEFINITION** block we set up the prior
distributions for the **STRUCTURAL** and **VARIABILITY** parameters of
the Model Object. All model parameters must have a prior distribution
specified, or a constant value set.

If model parameters are correlated or have a multivariate distribution
then it is common (although not mandatory) to specify multivariate prior
distributions. To do so, the user is likely to need to specify vectors
of means and matrices for covariances or correlations. The syntax for
specifying vectors and matrices is given in section 9.1.4.7.

The PRIOR\_VARIABLE\_DEFINITION block can contain assignment,
transformation and random variable definitions using ProbOnto
definitions for distributions.

For example (UseCase1\_PRIOR):

**PRIOR\_VARIABLE\_DEFINITION**{

\# prior on "THETA"

lMU\_POP\_CL = ln(MU\_POP\_CL)

*lPOP\_CL* \~ Normal(**mean**=lMU\_POP\_CL, **var**=VAR\_POP\_CL)

POP\_CL = exp(lPOP\_V)

lMU\_POP\_V = ln(MU\_POP\_V)

lPOP\_V \~ Normal(**mean**=lMU\_POP\_V, **var**=VAR\_POP\_V)

POP\_V = exp(lPOP\_V)

lMU\_POP\_KA = ln(MU\_POP\_KA)

lPOP\_KA \~ Normal(**mean**=lMU\_POP\_KA, **var**=VAR\_POP\_KA)

POP\_KA = exp(lPOP\_KA)

lMU\_POP\_TLAG = ln(MU\_POP\_TLAG)

lPOP\_TLAG \~ Normal(**mean**=lMU\_POP\_TLAG, **var**=VAR\_POP\_TLAG)

POP\_TLAG = exp(lPOP\_TLAG)

\# priors on "OMEGA"

R\_mat = [[ MU\_R\_CL, MU\_R\_V\_CL;

MU\_R\_V\_CL, MU\_R\_V ]]

TAU\_CL\_V \~ Wishart2(**inverseScaleMatrix**=R\_mat,
**degreesOfFreedom**=DF\_OMEGA)

OMEGA\_CL\_V = inverse(TAU\_CL\_V)

PPV\_CL = sqrt(OMEGA\_CL\_V[1,1])

PPV\_V = sqrt(OMEGA\_CL\_V[2,2])

PPV\_V\_CL = OMEGA\_CL\_V[1,2]

TAU\_KA \~ Gamma2(**shape**=0.001, **rate**=0.001)

PPV\_KA = sqrt(1/TAU\_KA)

\# prior on "SIGMA"

invRUV\_ADD \~ Gamma2(**shape**=a\_POP\_RUV\_ADD,
**rate**=b\_POP\_RUV\_ADD)

invRUV\_PROP \~ Gamma2(**shape**=a\_POP\_RUV\_PROP,
**rate**=b\_POP\_RUV\_PROP)

RUV\_ADD = sqrt(1/invRUV\_ADD)

RUV\_PROP = sqrt(1/invRUV\_PROP)

} \# end PRIOR\_VARIABLE\_DEFINITION

Note the following parameters have prior distributions assigned:
POP\_CL, POP\_V, POP\_KA, POP\_TLAG, PPV\_CL, PPV\_V, COV\_CL\_V,
PPV\_KA, RUV\_ADD, RUV\_PROP.

Note that priors on between subject variability are given on the
precision scale (= 1 / variance). This is to facilitate use in BUGS.
Here the distributions used are Wishart and Gamma on the precision
parameters, but Inverse-Wishart and Inverse-Gamma may alternatively be
used on the variance-covariance matrix and variance parameters.

Note also that priors for POP\_CL, POP\_V, POP\_KA and POP\_TLAG could
also be defined using logNormal1 distributions (using the ProbOnto
distribution) instead of transforming and back-transforming using
Normal(…) distributions.

To specify a matrix, we use double square brackets, and specify elements
row-wise, separated with a semi-colon. To specify elements of a matrix
we use the R convention of square bracket specifying row and column
entries. For example:

R\_mat = [[ MU\_R\_CL, MU\_R\_V\_CL;

MU\_R\_V\_CL, MU\_R\_V ]]

To specify the first row and column entry (corresponding to MU\_R\_CL):
R\_mat[1,1].

Note that a Gamma distribution is used to define the prior on the
between subject variability for PPV\_KA. This is a legacy from the early
days of fitting hierarchical models in BUGS where Gamma priors were
conjugate and easier to sample from. They have been somewhat discredited
as prior choices. Recent literature has favoured Half-Cauchy priors on
variance parameters of hierarchical models as they are robust against
smaller numbers of subjects.[^16]

Non-parametric and empirical distributions as priors – inline data.
-------------------------------------------------------------------

As an alternative to parametric distributions as priors, the user can
specify non-parametric distributions (specifying bins of values and
probabilities for each bin) or empirical distributions (specifying data
forming the basis of empirical sampling). Univariate and multivariate
sampling distributions have been defined in MDL for non-parametric and
empirical sampling distributions.

In both cases the source for the non-parametric or empirical sampling
can be specified inline via the **PRIOR\_PARAMETERS** block, or by
referencing an external data source in the **PRIOR\_SOURCE** block.

### Non-parametric distribution specification with inline data.

To specify a non-parametric distribution, MDL has a distributions called
NonParametric and MultiNonParametric. These map to the ProbOnto
RandomSample non-parametric distribution definition. To specify the
non-parametric distributions, the user must supply bins and
probabilities for sampling. To specify this inline we create a vector
(for NonParametric) or matrix (for MultiNonParametric) of bins and a
vector of probabilities. These are specified in the
**PRIOR\_PARAMETERS** block.

For example (Priors examples, Example3421dep)

**PRIOR\_PARAMETERS**{

…

\# For Non-Parametric distribution

bins\_POP\_K\_V =
matrix(**vector**=[2.006510,2.045465,2.084421,2.123377,2.162333,2.201288,2.240244,2.279200,2.318156,2.357111,
5.050013,5.050013,5.050013,5.050013,5.064166,5.064166,5.064166,5.064166,5.078318,5.078318],**ncol**=2,
**byRow** **is** **FALSE**)

p\_POP\_K\_V =
[0.033333,0.100000,0.100000,0.200000,0.100000,0.066667,0.166667,0.100000,0.066667,0.066667]

} \# end PRIOR

**PRIOR\_VARIABLE\_DEFINITION**{

\# prior on "THETA"

POP\_K\_V \~ MultiNonParametric(**probability**=p\_POP\_K\_V,
**bins**=bins\_POP\_K\_V)

POP\_K = POP\_K\_V[1]

POP\_V = POP\_K\_V[2]

…

} \# end PRIOR\_VARIABLE\_DEFINITION

Here a matrix of bins for POP\_K and POP\_V is created by specifying a
vector of values and then defining the number of columns and method for
filling the matrix. A vector of probabilities is also defined. Then in
the **PRIOR\_VARIABLE\_DEFINITION** block the multivariate
non-parametric sampling distribution is defined referencing the
probabilities and bins. Finally the Priors for POP\_K and POP\_V are
defined by referencing the elements of the POP\_K\_V vector.

### Empirical distribution specification with inline data.

Similarly, the user can specify the data source for empirical sampling
within the **PRIOR\_PARAMETERS** block and then refer to this in
defining the sampling distribution for the
**PRIOR\_VARIABLE\_DEFINITION**.

For example (Priors examples, Example 3422)

**PRIOR\_PARAMETERS**{

data\_POP\_K\_V =
matrix(**vector**=[2.006510,2.045465,2.084421,2.123377,2.162333,2.201288,2.240244,2.279200,2.318156,2.357111,
5.050013,5.050013,5.050013,5.050013,5.064166,5.064166,5.064166,5.064166,5.078318,5.078318],**ncol**=2,
**byRow** **is** **FALSE**)

…

} \# end PRIOR

\#

**PRIOR\_VARIABLE\_DEFINITION**{

\# prior on "THETA"

POP\_K\_V \~ MultiEmpirical(**data**=data\_POP\_K\_V)

POP\_K = POP\_K\_V[1]

POP\_V = POP\_K\_V[2]

…

} \# end PRIOR\_VARIABLE\_DEFINITION

Again, the data source is defined by specifying a matrix of values for
POP\_K and POP\_V and then using this as the basis for the
MultiEmpirical sampling distribution. For the univariate Empirical
sampling distribution, only a vector would be needed as the basis for
sampling.

**NON\_CANONICAL\_DISTRIBUTION** block
--------------------------------------

As an alternative to inline data specification for non-parametric or
empirical sampling distributions, the user may reference and external
dataset for bins and probabilities (for use with non-parametric sampling
distributions) or data for the basis of the empirical sampling
distribution.

### **PRIOR\_SOURCE** block

Similarly to the **SOURCE** block within the Data Object, the
**PRIOR\_SOURCE** block is a named list providing the file name, format
of the source data. However the **PRIOR\_SOURCE** block adds an argument
to the list to provide a vector of column names to be used in the data
source for the sampling distributions.

The syntax is as follows:

**PRIOR\_SOURCE**{

\<data source name\> : { file = \<”filename.csv”\>, **inputFormat is**
**csv**, column = [\<”variable name1”, “variable name2”, … , “variable
name k”]}

}

Multiple data sources may be defined within the **PRIOR\_SOURCE** block.

### **INPUT\_PRIOR\_DATA** block

The PRIOR\_SOURCE data objects can then be referenced in the
**INPUT\_PRIOR\_DATA** block to define how the data file columns map to
objects to be used in the **PRIOR\_VARIABLE\_DEFINITION** block sampling
distributions. This is done using anonymous lists.

The syntax is as follows:

**INPUT\_PRIOR\_DATA**{

:: { src = \<**PRIOR\_SOURCE** data variable\>, vector | matrix =
\<**PRIOR\_VARIABLE\_DEFINITION** object\>, column =
“\<**PRIOR\_SOURCE** data column name\>” }

}

For example :

**NON\_CANONICAL\_DISTRIBUTION**{

**PRIOR\_SOURCE**{

NonPar\_K\_V : { **file**="Nonparametric\_K\_V.csv", **inputFormat**
**is** **csv**,

**column**=["bins\_k", "bins\_v", "p\_k\_v"]}

Emp\_SIGMA : { **file**="Empirical\_Sigma.*csv*", **inputFormat** **is**
**csv**,

**column**=["data\_SIGMA2"]}

}

**INPUT\_PRIOR\_DATA**{

:: { **src**= NonPar\_K\_V, **vectorVar**=p\_k\_v, **column**="p\_k\_v"
}

:: { **src**= NonPar\_K\_V, **matrixVar**=bins\_k\_v,
**column**=["bin\_k", "bins\_v"] }

:: {**src**= Emp\_SIGMA, **column**="data\_SIGMA2",
**vectorVar**=data\_SIGMA2 }

}

}

**PRIOR\_VARIABLE\_DEFINITION**{

p\_k\_v::vector

bins\_k\_v::matrix

data\_SIGMA2::vector

POP\_k\_v \~ MultiNonParametric(**bins**=bins\_k\_v,
**probability**=p\_k\_v)

*POP\_SIGMA2* \~ Empirical(**data**=data\_SIGMA2)

POP\_K = POP\_k\_v[0]

POP\_V = POP\_k\_v[1]

}

In the above example, two sources are specified – one giving
non-parametric sampling bins and probabilities for K and V, the other
providing values for SIGMA to be used in the empirical sampling
distribution. In the definition of NonPar\_K\_V we want to read three
columns from the **PRIOR\_SOURCE** data file – “bins\_k” and “bins\_v”
to specify the bins for K and V, and “p\_k\_v” to specify the
probabilities for sampling these bins. In the definition of Emp\_SIGMA
we define the columns of the **PRIOR\_SOURCE** data file to use as the
basis of the empirical sampling distribution of SIGMA.

In the **INPUT\_PRIOR\_DATA** block we specify how the defined
**PRIOR\_SOURCE** information is to be mapped to vectors and matrices
defined in the **PRIOR\_VARIABLE\_DEFINITION** block and used in the
definition of the sampling distributions. Note that in the
**PRIOR\_VARIABLE\_DEFINITION** block we must define the type of the
objects p\_k\_v, bins\_k\_v and data\_SIGMA2 (vector, matrix and vector
respectively). POP\_k\_v is then sampled from a multivariate
non-parametric sampling distribution with bins specified by the matrix
bins\_k\_v and sampling probabilities by the vector p\_k\_v, while
POP\_SIGMA2 is sampled from an empirical sampling distribution with
values held in the vector data\_SIGMA2.

