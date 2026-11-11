VersionFormatter
================

The `VersionFormatter` is responsible for formatting a `Version` object 
according to specific rules and conventions.

It provides methods to convert a version string into a standardized
format, and to modify an existing format when you need to reuse the
existing format but with some slight modifications.

The best way to use the formatter is with the `Version::$formatter` property.

> [!CAUTION]
>
> Make sure to create a `Version` object before using the formatter property,
> as the formatter property is not available until a `Version` object is created.
>
> Merely calling any factory method on the `Version` class will create a `Version` object, 
> and thus the formatter property will become be initialized.

Syntax
------

The formatting syntax is defined as follows:

Identifiers are encoded as placeholders in the format string like `{major}` 
for example, certain placeholders can be omitted or made optional, 
and some might have additional options like casing.

Identifiers like `{major}`, `{minor}`, `{patch}`, `{pre-release}`, and 
`{build}` can be used in the format string.

Only `{patch}` and` {build}` identifiers can be omitted or made optional,
the `{pre-release}` identifier is only applied when a pre-release version 
is set in the `Version` instance. A pre-release is only displayed when 
provided, omitting the identifier from the format simply removes it.

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

Full examples:

- `v{major}.{minor}.{patch}{?build}-{pre-release:upper}x` (build is only displayed when provided)
- `{major}.{minor}.{patch}.{build}-{pre-release:lower}x`
- `{major}.{minor}.{?patch}` (patch is only displayed when provided)
- `{major}.{minor}` (minimal accepted version)

Formatting
----------

Version strings are formatted according to the [Semantic Versioning] specification.
If you want a slightly different format, you can use the [Version::format()](version.md#formatting) 
method.

The `format()` method allows you to omit certain identifiers, change the 
casing of pre-releases, and change the separator of the pre-release version.

See details in the [Syntax](#syntax) section.

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
