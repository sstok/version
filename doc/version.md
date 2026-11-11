Version class
=============

The `Version` class is the primary class used for handling Semantic Versioning.
It provides methods to create, manipulate, and validate version strings.

> [!NOTE]
>
> All examples assume Composer autoloading is used.
>
> `require 'vendor/autoload.php';`

Basic Usage
-----------

Every version must be a valid Semantic Version string; date-based versions
aren’t accepted or supported.

* A version might start with a case-insensitive prefix, such as “v” or “V”,
  which is ignored during validation and comparison.
* A version must always have a major and minor version.
* A version might omit the patch identifier; depending on the provided options
  it’s either set to 0 or null.
* A version might provide a build identifier; when omitted, depending on the
  provided options, it’s either set to 0 or null.
* A version might provide the pre-release in any casing with/without a leading dot.
* A pre-release might use a dot, dash, or nothing for the version separator
  like: 1.0.0-alpha1, 1.0.0-alpha-1, 1.0.0-alpha.1, and any casing.
* Finally, a version might have a trailing -dev suffix, as is used by Composer’s
  lock files. For the final result, this is always ignored.

> [!NOTE]
> Any stability markers like @dev are not accepted; please remove them
> before passing the version string to the Version::fromString() method.

The `Version` class is immutable (and read-only), meaning any mutation will
produce a new object instance with the changes applied.

To compare equality, use the `Version::equals()` method as described.

```php
use Rollerworks\Component\Version\Version;

// Creates an immutable Version value-object.
// See above's notes about acceptable formatting.
$version = Version::fromString('1.3.2');

// Prints 1.3.2
// Actual output depends on the version and configured outputFormat (provided with the constructor).
// patch (and build)
echo $version;

// Prints 1.3.2.0
// With the build (even when no build was provided)
echo $version->full;

// Prints 1.3.2
// Without the build (even if a build was provided)
echo $version->fullWithoutBuild;
```

### Omitting Patch and/or Build

By default, all version identifiers that are passed to `Version::fromString()`
are included in the object instance. If you only need certain identifiers,
you can either omit them from input like `Version::fromString('1.5')` or use
the `$options` parameter as shown in the example:

```php

// Omit the build identifier (by default)
$version = Version::fromString('1.3.2');

// Omit the patch identifier
$version = Version::fromString('1.3');

// This example is invalid, major *and* minor MUST always be provided
$version = Version::fromString('1');

// Alternatively, if you can't control the input directly
// You can use the bitmask `$options` parameter.

// Version is now processed as: 1.3
$version = Version::fromString('1.3.2', options: Version::INPUT_IGNORE_PATCH);

// Version is now processed as: 1.3
$version = Version::fromString('1.3.2-Beta.1', options: Version::INPUT_IGNORE_PATCH | Version::INPUT_IGNORE_PRE_RELEASE);

// Version is now processed as: 1.3
// INPUT_IGNORE_BUILD already implies INPUT_IGNORE_PATCH as a patch
// cannot be omitted when a build identifier is provided
$version = Version::fromString('1.3.2.5', options: Version::INPUT_IGNORE_BUILD);
```

Filling Patch and/or Build
--------------------------

By default, all version identifiers that were omitted with `Version::fromString()`
are marked as `null` for the object properties.

When comparing, this has no effect (as the patch `null` value is converted to 0).
It will, however, affect how the version is represented as a string.

To fill an omitted identifier, use the `$options` argument as shown below.

> [!WARNING]
> 
> Combining `Version::INPUT_IGNORE_PATCH` with `Version::INPUT_FILL_BUILD` is invalid.
> The patch identifier is required when a build is provided.
>
> Passing `Version::INPUT_FILL_BUILD` implies `Version::INPUT_FILL_PATCH` as well.

