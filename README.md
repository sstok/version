Rollerworks Semver Component
============================

A library for handling Semantic Versioning.

- Build-version support (`major.minor.patch.build`) 
- Validating whether versions are logically continues.
- Finding next possible version version increments.
- Version formatting.

Requirements
------------

You need at least PHP 8.3.

Installation
------------

To install this package, add `rollerworks/version` to your composer.json

```bash
$ php composer.phar require rollerworks/version
```

This command requires you to have Composer installed globally, as explained
in the [installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

Now, Composer will automatically download all required files, and install them
for you.

Basic usage
-----------

The [Version](doc/version.md) class is the primary class used for handling Semantic Versioning. 
It provides methods for creating, manipulating, and validating version strings.

The [ContinuesVersionsValidator](doc/continues_validator.md) allows to validate 
if a version is (logically) continues within a list of existing versions.

Finally the [VersionFormatter](doc/version_formatter.md) class provides some methods 
to format versions according to different styles, or modify an existing format to
either include or exclude version identifiers.

Versioning
----------

For transparency and insight into the release cycle, and for striving
to maintain backward compatibility, this package is maintained under
the Semantic Versioning guidelines as much as possible.

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

And constructed with the following guidelines:

* Breaking backward compatibility bumps the major (and resets the minor and patch)
* New additions without breaking backward compatibility bumps the minor (and resets the patch)
* Bug fixes and misc changes bumps the patch

For more information on SemVer, please visit <http://semver.org/>.

> [!IMPORTANT]
> Older versions of this library are no longer supported.
> Only critical bug fixes will be released for older versions.

License
-------

The source of this package is subject to the MIT license that is bundled
with this source code in the file [LICENSE](LICENSE).

This library is maintained by [Sebastiaan Stok](https://github.com/sstok).
