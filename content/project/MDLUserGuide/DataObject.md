The Data Object
===============

The Data Object defines the attributes of the source data file and
defines how values in the data source are to be used in the context of a
given Model Object. The Data Object is independent of other blocks,
including the Model Object. It must ultimately provide appropriate
information *to* the Model Object, but as with other Objects in MDL it
must be self-contained, so variables from other Objects which are
referenced must be declared in the `DECLARED_VARIABLES` block.

The following blocks are defined within the Data Object:
`DECLARED_VARIABLES`, `DATA_INPUT_VARIABLES`, `SOURCE`,
`DATA_DERIVED_VARIABLES`, `FUNCTIONS`.

Subsetting data for analysis
----------------------------

For quality assurance and audit purposes it is imperative that there be
a clear and traceable path between the original data source and data
used in analysis. This is normally achieved in one of two ways: through
having data manipulation steps performed in a scriptable language to
create the dataset for use in analysis, or through having the original
data as the input to the analysis and using filtering and subsetting
commands in the target analysis software code to define what records are
used in the task.

**The “NONMEM” method for dealing with outliers and filtering data –
commenting out data rows using a specific character as the first item on
a data line or specifying conditions under which data is accepted or
ignored – is not supported within MDL**.

We suggest that users use scriptable languages like R to subset, filter
and manipulate data prior to analysis, write the data for analysis to
file and then modify the MDL Data Object to reference the appropriate
source file in the `SOURCE` block. This aids reproducibility since the
data used in estimation will be equivalent across target software tools.
If the “ignored” or “dropped” data is saved separately in a file or
listing then it will also be possible to quickly and easily verify that
the data used in analysis, when combined with the dropped data matches
the original dataset.

