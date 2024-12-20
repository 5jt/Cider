---
title: Cider’s API functions
description: Complete reference to the Cider API, which lets you call the Dyalog APL project manager from your own code.
keywords: api, apl, cider, dyalog, link, nuget, parameter, source, tatin
---

# API functions

!!! abstract "Cider’s API lets you call it from your own code"

The API functions are similar to the [user-commands](user-commands.md), but not identical.
Not all have equivalent user commands.

<pre style="font-family: APL">
  AddAlias                            GetMyUCMDsFolder
  AddNuGetDependencies                GetNuGetDependencies
  AddTatinDependencies                GetProgramFilesFolder
  CloseProject                        GetTatinDependencies
  CreateCreateProjectParms            HasDotNet
  CreateOpenParms                     ListNuGetDependencies
  CreateProject                       ListOpenProjects
  DropAlias                           ListTatinDependencies
  GetCiderAliasFileContent            OpenProject
  GetCiderAliasFilename               ProjectConfig
  GetCiderGlobalConfigFileContent     ReadProjectConfigFile
  GetCiderGlobalConfigFilename        WriteProjectConfigFile
  GetCiderGlobalConfigHomeFolder      Version
</pre>

Unlike user commands, API function names are case-sensitive.

The API is exposed in `⎕SE.Cider` so, for example, call `AddAlias` as `⎕SE.Cider.AddAlias`.

??? warning "API code cache"

    The Cider code package is loaded into `⎕SE._Cider`, but the API is exposed via `⎕SE.Cider`. 

    Do not call functions in `⎕SE.Cider_`. 


---

## :fontawesome-solid-code: Add alias

{r}←AddAlias(projectpath alias)
{: .syntax}

Where

-   `projectpath` is a project path
-   `alias` is a string with no punctuation or spaces

