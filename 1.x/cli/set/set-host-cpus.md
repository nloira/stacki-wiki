## set host cpus

### Usage

`stack set host cpus {host}... {cpus} [cpus=string]`

### Description

Set the number of CPUs for a list of hosts.

### Arguments

* `{host}`

   One or more host names.

* `{cpus}`

   The number of CPUs to assign to each host.


### Parameters
* `[cpus=string]`

   Can be used in place of the cpus argument.

### Examples

* `stack set host cpus compute-0-0 2`

   Sets the CPU value to 2 for compute-0-0.

* `stack set host cpus compute-0-0 compute-0-1 4`

   Sets the CPU value to 4 for compute-0-0 and compute-0-1.

* `stack set host cpus compute-0-0 compute-0-1 cpus=4`

   Same as above.


