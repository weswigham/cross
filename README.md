cross
=====

cross is a cross-compiler builder and manager. With cross, generating a 
cross compiler for a new architecture is as simple as
```
cross make arm-none-eabi
```

Installing cross
----------------

Download cross:
```
git clone https://github.com/weswigham/cross.git ~/.cross
```

Then add the cross bin and shims to the end of your bash/zsh/other profile:
```
echo 'export PATH="$HOME/.cross/shims:$HOME/.cross/bin:$HOME/.cross/cmd:$PATH"' >> ~/.bash_profile
```

And then restart your shell.

Using cross
-----------
Cross comes bundled with a few useful commands.

_make_
```
cross make TARGET [BUILD_PARAMS ...]
```
'TARGET' is your standard target triple, such as 'x86_64-pc-mingw32'.

'BUILD_PARAMS' are parameters in the form KEY=VALUE which override any other configuration options. More on those later.

_version_
```
cross version
```
Outputs the `cross` version number.

_help_
```
cross help [subcommand]
```
Outputs either the list of subcommands or the help text for a specific subcommand.

Target Configurations
---------------------
Cross comes bundled with what could be considered 'sane defaults' for _embedded systems_-type target architectures. The head version of 
GCC is built first without headers, then newlib is built with it as the C-library to be includes, then GCC is rebuilt again 
with newlib as the included C library.

For quite a few targets these defaults work pretty okay, but if they don't, `cross` is highly configurable. 
`cross` looks in its `cmd/targets` directory for files named after the target given. If it exists, the file is loaded
as a lua table and searched for override parameters. Parameters it looks for include:

>SKIP_BISON or SKIP_FLEX
>>These, if present, make the build system skip building both flex and bison (prerequisites for building GCC)

>t_VERSION
>>Where 't' is any of FLEX, BISON, BINUTILS, GCC, or LIBC 
>>This, by default, indicates a specific version of that dependency is necessary. `cross` will try to checkout the given version from the repo (with hg/git/svn tags).

>t_OVERRIDE 
>>Where 't' is any of FLEX, BISON, BINUTILS, GCC, or LIBC 
>>This lets you specify a table to completely override that stage of the build.
>>The table has the fields 'target' and 'output' set to its output dir and target-triple,
>>Then, if the table specifies a 'method' (and, implicitly, a 'url') field, then
>>the `cross` internal source control function is used to download that version.
>>If `method` is not specified, `cross` expects the table to be callable, and return the directory that dependency's source is in when called. 
>>`cross` also expects the override table to have a 'builder' string. This string has the override table applied to it as a handlebars-esque context
>>and then is run from the shell from within the directory the source is in. This is usually something as simple as
>> "./configure; make; make install;"

Any combination of these can be present in the table, and they can be overridden again with parameters to the `make` subcommand.


How cross works
---------------
cross is based off of `rbenv`. As such, there are a number of similarities. Namely, shims. Unlike rbenv, you should never need 
to `rehash` yourself, just leave it to cross. Why does `cross` have shims? To enable a feature called a 'target override'. If you
want to `make` something for a new target, usually you'd need to make a new makefile referencing the new target's build executables.
cross alleviates this need. Instead, place a .cross_target file in the same directory as the makefile and make its contents your desired target.
i.e.:
```
echo "arm-none-eabi" >> .cross_target
```
cross's shims will look for this file when you run gcc, g++, or any executable compiled by cross and override it with the target-specific version when this file is present.