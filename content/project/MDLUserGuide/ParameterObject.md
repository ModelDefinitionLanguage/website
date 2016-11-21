The Parameter Object
====================

The Parameter Object defines model parameter values for use with the
Model Object. In estimation tasks, these are typically the initial
values for the estimation algorithm, or fixed parameter values within
the model.

The Parameter Object should provide a value for each parameter listed in
the Model Object **STRUCTURAL\_PARAMETERS** and
**VARIABILITY\_PARAMETERS** blocks.

**STRUCTURAL** and **VARIABILITY** parameter blocks are kept separate to
allow the user to quickly identify the function of each parameter in the
model and to facilitate certain tasks, for example fixing variability
parameters for simulation.

<span id="_Toc303739536" class="anchor"><span id="_Toc387511065" class="anchor"><span id="_Toc405020429" class="anchor"><span id="_Toc459299501" class="anchor"></span></span></span></span>**STRUCTURAL** Block
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The **STRUCTURAL** block defines the numerical values of the structural
parameters with optional constraints (low and high values) and whether
the value is fixed or to be estimated. Each structural parameter must
have the **value** argument assigned a numeric value.

For each structural parameter the typical construct will be

> \<PARAMETER NAME\> : { **value** = \<numeric\> }

Or, with additional optional attributes

> \<PARAMETER NAME\> : { **value** = \<numeric\>, **lo** = \<numeric
> (lower bound)\>, **hi** = \<numeric (upper bound)\>, **fix** = \<true
> | false\> }

This provides a numerical value for a parameter which may be used as an
initial estimate for estimation or as a value for simulation. Numerical
values may be expressed in scientific notation.

The **lo** and **hi** attributes are optional and are used to define
lower and upper boundaries for estimation.

The **fix** attribute is optional. It may be set to a logical value of
true or false. The default value of **fix** is false. When **fix** is
true the parameter will not be estimated in an estimation task.
Specifying “**fix** = true” overrides any setting of **lo** and **hi**.

> **STRUCTURAL** {
>
> POP\_CL : { **value** = 0.1, **lo** = 0.001 }
>
> POP\_V : { **value** = 8, **lo** = 0.001 }
>
> POP\_KA : { **value** = 0.362, **lo** = 0.001 }
>
> POP\_TLAG : { **value**=1, **lo**=0.001 }
>
> BETA\_CL\_WT : { **value** = 0.75, **fix** = true }
>
> BETA\_V\_WT : { **value** = 1, **fix** = true }
>
> ~~RUV\_PROP : { **value** = 0.1, **lo** = 0 }~~
>
> ~~RUV\_ADD : { **value** = 0.1, **lo** = 0 } ~~
>
> } \# end STRUCTURAL

### Note on parameter values

It is typical to specify log-Normal distributions for parameters, but
the user should be aware that in some models, parameters may be
negative. As with other languages, the user should be careful to avoid
parameterisations that would lead to taking logs of a negative number.

<span id="_Toc303739538" class="anchor"><span id="_Toc387511067" class="anchor"><span id="_Toc405020431" class="anchor"><span id="_Toc459299503" class="anchor"></span></span></span></span>**VARIABILITY** Block
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The **VARIABILITY** block defines the names and values of random effect
parameters (including covariance or correlation parameters) that are to
be used in the Model Object. Similar to the **STRUCTURAL** block above,
the **VARIABILITY** block provides initial values for estimation. Each
variable must have the **value** argument assigned a numeric value.

~~The **VARIABILITY** block has a more complex structure because it
needs to express both values for variance parameters, but also any
correlations or covariances between these variance parameters (random
effects).~~

~~Currently, MDL supports definition of the VARIABILITY (random effect)
parameters and definition of the covariance or correlation between these
parameters (see section 3.2.1).~~

Similar to the STRUCTURAL block, the VARIABILITY block requires
attributes for each random effect used in the model.

For each random effect parameter the typical construct is