For legacy data and NMTRAN models, we recommend using the “ignored”
function within the “metrumrg” R package
(<https://r-forge.r-project.org/projects/metrumrg/>). This function
reads a NONMEM control stream and creates a TRUE / FALSE logical flag
for whether the records meet IGNORE and/or ACCEPT criteria specified in
the NMTRAN code. This will allow the user to identify which data records
have been dropped by NONMEM. Filtering on the original data based on
this criteria will allow them to create a dataset ready for analysis
with MDL.

For example, if the NMTRAN control file had the following \$DATA
statement:

~~~
$DATA mx2007.csv IGNORE=@

IGNORE ID.EQ.1

ACCEPT VISI.EQ.3
~~~

This code means that any data rows which start with “@” are to be
omitted, that the subject with ID == 1 should be omitted from analysis,
and only VISI == 3 is to be included.

Using the metrumrg function ignored, we have the following code which
can be used within the R script. This assumes that the \$DATA statements
above are in a control file called “run1.ctl”:

~~~
library(metrumrg)

mx2007 <- read.csv("mx2007.csv", header=T)

mx.dropped <- mx2007[ignored(ctlfile="run1.ctl"),]

mx.kept <- mx2007[!ignored(ctlfile="run1.ctl"),]
~~~
This separates out the dropped records into the mx.dropped data frame,
and the kept records into the data frame mx.kept. The user can then
write mx.kept to a .csv file and use this as the file in the `SOURCE`
block.

`DATA_INPUT_VARIABLES` Block
------------------------------------------------------------------------------------------------------------------------------

This block defines the columns in the dataset described in `SOURCE`
block and how these map to model variables.

-   All columns of the data file defined in the `SOURCE` block must be
    defined in `DATA_INPUT_VARIABLES.` This aids clarity and
    readability.

-   **In the current MDL , variables in the DATA_INPUT_VARIABLES block
    must be defined *in the column order they appear in the* SOURCE
    *block data file*.**

-   **In the current MDL, variable names in the DATA_INPUT_VARIABLES
    block must *match the names in the header row of the data file*
    named in the SOURCE block. For interoperability with Monolix the
    case of the names in the header row must match the case of the data
    variable in the DATA_INPUT_VARIABLES block.**

-   `DATA_INPUT_VARIABLES` cannot have more than one `use`
    defined.

-   All `DATA_INPUT_VARIABLES` must have a `use` defined.

-   Columns in the data which are not required in the model should have
    “use is ignore”.

The typical syntax for defining items in the `DATA_INPUT_VARIABLES`
block is:

~~~
<Variable name> : { use is <use type> }
~~~

Define types for `use type` are:

  Use         **Defines**
  ----------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  id          Individual identifier. Typically subject ID in clinical trials. Defines the indvidual level of parameter variability.
  idv         Independent variable. Typically TIME. In the Model Object, the reserved variable T is used as the integrator variable in the specification of differential equations.
  amt         Dose amount
  dv          Dependent variable
  varLevel    Defines a level of variability in the model. Not required for `DATA_INPUT_VARIABLES` with `use` is `id` or `use` is `dv` . Used to define all other variability levels such as interoccasion variability.
  covariate   Covariate for use in definition of fixed effect variable
  catCov      Categorical covariate for use in definition of fixed effect variables based on categories
  variable    Defines any model input not covered by other types. Corresponds to the “regressor” variable type in Monolix.
  dvid        Dependent variable identifier when there is more than one type of observation.
  evid        Event identifier. This type is defined in MDL, but not yet implemented by converters in the current SEE.
  mdv         Missing dependent variable
  cmt         Compartment identifier
  ss          Steady State indicator
  ii          Inter-dose interval i.e. time between repeated doses
  addl        Number of additional doses in repeated dose administration
  rate        Zero-order input rate
  ignore      Ignores a data variable

The `use` types correspond to those used by NONMEM and Monolix.
`varLevel` corresponds to OCC in Monolix, `catCov` corresponds to
CAT in Monolix. (NMTRAN does not have equivalent reserved words for
these types of variables – their use is implicit in the implementation
of the model).

Use “`use is` `covariate`” and “`use is catCov`” for data
variables that are to be used in “linear” definition of
`INDIVIDUAL_VARIABLES` (see section 4.9). Note that covariates and
categorical covaraites used in “linear” definitions within the
`INDIVIDUAL_VARIABLES` block should not vary with time, although they
may vary by occasion.

Use “`use` is `variable`” for data variables that are to be used
directly in the calculations within the `MODEL_PREDICTION` block –
for example, individualised PK parameter predictions for use in a PD
model. These may be time-varying covariates or inputs to the
`MODEL_PREDICTION` equations.

“`use` is `variable`” corresponds to the “regressor” variable type
in Monolix.

MDL does not have “reserved” names for variables in the Model Object
other than the default for the independent variable T. The intended use
for variables is defined via the “`use` is …” attribute as described
above. The choice of names for DATA_INPUT_VARIABLES should be
meaningful to the user and clear for any third party reading the code.

**~~However, within the current Standalone Execution Environment (SEE)
interoperability framework, certain names for DATA_INPUT_VARIABLES are
required to facilitate translation to NONMEM, Monolix and on to
downstream R packages such as Xpose. It is expected that future versions
will relax these constraints. The variables under constraint are AMT,
DOSE, TIME and DV. See below for more information.~~**

### Defining dose amount

The dosing amount is defined through `DATA_INPUT_VARIABLES` with
“`use is amt`”. It is assumed that when the column has a non-zero /
non-missing value, this amount is assigned to the relevant Model Object
dosing variable.

The Model Object variables to which dose is assigned should be declared
in the `DECLARED_VARIABLES` block and should have type
::dosingTarget. This applies equally when doses are assigned to
differential equation variables (UseCase1), PK input compartments with
“`type is depot`” or “`type is direct`” (UseCase4) or variables in
analytical models (UseCase2).

The syntax for defining the dose amount is:

~~~
<Variable> : { use is amt, variable = <mdlObject variable>}
~~~

For example:
~~~
> AMT : { use is amt, variable = GUT }
~~~

**~~For the current version of the interoperability framework SEE, the
name of this variable must be AMT or DOSE.~~**

Within the Model Object, the dose amount is defined when AMT > 0 if the
`DATA_INPUT_VARIABLES` variable is defined as “`use is amt`”. It
is not necessary to conditionally assign a Model Object variable to this
value when AMT > 0. This is taken care of in translation to PharmML
behind the scenes. If, however, the user chooses to treat the dose
amount column as “`use is covariate`” or “`use is variable`” then
this **will** need conditional assignment within the model. Also, in
that case the dosing amount variable should be declared in the
`COVARIATES` block of the Model Object.

For analytical models (such as UseCase2) the dosing amount may be
defined with respect to a dosing variable in the Model Object, rather
than as an initial amount in a differential equation or compartment. In
that case it is necessary to declare the dosing variable within the
`MODEL_PREDICTION` block:

The declared dosing variable for analytical models should have type
::dosingTarget. This maps into variables of type ::dosingVar and
::dosingTarget in the Model Object. Examples of ::dosingTarget are
COMPARTMENT variables with “type is depot”, “type is direct” and also
variables defined by differential equations.

For example:

In the Data Object:
~~~
> DECLARED_VARIABLES{ D::dosingTarget }
>
> DATA_INPUT_VARIABLES{
>
> AMT : { use is amt, variable=D }
>
> }
~~~

In the Model Object:

~~~
> MODEL_PREDICTION {
>
> D :: dosingVar # dosing variable
>
> k = CL/V
>
> CC = if ( T < TLAG) then 0
>
> else (D/V) * KA/(KA-k) * (exp(-k * (T - TLAG))- exp(-KA*(T-TLAG))
> )
>
> } # end MODEL_PREDICTION
~~~

MDL supports definition of multiple doses via `DATA_INPUT_VARIABLES`
defined as “`use is ii`”, “`use is addl`”, “`use is ss`”.

MDL supports infusion rate (zero-order input rate) specification via
`DATA_INPUT_VARIABLES` with “`use is rate`”. **The current version
of MDL does not support negative values for “use is rate” in order to
allow model defined rate or duration.**

Please also read the section 2.2.7 and 2.2.8 on assignment using
“`variable` = …” compared to using “`define` = …”.

### Defining the independent variable

The independent variable is defined through a `DATA_INPUT_VARIABLES`
with “`use is idv`”. The Model Object has an `IDV` block where the
independent variable in the model is defined and the model variable
defined in this block is automatically mapped to the Data Object
variable defined as “`use is idv`”. This is used to link the model
independent variable in the Model Object `IDV` block and the event
(observation, dosing) times specified in the `DATA_INPUT_VARIABLES`.

Syntax:
~~~
<Variable> : { use is idv }
~~~

Typically the independent variable will be time. UseCase11 shows a
pharmacodynamic model with the plasma concentration as the independent
variable.

**~~For the current version of the SEE, the name of the independent
variable must be TIME.~~**

### Defining the dependent variable

The dependent variable is defined through a `DATA_INPUT_VARIABLES`
with “`use is dv`”. This data variable is the observation and must
be mapped to the Model Object `OBSERVATION` block prediction variable
using the Data Object DECLARED_VARIABLES block

**~~For the current version of the interoperability framework SEE, the
name of the dependent variable must be DV.~~**

#### Continuous data, single Model Object OBSERVATION prediction

If there is only one observation type and it is continuous then the user
should map the `DATA_INPUT_VARIABLES` “`use is dv`” variable to a
prediction variable name in the Model Object `OBSERVATION` block. It
is possible to map the dependent variable to a single, specified Model
Object `OBSERVATION` prediction variable using “`variable = <NAME>`”

The Model Object `OBSERVATION` block prediction variable name must be
declared in the Data Object `DECLARED_VARIABLES` block.

The observation variable in the `DECLARED_VARIABLES` block must have
type ::observation.

For example:

In the Data Object:
~~~
> DECLARED_VARIABLES{ Y::observation }
>
> DATA_INPUT_VARIABLES{
>
> …
>
> DV : { use is dv, variable = Y }
>
> …
>
> }
~~~
In the Model Object:

~~~
> OBSERVATION{
> 
> Y : {type is additiveError, additive=SD_ADD, eps=EPS_Y, prediction= CONC}
> 
> }# end OBSERVATION
~~~

#### Multiple Model Object OBSERVATION predictions

If mapping the single dataset dependent variable column (with “`use is
dv`”) to multiple Model Object `OBSERVATION` prediction variables, it
is necessary to also define a `DATA_INPUT_VARIABLE` with “`use is
dvid`” that identifies which records of the dataset belong to each
observation type. The user must also define how to map the Model Object
`OBSERVATION` block variables to values in the
`DATA_INPUT_VARIABLE` with “`use is dvid`”.

The syntax is as follows `define = {<value> in <data column name with 
“use is dvid”> as <DECLARED_VARIABLE variable>, etc.}`.

For example (UseCase3):

In the Data Object:
~~~
> DECLARED_VARIABLES{ CP_obs::observation PCA_obs::observation }
>
> DVID : { use is dvid }
>
> DV : { use is dv, define={1 in DVID as CP_obs, 2 in DVID as PCA_obs} }
~~~

In the Model Object:
~~~
> OBSERVATION{
> 
> CP_obs : {type is combinedError1,
> 
> additive = RUV_ADD, proportional = RUV_PROP,
> 
> eps = EPS_CP, prediction = CC }
> 
> PCA_obs : {type is additiveError, additive = RUV_FX, eps = EPS_PCA, prediction = PCA }
> 
> }\# end OBSERVATION
~~~

This means when the data variable with “`use is dvid`” has the
value 1 then the observation in the data variable with “`use is dv`”
***within the same data record*** will be mapped to the Model Object
`OBSERVATION` block variable CP_obs, and when this variable has the
value 2 it will be mapped to PCA_obs.

All continuous observation variables must be declared in the
`DECLARED_VARIABLES` block as ::observation.

When declaring an outcome variable that is binary (UseCase12) or
categorical (UseCase13), it is necessary in the DECLARED_VARIABLES
block to define the category name for the Model Object variables which
define the prediction.

For example (UseCase12):

In Data Object:

`DECLARED_VARIABLES{ Y withCategories {none, event} }`

Using this convention, the type is implicit in the “withCategories”
keyword.

Note that when declaring an outcome variable with a Poisson count or
Binomial number of successes outcome, Y is declared as the variable
attribute of an anonymous OBSERVATION block list.

For example (UseCase11):

In the Data Object:

~~~
> DECLARED_VARIABLES{ Y::observation }
~~~

In the Model Object:
~~~
> RANDOM_VARIABLE_DEFINITION(level = DV){
> 
> Y ~ Poisson1(rate = LAMBDA)
> 
> }
> 
> OBSERVATION{
> 
> :: {type is count, variable = Y}
> 
> }# end ESTIMATION
~~~

### Mapping data variables to model variability levels 

The hierarchy of model levels of variability is defined within the Model
Object in the `VARIABILITY_LEVELS` block. Within the Data Object we
match the levels in the model to DATA_INPUT_VARIABLES to identify the
data variables where changing values signify new individuals, occasions
or observations (and other levels of variability in the model). By
default the lowest level of the hierarchy is the observation level with
`DATA_INPUT_VARIABLES` defined as “`use is dv`”. The other
variability level commonly used is the experimental unit, in clinical
trials this is typically the subject with `DATA_INPUT_VARIABLES`
defined as “`use is id`”.

The level in the hierarchy is defined by a numerical level value in the
Model Object VARIABILITY LEVELS block.

For example (UseCase1):

In Data Object:
~~~
> DATA_INPUT_VARIABLES{
> 
> ID : { use is id }
> 
> …
> 
> DV : { use is dv, variable = Y }
> 
> …
> 
> }\# end DATA_INPUT_VARIABLES
~~~

In the Model Object:
~~~
> VARIABILITY_LEVELS{
> 
> ID : { level =2, type is parameter }
> 
> DV : { level =1, type is observation }
> 
> }
~~~

Other variability levels may be defined in `DATA_INPUT_VARIABLES` as
“`use is varLevel`”. This will be used to define variability levels
such as occasion, study (if modelling across more than one study) etc.

For example, when defining occasions for use in between occasion
variability models:

`OCC : { use is varLevel }`

For example (UseCase8):

In Data Object:
~~~
> DATA_INPUT_VARIABLES{
> 
> ID : { use is id }
> 
> TIME : { use is idv }
> 
> WT : { use is covariate }
> 
> AGE : { use is covariate }
> 
> SEX : { use is catCov withCategories {female when 1, male when 0} }
> 
> AMT : { use is amt, variable = INPUT_KA}
> 
> OCC : { use is varLevel }
> 
> DV : { use is dv, variable = Y }
> 
> MDV : { use is mdv }
> 
> }# end DATA_INPUT_VARIABLES
~~~

In Model Object:
~~~
> VARIABILITY_LEVELS{
> 
> ID : { level = 3, use is parameter }
> 
> OCC : { level = 2, use is parameter }
> 
> DV : { level = 1, use is observation }
> 
> }
~~~
MDL does not place any limits on the number of levels of variability
within the Model Object. However some constraints may exist within the
target software used for estimation.

Note that occasionally, if modelling summary level data in a model-based
meta-analysis, the experimental unit defined in the data may be
different, for example, the treatment arm rather than an individual
subject.

For example:

In Data Object:
~~~
> DATA_INPUT_VARIABLES{
> 
> ARM : { use is id }
> 
> DV : { use is dv, variable = Y }
> 
> }# end DATA_INPUT_VARIABLES
~~~

In Model Object:
~~~
> VARIABILITY_LEVELS{
> 
> ARM : { level=2, type is parameter }
> 
> DV : { level=1, type is observation }
> 
> }
~~~

See section 4.6 for discussion of identifying the reference level of
variability when using a model with a Design Object.

### span>Defining covariates

#### Defining continuous covariates

Continuous covariates (including time-varying covariates) are defined as
“`use is covariate`”.

Note the discussion in sections 4.3 and 4.9 about the requirements of
covariates that are to be used in definition of
`INDIVIDUAL_PARAMETERS`.

Interpolation can be specified for continuous covariates through the
argument “interp = &constInterp | &cubicInterp | &lastValueInterp |
&linearInterp | &nearestInterp | &pchipInterp | &splineInterp”.

Note the “&” in front of the interpolation type to signify that a
function is being referenced.

Interpolation functions take as arguments t0 (start of time interval),
t1 (end of time interval), x (variable for interpolation), x0 (variable
value at t0) and x1 (variable value at t1).

#### Defining categorical covariates

Categorical covariates are defined as “`use is catCov`” and must have
a mapping between the values in the data column and categories to be
used in the model. This serves a dual purpose: firstly, providing
clarity on how numeric codes in the data map to named categories and
secondly, transparency in model description by allowing us to use those
named category labels in the model when referring to the categories.

The mapping is performed by using the keyword “withCategories” and then
specifying the mapping between categories and values using <category>
when <value> in a comma separated list.

~~~
> SEX : {use is catCov withCategories {female when 1, male when 0} }
~~~

The categories defined (female, male) must match those defined within
the Model Object `COVARIATES` block. Note that the category labels
(female, male) are not character strings. They are enumerated variables,
and can be referred to in the model as SEX.female or SEX.male. For
example to define an action dependent on the SEX being female in the
Model Object we would use the Boolean comparison SEX == SEX.female. This
evaluates to true when the value in the SEX column matches the value
defined (in the `DATA_INPUT_VARIABLES` as above) that corresponds to
the female category of the SEX variable.

In the current version of MDL the categories defined must be unique i.e.
it is not possible to assign more than one value to a category, nor is
it possible to define several categories with the same name. This means
that each category maps to only one data value. The following is code
NOT valid:

~~~
> food : {use is catCov withCategories {fed when 2, fasted when 1, fasted when -999}
~~~

Nor is:
~~~
> food : {use is catCov withCategories {fed when 2, fasted when [-999,1] }
~~~

The user should identify all categories in the DATA_INPUT_VARIABLE
block:
~~~
> food : {**use is** catCov withCategories {fed when 2, fasted when 1,
missing when -999}
~~~

### Defining model inputs or time-varying covariates

Often we will want to pass data variables to the `GROUP_VARIABLES` or
`MODEL_PREDICTION` blocks in the Model Object which will not fit the
definition of covariates as defined in section 2.2.5. This might be the
case if we want to pass individualised predictions of PK parameters into
a PD model, or a time-varying covariate such as plasma concentration or
age for use in a maturation model. These variables are sometimes
referred to as regressors or model input variables.

These variables are defined in MDL as “`use is variable`”.

Variables with “`use is variable`” should not be used with definition
of INDIVIDUAL_PARAMETERS with “`type is linear`”.

### Assignment to a single variable using “variable = <NAME>”

If the value of the variable within the data is to be mapped to a single
model variable e.g. dosing amount D, then the variable attribute must be
assigned, and the associated variable declared in
`DECLARED_VARIABLES`:
~~~
> DECLARED_VARIABLES{ D::dosingTarget Y::observation }
>
> DATA_INPUT_VARIABLES{
>
> …
>
> AMT : { use is amt, variable = D }
>
> DV : { use is dv, variable = Y }
>
> …
>
> }
~~~

### Assignment to multiple variables using “define = < … >”

In the case of mapping data values to ***multiple*** variables depending
on the values of another variable, the syntax is as follows
`define = {<value> in <data variable name> as <declared_variable>, etc.}`.

The Model Object variables used in this
definition must also be declared in the `DECLARED_VARIABLES` block.

~~~
> DECLARED_VARIABLES{ CP_obs::observation PCA_obs::observation }
>
> DATA_INPUT_VARIABLES{
>
> …
>
> DVID : { use is dvid }
>
> DV : { use is dv, define={1 in DVID as CP_obs, 2 in DVID as
> PCA_obs} }
>
> …
>
> }
~~~

`SOURCE` Block
--------------------------------------------------------------------------------------------------------------

This block defines the source data file for use with the model. It
defines the file name and file format.

**In the current version of MDL it is assumed that the SOURCE data file
will be present as an ASCII comma-delimited text file (.csv). We also
assume that the dataset conforms to NONMEM data standards. The data file
should have a header row with names matching those in the
DATA_INPUT_VARIABLES block. Data values should be numeric. Missing
values should be denoted by “.”.**

The MDL syntax is as follows:

 `<source object name> : {file = <filename>, inputFormat is nonmemFormat }

For example:
~~~
> SOURCE {
>
> srcfile : {file = "warfarin_conc.csv",
>
> inputFormat is nonmemFormat }
>
> } \ end SOURCE
~~~

**For the current version of the interoperability framework SEE, data
files must be in the same folder and workspace as the model file. **

`DECLARED_VARIABLES` Block
-----------------------------

This block links variables defined in the Model Object with variables
defined within the Data Object - occasionally we need to refer to Model
Object variables while describing the data constructs. For example: When
defining the Pharmacokinetic model (UseCase1) we need to define which
Model Object variable receives the dosing amount – in this case the
differential equation specifying the amount in the GUT, and an
observation variable Y.

Since the MDL objects are independent of each other (i.e. the Data
Object is not “aware” of Model Object variables) we must explicitly
declare Model Object variables within the Data Object if we need to
refer to them.

When declaring the variable, the user must also define the variable
type. A controlled vocabulary of types is provided through the MDL-IDE.
Type is defined via the double colon:

<Variable name> :: <type>

For example: GUT :: dosingTarget Y :: observation.

This improves validation of the MOG since we can check that Data Object
`DATA_INPUT_VARIABLE` use is mapped to an appropriate type of
variable and that this passes through to the appropriate type of
variable in the Model Object. This change is also required to ensure
that the Data Object and the Design Object handle definition and
declaration of variables in an equivalent way.

The following types will be typically of use in DECLARED_VARIABLES:

dosingTarget, observation.

As has been described above, when declaring an outcome variable that is
binary (UseCase12) or categorical (UseCase13), it is necessary in the
DECLARED_VARIABLES block to define the categories for the outcome
variable.

Note that when declaring an outcome variable with a Poisson count or
Binomial number of successes outcome, Y is declared as observation.

Note that no delimiter is required between variables defined in the
`DECLARED_VARIABLES` block.

**In the current MDL, variable names mapped across MDL Objects must
match e.g. if we declare variable Y in the Data Object, then this will
be linked to the Model Object OBSERVATION block variable Y**.

DATA_DERIVED_VARIABLES Block
----------------------------------------------------------------------------------------------------------------------------

Occasionally, the user will need to define new variables that depend on
existing information in the dataset for example the dose amount or dose
time when this is to be used as a covariate in the model. The
`DATA_DERIVED_VARIABLES` block allows the user to define new
variables using data in the columns with “`use is amt`” and
“`use is idv`”. The variables defined must have unique names,
different from those specified in the `DATA_INPUT_VARIABLES`.

Syntax is

`<variable> : {use is <doseTime / covariate / variable / doseInterval>, … }`

The subsequent arguments of the list depend on its use.

~~~
<variable> : {use is doseTime, 
idvColumn = <DATA_INPUT_VARIABLE variable with “use is idv”>,
dosingVar = <dosing variable defined with type ::dosingTarget>}
~~~

**In the current MDL, “`use is doseInterval” `below cannot currently
be mapped to PharmML.**
~~~
<variable> : {use is doseInterval, idvColumn =
<DATA_INPUT_VARIABLE variable with “use is idv”>,
dosingVar = <dosing variable defined with type ::dosingTarget> }

<variable> : {use is covariate, column =
<DATA_INPUT_VARIABLE variable> }

<variable> : {use is catCov, column = <DATA_INPUT_VARIABLE
variable> }

<variable> : {use is variable, column =
<DATA_INPUT_VARIABLE variable> }
~~~

For example (UseCase2_1):
~~~
> DATA_DERIVED_VARIABLES{
> 
> # Like 'use is *amt*' we assume that the DT variable is only assigned
> when AMT > 0.
> 
> # The typing ensured that the attributes reference a column with the
> correct 'use'.
> 
> DT : { use is doseTime, idvColumn=TIME, amtColumn=AMT }
> 
> }
~~~