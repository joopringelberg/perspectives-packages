# perspectives-packages
The shared `packages.dhall` file for all Purescript Perspective packages.

This package will be installed in all Purescript Perspectives projects, as a developer dependency.

The file `packages.dhall` will be referred to in the spago.dhall file of each such project:

```
 packages = ./node_modules/perspectives.packages/packages.dhall
```

While developing the Purescript projects, I will keep `packages.dhall` in a modified state where Purescript packages that have been added to the base package-set are read from their local directories. Instead of this:

```
let upstream =
      https://github.com/purescript/package-sets/releases/download/psc-0.14.7-20220404/packages.dhall
        sha256:75d0f0719f32456e6bdc3efd41cfc64785655d2b751e3d080bd849033ed053f2

in  upstream
  with avar-monadask =
    { dependencies =
      [ "prelude"
      , "avar"
      , "aff"
      , "transformers"
      ]
    , repo =
       "https://github.com/joopringelberg/purescript-avar-monadask.git"
    , version =
        "v2.2.0"
    }

```

We'll have this:

```
let upstream =
      https://github.com/purescript/package-sets/releases/download/psc-0.14.7-20220404/packages.dhall
        sha256:75d0f0719f32456e6bdc3efd41cfc64785655d2b751e3d080bd849033ed053f2

in  upstream
  with avar-monadask =
    { dependencies =
      [ "prelude"
      , "avar"
      , "aff"
      , "transformers"
      ]
    , repo =
       "~/Code/purescript-avar-monadask"
    , version =
        "v2.2.0"
    }

```

This means that all Purescript projects that import avar-monadask will work with _the latest version in development_.

Suppose one of the Purescript projects has been updated and that this creates a new stable release of MyContexts. To preserve (and publish) this state, one updates the version numbers in the `packages.dhall` file before pushing it to github and tagging it. **However**, the _overrides with local packages_ should first be restored to the git references! 

For this, we have two scripts in the package.json file: `localize` and `delocalize`. The workflow is then:

* update package versions as needed;
* run npm script `delocalize`;
* update the version number in `package.json`;
* commit and push to github;
* create and push a tag (tag name based on the version number);
* run npm script `localize`.

Notice that in the Purescript projects that depend on `perspectives-packages`, we then have to update the version of `perspectives-packages` in their `package.json` files. Does this require a particular order? Should one publish a new version of `perspectives-packages` before or after a new project? In fact, __it doesn't matter__. Remember that all these files are _descriptions_ of packages or projects that are not used on publishing other descriptive files. Order is unimportant; but the end result should be coherent in the sense that

* a new version of a Purescript project should be in the `packages.dhall` file in `perspectives-packages`;
* each Purescript project should refer to the latest version of `perspectives-packages`.