Lifecycle algorithm
===================

This is the lifecycle processing algorithm for a single-phase
implementation. For two-phase planning/execution, ordering of
operations should be decided in the planning phase and any
action that changes the filesystem should be handled in the
execution phase.

::

  __main(target_step, parts):
      validate parts schema
      sort parts
      run __lifecycle(target_step, parts)


  __lifecycle(target_step, parts):
      for each step up to target_step:
          if step is STAGE:
              check for collisions
          for each part in parts:
              if __didnt_run_yet(part, step):
                  __run_step(part, step)
                  end

              if (part, step) was explicitly requested:
                  __rerun_step(part, step)
                  end

              if __is_dirty(part, step):
                  __rerun_step(part, step)
                  end

              if __is_outdated(part, step):
                  if step is PULL or step is BUILD:
                      __update_step(part, step)
                  else:
                      __rerun_step(part, step)

                  end

              skip action




  __clean(part, step):
      if step <= PRIME:
          remove only this part's files from the prime directory
          remove state for (part, PRIME)

      if step <= STAGE:
          remove only this part's files from the stage directory
          remove state for (part, STAGE)

      if step <= BUILD:
          remove part build tree
          remove part install tree
          remove state for (part, BUILD)

      if step <= PULL:
          remove part snaps tree
          remove part source tree
          remove stage packages tree
          remove state for (part, PULL)



  __prepare(part, step):
      reset environment
      get dependencies for this part that should run STAGE or PRIME
      if there are dependencies, run lifecycle again for this step and dependent parts
      remove part install tree

      if step is PULL:
          fetch and install stage packages
          fetch and install stage snaps

      if step is STAGE:
          install stage packages
          install stage snaps



  __run_step(part, step):
      __prepare(part, step)

      if step is PULL:
          clear any previously failed pull
          __pull_handler(part) or override-pull scriptlet for this part
          update state for (part, step)

      if step is BUILD:
          wipe and repopulate part build dir
          __build_handler(part) or override-build scriptlet for this part
          organize installed files
          update state for (part, step)

      if step is STAGE:
          __stage_handler(part) or override-stage scriptlet for this part
          if (part, STAGE) didn't already run:
              update state for (part, step) with no snaps, no dirs

      if step is PRIME:
          __prime_handler(part) or override-prime scriptlet for this part
          if (part, STAGE) didn't already run:
              update state for (part, step) with empty data

      set (part, step) as ran



  __rerun_step(part, step):
      __clean(part, step)
      for each step starting at this one:
          remove step from list of steps we already ran for this part
      __run_step(part, step)



  __update_step(part, step):
      if step is PULL:
          // this is like __run_step with some extra stuff added
          __prepare(part, step)
          run scriptlet for (part, step)
          update according to source-type
          update state for (part, step)
          end

      if step is BUILD:
          // this is like __run_step with some extra stuff added
          __prepare(part, step)
          ?? do some source check and return if needed
          update according to source-type
          run scriptlet for (part, step)
          organize (overwriting if needed)
          update state for (part, step)
          end



  __pull_handler(part):
      source pull (if applicable)



  __build_handler(part):
      generate and run the plugin build script



  __stage_handler(part):
      migrate installed files to staging area
      update state for (part, step) if everythig is ok



  __prime_handler(part):
      migrate files from stage to prime
      update state for (part, step)



  __didnt_run_yet(part, step):
      // This is implemented in a much more confusing way in snapcraft
      yes if step is larger than the latest step that ran for this part



  __should_run_step(part, step):
      yes, if it didn't run yet
      yes, if __is_outdated(part, step)
      yes, if __is_dirty(part, step)
      yes, if __should_run_step(part, previous step)
      otherwise no



  __is_dirty(step, part):
      check if properties or options of interest from (step, part) have changed
      if they've changed:
         // Shouldn't we return here if we know we're dirty?
         // this is not how it's currently implemented in snapcraft
         return the result (along with reason)

      get dependencies for this part
      list of changed dependencies is empty
      for each dependency:
          if this step is STAGE:
              if state for (step, part) is newer than PRIME, or (dependency, PRIME) should run:
                  add (dependency, PRIME) to list of changed dependencies
          else:
              if state for (step, part) is newer than STAGE, or (dependency, STAGE) should run:
                  add (dependency, STAGE) to list of changed dependencies

      if we have changed dependencies:
          return this result (along with reason)

      not dirty



  __is_outdated(step, part):
      if step is PULL:
          ask if outdated according to source-type
          return the result

      check if a previous step have a newer timestamp than this step
      return the result
