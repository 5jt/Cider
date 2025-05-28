---
title: Cider worked example
description: A worked example of using Cider to develop four Tatin packages
keywords: api, apl, cider, dyalog, example, link, source, tatin
---

# A worked example using Cider and Tatin

!!! abstract "Goal"

    Use Cider as the project manager to publish some Tatin packages, with test scripts, and dependencies on each other, all with public GitHub repos.


## Four packages

A worked example needs packages that are simple but not trivial
– and preferably actually useful.

---|---
[NiceTime](https://github.com/5jt/nicetime) | Take a `⎕TS` timestamp as argument and return a text string describing the offset from the present.
[Translate](https://github.com/5jt/translate) | For a text string, return from a dictionary its equivalent in another language.
[Text](https://github.com/5jt/text) | An APL equivalent of Python’s interpolated string, e.g. `f'2+2={2+2}'` => `'2=2=4'`.
[TinyTest](https://githb.com/5jt/tinytest)| A minimal test framework: operators to compare against an expected result, and catch an expected error.

Examples:

```apl
      #.nicetime.was (-∘1@3)⎕TS
yesterday

      'fr' #.translate.text 'yesterday'
hier

      N←2
      #.text.f'depuis {N} mois'
depuis 2 mois

      ec←0 ⍝ error count
      ec+←2(+ #.tinytest.match 4)2
      ec+←2 3(+ #.tinytest.catch 5 'LENGTH ERROR')4 5 6
      #.tinytest.report ec
All tests passed
```

### Interdependencies

The first three packages all use TinyTest for their test scripts,
so all need TinyTest as a _development_ dependency.

NiceTime depends on Translate, which, in turn, depends on Text.

    NiceTime
    - TinyTest (dev)
    - Translate
      - TinyTest (dev)
      - Text
        - TinyTest (dev)

!!! tip "Where package A depends upon package B, publish B first."

The APL code for these packages can be seen in their GitHub repos, linked to above.
But we start at the beginning.

## Workflow

The workflow for each package follows the same steps.

1.  **Create a Cider project:** links a folder in your local file system to a namespace in your workspace and drafts a Cider configuration for you.

1.  **Write and test** your APL code in the namespace. (Link saves it in `APLSource/` within the project folder.)

1.  **Put under version control:** Initialise the project folder as a Git repository and link it to a repo on your GitHub account.

1.  **Configure as a Tatin package**: write a Tatin configuration file in the project folder.

1.  **Create a public API** if only some objects are to be exposed.

1.  **Include dependencies:** any Tatin or Nuget packages on which the APL code depends.

1.  **Build** the package as a ZIP in the project’s distribution folder.

1.  **Publish** the package to a Tatin server.

For the first package in this worked example, these steps are described in detail.
For subsequent packages, only the variations.


## TinyTest

!!! abstract "The smallest testing framework that could possibly work"

The other three packages depend on TinyTest, which has no dependencies of its own.
So we publish TinyTest first.


### Create a Cider project

We begin in my `examples/` folder.
(You will of course use your own file path.)

```apl
      ]CD /Users/sjt/Projects/Dyalog/examples
/Users/sjt
      ]CIDER.CreateProject tinytest
"/Users/sjt/Projects/dyalog/examples/tinytest" does not exist yet - create? (Y/n) y
```

Cider uses the Editor to show me its draft of the Cider configuration

??? example "tinytest/cider.config"

    ```json
    {
      CIDER: {
        cider_version: "0.48.0",
        dependencies: {
          nuget: "",
          tatin: "",
        },
        dependencies_dev: {
          tatin: "",
        },
        distributionFolder: "Dist",
        init: "",
        make: "",
        parent: "#",
        projectSpace: "tinytest",
        project_url: "",
        source: "APLSource",
        tests: "",
      },
      SYSVARS: {
        io: 1,
        ml: 1,
      },
      USER: {
      },
    }
    ```

This looks all right.
The `CIDER` subkeys `dependencies.tatin` and `dependencies_dev.tatin` are empty, which is fine:
TinyText has no dependencies.

<kbd>Esc</kbd> to save the config as is and close the Editor.

```
Project successfully created; open as well? (Y/n) y
Project successfully opened and established in: #.tinytest
```

Now we see in the `examples/` folder:

```
tinytest
├── APLSource
└── cider.config
```

Nothing in the `APLSource/` folder yet.
We can fix that.


### APL source

```apl
      )CS #.tinytest
```

In the Editor we define [two d-ops and a d-fn](https://github.com/5jt/TinyTest/tree/main/APLSource).
Link saves them in the file system.

```txt hl_lines="4-6"
tinytest
├── apl-package.json
├── APLSource
│  ├── catch.aplo
│  ├── match.aplo
│  └── report.aplf
└── cider.config
```

Now we have something worth backing up.


### Git and GitHub

On GitHub we create a new repo for TinyTest, with a `README.md` and a licence acceptable to Tatin.
(Below, the GitHub username is `5jt`; you will, of course, use your own account and email address.)

On our local machine we open a command shell in `tinytest/`, run `git init`, and register the GitHub repo as a remote:

```bash
# initialise as a Git repository
git init
# register the GitHub repo as a remote
git remote add origin git@github.com:5jt/TinyTest.git
# pull content from GitHub
git pull origin main
# include and commit local content
git add .
git commit -m 'Cider project'
# push changes to GitHub
git push origin main
```
Now the project looks like this in the file system.

```txt hl_lines="7 8"
tinytest
├── APLSource
│  ├── catch.aplo
│  ├── match.aplo
│  └── report.aplf
├── cider.config
├── LICENSE
└── README.md
```


### Initialise as a Tatin package

So far we have working APL code, managed by Cider, and backed by a GitHub repo.
We do not yet have a Tatin package.
Back to the APL session.

```apl
      ]CD
/Users/sjt/Projects/dyalog/examples
      ]TATIN.Init tinytest

There is no file /Users/sjt/Projects/dyalog/examples/tinytest/apl-package.json yet; would you like to create it? (Y/n) Y
Enter the group name (mandatory): sjt
Enter the package name (mandatory) [tinytest]:
Enter a description of what the package is doing (mandatory): Minimal testing framework
Enter the email address of the maintainer: sjt@5jt.com
Enter a comma-separated list of tags: framework,test
Enter name of the license (enter "?" for a list of options): MIT

Is the project (going to be) hosted on GitHub? (a URL will then be compiled for the "project_url" property) (Y/n) Y
```
Above, the Tatin group name is `sjt` and the maintainer’s email address is `sjt@5jt.com`; you will, of course, use your own.

Tatin uses the Editor to display its draft configuration.

??? example "tinytest/apl-package.json"

    ```json hl_lines="5 8 13 19"
    {
      api: "",                                           // The public interface of the package
      assets: "",                                        // Empty or a single folder holding assets required by the package
      description: "Minimal testing framework",          // Mandatory. Maximum length is 120 chars.
      documentation: "",                                 // A URL or a local path or a function providing documentation
      files: "",                                         // Optional. For files like "ReadMe.txt" to go to the root of the package
      group: "sjt",                                      // Mandatory. Maximum length is 120 chars.
      io: 0,                                             // Value for index origin
      license: "MIT",                                    // Name of the license; see ]Tatin.ListLicenses
      lx: "",                                            // Function to be called when the package is established in the WS
      maintainer: "sjt@5jt.com",                         // The email address of the maintainer
      minimumAplVersion: "18.2",                         // Minimum version required to run the package
      ml: 0,                                             // Value for migration level
      name: "tinytest",                                  // Mandatory. The project's name
      os_lin: 1,                                         // 1=the package runs under Linux
      os_mac: 1,                                         // 1=the package runs under Mac OS
      os_win: 1,                                         // 1=the package runs under Windows
      project_url: "https://github.com/auser/tinytest",  // The project's home for example
      source: "",                                        // Mandatory. Either a code file (.aplc, .apln, etc.) or a folder
      tags: "framework,test",                            // A comma-separated list of tags
      userCommandScript: "",                             // Path to the user command script within the project
      version: "0.1.0+0",                                // Mandatory. The project's version number
    }
    ```

This is nearly right.

-   The GitHub README will serve as documentation for TinyTest.

            documentation: "https://github.com/5jt/tinytest",

-   The zero values for `⎕IO` and `⎕ML` are not the Dyalog defaults.
    Cider will overwrite them with 1 anyway but it will do no harm and remove any uncertainty if we set them here explicitly to 1.

            io: 1,
            ml: 1,

-   We must specify where the APL source files are, i.e. `APLSource/`.

            source: "APLSource",


??? danger "Minimum APL version"

    The default Tatin configuration in `apl-package.json` specifies a minimum APL version of 18.2, the lowest that will support Tatin.

    If you are developing in V20.0 or higher, your APL source files might use array notation.
    If so, specify in `apl-package.json` a minimum version of 20.0.

<kbd>Esc</kbd> to save the configuration and finish the dialogue.

```
Add a build number to the version (for telling different builds with same version no. apart)? (Y/n) Y

Does the package come with a user command script? (y/N) N
```
Tatin configuration file `apl-package.json` now appears in the project folder.

TinyTest has no dependencies, so we are ready to build the package.


### Build the package

The last step before publishing is to build the Tatin package as a ZIP in the distribution folder specified in the Cider configuration.

```
      ]TATIN.BuildPackage

Sure that you want to pack
   /Users/sjt/Projects/dyalog/examples/tinytest/
into
/Users/sjt/Projects/dyalog/examples/tinytest/Dist/? (Y/n) y

Target directory
/Users/sjt/Projects/dyalog/examples/tinytest/Dist/
does not exist yet; create it? (Y/n) y
/Users/sjt/Projects/dyalog/examples/tinytest/Dist/sjt-tinytest-0.1.0.zip
```
Now our project looks like this:
```txt hl_lines="2 8 9"
tinytest
├── apl-package.json
├── APLSource
│  ├── catch.aplo
│  ├── match.aplo
│  └── report.aplf
├── cider.config
├── Dist
│  └── sjt-tinytest-0.1.0.zip
├── LICENSE
└── README.md
```
In the distribution folder `Dist/` we see a ZIP of version 0.1.0.
The package is ready to publish to the Test server.


### Publish to a Tatin server

The working directory is still the `examples/` folder, so the path to the project folder is simply `tinytest`.

```
      ]TATIN.PublishPackage tinytest [tatin-test]
Package published on https://test.tatin.dev/
      ]TATIN.ListPackages [tatin-test] -group=sjt
 Registry: [tatin-test]               ≢ 1
 Group & Name            # major versions
 ------------            ----------------
 sjt-tinytest                           1
```

## Text

??? abstract "Interpolate a string, much like Python’s `f` does."

    (By _string_ we mean a character vector.)

    ```apl
          text.f'{2} + {2} = {2+2}'
    2 + 2 = 4

          (A B)←2 3
          text.f'{A} + {B} = {A+B}'
    2 + 3 = 5
    ```


The Text package has three features that TinyTest does not:

1.  It has a test script, `tests`.
1.  Not all its objects should be exposed for use: `tests` is only for developing Text itself.
1.  The test script depends upon another package (TinyTest).

We deal with these as follows:

1.  **Declare the test script** in the Cider configuration.
1.  Create a **public API** to expose only specified objects.
1.  **Specify the dependencies:** the packages Text depends on.

We follow the same steps as for TinyTest but stop before building the Text package.


### Declare the test script

Create the Cider project.

In the configuration declare the test script `tests`.

??? example "text/cider.config"

    ```json
    {
      CIDER: {
        cider_version: "0.48.0",
        dependencies: {
          nuget: "",
          tatin: "",
        },
        dependencies_dev: {
          tatin: "tatin-packages-dev",
        },
    ...
    ```

As for TinyTest, create the APL source files and set up the GitHub repository.


### Create the public API

If Text were a class it would expose `f` as a method and `DEBUG` as an instance property.
But for a (lightweight) namespace, we follow the Tatin convention of listing the exposed objects in a constant `Public`.
<!-- FIXME Link to Public Interface page in Tatin user guide -->
```apl
 Z←Public
⍝ niladic function as an immutable constant
 Z←'f' 'DEBUG'
```
and use a Tatin API function to create a public API for Text.
```apl
      ]CD
/Users/sjt/Projects/Dyalog/examples
      cfg←0(⎕JSON⍠'Dialect' 'JSON5')⊃⎕NGET 'text/apl-package.json'
      ⎕SE.Tatin.CreateAPIfromCFG (#.text cfg)
```


### Install the dependency

A package’s dependencies are listed in the file `apl-dependencies.txt` in its root.

The Text namespace requires TinyTest for development only: it is not needed in production.
So we declare the `tatin-packages-dev/` folder that will hold TinyTest.

??? example "text/cider.config"

    ```json
        ...
        dependencies_dev: {
          tatin: "tatin-packages-dev",
        },
        ...
    ```

and install TinyTest in it:
```
      ]CIDER.AddTatinDependency sjt-tinytest -development
```
<!-- FIXME Confirm the command installs the dependency as well as listing it. -->

We have no interest in using Git to control Tatin source code.

!!! example "text/.gitignore"

        tatin-packages-dev/

Our project now looks like this:
``` hl_lines="2 3 6-8 16-24"
text
├── .gitignore
├── apl-dependencies.txt
├── apl-package.json
├── APLSource/
│  ├── API/
│  │  ├── DEBUG.aplf
│  │  └── f.aplf
│  ├── DEBUG.apla
│  ├── f.aplf
│  ├── Public.aplf
│  └── tests.aplf
├── cider.config
├── LICENSE
├── README.md
└── tatin-packages-dev/
    ├── apl-buildlist.json
    ├── apl-dependencies.txt
    └── sjt-tinytest-0.1.0
        ├── apl-package.json
        ├── APLSource/
        │  ├── catch.aplo
        │  └── match.aplo
        └── LICENSE
```
We see

-   a `.gitignore` to exclude the dependencies from Git version control
-   `apl-dependencies.txt` listing the dependency on TinyTest
-   the API defined in a child folder of `APLSource/`
-   TinyTest installed in `tatin-packages-dev/`

### Build and publish

The Text package is ready to be built and published.

The first time you publish a package, you can omit the build:
the command will do it for you.
```
      ]TATIN.Publish text [tatin-test]
```


## Translate

??? abstract "For a string, return from a dictionary its equivalent in another language."

    ```apl
          'fr' #.translate.text 'yesterday'
    hier
    ```

1.  Create the Cider project.

1.  Create the APL source files in `APLSource/`.

1.  **Initialise as a Tatin package**

    -   The documentation for Translate is in a variable `help`.
    -   The APL source files use array notation.

    So for the Tatin configuration:

    ??? example "translate/apl-package.json"

        ```json
        documentation: "help",
        ...
        minimumAplVersion: "20.0",
        ```

1.  **Install the dependencies**

    Translate depends on Text and, in development, on TinyTest.

    Specify dependency folders in the Cider configuration:

    ??? example "translate/cider.config"

        ```json
        ...
        dependencies: {
          nuget: "",
          tatin: "tatin-packages",
        },
        dependencies_dev: {
          tatin: "tatin-packages-dev",
        },
        ...
        ```

    and install the dependencies there.
    ```apl
          ]CIDER.AddTatinDependency sjt-text
          ]CIDER.AddTatinDependency sjt-tinytest -development
    ```

    Use `.gitignore` to exclude the dependency folders from Git version control.

    ??? example "translate/.gitignore"

            tatin-packages*/


1.  Create the public API: `DEBUG`, `dictionary`, `help`, and `text`.

1.  Build and publish the package.

              ]TATIN.Publish translate [tatin-test]

    The ZIP is made and published to the Test server.


## NiceTime

1.  Create the Cider project.

1.  Create the APL source files in `APLSource/`.

1.  **Initialise as a Tatin package**

    -   The documentation for NiceTime is in the repo README.
    -   The namespace has an initialisation function `init` that writes `dictionary` into Translate.
    -   The APL source files use array notation.

    ??? example "nicetime/apl-package.json"

        ```json
        documentation: "https://github.com/5jt/NiceTime",
        ...
        lx: "init",
        ...
        minimumAplVersion: "20.0",
        ```

1.  **Install the dependencies**

    NiceTime depends on Translate and, in development, on TinyTest.

    ??? example "nicetime/cider.config"

        ```json
        ...
        dependencies: {
          nuget: "",
          tatin: "tatin-packages",
        },
        dependencies_dev: {
          tatin: "tatin-packages-dev",
        },
        ...
        ```

    and install the dependencies there.
    ```apl
          ]CIDER.AddTatinDependency sjt-translate
          ]CIDER.AddTatinDependency sjt-tinytest -development
    ```

    Translate itself depends on Text, which Cider therefore also installs.

    Use `.gitignore` to exclude the dependency folders from Git version control.

    ??? example "nicetime/.gitignore"

            tatin-packages*/


1.  Create the public API: `sundayStart` (not yet implemented) and `was`.

1.  Build and publish the package.

              ]TATIN.Publish nicetime [tatin-test]

    The ZIP is made and published to the Test server.

On your local machine the NiceTime project now looks something like this:

!!! example "nicetime/"

    ```
    ├── .gitignore
    ├── apl-package.json
    ├── APLSource
    │  ├── API
    │  │  ├── sundayStart.aplf
    │  │  └── was.aplf
    │  ├── dictionary.apla
    │  ├── init.aplf
    │  ├── monthnames.apla
    │  ├── Public.apla
    │  ├── sundayStart.apla
    │  ├── tests.aplf
    │  ├── was.aplf
    │  └── weekdays.apla
    ├── banner.png
    ├── cider.config
    ├── dictionary.csv
    ├── Dist
    │  └── sjt-nicetime-0.1.0.zip
    ├── LICENSE
    ├── README.md
    ├── SPEC.md
    ├── tatin-packages
    │  ├── apl-buildlist.json
    │  ├── apl-dependencies.txt
    │  ├── sjt-text-0.1.0
    │  │  ├── apl-package.json
    │  │  ├── APLSource
    │  │  │  ├── API
    │  │  │  │  ├── DEBUG.aplf
    │  │  │  │  └── f.aplf
    │  │  │  ├── DEBUG.apla
    │  │  │  ├── f.aplf
    │  │  │  ├── Public.apla
    │  │  │  └── tests.aplf
    │  │  └── LICENSE
    │  └── sjt-translate-0.1.4
    │      ├── apl-dependencies.txt
    │      ├── apl-package.json
    │      ├── APLSource
    │      │  ├── API
    │      │  │  ├── DEBUG.aplf
    │      │  │  ├── dictionary.aplf
    │      │  │  ├── help.aplf
    │      │  │  └── text.aplf
    │      │  ├── DEBUG.apla
    │      │  ├── dictionary.apla
    │      │  ├── help.apla
    │      │  ├── Public.apla
    │      │  ├── tests.aplf
    │      │  ├── text.aplf
    │      │  └── validDictionary.aplf
    │      └── LICENSE
    └── tatin-packages-dev
        ├── apl-buildlist.json
        ├── apl-dependencies.txt
        └── sjt-tinytest-0.1.1
            ├── apl-package.json
            ├── APLSource
            │  ├── catch.aplo
            │  └── match.aplo
            └── LICENSE
    ```

