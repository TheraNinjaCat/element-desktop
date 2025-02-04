Element Desktop
===============

Element Desktop is a Matrix client for desktop platforms with Element Web at its core.

First Steps
===========
Before you do anything else, fetch the dependencies:

```
yarn install
```

Fetching Element
================
Since this package is just the Electron wrapper for Element Web, it doesn't contain any of the Element Web code,
so the first step is to get a working copy of Element Web. There are a few ways of doing this:

```
# Fetch the prebuilt release Element package from the element-web GitHub releases page. The version
# fetched will be the same as the local element-desktop package.
# We're explicitly asking for no config, so the packaged Element will have no config.json.
yarn run fetch --noverify --cfgdir ""
```

...or if you'd like to use GPG to verify the downloaded package:
```
# Fetch the Element public key from the element.io web server over a secure connection and import
# it into your local GPG keychain (you'll need GPG installed). You only need to to do this
# once.
yarn run fetch --importkey
# Fetch the package and verify the signature
yarn run fetch --cfgdir ""
```

...or either of the above, but fetching a specific version of Element:
```
# Fetch the prebuilt release Element package from the element-web GitHub releases page. The version
# fetched will be the same as the local element-desktop package.
yarn run fetch --noverify --cfgdir "" v1.5.6
```

If you only want to run the app locally and don't need to build packages, you can
provide the `webapp` directory directly:
```
# Assuming you've checked out and built a copy of element-web in ../element-web
ln -s ../element-web/webapp ./
```

[TODO: add support for fetching develop builds, arbitrary URLs and arbitrary paths]


Building
========
Now you have a copy of Element, you're ready to build packages. If you'd just like to
run Element locally, skip to the next section.

If you'd like to build the native modules (for searching in encrypted rooms and
secure storage), do this first. This will take 10 minutes or so, and will
require a number of native tools to be installed, depending on your OS (eg.
rust, tcl, make/nmake).

You'll also to need to make sure you've built the native modules for the same
architecture as your package, so for anything more advanced than just building
the modules and app for the host architecture see 'Other Architectures'.

If you don't need these features, you can skip this step.

To just build these for your native architecture:
```
yarn run build:native
```

Now you can build the package:

```
yarn run build
```
This will do a couple of things:
 * Run the `setversion` script to set the local package version to match whatever
   version of Element you installed above.
 * Run electron-builder to build a package. The package built will match the operating system
   you're running the build process on.

This build step will not build any native modules.

You can also build using docker, which will always produce the linux package:
```
# Run this once to make the docker image
yarn run docker:setup

yarn run docker:install
# if you want to build the native modules (this will take a while)
yarn run docker:build:native
yarn run docker:build
```

After running, the packages should be in `dist/`.

Starting
========
If you'd just like to run the electron app locally for development:
```
# Install electron - we don't normally need electron itself as it's provided
# by electron-builder when building packages
yarn add electron
yarn start
```

Other Architectures
===================
Building the native modules will build for the host architecture (and only the
host architecture) by default. On Windows, this will automatically determine
the architecture to build for based on the environment. Make sure that you have
all the [tools required to perform the native modules build](docs/windows-requirements.md)


On macOS, you can build universal native modules too:
```
yarn run build:native:universal
```

...or you can build for a specific architecture:
```
yarn run build:native --target x86_64-apple-darwin
```
or
```
yarn run build:native --target aarch64-apple-darwin
```

You'll then need to create a built bundle with the same architecture.
To bundle a universal build for macOS, run:

```
yarn run build:universal
```

If you're on Windows, you can choose to build specifically for 32 or 64 bit:
```
yarn run build:32
```
or
```
yarn run build:64
```

Note that the native module build system keeps the different architectures
separate, so you can keep native modules for several architectures at the same
time and switch which are active using a `yarn run hak copy` command, passing
the appropriate architectures. This will error if you haven't yet built those
architectures. eg:

```
yarn run build:native --target x86_64-apple-darwin
# We've now built & linked into place native modules for Intel
yarn run build:native --target aarch64-apple-darwin
# We've now built Apple Silicon modules too, and linked them into place as the active ones

yarn run hak copy --target x86_64-apple-darwin
# We've now switched back to our Intel modules
yarn run hak copy --target x86_64-apple-darwin --target aarch64-apple-darwin
# Now our native modules are universal x86_64+aarch64 binaries
```

The current set of native modules are stored in `.hak/hakModules`,
so you can use this to check what architecture is currently in place, eg:

```
$ lipo -info .hak/hakModules/keytar/build/Release/keytar.node 
Architectures in the fat file: .hak/hakModules/keytar/build/Release/keytar.node are: x86_64 arm64 
```

