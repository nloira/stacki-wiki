## add host alias

### Usage

`stack add host alias {host} {name} [name=string]`

### Description

Adds an alias to a host

### Arguments

* `{host}`

   Host name of machine

* `{name}`

   The alias name for the host.


### Parameters
* `[name=string]`

   Can be used in place of the name argument.

### Examples

* `stack add host alias compute-0-0 c-0-0`

   Adds the alias 'c-0-0' to the host 'compute-0-0'.

* `stack add host alias compute-0-0 name=c-0-0`

   Same as above.


