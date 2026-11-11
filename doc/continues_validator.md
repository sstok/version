Continues Versions Validator
============================

The `ContinuesVersionsValidator` class validates whether a version is
a continuation of any previous versions aka whether there are no gaps.

[HuPKit] uses the `ContinuesVersionsValidator` for the `release` command to 
ensure a new release follows a logical order and no major jumps are made.

```
 Provided version: 4.0.0

 [WARNING] It appears there is a gap compared to the last version.

           Expected one of: 2.0.1, 2.1.0-BETA1, 2.1.0, 3.0.0-ALPHA1, 3.0.0-BETA1, 3.0.0
```

> [!NOTE]
> 
> The `ContinuesVersionsValidator` is not meant to be used as a service, 
> but rather as a standalone validator. Existing versions are expected to be 
> provided as variadic arguments to the constructor.

```php
use Rollerworks\Component\Version\ContinuesVersionsValidator;
use Rollerworks\Component\Version\Version;

// The ordering of versions is unimportant, they always sorted internally.
$existingVersions = [
    Version::fromString('0.1.0'),
    Version::fromString('v1.0.0-beta1'),
    Version::fromString('v1.0.0-beta2'),
    Version::fromString('v1.0.0-beta6'),
    Version::fromString('v1.0.0-beta7'),
    Version::fromString('1.0.0'),
    Version::fromString('v1.0.1'),
    Version::fromString('v1.1.0'),
    Version::fromString('v2.0.0'),
    Version::fromString('v3.5-beta1'),
];

// Expects the versions as a variadic arguments
$validator = new ContinuesVersionsValidator(...$existingVersions);

// Pass no argument when there are no existing versions
$validator = new ContinuesVersionsValidator();

// This version would be major jump from 1.1.0!
$newVersion = Version::fromString('1.5.0');


// This version seems more logical;
//
// while new major versions exist, the validator only
// checks within the range of the same logical version (like major in this case).
// 
// And 1.1.1 would also be a logical increment.
// $newVersion = Version::fromString('1.2.0');

// First check if the version is continuing
if (! $validator->isContinues($newVersion)) {
    echo sprintf('Version "%s" does not seem to be a logical increment.'. PHP_EOL;
    echo 'Expected either: ' . implode(', ', $validator->getPossibleVersions()) . PHP_EOL;
}
```


> [!TIP]
>
> To create a list of existing versions use the `Version::createFromList()` 
> factory method. See [Version::createFromList()](version.md#createfromlist)
> for details.

Excluding Candidates
--------------------

If you need to exclude certain increment types, use the `$options` parameter
with `isContinues()` to exclude certain version candidates.

> [!NOTE]
>
> Certain candidates are already excluded by default when their identifier
> was omitted during creation, for example, when no patch version was provided
> any patch candidate is automatically excluded.
>
> For the best result either omit or fill the patch (and build) identifier
> when you expect that not all versions share the same formatting.
>
> A build candidate is only considered when a build version was originally
> provided. *When the patch was omitted (`null`) it will not be considered.*

```php
use Rollerworks\Component\Version\Version;

if (! $validator->isContinues($newVersion, Version::CANDIDATE_EXCLUDE_PATCH | Version::CANDIDATE_EXCLUDE_BETA)) {
    // ...
}
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

[HuPKit]: https://hupkit.dev
