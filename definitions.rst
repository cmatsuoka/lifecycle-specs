Definitions
===========

Part
  Each separately specified component of the project we're processing.

Step
  Each of the tasks we need to fulfill in order to obtain the final
  artifacts resulting from processing the project specification. Steps
  are, in this order: ``PULL``, ``BUILD``, ``STAGE``, and ``PRIME``.

State
  A collection of metadata gathered from the context used in the
  execution of each step for a given part. A state must exist for every
  step executed for a part.

Part dependency
  Parts may depend on other parts, meaning that the part we're depending
  on must complete its ``STAGE`` step before we execute our ``BUILD``
  step.

Part order
  The sorting order of parts places dependent parts after the parts
  they depend on. If no dependency relation exists, parts are sorted
  alphabetically by name. Dependency cycles are not allowed.

Ṕlanning phase

Execution phase

Action
  Any of the operations handled during the execution phase.
