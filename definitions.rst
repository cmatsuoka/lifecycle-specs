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
  The execution of the algorithm without changes to the disk (except for
  package list updates from remote repositories). The outcome of the
  planning phase is a list of actions to be handled at a later moment.
  The initial state is read from the persistent state on disk. All state
  changes are carried in memory and can be added as a action parameter
  if the state is required to execute the action.

Execution phase
  The handling of the list of actions obtained in the Planning Phase. The
  Execution Phase is stateless, states are received as part of the action
  to be executed and updated to disk as needed.

Ephemeral state
  The lifecycle state in memory used in the Planning Phase.

Persistent state
  The lifecycle state on disk written in the Execution Phase.

Action
  Any of the operations required to run the lifecycle, sequenced during
  the Planning Phase and handled during the Execution Phase. The list of
  possible actions is:

  * ``CLEAN``
  * ``PULL``
  * ``BUILD``
  * ``STAGE``
  * ``PRIME``
  * ``SKIP``: an operation was skipped
  * ``REMOVE``: remove files or directories
  * ``INSTALL_PKG``: fetch and install packages on this system
  * ``INSTALL_STAGE_PKG``: fetch and install packages in the staging area
  * ``UPDATE_PULL``
  * ``UPDATE_BUILD``
  * ...
