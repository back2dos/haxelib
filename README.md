# Haxelib Rewrite

A fresh rewrite of the haxelib client.

## Strategy

The goal is to decouple Haxe entirely from haxelib, much like nodejs has no dependency on npm. Instead, nodejs has a clearly defined rule where it's `require` function expects modules to be found, and npm installs modules in that very location.

In the future, the haxe compiler should be able to resolve `-lib name` without `haxelib path`, by defining a simple convention that haxelib (and possibly other package managers) will respect. For transition, `haxelib` will still implement the `path` command, according to the new convetion.

## Library Resolution Convention

The resolution of `-lib name` will proceed like so:

1. Starting at the cwd, recurse upwards until a `.haxelibs` directory is found. 
2. Within that directory, seek a `name.hxml` and use the paremeters in there
3. Follow up on other `-lib` arguments found therein.

Support for `-lib name:version` is dropped.

## Future `haxelib install` command

In the future `haxelib install name versionRange` will be implemented like so:

1. Locate the `.haxelibs` directory according to the above convention. Fail if not found.
2. Get current dependencies from `haxelib.json` in the parent directory containing `.haxelibs`. If not present, create it. Resolve all dependency constraints to the projects and versions that need to be installed. Update `haxelib.json` accordingly.
3. Download the respective sources to a central location, for each library that is `%HAXELIBS%/name/version/` if not yet present there or if a `--force` parameter is passed.
4. In the `.haxelibs` directory write a `name.hxml` with the following content:

```
# command that allows installing this particular version again, i.e. `haxelib install name version --no-dependencies`
-D name=version
-cp %HAXELIBS%/name/version/classPath_from_haxelib.json
# contents of extraParams.hxml
# -lib commands based on dependencies as spefified by haxelib.json
```

## Motivation

This approach allows for a setup that is both project local and complete - and thus replicable and versionable. Storing sources in a central location avoid duplicate downloads and makes branch switching work instantaneously.

Cloning a repository and running `haxelib install all` will be able to fully restore the state by relying on the commands found in the respective `.haxelibs/project.hxml` files (reading the command from the `project.hxml` allows also getting libraries that were installed by other means than haxelib itself).

Also, `haxelib update all` will be able to pull in the newest libraries within the dependency ranges stored in `haxelib.json`.

The resulting dependency management should prove both faster and more reliable.
