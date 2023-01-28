This repository contains TLA+ files to reproduce the glibc condition variable bug.

File 0_ocaml_mutex.tla shows the bug by implementing the Ocaml masterlock using pthread condition variables.

1_with_barrus_patch.tla and following files show the fixed code, with each patch in my series. The first one already fixes the bug, the later files just confirm that no other problems get introduced.

# Steps to Reproduce

* Download TLA+ https://github.com/tlaplus/tlaplus/releases/tag/v1.7.1#latest-tla-files
* Run the toolbox
* Click “File -> Open Spec -> Add New Spec” then select ocaml_mutex.tla
* Click “TLC Model Checker -> New Model”, click OK
* In the model under
* * What is the behavior spec: leave as default “Spec”
* * What is the model?
* * * UseSpinlockForCVMutex <- TRUE
* * * Procs <- {P0, P1, P2}, also select “Set of model values” and “Symmetry set”
* * * UseSpinlockForMutex <- TRUE
* * * UsePatch <- TRUE (this flag controls whether the mitigation patch should be used)
* * * MaxNumLoops <- 3
* * What to check?
* * * Check “Deadlock” (should be checked by default)
* * * Invariants: leave empty
* * * Properties: leave empty
* Click “TLC Model Checker -> Run Model”
* Watch the progress in the “Model Checking Results” column. Once per minute it should populate the table “Time, Diameter, States Found, Distinct States, Queue Size.” After ~2.5 hours, when “Diameter” reaches 342, a new panel should open up titled “TLC Errors”, and it should say “Deadlock reached.” in the first window.
* Now you have 341 states to look at in the Error-Trace, in full detail. The “pc” variable shows you which line each process was on for every state. All other variables are also visible. They are a pretty direct translation from the C code. It should be possible to figure out by having the C code and TLA+ code side-by-side to see the minor differences. I uploaded a table with only the relevant information here. (this was from a run with slightly different source code, with the AlwaysSetGrefs commented out. This means your trace will be slightly different and the line numbers of the spreadsheet won’t align if you try to add more information. You’ll need to create your own by exporting the trace)
* (optional step before running the model: Click on “Additional TLC Options” and increase “Number of worker threads” for your machine, also “Fraction of physical memory allocated to TLC” and set “Profiling” to “Off”, this should speed up the checking)