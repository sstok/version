Version class
=============

The `Version` class is the primary class used for handling Semantic Versioning. 
It provides methods for creating, manipulating, and validating version strings.



```php
require 'vendor/autoload.php';

use Rollerworks\Component\Version\Version;
use Rollerworks\Component\Version\VersionsValidator;

// Creates an immutable Version value-object.
// Any call to this object will produce a new Version object
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
// Increasing minor or patch is prohibited until the meta-ver (alpha,beta,rc) is 0
// For patch this resolves to "next".

$version = Version::fromString('v1.4.0-BETA1');
$newVersion = $version->increase('beta');   // v1.4.0-BETA2
$newVersion = $version->increase('rc');     // v1.4.0-RC1
$newVersion = $version->increase('major');  // v1.4.0
$newVersion = $version->increase('next');   // v1.4.0-BETA2
$newVersion = $version->increase('patch');  // v1.4.0-BETA2
$newVersion = $version->increase('stable'); // v1.4.0

// Version validation
// ... //

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

$validator = new ContinuesVersionsValidator(...$existingVersions); // Expects the versions as a variadic arguments
//$validator = new ContinuesVersionsValidator(); // No existing versions

VersionsValidator::isVersionContinues(Version::fromString('v1.1.1'));      // true
VersionsValidator::isVersionContinues(Version::fromString('1.0.2'));       // true
VersionsValidator::isVersionContinues(Version::fromString('1.1.1.'));      // true
VersionsValidator::isVersionContinues(Version::fromString('2.0.1.'));      // true
VersionsValidator::isVersionContinues(Version::fromString('3.5.0-beta2')); // true
VersionsValidator::isVersionContinues(Version::fromString('3.5.0'));       // true

// A new minor or major version is not considered acceptable when there are already higher
// versions. Only patch releases are accepted then.
VersionsValidator::isVersionContinues(Version::fromString('0.2.0'));        // false
VersionsValidator::isVersionContinues(Version::fromString('v1.0.0-beta8')); // false
VersionsValidator::isVersionContinues(Version::fromString('v1.2'));         // false
VersionsValidator::isVersionContinues(Version::fromString('v2.1'));         // false
VersionsValidator::isVersionContinues(Version::fromString('v3.5-alpha1'));  // false
VersionsValidator::isVersionContinues(Version::fromString('v3.5-beta3'));   // false
VersionsValidator::isVersionContinues(Version::fromString('v3.6'));         // false

// A list of possible versions with respect to the major.minor bounds of any existing version
// For higher major.minor versions then validated only suggests a patch release, otherwise
// all possible increments till the next stable major are suggested.
$possibleVersions = $validator->getPossibleVersions();
```
