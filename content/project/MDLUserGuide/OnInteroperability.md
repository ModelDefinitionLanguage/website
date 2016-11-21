Interoperability Guide
======================

On interoperability
-------------------

A key goal of the DDMoRe project is to have an intoperability framework
in which models are written in a consistent language, translated to
PharmML and from there converted to target software code. Before the
DDMoRe project no existing language standard existed across target
software used in pharmacometrics modelling, and while the underlying
models could be expressed consistently in mathematical and statistical
terms, the implementation of any given model varied by tool and by user
according to their experience with a given target software tool.

There is some flexibility within MDL around how the user can express the
mathematical and statistical models. Having flexibility allows the user
to encode models quickly in a common language (MDL) which can then be
shared with others and mutually understood. This flexibility also
facilitates encoding in a given target when that language construct does
not have a parallel in other tools. However, we STRONGLY encourage the
user to encode the majority of models in a way that will facilitate
interoperability. There are MDL constructs that facilitate
interoperability – these generally appear as built-in functions which
translate to specific constructs in PharmML and the target software.
These constructs cover many typical models and are designed to allow the
user to generate code quickly and have high confidence that it will be
interoperable across tools.

The Model Description Language Interactive Development Environment
(MDL-IDE) should assist the user in ensuring that the models encoded are
valid MDL (and as a consequence, also valid PharmML). Not all models
will result in code which can be readily converted to all target tools.

These interoperability constructs will be highlighted in the subsequent
sections, but users should pay particular attention to sections on the
use of GROUP\_VARIABLES, INDIVIDUAL\_VARIABLES and the
MODEL\_PREDICTION.

Dataset conventions
-------------------

There are a number of conventions in preparing data for use in MDL and
for target software.

-   It is assumed that the **SOURCE** data file will be present as an
    ASCII comma-delimited text file (.csv).

-   The data file should have a header row with names matching those in
    the **DATA\_INPUT\_VARIABLES** block.

-   Data values should be numeric.

-   Data columns with string or date:time values should have “use is
    ignore” for MDL.

-   Null or missing values should be denoted by “.”.

Generally speaking, MDL follows NONMEM dataset conventions.

In addition the following restrictions should be observed for
interoperability reasons:

-   A column with “use is id” is mandatory. Values should be positive,
    non-zero integer, unique and contiguous.

-   A column with “use is idv” is mandatory. Values should be positive,
    real. When the model is expressed using **DEQ** or **COMPARTMENTS**
    block the values must be monotonic increasing within ID. This
    constraint does not apply to analytical models. The first idv value
    is taken as the initial time for the model. The initial value does
    not need to be the same for all individuals, but it must not be
    lower than that of the first individual. date:time format is not
    supported for time.

-   A column with “use is dv” is mandatory. This column can be any real
    value. This must have a null value for dosing records.

-   When modelling multiple outcomes a column with “use is dvid” is
    required. Values should be positive, integer. Values should not be
    null for observation records.

-   A column with “use is mdv” is optional. Valid values are 0
    (observed),1 (missing). When the observation is null or missing this
    column should have the value 1. It can take the value 1 when
    observations are present if this observation is to be ignored.

-   A column with “use is evid” is optional. Valid values are 0
    (observation record), 1 (dosing record), 4 (reset and dose record).

-   A column with “use is amt” is optional. For dosing records this
    column must be have positive, real value.

-   A column with “use is rate” is optional. This column must have
    positive, real values. This column can only be used in combination
    with a column with “use is amt”. If the value is zero then a bolus
    dose is assumed.

-   A column with “use is addl” is optional. This column must have
    positive, real values. This column can only be used in combination
    with columns with “use is amt” and “use is ii”.

-   A column with “use is ii” is optional. This column must have
    positive, real values. This column can only be used in combination
    with columns with “use is amt” and “use is addl” or “use is ss”.

-   A column with “use is ss” is optional. Valid values are 0 (not at
    steady state), 1 (at steady state). This column can only be used in
    combination with columns with “use is amt” and “use is ii”.

-   A column with “use is cmt” is optional unless there is more than one
    route of administration. This column must have positive, integer
    values. Values in this column should start at 1 and correspond to
    the order of ODEs specified in the **DEQ** block of the Model
    Object.

-   Columns with “use is covariate”, “use is catCov” and “use is
    variable” are optional. These columns must not have missing values.
    Columns with “use is catCov” must have integer values. Covariate
    names in the **DATA\_INPUT\_VARIABLES** block must match the same
    name (including matching case) as the header name in the source file
    .csv. Please see sections 2.2.5, 4.3 and 4.9 for details on the use
    of these variables. If the column has “use is covariate” or “use is
    catCov” but this variable is not declared in the **COVARIATES**
    block of the Model Object then it will be “dropped” and ignored.

-   Columns with “use is catCov” can only have values 0,1. This implies
    that categorical covariates with k values should be converted to k-1
    indicator variables.

-   A column with “use is varLevel” is optional. This column should not
    have missing values. Columns with this type should not have an
    underscore in the column name. The change in value of this variable
    denotes when to sample new values of the random variable.

Multiple uses of dataset columns
--------------------------------

Dataset columns cannot have multiple uses defined in
**DATA\_INPUT\_VARIABLES**. The **DATA\_DERIVED\_VARIABLES** block can
be used to specify additional uses for dataset variables. In the current
MDL, scope for using **DATA\_DERIVED\_VARIABLES** is limited.

For example, if the user wants to specify different outcomes /
observations conditional on a dataset variable like CMT, i.e. using CMT
as DVID then they will need to create a dataset variable DVID mapping
into CMT values appropriately.

Defining constants in the model 
--------------------------------

For interoperability, constant values in the model should be defined as
STUCTURAL\_PARAMETERS and fixed to a value in the Parameter Object.

For models expressed as systems of differential equations (**DEQ**
block), model parameters can be set to constant values in the
**MODEL\_PREDICTION** block, but this may be inefficient in the target
software translation.

In the current SEE, to ensure interoperability, model parameters should
not be assigned a constant value in the **GROUP\_VARIABLES** or
**INDIVIDUAL\_VARIABLES** block.

**MODEL\_PREDICTION** block
---------------------------

To ensure Monolix interoperability, any variable used in the
MODEL\_PREDICTION block must be either:

• the independent variable

• defined in MODEL\_PREDICTION

• declared in INDIVIDUAL\_VARIABLES using {type is linear, … }

• defined as “use is variable” in the DATA\_INPUT\_VARIABLES block of
the Data Object

This implies in particular that STRUCTURAL\_PARAMETERS,
VARIABILITY\_PARAMETERS, GROUP\_VARIABLES and random variables defined
in RANDOM\_VARIABLES\_DEFINITION cannot be used in MODEL\_PREDICTION.

**OBSERVATION** block
---------------------

For Monolix interoperability, different observations / outcomes must not
share VARIABILITY\_PARAMETERS and RANDOM\_VARIABLE\_DEFINITIONS.

For interoperability with Monolix, the residual error(s) σ^2^~ij~ must
be Normal(0,1) random variables.