> \<PARAMETER NAME\> : { **value** = \<numeric\> ~~, **type** is
> \<**sd** | **var**\>~~ }

With additional **fix** attribute

> \<PARAMETER NAME\> : { **value** = \<numeric\>, ~~**type** is \<**sd**
> | **var**\>,~~ **fix** = true }

~~The **type** argument specifies whether the initial values ***and***
parameter estimation are specified on the standard deviation scale.
**The parameter value type must correspond to the type used in the Model
Object RANDOM\_VARIABLE\_DEFINITION block.** ~~

Within the Parameter Object **VARIABILITY** block, we no longer need to
specify the type of variability (variance or sd). The Parameter Object
simply defines values for the parameters used in the Model Object
**RANDOM\_VARIABLE\_DEFINITION** block. The user then needs to ensure
that the parameter values are on the appropriate scale.

**Note: that in the version of PsN used in the current version of the
SEE, bootstrap estimates of variability parameters are not available on
the standard deviation scale. Returned variability parameters from
bootstrap estimation will be on the *variance* scale.**

An example **VARIABILITY** block:

> **VARIABILITY** {
>
> PPV\_CL : { **value** = 0.1~~, **type** is **sd**~~ }
>
> PPV\_V : { **value** = 0.1~~, **type** is **sd**~~ }
>
> CORR\_CL\_V : { **value** = 0.01 }
>
> PPV\_KA : { **value** = 0.1~~, **type** is **sd**~~ }
>
> PPV\_TLAG : { **value** = 0.1, ~~**type** is **sd**,~~ **fix**=true }

RUV\_PROP : { **value** = 0.1, **lo** = 0 }

> RUV\_ADD : { **value** = 0.1, **lo** = 0.0001 }
>
> } \# end VARIABILITY

Note that parameter estimates for residual errors e.g. RUV\_PROP and
RUV\_ADD above are now specified as **VARIABILITY** parameters, and not
**STRUCTURAL**. Typically, these parameters are defined as multipliers
of standard Normal(0,1) random variables in definition of the residual
error model. The purpose of the **VARIABILITY** block definition in the
Parameter Object is to make it easier to identify those parameters
associated with variability and if required “turn off” variability by
fixing these parameters to zero. Placing the residual error parameters
in the **STRUCTURAL** parameter block made this difficult, since these
parameters can have arbitrary names. Nevertheless, the models are
equivalent whether residual errors are defined by parameters specified
in the STRUCTURAL or in the VARIABILITY block.

Note also that correlations between parameters are given parameter
values in the **VARIABILITY** block, but definition of correlation and
covariance now occurs in the Model Object
**RANDOM\_VARIABLE\_DEFINITION** block.

When defining values for multivariate distributions, the user may need
to define vectors and matrices to define the mean and covariance matrix
(respectively). To do so the user defines a list with the following
syntax:

To define a vector of length k:

\< VARIABLE NAME \> : {vectorValue = [ \<value1\>, \<value2\>, …
\<valuek\>] }

The square brackets denote that the result is a vector.

To define a matrix of size n rows by p columns:

\< VARIABLE NAME\> = [[ \<value\_1\_1\>, \<value\_1\_2\>, … ,
\<value\_1\_p\>;

\<value\_2\_1\>, \<value\_2\_2\>, …, \<value\_2\_p\>;

…

\<value\_n\_1\>, \<value\_n\_2\>,…, \<value\_n\_p\>]]

Note the double square brackets to define the matrix type, comma
separated values to signify individual elements and semi-colon to
specify the end of a row.

Alternatively to create a matrix, it is possible to use functions
“diagonal”, “triangle”, “matrix”. These take a vector as input and
return a matrix.

For example (UseCase6\_2.mdl):

**VARIABILITY** {

PPV\_CL\_V\_KA : {matrixValue = triangle([0.1,

0.01, 0.1,

0.01, 0.01, 0.1], 3, true)}

…

} \# end VARIABILITY

### <span id="_Toc459299504" class="anchor"><span id="_Ref433618161" class="anchor"></span></span>Parameter naming