```php

// Fill the build identifier
// Version is now processed as: 1.3.0.0
$version = Version::fromString('1.3', options: Version::INPUT_FILL_BUILD);

// Fill the patch identifier
// Version is now processed as: 1.3.0
$version = Version::fromString('1.3', options: Version::INPUT_FILL_PATCH);

// This example is invalid, major *and* minor MUST always be provided
$version = Version::fromString('1');
```

CreateFromList
--------------

TBD.

Formatting
----------

Version strings are formatted according to the [Semantic Versioning] specification.
If you want a slightly different format, you can use the `Version::format()` method.

The `format()` method allows you to omit certain identifiers, change the 
casing of pre-releases, and change the separator of the pre-release version.

The default format only displays the patch and build when they are not omitted
from the version (`null`), when they are made "mandatory" in the format they 
are displayed as 0-values.

> [!NOTE]
>
> A version format must still adhere to the specifications of [Semantic Versioning].
> You cannot change the order of identifiers; the major and minor identifiers cannot 
> be omitted, and only certain separators can be used.
>
> See the formatting details in the <a href="#basic-usage">Basic Usage</a> section.
>
> Use the object properties if you need a specific formatting like `Version 1.0.5, build 5.`

Identifiers like `{major}`, `{minor}`, `{patch}`, `{pre-release}`, and `{build}` can be 
used in the format string.

Note that only patch and build identifiers can be omitted or made optional,
the pre-release identifier is only applied when a pre-release version is set.

In short, the following syntax is allowed:

- Must not contain any whitespace characters;
- May optionally start with a leading `v` character (lower or uppercase);
- Must precede with `{major}.{minor}`;
- May proceed with a `{patch}` identifier (use `{?patch}` to only use when a patch was provided);
- May proceed with a `{build}` identifier (use `{?build}` to only use when a build was provided);
- May proceed with `-{pre-release}x` (prefix `dash` and `x` are mandatory) x maybe separated 
  by a dot (`.`) character or dash (`-`) character, or no character (as shown);

To change the casing of the pre-release, use `{pre-release:<case>}`,
`<case>` can be `lower`, `upper`, or `title`, like `{pre-release:upper}`.

No other suffix is allowed after the pre-release identifier.

> [!NOTE]
>
> Identifiers (except for pre-release) must be separated by a dot (`.`) character
> and cannot be reordered. Cannot use `{build}` before (or instead of) a `{patch}`.

A pre-release is only displayed when provided, omitting the identifier from the
format simply removes it.

Full examples:

- `v{major}.{minor}.{patch}{?build}-{pre-release:upper}x` (build is only displayed when provided)
- `{major}.{minor}.{patch}.{build}-{pre-release:lower}x`
- `{major}.{minor}.{?patch}` (patch is only displayed when provided)
- `{major}.{minor}` (minimal accepted version)

```php
use Rollerworks\Component\Version\Version;

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
>
> Need to change an existing format without worrying about syntax rules?
> Use the [VersionFormatter::modify() method](version_formatter.md).
>
> By default, any version increment created with `Version::getNextIncreaseOf()` 
> automatically inherits the previous format with the patch and build identifiers 
> marked as optional; the pre-release format is kept as-is.
>
> The same applies for any version created with `Version::getNextVersionCandidates()`.

Comparisons
-----------

The best way to compare versions is with the various comparator methods.

By default, a build number is ignored when comparing versions (by Semver 
conventions). Any variation in formatting is ignored when comparing.

The method names are self-explanatory; `equalTo` is an alias for `equals`.

```php
use Rollerworks\Component\Version\Version;

$version1 = Version::fromString('1.3.2');

 // patch might be omitted, the full version becomes 1.5.0.0
$version2 = Version::fromString('1.5');

$version->lowerThan($version2);
$version2->higherThan($version1);

