# Semantic Versioning 2.0.0

> Source: https://semver.org/
> Collected: 2026-08-10
> Published: Unknown

Software using Semantic Versioning MUST declare a public API. This API could be declared in the code itself or exist strictly in documentation. However it is done, it SHOULD be precise and comprehensive.

A normal version number MUST take the form X.Y.Z where X, Y, and Z are non-negative integers, and MUST NOT contain leading zeroes. X is the major version, Y is the minor version, and Z is the patch version. Each element MUST increase numerically.

Once a versioned package has been released, the contents of that version MUST NOT be modified. Any modifications MUST be released as a new version.

Patch version Z (x.y.Z | x > 0) MUST be incremented if only backward compatible bug fixes are introduced.

Minor version Y (x.Y.z | x > 0) MUST be incremented if new, backward compatible functionality is introduced to the public API.

Major version X (X.y.z | X > 0) MUST be incremented if any backward incompatible changes are introduced to the public API.

A pre-release version MAY be denoted by appending a hyphen and a series of dot separated identifiers immediately following the patch version.

Build metadata MAY be denoted by appending a plus sign and a series of dot separated identifiers immediately following the patch or pre-release version.
