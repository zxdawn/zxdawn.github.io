---
title: Set decayed tracer for WRF
date: 2018-07-31 10:09:31
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Set decayed tracer for WRF"
img: "images/sci-tech/2017-10/wrfchem_logo.png"
---

Because tracer_opt in WRF-Chem support more functions than WRF, we plan to clone and edit it for WRF (which is much quickier than WRF-Chem).

1. Copy `/chem/module_input_tracer_data.F` and `chem/module_input_tracer.F` into `/WRFV3/phys`.

2. After compile, I get an error about `TRACER_TEST1 not found`. It's related to `module_state_description.F`

       module_state_description.F : ../Registry/$(REGISTRY)
                         ( cd .. ; tools/registry $(ARCHFLAGS) $(ENVCOMPDEFS) -DNEW_BDYS Registry/$(REGISTRY) ) ; 

3. `Registry/Registry.EM_COMMON`

<!--more-->

  ```
  # Lines that start with the word 'state' form a table that is
  # used by the script use_registry to generate module_state_descript.F
  # and other files.  Also see documentation in use_registry.
  ```

4. Add these lines to `Registry/Registry.EM_COMMON`:

```
# Copy WRF-Chem tracers options
state   real    -          ikjftb  tracer        1         -     -     -
state   real    smoke      ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "smoke"         "tracing smoke"     "Dimensionless"
state   real    tr17_1     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_1"         "tr17_1"     "Dimensionless"
state   real    tr17_2     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_2"         "tr17_2"     "Dimensionless"
state   real    tr17_3     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_3"         "tr17_3"     "Dimensionless"
state   real    tr17_4     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_4"         "tr17_4"     "Dimensionless"
state   real    tr17_5     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_5"         "tr17_5"     "Dimensionless"
state   real    tr17_6     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_6"         "tr17_6"     "Dimensionless"
state   real    tr17_7     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_7"         "tr17_7"     "Dimensionless"
state   real    tr17_8     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr17_8"         "tr17_8"     "Dimensionless"
state   real    tr18_0     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_0"         "tr18_0"     "Dimensionless"
state   real    tr18_1     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_1"         "tr18_1"     "Dimensionless"
state   real    tr18_2     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_2"         "tr18_2"     "Dimensionless"
state   real    tr18_3     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_3"         "tr18_3"     "Dimensionless"
state   real    tr18_4     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_4"         "tr18_4"     "Dimensionless"
state   real    tr18_5     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_5"         "tr18_5"     "Dimensionless"
state   real    tr18_6     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_6"         "tr18_6"     "Dimensionless"
state   real    tr18_7     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_7"         "tr18_7"     "Dimensionless"
state   real    tr18_8     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_8"         "tr18_8"     "Dimensionless"
state   real    tr18_9     ikjftb  tracer        1         -     irhusdf=(bdy_interp:dt)    "tr18_9"         "tr18_9"     "Dimensionless"


# tracer options. (smoke) only has one tracer for smoke (has to run with biomassburning).
#                 (test1) has lateral boundary tracer(1,2), stratospheric tracer(5,6), 
#                     boundary layer tracer(7,8), surface tracer (3,4)
#                 (test2) has surface tracer replaced by smoke tracer
package   tracer_smoke  tracer_opt==1       -             tracer:smoke
package   tracer_test1  tracer_opt==2       -             tracer:tr17_1,tr17_2,tr17_3,tr17_4,tr17_5,tr17_6,tr17_7,tr17_8
package   tracer_test2  tracer_opt==3       -             tracer:tr17_1,tr17_2,tr17_3,tr17_4,tr17_5,tr17_6,tr17_7,tr17_8
package   tracer_test3  tracer_opt==4       -             tracer:tr17_1,tr17_2,tr17_3,tr17_4,tr17_5,tr17_6,tr17_7,tr17_8,tr18_1,tr18_2
```

5. Add `module_input_tracer` to `dyn_em/solve_em.F`

   ```
   ------before------
   !   USE module_diagnostics
   #if (WRF_CHEM == 1)
      USE module_input_chem_data
      USE module_input_tracer
      USE module_chem_utilities
   #endif
      USE module_first_rk_step_part1
      USE module_first_rk_step_part2
   ------after------
   !   USE module_diagnostics
   #if (WRF_CHEM == 1)
      USE module_input_chem_data
      USE module_input_tracer
      USE module_chem_utilities
   #endif
      USE module_input_tracer
      USE module_first_rk_step_part1
      USE module_first_rk_step_part2
   ```

6.  Add `    module_input_tracer_data.o` and `module_input_tracer.o` to `phys/Makefile`