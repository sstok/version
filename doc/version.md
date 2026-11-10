Version class
=============

The `Version` class is the primary class used for handling Semantic Versioning. 
It provides methods for creating, manipulating, and validating version strings.

Basic Usage
-----------

Every version must be a valid Semantic Version string, date-based versions
aren not accepted or supported.

- A version might start with a case-insensitive prefix, such as "v" or "V", 
  which is ignored during validation and comparison.
- A version must always have a major and minor version.
- A version might omit the patch identifier, depending the provided options
  it's either turned to 0 or null.
- A version might provide a build identifier, when omitted depending the 
  provided options it's either turned to 0 or null.
- A version might provide the pre-release in any casing with/without a leading
  dot.
- A pre-release might use a dot, dash or nothing for the version separator
  like: `1.0.0-alpha1`, `1.0.0-alpha-1`, `1.0.0-alpha.1`, and any casing.
- Finally a version might have trailing `-dev` suffix, as is used by Composer's
  lock files. For the final result this is always ignored.

> [!NOTE]
> Any stability markers like `@dev` are not accepted, please remove them
> before passing the version string to the `Version::fromString()` method.

The `Version` class is immutable (and readonly) meaning any mutation will 
produce a new object instance with the changes applied.

To compare equality use the `Version::equals()` method as described.

```php
require 'vendor/autoload.php';

use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionsValidator;

// Creates an immutable Version value-object.
// See above's notes about acceptable formatting.
$version = Version::fromString('1.3.2');

// Prints 1.3.2
// Actual output depends on the version and configured outputFormat (provided with the constructor).
// patch (and build)
echo $version;

// Prints 1.3.2.0
// With the build (even when no a build was provided)
echo $version->full;

// Prints 1.3.2
// Without the build (even if a build was provided)
echo $version->fullWithoutBuild;
```

Formatting
----------

Version strings are by formatted according to the [Semantic Versioning] specification.
If you want a different format you can use the `Version::format()` method.

The `format()` method allows to omit certain identifiers, change the casing of pre-releases, 
and change the separator of the pre-release version.

> [!NOTE]
> A version formatting must still adhere the specification of [Semantic Versioning].
> You cannot change the order of identifiers, the major and minor identifiers cannot 
> be omitted, and only certain separators can be used.
>
> See the formatting details in <a href="#basic-usage">Basic Usage</a> section.
>
> Use the object properties if you need a specific formatting like `Version 1.0.5, build 5.`.

Identifiers like `{major}`, `{minor}`, `{patch}`, `{pre-release}`, and `{build}` can be 
used in the format string.

Note that only patch and build identifiers can be omitted or made optional,
the pre-release identifier is only applied when a pre-release version is set.

To change the casing of pre-releases use the following syntax `{pre-release:lower}` or 
`{pre-release:upper}`.

In short the following syntax is allowed:

- Must not contain any whitespace characters;
- May optionally start with a leading `v` character (lower or uppercase);
- Must precede with `{major}.{minor}`;
- May proceed with a `{patch}` identifier (use `{?patch}` to only use when a patch was provided);
- May proceed with a `{build}` identifier (use `{?build}` to only use when a build was provided);
- May proceed with `-{pre-release}x` (prefix `dash` and `x` are mandatory) x maybe separated 
  by a dot (`.`) character or dash (`-`) character, or no character (as shown);

To change the casing of the pre-release, use `{pre-release:<case>}`,
`<case>` can be `lower`, `upper` or `title`, like `{pre-release:upper}`.

No other suffix is allowed after the pre-release identifier.

> [!NOTE]
> Identifiers (except for pre-release) must be separated by a dot (`.`) character
> and cannot be reordered. Cannot use `{build}` before (or instead of) a `{patch}`.

A pre-release is only displayed when provided.

Full examples:

- `v{major}.{minor}.{patch}{?build}-{pre-release:upper}x` (build is only displayed when provided)
- `{major}.{minor}.{patch}.{build}-{pre-release:lower}x`
- `{major}.{minor}.{?patch}` (patch is only displayed when provided)
- `{major}.{minor}` (minimal accepted version)

```php
require 'vendor/autoload.php';

use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionsValidator;

// v1.3.2
echo Version::fromString('1.3.2')->format('v{major}.{minor}.{patch}{?build}-{pre-release:upper}x');

// v1.3.2.5
echo Version::fromString('1.3.2.5')->format('v{major}.{minor}.{patch}{?build}-{pre-release:upper}x');

// v1.3.2.0
echo Version::fromString('1.3.2')->format('v{major}.{minor}.{patch}{build}-{pre-release:upper}x');

// v1.3.2.0-Beta.6
echo Version::fromString('1.3.2-BETA6')->format('v{major}.{minor}.{patch}{build}-{pre-release:title}.x');
```

> [!TIP]
> Need to change an existing format without worrying about syntax rules?
> Use the [VersionFormatter::modify() method](doc/version_formatter.md).

Version Increases
-----------------



```php
require 'vendor/autoload.php';

use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionsValidator;

$version = Version::fromString('v1.3.2');

$newVersion = $version->increase('major');  // v2.0.0
$newVersion = $version->increase('minor');  // v1.4.0
$newVersion = $version->increase('next');   // v1.4.0
$newVersion = $version->increase('patch');  // v1.3.3
$newVersion = $version->increase('stable'); // v1.4.0

$newVersion = $version->increase('alpha'); // v1.4.0-ALPHA1
$newVersion = $version->increase('beta');  // v1.4.0-BETA1
$newVersion = $version->increase('rc');    // v1.4.0-RC1

// ...
// Increasing minor or patch is prohibited until the pre-release (alpha,beta,rc) is 0
// For patch this resolves to "next".

$version = Version::fromString('v1.4.0-BETA1');

$newVersion = $version->increase('beta');   // v1.4.0-BETA2
$newVersion = $version->increase('rc');     // v1.4.0-RC1
$newVersion = $version->increase('major');  // v1.4.0
$newVersion = $version->increase('next');   // v1.4.0-BETA2
$newVersion = $version->increase('patch');  // v1.4.0-BETA2
$newVersion = $version->increase('stable'); // v1.4.0
```


[Semantic Versioning]: http://semver.org/
