The Design Object
=================

Design Object overview
----------------------

The Design Object is intended to provide information for creating
simulation or evaluation of trial designs, and design space information
required for optimal design. For trial simulation or design evaluation,
the trial design is fixed and should be completely defined. For optimal
design the user should specify a design space to optimise over in the
**DESIGN\_SPACES** block.

The user should specify all required interventions (drug
administrations), sampling (or observation) schedules, populations,
covariate distributions, and design parameters. These are then combined
in the **STUDY\_DESIGN** block to completely define study design(s). For
simulation and evaluation only one design should be specified. For
optimal design, many study designs can be considered and the
**DESIGN\_SPACES** block will determine what attributes the optimisation
algorithm will search over.

An elementary design is given by the combination of INTERVENTION +
SAMPLING, and can describe the design for groups of subjects e.g.
treatment arms, or can define a unique design for an individual. In
optimal design the combination of treatment arms and numbers of subjects
in each is called the population design.

**DECLARED\_VARIABLES** block
-----------------------------

As with the **DECLARED\_VARIABLES** block in other MDL objects, the
block should contain any variables defined in other MDL Objects
(typically the Model Object) that are required for complete definition
of the Design Object. See section **Error! Reference source not found.**
for further details.

**INTERVENTION** block
----------------------

The **INTERVENTION** block contains details of administration schedules
and types and how these are combined to form treatment interventions.
Note here that we distinguish between treatment interventions (including
placebo treatment, non-medical treatments, drug-free periods) and
treatment arms, since an arm of a study may be purely observational i.e.
with no treatment intervention. For the optimisation task of optimal
design, the **INTERVENTION** block defines the treatment definitions for
the starting design (if required by the algorithm).

The syntax for this block is:

\<name\> : { **type is** **bolus**,

**input** = \<Model Object dosingTarget\>,

**amount** = \<real value or vector of real values\>,

**doseTime** = \<vector of real values\>,

***ssInterval** = \<real value\>,*

***timeLastSSDose** = \<real value\>,*

***doseIntervalVar** = \<dosing interval variable reference\>,*

***lastDoseTimeVar** = \<time of last dose variable reference\>*

}

In the above, ssInterval and timeLastSSDose are optional;
doseIntervalVar, lastDoseTimeVar are optional.

\<name\> : { **type is infusion**,

**input** = \<Model Object dosingTarget\>,

**amount** = \<real value\>,

**rate** = \<vector of real values \>,

**doseTime** = \<vector of real values\>,

**duration** = \< vector of real values \>,

***ssInterval** = \<real value\>,*

***timeLastSSDose** = \<real value\>,*

***timeStopSSInfusion** = \<real value\>,*

***doseIntervalVar** = \<dosing interval variable reference\>,*

***lastDoseTimeVar** = \<time of last dose variable reference\>*

}

In the above, one of duration or rate ***must*** be present. If doseTime
is not present, then timeLastSSDose or timeStopSSInfusion ***must*** be
present.

\<name\> : { **type is combi**,

**combination** = \<vector of previously defined interventions\>,

**start** = \<vector of start times\>,

**end** = \<vector of end times\>

}

The “combi” type is used to combine existing (already defined)
intervention schedules to form more complex dosing regimens.

\<name\> : {**type is reset**,

**reset** = { **variable** = \<Model Object dosingTarget variable\>,

**resetTime**=\<real value\>,

**value**=\<real value\>}

}

The “reset” type is used to reset the dosing variable in the Model
Object at a given time.

\<name\> : {**type is resetAll**}

The “resetAll” type is used to reset all dosing variables.

For example, defining a single oral administration to the GUT at time 0
for use with the warfarin example (/Design/UseCase1\_design1\_eval):

**INTERVENTION**{

admin1 : {**type** **is** **bolus**, **input**=GUT, **amount**=100,
**doseTime**=[0] }

}

Another example showing single dose administration with an oral dose to
the GUT and an infusion to the CENTRAL compartment
(UseCase4\_design1\_eval):