$version->lowerThanOrEqual($version2);
$version2->higherThanOrEqual($version1);
```

To make sure the build is part of the comparison, pass `false` to the 
second argument like `$version->lowerThan($version2, ignoreBuild: false)`.

> [!TIP]
> 
> To sort versions, use the special `compareTo()` method, as shown in the
> "full" example below:
>
> ```php
> usort($versions, static fn (Version $a, Version $b): int => $a->compareTo($b, ignoreBuild: false, sortDirection: \SortDirection::Descending));
> ```

Version Increments
------------------

To get the next logical version increment, use the `getNextIncreaseOf()` method with
the requested version identifier to increment.

The `getNextIncreaseOf()` expects a `VersionIdentifier` enum instance.

> [!NOTE]
> 
> When the patch and/or build were omitted, they're treated as 0.

```php
use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionIdentifier;

$version = Version::fromString('1.3.2');

$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Major);  // 2.0.0
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Minor);  // 1.4.0
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Patch);  // 1.3.3
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Build);  // 1.3.3.1

$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Alpha); // v1.4.0-ALPHA1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Beta);  // v1.4.0-BETA1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Rc);    // v1.4.0-RC1

// ---------------------

$version = Version::fromString('1.3');

$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Major);  // 2.0.0
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Minor);  // 1.3.0
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Patch);  // 1.3.1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Build);  // 1.3.0.1

$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Alpha); // 1.4-ALPHA1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Beta);  // 1.4-BETA1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Rc);    // 1.4-RC1
$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Stable); // v1.4.0
```

> [!NOTE]
>
> Depending on the "current" version, "stable", as the name implies, will provide 
> the next (logical) stable release. Which is either a minor version increase 
> for `1.5`, `1.5` for `1.5-beta1` (or any other pre-release), and `1.0` for 
> any unstable version like `0.1`.

### Next Logical Version

Besides the "stable" increment, you can also increase to the "next" logical
version, which is either minor, or a pre-release increase, as follows:

| Current | Next    |
| -------------- | ------------- |
| 0.4.0   | 0.5.0  |
| 1.4.0   | 1.5.0  |
| 1.4.1   | 1.5.0  |
| 1.4.5.7   | 1.5.0.0  |
| 1.4.0-Beta5   | 1.4.0-Beta6  |
| 1.4.0.5-Beta5  | 1.4.0.0-Beta6 |

```php
use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionIdentifier;

$newVersion = $version->getNextIncreaseOf(VersionIdentifier::Next);
```

Next Version Candidates
-----------------------

The `Version` class can produce a list of possible next (logical) 
versions, known as "next" version candidates.

The candidates are used by the [ContinuesVersionsValidator](continues_validator.md)
to validate whether versions are logically incremental to existing ones. 
Aka, that are no gaps between the current version and the next version.

See the dedicated chapter for more details.

To get a list of these version candidates use the `getNextVersionCandidates()`
method, which returns an array of `Version` instances; with the output formatting
of the original `Version` instance.

If you need to exclude certain increment types, use the `$options` parameter
to exclude certain version candidates.

> [!NOTE]
>
> Certain candidates are already excluded by default when their identifier
> was omitted during creation, for example, when no patch version was provided
> any patch candidate is automatically excluded.
>
> A build candidate is only considered when a build version was originally
> provided. *When the patch was omitted (`null`) it will not be considered.*

```php
use Rollerworks\Component\Version\Version;

$versions = $version->getNextIncreaseOf(Version::CANDIDATE_EXCLUDE_PATCH | Version::CANDIDATE_EXCLUDE_BETA);
```

All options (class constants of the `Version` class):

* `CANDIDATE_EXCLUDE_PATCH` 
* `CANDIDATE_EXCLUDE_BUILD` 
* `CANDIDATE_EXCLUDE_ALPHA` 
* `CANDIDATE_EXCLUDE_BETA` 
* `CANDIDATE_EXCLUDE_RC`
* `CANDIDATE_EXCLUDE_PRE_RELEASES`

The `CANDIDATE_EXCLUDE_PRE_RELEASES` includes all pre-release types 
(alpha, beta, and rc).

[Semantic Versioning]: http://semver.org/