Unlike some target software, MDL does not have reserved names for
parameters, nor is any meaning extracted from parameter names.

In the MDL documentation, we have used the convention that variability
parameters describing the population parameter variability from the
combination of between subject and within subject (between occasion)
random effects are named PPV\_. The individual level random effects
we’ve named ETA\_ since this is a familiar convention for many analysts.
The residual unexplained variability parameters have been named RUV\_
and the random variable associated with these has been named EPS\_ again
to following a familiar convention.

### ~~Covariances and Correlations~~

~~Random variability parameters and any covariances or correlations are
defined separately, rather than as a combined matrix. ~~

~~The covariance (or correlation) between random effects is defined as
follows:~~

~~\<PARAMETER NAME\> : { **parameter** = \<vector of random effect
variables\> , **value** = \<vector of values\>, **type** is \<**cov** |
**corr**\> }~~

~~The random effects variables must be declared in the
**DECLARED\_VARIABLES** block within the Parameter Object so they can be
mapped to the random effect variables in the Model Object.~~

**~~Note that value expects a vector, so even if a single correlation is
specified (one value) then this must be enclosed in square brackets to
signify this is a vector with one element.~~**

~~So for a simple example where the between subject variance parameters
for CL, V and KA are on the standard deviation scale and the correlation
between these parameters is to be specified, the standard deviation -
correlation matrix (standard deviation on the diagonal, correlation off
diagonal) is given by ~~

$$\begin{bmatrix}
PPV\_ CL \\
PPV\_ V \\
PPV\_ KA \\
\end{bmatrix} = \ \begin{bmatrix}
sd = \ 0.1 & 0.01 & 0.01 \\
\mathbf{corr = 0.01} & sd = \ 0.1 & 0.01 \\
\mathbf{corr = 0.01} & \mathbf{corr = 0.01} & sd = \ 0.1 \\
\end{bmatrix}$$

~~And the corresponding MDL code is:~~

> ~~warfarin\_PK\_CORR\_par = parObj {~~
>
> ~~**DECLARED\_VARIABLES**{ ETA\_CL ETA\_V ETA\_KA}~~
>
> ~~**STRUCTURAL** {~~
>
> ~~…~~
>
> ~~} \# end STRUCTURAL~~
>
> ~~**VARIABILITY** {~~
>
> ~~PPV\_CL : {**value**=0.1, **fix**=true, **type** is **sd**}~~
>
> ~~PPV\_V : { **value** = 0.1, **type** is **sd** }~~
>
> ~~PPV\_KA : { **value** = 0.1, **type** is **sd** }~~
>
> ~~PPV\_TLAG : { **value** = 0.1, **type** is **sd**, **fix**=true } ~~
>
> ~~\# correlation between CL, V, KA~~
>
> ~~OMEGA1 : {**type** is **corr**, **parameter**=[ETA\_CL, ETA\_V,
> ETA\_KA], ~~
>
> ~~**value**=[0.01, 0.01, 0.01]}~~
>
> ~~} \# end VARIABILITY~~
>
> ~~} \# end of parameter object ~~

~~In the code above, the variable OMEGA1 is defined as the lower
triangle of the matrix above (correlation entries only) and three values
are required to define the correlations between the parameters.
Specifying the between subject variability parameters separately from
covariances and correlations allows the user to change the covariance or
correlation structure independently of the other variance parameter
definitions.~~

**~~If any VARIABILITY block variable associated with the correlation or
covariance list definition has the attribute fix=true, then all diagonal
and off-diagonal elements of the standard deviation – correlation matrix
will be assumed to have the attribute fix=true.~~**

~~Note that the parameters correlated are the random effects rather than
the parameters defining the distribution of the random effects. Thus it
is these random effect variables that are declared in the
**DECLARED\_VARIABLES** block.~~

**~~In the current version of MDL, if a covariance or correlation is
specified between three or more parameters then all elements of that
covariance or correlation must be estimated. ~~**