if the project folder exists the alias is registered in the file returned by [`GetCiderAliasFilename`](#get-alias-filename).
The shy result is an error message, empty if successful.

If the alias is already in use Cider asks you to confirm the change.


## :fontawesome-solid-code: Add NuGet dependencies

list←AddNuGetDependencies(packages project)
{: .syntax}

Where

-   `packages`is one or more NuGet packages
-   `project` is an alias or project path

Cider installs NuGet packages in the (single) NuGet dependency folder defined in the project config and returns their names as a list of strings.
<!-- FIXME confirm list of strings -->

Specify `packages` as either a list of strings or a comma-separated string.

!!! warning "NuGet package names" 

    NuGet package names are not case sensitive when they are loaded so, for example, you can load `Clock` by the name `clock`. 

    However, the correct name is returned, and is required for using a package.

:fontawesome-solid-terminal: 
[`]CIDER.AddNugetDependencies`](user-commands.md#add-nuget-dependencies)


## :fontawesome-solid-code: Add Tatin dependencies

r←AddTatinDependencies(packages project dev)
{: .syntax}

Where

-   `packages` is one or more Tatin packages
-   `project` is an alias or project path
-   `dev` is a flag

Cider installs the packages in one of the Tatin dependency folders defined in the project’s configuration file. 
(The default is `dependencies.tatin`.)

Specify packages as either a list of strings or a comma-separated string.

Setting the `dev` flag switches the installation folder from `dependencies` (default) to`dependencies_dev`.

<!-- FIXME Result? -->

:fontawesome-solid-terminal: 
[`]CIDER.AddTatinDependencies`](user-commands.md#add-tatin-dependencies).



## :fontawesome-solid-code: Close project

r←{x} CloseProject projects
{: .syntax}

Where 

-   `x` (optional) is a list of projects and/or a flag
-   `projects` is one or more open projects

Cider closes the projects (unlinks the source files) and returns the number of projects closed.

Identify projects as (any of)

-   fully qualified namespace names
-   aliases
-   project paths

Specify `projects` as either a string or list of strings.

The __optional left argument__ can be either or both (in any order) of

-   a list of projects as returned by [`ListOpenProjects`](#list-open-projects)
-   a flag (defaults to 1): whether Dropbox conflict checks are made.


<!-- FIXME specify effect of listing projects in `x`. -->

:fontawesome-solid-gear: 
[`CheckForDropboxConflicts`](configuration.md#checkfordropboxconflicts)<br>
:fontawesome-solid-terminal: 
[`]CIDER.CloseProject`](user-commands.md#close-project)


## :fontawesome-solid-code: Create `CreateProject` parms

parms←{parms} CreateCreateProjectParms folder
{: .syntax}

Where

-   `parms` (optional) is a namespace of parameters
-   `folder` is a project path

Cider returns a namespace with parameters required by [`CreateProject`](#create-project), by default:

    acceptConfig   - 0
    folder         - project path
    ignoreUserExec - 0
    namespace      - name of the project folder

Defaults are overwritten by any specified in the `parms` argument.

:fontawesome-solid-terminal: 
[`]CIDER.CreateProject`](user-commands.md#create-project)


## :fontawesome-solid-code: Create Open parms

parms←CreateOpenParms y
{: .syntax}

Where `y` is either an empty vector or a namespace of parameters, returns a namespace of parameters required by the `OpenProject` function. 

Parameters in `y` overwrite the defaults, which are:

    alias                 ''      
    batch                 0
    checkPackageVersions  ⍬
    folder                ''  
    ignoreUserExec        0
    importFlag            0
    noPkgLoad             0
    parent                '' 
    projectSpace          ''  
    quietFlag             0 
    suppressInit          0
    verbose               0
    watch                 0

!!! detail "Setting `watch` to 0 shows Cider you have not set it. Eventually 0 becomes `both`, the default."


## :fontawesome-solid-code: Create project

r←CreateProject parms
{: .syntax}

Where `parms` is a namespace of parameter values, typically the result of [`CreateCreateProjectParms`](#create-createproject-parms), Cider creates a project.

:fontawesome-solid-terminal: 
[`]CIDER.CreateProject`](user-commands.md#create-project)


## :fontawesome-solid-code: Drop alias

{flag}←DropAlias alias
{: .syntax}

Where `alias` is a project alias, Cider removes it from the file named by `GetCiderAliasFilename` and returns a flag indicating success.


## :fontawesome-solid-code: Get alias file content

r←{filename} GetCiderAliasFileContent dummy
{: .syntax}

Cider ignores `dummy` and returns as a matrix of strings the contents of the file named by `GetCiderAliasFilename`.

```
      ⍴⎕←⎕SE.Cider.GetCiderAliasFileContent 'blah'
┌───┬──────────────────┐
│bar│/Users/sjt/tmp/foo│
└───┴──────────────────┘
1 2
```

If the file is empty the result has zero rows.

:fontawesome-solid-terminal: 
[`]CIDER.ListAliases`](user-commands.md#list-aliases)



## :fontawesome-solid-code: Get alias filename

filename←GetCiderAliasFilename
{: .syntax}

Returns the path to the file used to record alias names and their paths.



## :fontawesome-solid-code: Get global config file content

parms←GetCiderGlobalConfigFileContent
{: .syntax}

Cider returns the [global config](configuration.md#global) – if found – as a parameter namespace; otherwise the result is `⍬`.

:fontawesome-solid-terminal: 
[`]CIDER.Config`](user-commands.md#config)


## :fontawesome-solid-code: Get global config filename

path←GetCiderGlobalConfigFilename
{: .syntax}

Returns the path to Cider’s global config file.


## :fontawesome-solid-code: Get global config home folder

path←GetCiderGlobalConfigHomeFolder
{: .syntax}

Returns the path to the parent folder of Cider’s global config file.

On Windows, this is typically `C:/Users/<⎕AN>/.cider/config.json`



## :fontawesome-solid-code: Get MyUCMDs folder

path←GetMyUCMDsFolder
{: .syntax}

Returns the path to the `MyUCMDs/` folder.

!!! warning "The folder might not exist :fontawesome-brands-linux: :fontawesome-brands-apple:"

    On Windows, this folder is created by the installer. 
    Not so on other platforms.


## :fontawesome-solid-code: Get NuGet dependencies

r←name GetNuGetDependencies config
{: .syntax}

Where 

-   `name` is `'development'` or `'development_dev'`
-   `config` is a parameter namespace

returns either the value of `nuget` in the given branch or an empty vector if `nuget` is not defined.

The `config` argument is typically derived from a project’s configuration file.

:fontawesome-solid-terminal: 
[`]CIDER.ListNuGetDependencies`](user-commands.md#list-nuget-dependencies)



## :fontawesome-solid-code: Get Program Files folder

path←{current}GetProgramFilesFolder suffix
{: .syntax}

Where

-   `current` (optional) is a flag
-   `suffix` is a suffix to the folder filepath

returns the path to the Dyalog files folder with any `suffix` specified.

The `current` flag (default 0) specifies whether the result is specific to the currently running version of Dyalog.


```
      ⍝ Version agnostic
      Cider.GetProgramFilesFolder ''
C:\Users\kai\Documents\Dyalog APL Files             
      Cider.GetProgramFilesFolder 'CiderTatin'
C:\Users\kai\Documents\Dyalog APL Files/CiderTatin

      ⍝ Version specific
      1 Cider.GetProgramFilesFolder ''              
C:\Users\kai\Documents\Dyalog APL-64 18.2 Unicode Files
```


## :fontawesome-solid-code: Get TatinD dependencies

r←name GetTatinDependencies config
{: .syntax}

Where

-   `name` is `'development'` or `'development_dev'`
-   `config` is a parameter namespace

returns either the value of `tatin` in the given branch or an empty vector if `tatin` is not defined.

The `config` argument is typically derived from the project config.



## :fontawesome-solid-code: Has DotNet

flag←HasDotNet
{: .syntax}

Result indicates whether .NET Core or .NET is available and the bridge DLL was successfully loaded.


## :fontawesome-solid-code: List NuGet dependencies

r←ListNuGetDependencies projectPath
{: .syntax}

Where `projectPath` is a project path, returns a matrix of names and versions of its NuGet dependencies.


## :fontawesome-solid-code: List open projects

r←ListOpenProjects verbose
{: .syntax}

Where `verbose` is a flag, returns the open projects as a matrix of 2 or 4 columns:

1. Fully qualified project namespace
1. Path the project was loaded from
1. Number of objects belonging to the project
1. Alias (if any)

```
      ⎕SE.Cider.ListOpenProjects 0
 #.Cider  /path/to/Cider

      ⎕SE.Cider.ListOpenProjects 1
 #.Cider  /path/to/Cider  32  cider 
```

:fontawesome-solid-terminal:
[`]CIDER.ListOpenProjects`](user-commands.md#list-open-projects).


## :fontawesome-solid-code: List Tatin dependencies

r←ListTatinDependencies projectpath
{: .syntax}

Where `projectpath` is a project path, returns as a 5-column matrix the dependencies installed in the Tatin installation folders.

1. Full package ID
1. A flag: whether the package is a principal or a dependency
1. URL from which the package was loaded
2. ==???==
3. ==???==

<!-- 
FIXME
Describe the other two columns
 -->

!!! detail "Until version 0.34.0 this was named `ListTatinPackages`"

:fontawesome-solid-terminal:
[`]CIDER.ListTatinDependencies`](user-commands.md#list-tatin-dependencies).



## :fontawesome-solid-code: Open project

(flag log)←OpenProject y
{: .syntax}

Where `y` is either

-   an alias or project path
-   a parameter namespace (typically created by  `CreateOpenParms`)

Cider [opens the project](open-project.md), and returns a 2-item result:

    flag - 1 for success
    log  - list of strings as printed to the session

<!-- 
### Actions

1.  Creates the `projectSpace` namespace in `parent` if it does not already exist. This is the __project root__.
1.  Sets the projects’s [system variables](configuration.md#system) (at least `⎕IO` and `⎕ML`) in the project root.
1.  Brings all APL code and variables into the project root, linked to their source files according to the project’s [`watch`](configuration.md#watch) setting unless the `importFlag` parameter is set.
1.  Loads all Tatin packages from the Tatin installation folders defined by [`dependencies:tatin`](configuration.md#dependencies) and [`dependencies_dev:tatin`](configuration.md#dependencies-dev).
1.  Loads all NuGet packages from the NuGet installation folder defined by [`dependencies:nuget`](configuration.md#dependencies).
1.  Injects a namespace `CiderConfig` into the project root and populates it with the contents of the configuration file.
1.  Injects a namespace `TatinVars` into the project root, and a ref pointing to that `TatinVars` (with the same name) if parameter [`tatinVars`](configuration.md#tatinvars) is defined in the project config `CIDER` section and points to a subnamespace of the project space.
1.  Changes the current directory as specified by [`AskForDirChange`](configuration.md#askfordirchange) in the global config.
1.  Adds a variable `HOME` to `CiderConfig` specifying the path from which the project was loaded.
1.  Executes the function specified in the project’s [`init`](configuration.md#init) setting.
1.  Executes the function specified in the global [`ExecuteAfterProjectOpen`](configuration.md#executeafterprojectopen) setting.
1. If it finds a variable `ToDo` in the project root, displays it for editing.
1. Reports the Git status of the project according to the global [`ReportGitStatus`](configuration.md#reportgitstatus) setting.
### Parameters
 -->

All parameters are optional except `folder`.


`folder`

: String. Alias or project path. May not be empty, but could be `./` indicating the current directory.

`alias`

: String. An alias by which you can refer to the project.

    !!! detail "Special syntax"

        If `alias` is just a dot, the name of the project folder becomes the alias. 
        Example:

        ```apl
        p←Cider.CreateOpenParms
        p.folder←'/path/2/projects/foo'
        p.alias←'.'  ⍝ `foo` becomes the (new) alias
        ```

    If you specify an alias on `folder` but also on `alias`, then Cider expects the alias on `folder` to be defined, and will use that one to open the project. It will then overwrite the former alias with the one defined on `alias`.

`checkPackageVersions`

: By default Cider proposes to check principal packages for later versions and, if found, to update them. 

        ⍬ - Ask me whether to check (default)
        0 - Do not check at all
        1 - Check and report findings but prompt for updating
        2 - Check and update without consulting me

    This parameter is ignored if the project has no Tatin installation folder, or if `importFlag` is set.


`ignoreUserExec`

: Set the flag to stop Cider executing at the end of opening a project a function named in the global  [`ExecuteAfterProjectOpen`](configuration.md#executeafterprojectopen) setting. Defaults to 0.


`importFlag`

: Set the flag to stop Cider from linking APL objects to their source files. Defaults to 0.

    :fontawesome-solid-gear:
    [`watch`](configuration.md#watch)

    !!! warning "Setting this flag has implications for how Cider deals with Tatin packages."
<!-- 
, see there. 
FIXME Where? What implications?
 -->

`noPkgLoad`

: Set the flag to stop Cider from loading Tatin dependencies as specifed in the config file’s `dependencies` and `dependencies_dev` settings. Defaults to 0.


`parent`

: String. Defaults to `#` but could be something like `⎕SE` or `#.Foo.Goo.Boo`. All namespaces listed must exist.

    :fontawesome-solid-gear:
    [`parent`](configuration.md#parent)


`projectSpace`

: String. The name of the namespace the project is injected into. If this is empty it is going to be `#` or `⎕SE`, depending from where the function was called from.

    :fontawesome-solid-gear:
    [`projectSpace`](configuration.md#projectspace)


`quietFlag`

: Set the flag to stop Cider printing messages to the session. (They are still returned in the function‘s result.)


`suppressLX`

: Set the flag to stop Cider executing the project’s [initialisation](configuration.md#init) function. Defaults to 0.

: For example, an automated build process might open a project without initialising it.

`watch`

: String. See [`watch`](configuration.md#watch) in the project config for setting values. 

    :fontawesome-solid-bomb:
    [How Link watches for changes](troubleshooting.md#how-link-watches-for-changes)

:fontawesome-solid-terminal:
[`]CIDER.OpenProject`](user-commands.md#open-project)


## :fontawesome-solid-code: Project config

{r}←ProjectConfig projectpath
{: .syntax}

Where `projectpath` is a project path, Cider displays the project config for editing.

Asks your permission before writing changes back to file, and performs checks before doing so.


## :fontawesome-solid-code: Read project config file

config←ReadProjectConfigFile projectpath
{: .syntax}

Where `projectpath` is a project path, Cider returns its project config as a parameter namespace.

The path may or may not terminate in the filename `cider.config`.

__Side effect__ If the function does not find the sub-keys `dependency.tatin` and `dependency.nuget` in the file it creates them and writes them there.

:fontawesome-solid-terminal:
[`]CIDER.ProjectConfig`](user-commands.md#project-config).


## :fontawesome-solid-code: Write project config file

{r}←config WriteProjectConfigFile project
{: .syntax}

Where 

-   `config` is a parameter namespace
-   `project` is a project path

Cider writes the contents of `config` as the project’s configuration file.

The path may or may not terminate in the filename `cider.config`.


## :fontawesome-solid-code: Version

r←Version
{: .syntax}

Returns a string with major and minor versions, patch number and timestamp, e.g.

          ⎕SE.Cider.Version
    0.44.0+835

This could be just e.g. `1.2.3`,  but might be something like `1.2.3-beta-1+113`.





