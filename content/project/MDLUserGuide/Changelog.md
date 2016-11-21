##### Changes from Public Beta release – 11 Dec 2015

Changes have been made to the MDL since the Public Beta release on 11
Dec 2015. The changes are reflected in this documentation by striking
out comments that are not applicable any more, and by adding new syntax
in red, boxed-out text, as seen here.

The primary focus of MDL in this release is translation to valid
PharmML, rather than conversion to target software. The previous release
was primarily concerned with demonstrating interoperability across key
software targets. In this version of MDL there may be MDL features which
are not supported in conversion from PharmML to certain target software,
but which are valid for model description and which generate valid
PharmML. The aim is to widen the scope of models which can be encoded in
MDL and generate PharmML for uploading to the DDMoRe repository and for
future interoperability. Translation of these models to target software
will follow with updates to the interoperability framework converters.

The changes to MDL since draft 7 (v0.7) enable integration of the Prior
and Design Objects and improved validation of MDL giving increased
confidence in generation of valid PharmML. In order to facilitate this,
certain changes to syntax have been made that are ***NOT*** backwards
compatible. This is regrettable since it means that existing models
required changes. We do not make these changes lightly.

Key changes which break backwards compatibility:

**- DECLARED\_VARIABLES** block must now have type assigned to variables
to enable validation of variable types between MDL objects, particularly
Design Object variables.

- Correlations and covariances between parameters are now specified in
the Model Object and must be named parameters. This is to facilitate
specification of priors on these parameters.

- In the Parameter Object, user should not specify the type of
variability definition (**type is** **sd**, **type is** **var**, **type
is** **corr**, **type is** **cov**) for **VARIABILITY** parameters.The
variability, covariance or correlation type is specified and used in the
RANDOM\_VARIABLE\_DEFINITION block where these parameters are defined.

- Left hand side transformations for **INDIVIDUAL\_PARAMETERS** are no
longer valid. These were felt to be confusing.

- Right hand side functions for **INDIVIDUAL\_PARAMETER** and
**OBSERVATION** definitions are now list definitions with the matching
type to the function. This, combined with conditional statements allows
more flexibility in parameter and observation definitions.

- Non-continuous outcomes (binary, count, categorical) must be defined
as **RANDOM\_VARIABLE\_DEFINITION**(level=DV){ … } and then the variable
defined assigned as an anonymous list in the OBSERVATION block. This is
to ensure that outcome variables are always defined with variability at
the DV level. (This is implicit in continuous outcomes due to definition
of residual error at the DV level).

- Arbitrary equations defining the outcome variable are not allowed in
the **OBSERVATION** block. Use “**type is userDefined”** instead.

Additional features in the new version:

- Prior Object definition

- Design Object definition

- Support for ProbOnto distributions in
**RANDOM\_VARIABLE\_DEFINITION**.

- Support for target software specific Task Properties object.

- Support for **DATA\_DERIVED\_VARIABLES** where dose amounts and dose
times can be derived from data columns which are being used otherwise as
“**use is** **amt**” or “**use is** **idv**”.

- Support for vectors and matrices

- Support for conditional assignment to lists.

- Support for definition of parameters in
**RANDOM\_VARIABLE\_DEFINITION** i.e. definition of CL \~
Normal(mean=POP\_CL, sd=PPV\_CL) for subsequent use in
**INDIVIDUAL\_PARAMETERS**. This combined with support for ProbOnto
definitions allows the user to define multivariate distributions of
parameters, mixture distributions etc.

- Support for combination of compartment definitions with differential
equations.

- Support for model input variables passed from the Data Object with
“**use is** **variable**”. This equates to “regressor” type inputs to
models.

- Support for userDefined specification of the relationship between
model predictions and residual error random variables in the
**OBSERVATION** block.