Config
======
If you'd like the packaged Element to have a configuration file, you can create a
config directory and place `config.json` in there, then specify this directory
with the `--cfgdir` option to `yarn run fetch`, eg:
```
mkdir myconfig
cp /path/to/my/config.json myconfig/
yarn run fetch --cfgdir myconfig
```
The config dir for the official Element app is in `element.io`. If you use this,
your app will auto-update itself using builds from element.io.

Profiles
========

To run multiple instances of the desktop app for different accounts, you can
launch the executable with the `--profile` argument followed by a unique
identifier, e.g `element-desktop --profile Work` for it to run a separate profile and
not interfere with the default one.

Alternatively, a custom location for the profile data can be specified using the
`--profile-dir` flag followed by the desired path.

User-specified config.json
==========================

+ `%APPDATA%\$NAME\config.json` on Windows
+ `$XDG_CONFIG_HOME\$NAME\config.json` or `~/.config/$NAME/config.json` on Linux
+ `~/Library/Application Support/$NAME/config.json` on macOS

In the paths above, `$NAME` is typically `Element`, unless you use `--profile
$PROFILE` in which case it becomes `Element-$PROFILE`, or it is using one of
the above created by a pre-1.7 install, in which case it will be `Riot` or
`Riot-$PROFILE`.

See https://github.com/vector-im/element-web/blob/develop/docs/config.md

Electron Config
===============

Electron config is stored in the same folder as the [user specified config](#user-specified-configjson) 
in `electron-config.json`.
This is for configuring Electron options as opposed to Matrix/Element options from `config.json`. 

This has the following config options:

* `warnBeforeExit`: boolean, optional (default: `true`)
  * `true`: Element will display a confirmation box before exiting.
* `minimizeToTray`: boolean, optional (default: `true`)
  * `true`: Element will enable the tray.
* `spellCheckerEnabled`: boolean, optional (default: `true`)
  * `true`: Element spell checking is enabled.
* `autoHideMenuBar`: boolean, optional (default: `true`)
  * `true`: Element will automatically hide the menu bar.
* `locale`: string[], optional
  * A list of locales for Element to enable.
* `proxy`: object, optional (default: `null`)
  * Proxy configuration for Electron to use. [More info](https://www.electronjs.org/docs/api/session#sessetproxyconfig).
  * `mode`: string, optional (default: `null`)
    * Proxy mode. If it's unspecified, it will be automatically determined based on other specified options.
    * `direct`: All connections go direct without a proxy involved.
    * `auto_detect`: The proxy configuration is determined by a PAC script that will be downloaded from `wpad/wpad.dat`.
    * `pac_script`: The proxy configuration is determined by a PAC script specified in `pacScript`.
      This is the default mode if `pacScript` is specified.
    * `fixed_servers`: The proxy configuration is specified in `proxyRules`.
      This is the default mode if `proxyRules` is specified and `pacScript` is not specified.
    * `system`: The proxy configuration is taken from the operating system.
  * `pacScript`: string, optional (default: null)
    * The URL for a PAC file.
    * This supports local files through the `file:` URI scheme,
      e.g. `file:///home/$USER/proxy.pac` or `file://c:/Users/Username/Documents/proxy.pac`,
      as well as `data:`, `http:`, and `https:` URI schemes.
    * A `data:` URI should follow the format `data:application/x-javascript-config;base64,$BASE64_PAC_CONTENT`;
  * `proxyRules`: string, optional (default: `null`)
    * Rules indicating which proxies to use.
  * `proxyBypassRules`: string, optional (default: `null`)
    * Rules indication which URLs should bypass the proxy settings.

Translations
==========================

To add a new translation, head to the [translating doc](https://github.com/vector-im/element-web/blob/develop/docs/translating.md).

For a developer guide, see the [translating dev doc](https://github.com/vector-im/element-web/blob/develop/docs/translating-dev.md).

[<img src="https://translate.element.io/widgets/element-desktop/-/multi-auto.svg" alt="translationsstatus" width="340">](https://translate.element.io/engage/element-desktop/?utm_source=widget)

Report bugs & give feedback
==========================

If you run into any bugs or have feedback you'd like to share, please let us know on GitHub.

To help avoid duplicate issues, please [view existing issues](https://github.com/vector-im/element-web/issues?q=is%3Aopen+is%3Aissue+sort%3Areactions-%2B1-desc) first (and add a +1) or [create a new issue](https://github.com/vector-im/element-web/issues/new/choose) if you can't find it.  Please note that this issue tracker is associated with the [element-web](https://github.com/vector-im/element-web) repo, but is also applied to the code in this repo as well.