**INTERVENTION**{

admin1 : {**type** **is** **infusion**, **input**=CENTRAL,
**amount**=100, **doseTime**=[0], **duration**=100 }

admin2 : {**type** **is** **bolus**, **input**=GUT, **amount**=150,
**doseTime**=[0] }

}

A more complex example showing how interventions can be combined to
define sequences of interventions and actions.

**INTERVENTION**{

ivbolus : {**type** **is** **bolus**, **input**=CENTRAL, **amount**=100,
**doseTime**=0 }

oral4 : {**type** **is** **bolus**, **input**=GUT, **amount**=150,
**doseTime**=[0,24,48,72] }

ivinf : {**type** **is** **infusion**, **input**=CENTRAL,
**amount**=100, **doseTime**=0, **duration**=1 }

wash1 : {**type** **is** **resetAll**}

ivoral : {**type** **is** **combi**, **combination** = [ivbolus, wash1,
oral4], **start**=[0, 24, 48]}

oralinf : {**type** **is** **combi**, **combination** = [oral4, wash1,
ivinf], **start**=[0, 96, 120]}

}

The above example describes three different drug administrations – an IV
bolus dose, four doses of oral drug separated by 24 hours, and an IV
infusion over the space of 1 hour. It also describes a washout “event”
which resets the amount in all dosing variables in the model. We can
then combine these four interventions and actions into a treatment arm
definition. So the ivoral arm receives the ivbolus dose at time zero,
then a washout / reset at 24 hours followed by the four oral doses
starting at 48 hours. Note that the **doseTime** argument in each
intervention definition defines the dosing time relative to the
respective value in the start argument of the combination definition.
Similarly, the oralinf arm receives the four doses of oral drug,
followed by a washout / reset event at 96 hours then the ivinf treatment
starting at 120 hours.

That is:

  Time      0          24         48         72         96         120
  --------- ---------- ---------- ---------- ---------- ---------- ----------
  ivoral    ivbolus    wash1      oral4[1]   oral4[2]   oral4[3]   oral4[4]
  oralinf   oral4[1]   oral4[2]   oral4[3]   oral4[4]   wash1      ivinf

Note that for vectors of times, the seq(…) function can be used to
specify a sequence of times with starting time, stopping time and
interval size (MDL\_Reference Guide section 5.80).

**SAMPLING** block
------------------

Similar to the INTERVENTION block, the SAMPLING block provides a means
to describe observation schedules which can then be used for
individuals, for treatment arms or in combination to define complex
patterns of observation. For the optimisation task in optimal design,
the SAMPLING block defines the starting design sample times (if required
by the algorithm).

The syntax for this block is:

\<name\> : {**type is simple**,

**outcome** = \<Model Object OBSERVATION block variable\>,

***sampleTime** = \<vector of real values\>,*

***numberTimes** = \<integer\>,*

***deltaTime** = \<real value\>,*

***blq** = \<real value\>,*

***ulq** = \<real value\>,*

}

In the above definition, the outcome argument is mandatory, all other
arguments are optional. **numberTimes** defaults to the length of the
**sampleTime** vector. Either **numberTimes** or **sampleTime** should
be used, not both. Note that **numberTimes** should only be used in the
context of Optimal Design optimisation tasks and not for design
evaluation or simulation. **deltaTime** is the minimum time between two
samples (for use in optimal design).

To combine a number of sampling definitions, the user can use the
following syntax:

\<name\> { **type is combi**,

**combination** = [\<vector of previously defined sampling schedules\>,

***numberTimes** = \<integer\>,*

***start** = \<vector of real values\>,*

***relative** = \< Boolean\>*

}

The combination argument is mandatory, other arguments are optional.

For an example of definition of “simple” sampling scheme
(UseCase1\_design1\_eval):

**SAMPLING**{

window1 : {**type** **is** **simple**, **outcome**=Y,

**sampleTime** = [0.0001, 24, 36, 48, 72, 96, 120] }

}

For an example of combining sampling schedules
(UseCase3\_design1\_eval):

**SAMPLING**{

winPK : {**type** **is** **simple**, **outcome**=CC,

**sampleTime** = [0.0001, 24, 36, 48, 72, 96, 120] }

winPD : { **type** **is** **simple**, **outcome**=PCA,

**sampleTime** = [0.0001, 24, 36, 48, 72, 96, 120] }

\# implies concurrent start, both start 0 unless define times

sampPKPD : {**type** **is** **combi**, **combination**=[winPK,winPD] }

}

In the above, the sampling schedules for winPK and winPD start
concurrently (at time 0) – there is no vector of **start** times to
offset winPK and winPD.

**POPULATION** block
--------------------

The **POPULATION** block initialises and defines population
characteristics or distributions for use in defining covariate
distributions within the Model Object **COVARIATES** block. The
**POPULATION** block acts like a **COVARIATES** block for the Design
Object, where the user defines covariate distributions and how these may
differ for each study arm. The **COVARIATES** block in the Model Object
defines how (observed) covariates are used in the model.

The syntax for the POPULATION block is:

\<population\_name\> : { **type is** **template**,

**covariate** = [\<list of covariate definitions\>] }

Each covariate definition should have the following syntax. For
continuous covariates:

{**cov** = \<covariate name\>, **rv** \~ \<ProbOnto distribution\>}

For discrete covariates:

{**catCov** = \<covariate name\>, **discreteRv** \~ \<ProbOnto
distribution\>}

It is possible to define an arm to contain only one level of a
categorical covariate (defined in the **POPULATION** block) by using the
following syntax:

{**catCovValue** = \<categorical covariate category\>}

Note that covariates defined in the **POPULATION** block should be
declared in the **DECLARED\_VARIABLE** block.

For example, for the Design Object to be used with UseCase5 which uses
WT, SEX and AGE as covariates (UseCase5\_design2\_opt.mdl):

**DECLARED\_VARIABLES**{

…

SEX **withCategories**{female, male}

}

**POPULATION**{

default : { **type** **is** **template**, **covariate** = {
**catCov**=SEX,

**discreteRv** \~ Bernoulli1( **probability** = 0.5) } }

arm2Pop : { **type** **is** **template**, **covariate** = {
**catCovValue**=SEX.female } }

}

In the above example, a default distribution is defined for the SEX
covariate which has categories female and male and defines that 50% of
subjects should be male. (The Bernoulli distribution defines probability
of the second category). arm2Pop is defined as having ONLY female
subjects.

Another example for the same model:

**POPULATION**{

default : { **type** **is** **template**,

**covariate**=[{ **catCov**=SEX, **discreteRv** \~
Bernoulli1(**probability** = 0.5) },

{**cov** = WT, **rv** \~ Normal1(**mean**=

**piecewise**{{ 70 **when** SEX==SEX.male;

**otherwise** 60 }},

**stdev**=10)

}

]

}

arm2Pop : { **type** **is** **template**,

**covariate**=[{ **catCovValue**=SEX.female },

{**cov** = WT, **rv** \~ Normal1(**mean**=55, **stdev**=5) }]

}

}

In the above example, SEX is defined as having 50% probability of being
male, and WT is defined as having a Normal1 distribution where the mean
depends on the value of SEX. If SEX is male then the mean is 70, whereas
if female then the mean is 60. Both populations have a standard
deviation of 10 for WT. arm2Pop is exclusively female and WT is defined
separately for this group.

The populations defined in the **POPULATION** block are used in
definition of the study arms within the **STUDY\_DESIGN** block. For
example:

**STUDY\_DESIGN**{

arm1 : {

**armSize**=16,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

**population** = default

}

arm2 : {

**armSize**=17,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

**population** = arm2Pop

}

**STUDY\_DESIGN** block
-----------------------

The **STUDY\_DESIGN** block defines how the **SAMPLING**,
**INTERVENTION** and **POPULATION** blocks are used to define arms in a
study design. For the optimisation task of optimal design, it defines
the starting design (if required by the algorithm). For evaluation and
simulation tasks it defines the study design to be evaluated or
simulated.

The syntax for defining each arm is as follows:

\<name\> : {***armSize** = \<integer value\>, *

***sameTimes** = \<Boolean\>,*

***occasionSequence** = [{**occasion**=\<vector\>, **level**=\<reference
varLevel\>, **start**=\<vector\>}],*

**interventionSequence**=[{**admin** = \<**INTERVENTION** block list
name\>, **start** = \<real value\> }],

**samplingSequence** = [{**sample** = \<**SAMPLING** block list name\>,
**start** = \<real value\> }],

**population** = \<**POPULATION** block list name\> }

Recall that the intervention (**doseTime** argument) and sampling times
(**sampleTime** argument) specified are ***relative***. Within the
**STUDY\_DESIGN** block definitions we can specify start times for these
that define in study time, when each starts. Using this construct it is
possible to define interventions and samples that happen concurrently,
and also those that happen sequentially. If the **start** argument is
not given in **interventionSequence** or **samplingSequence** then it is
assumed to be zero and the start times are taken from the relevant
**INTERVENTION** and **SAMPLING** block definitions.

**armSize** defines the size of each arm.

If multiple outcomes are defined, then **sameTimes** (if **true**)
denotes that the same observation times are to be used for all outcomes.

Additionally and optionally, arguments may be given pertaining to the
study design as a whole. These are defined using the **set** keyword and
are comma separated. Arguments used are:

***totalSize** = \<integer value\>*

***numberSamples** = \<vector of integer\>*

***totalCost** = \<real value\>*

***numberArms** = \<vector of integer\>*

***sameTimes** = \<Boolean\>*

**totalSize** defines the overall size of the study. This defaults to
the sum of the individual armSize arguments.

**numberSamples** defines the constrain on the number of samples for one
subject. If one value is given, the number of samples should be equal to
that value for all designs. If several values are given then this
defines the set of number of samples allowable in designs (primarily
used in optimal design).

**totalCost** defines the total cost for the entire population design.

**numberArms** defines the constraint on the number of arms in the
study. If a single value is given then the final design will have
exactly the required number of arms. If several values are given then
this defines the set of number of arms allowable in designs (primarily
used in optimal design).

**sameTimes** defines whether the sample sampling times are to be used
for all observed outcomes.

As an example of the STUDY\_DESIGN block (UseCase5\_design2\_opt.mdl):

**STUDY\_DESIGN**{

arm1 : {

**armSize**=16,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

**population** = default

}

arm2 : {

**armSize**=17,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

**population** = arm2Pop

}

}

Another example showing how arguments like totalSize and numberSamples
should be used (when optimising a study design):

**STUDY\_DESIGN**{

**set** **totalSize**=40,

**numberSamples**=4

arm1 : {

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

**population** = default

}

…

}

**DESIGN\_SPACES** block
------------------------

The **DESIGN\_SPACES** block defines which design elements should be
optimised in finding the optimal design. A design space defines the
possible values of the design variables specified in **SAMPLING**,
**INTERVENTION**, **POPULATION** blocks. It should not be used with
design evaluation (**EVALUATE** task in the Task Properties object) or
simulation. If no design space is defined for a variable e.g. amount or
sampling schedule then it is assumed fixed for the optimisation process.

The syntax for DESIGN\_SPACES definitions is as follows:

\<name\> : {**objRef** = \<named object in INTERVENTION, SAMPLING or
POPULATION\>,

**element is** \<variable from named object above\>,

**discrete** = \<vector of values, with type dependent on original
argument \>,

**range** = \<vector of values defining upper and lower range of values
to be explored,

with type dependent on original argument\>

}

Valid choices for element depend on the object type (**SAMPLING**,
**INTERVENTION**, **POPULATION**) that is being referenced: **bolusAmt,
infAmt, duration, sampleTime, numberTimes, covariate, numberArms,
armSize, parameter, doseTime**.

For example (UseCase1\_design2\_optFW.mdl):

warfarin\_PK\_ODE\_design = **designObj**{

…

**SAMPLING**{

window1 : {**type** **is** **simple**, **sampleTime** = [0.0001, 36, 96,
120], **outcome**=Y }

}

**DESIGN\_SPACES**{

DS1 : { **objRef**=[window1], **element** **is** **sampleTime**,

**discrete**=[0.0001,24, 36,48,72,96, 120] }

DS2 : { **objRef**=[window1], **element** **is** **numberTimes**,

**discrete**=[4,5] }

}

…

}

In the above example the **DESIGN\_SPACES** defines how the optimisation
algorithm will investigate the optimal design based on the window1
defined within the **SAMPLING** block, examining designs with 4 or 5
sample points chosen from the sample times defined in DS1 i.e. [0.0001,
24, 36, 48, 72, 96, 120]. The sample times defined in window1 of the
**SAMPLING** block are used as the starting design for optimisation.

Another example:

warfarin\_PK\_SEX\_design = **designObj**{

…

**INTERVENTION**{

admin1 : {**type** **is** **bolus**, **input**=INPUT\_KA,
**amount**=100, **doseTime**=[0] }

}

**SAMPLING**{

window1 : {**type** **is** **simple**, **sampleTime** = [0.0001, 24, 36,
48, 72, 96, 120], **outcome**=Y }

}

**STUDY\_DESIGN**{

arm1 : {

**armSize**=16,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

}

arm2 : {

**armSize**=17,

**interventionSequence**=[{

**admin**=admin1,

**start**=0

}],

**samplingSequence**=[{

**sample**=window1,

**start**=0

}],

}

}

}

**DESIGN\_SPACES**{

DS1 : { **objRef**=[window1], **element** **is** **sampleTime**,

**discrete**=[0.0001, 24, 36, 48,72,96, 120] }

DS2 : { **objRef**=[window1], **element** **is** **numberTimes**,
**discrete**=[4,5] }

DS3 : { **objRef**=[admin1], **element** **is** **bolusAmt**,
**discrete**=seq(100,300,100) }

DS4 : { **objRef**=[arm1, arm2], **element** **is** **armSize**,
**range**=[0,30] }

}

In the above example several design space elements are defined. DS1
defines possible sampling times for window1, while DS2 defines that
designs of 4 or 5 samples are to be considered. DS3 specifies that the
amount to be dosed should vary between 100 and 300 with steps of 100.
Finally DS4 specifies how the number of subjects in each arm should be
in the range [0,30] (inclusive) for both arms.

**DESIGN\_PARAMETERS** block
----------------------------

This block defines parameter values (constants) required for use in
defining the design or to pass as constants to the model. Values are
defined by assignment \<name\> = \<value\> or by equation \<name\> =
\<equation\>.

Variables defined in **DESIGN\_PARAMETERS** can be used in definition of
covariate distributions defined in Design Object blocks.

Recall the example above where the distribution of covariates was
defined in the **POPULATION** block:

**POPULATION**{

default : { **type** **is** **template**,

**covariate**=[{ **catCov**=SEX, **discreteRv** \~
Bernoulli1(**probability** = 0.5) },

{**cov** = WT, **rv** \~ Normal1(**mean**=

**piecewise**{{ 70 **when** SEX==SEX.male;

**otherwise** 60 }},

**stdev**=10)

}

]

}

arm2Pop : { **type** **is** **template**,

**covariate**=[{ **catCovValue**=SEX.female },

{**cov** = WT, **rv** \~ Normal1(**mean**=55, **stdev**=5) }]

}

}

We can define the population means for weight in each population in the
**DESIGN\_PARAMETERS** block and pass these values into the
**POPULATION** block definitions.

**DESIGN\_PARAMETERS**{

maleMeanWT = 70

femaleMeanWT = 60

femaleMeanWTArm2 = 55

stdevWT = 10

}

**POPULATION**{

default : { **type** **is** **template**,

**covariate**=[{ **catCov**=SEX, **discreteRv** \~
Bernoulli1(**probability** = 0.5) },

{ **cov** = WT, **rv** \~ Normal1(**mean**=

**piecewise**{{ maleMeanWT **when** SEX==SEX.male;

**otherwise** femaleMeanWT }},

**stdev**=stdevWT) }] }

arm2Pop : { **type** **is** **template**,

**covariate**=[{ **catCovValue**=SEX.female },

{ **cov** = WT, **rv** \~ Normal1(**mean**=femaleMeanWTArm2,
**stdev**=stdevWT) }]

}

}

1.  

2.  The MOG Object
    ==============

The MOG Object is where the user defines the MDL objects required for a
particular task: estimation, simulation, design evaluation or
optimisation.

**INFO Block**
--------------

The **INFO** block provides a name and/or problem statement to the
associated MOG. The **name** attribute populates the Name tag in
PharmML, while the **problemStmt** attribute populates the Description
tag in PharmML. This information can then be passed forward to target
software that support names or problem statement definition. For
example, NONMEM conversion uses the **problemStmt** attribute to
populate the \$PROB statement, while the **name** attribute is converted
to metadata in the comment header of the control stream file.

By default the Name tag in PharmML is “Generated from MDL. MOG ID: \<MOG
object name\>”.

The problemStmt attribute can be set via the ddmore R package function
writeMDL( … , problemStmt = “Problem statement text”).

The syntax for the INFO block is:

**INFO**{

**set** **problemStmt** = \<text string\> ,

**name** = \<text string\>

}

The statements can be comma separated or the **set** command can be used
for each line.

For example (UseCase1\_1.mdl):

**INFO**{

**set** **problemStmt** = "my Problem Statement"

**set** **name** = "10May2016 Task Properties check"

}

**OBJECTS** Block
-----------------

~~In the current MDL version, the only block supported is the
**OBJECTS** block. ~~

The **OBJECTS** block defines the objects (defined in the current .mdl
file) that are to be used in defining the Modelling Objects Group for
use in the desired task. The MDL-IDE checks that these named objects
exist in the current file.

The MDL-IDE also uses the MOG Object to “tie together” variable
definitions across objects – it checks that variables used in the model
are defined. So for example, if the model expects a covariate called
logtWT but this is not defined in the Data Object then an error is
given. Without a MOG Object, no validation check of this type is
possible. Without the MOG Object, the MDL-IDE can only perform
rudimentary syntax checking of MDL statements. With the MOG Object
defined the MDL-IDE can check that the resulting model will result in
valid PharmML.

The syntax for statements in this block is :

\<Object name within the current MDL file\> : {**type** is \<**dataObj |
designObj | mdlObj | parObj | priorObj | taskObj\>**}

Note that in the MOG Object, the user must specify **dataObj** ***OR***
**designObj**; **parObj** ***OR*** **priorObj**. As stated previously,
for simulation, design evaluation or optimisation the Design Object
takes the place of the Data Object. Similarly for estimation with BUGS
or other Bayesian estimation software the Prior Object takes the place
of the Parameter Object.

For example (UseCase1.mdl):

> warfarin\_PK\_ODE\_mog = **mogObj** {
>
> **OBJECTS**{
>
> warfarin\_PK\_ODE\_dat : { **type** is **dataObj** }
>
> warfarin\_PK\_ODE\_mdl : { **type** is **mdlObj** }
>
> warfarin\_PK\_ODE\_par : { **type** is **parObj** }
>
> warfarin\_PK\_ODE\_task : { **type** is **taskObj** }
>
> }
>
> }

Mapping of variable names between MDL Objects
---------------------------------------------

**The current version of MDL requires that variable names in each object
are consistently named. Future versions of MDL may allow mapping between
variable names across objects.**

