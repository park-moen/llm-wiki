# Swagger UI Introduction

> Source: https://github.com/swagger-api/swagger-ui
> Original: https://raw.githubusercontent.com/swagger-api/swagger-ui/master/README.md
> Collected: 2026-10-13
> Published: Unknown
> Format: Official repository Markdown source, preserved verbatim below.

# <img src="https://raw.githubusercontent.com/swagger-api/swagger.io/wordpress/images/assets/SWU-logo-clr.png" width="300">

[![NPM version](https://badge.fury.io/js/swagger-ui.svg)](http://badge.fury.io/js/swagger-ui)
[![Build Status](https://github.com/swagger-api/swagger-ui/actions/workflows/nodejs.yml/badge.svg)](https://github.com/swagger-api/swagger-ui/actions/workflows/nodejs.yml)
[![security scan](https://img.shields.io/github/actions/workflow/status/swagger-api/swagger-ui/security-scan.yml?label=Security%20Scan)](https://github.com/swagger-api/swagger-ui/actions/workflows/security-scan.yml)
[![total GitHub contributors](https://img.shields.io/github/contributors-anon/swagger-api/swagger-ui.svg)](https://github.com/swagger-api/swagger-ui/graphs/contributors)

[![monthly npm installs](https://img.shields.io/npm/dm/swagger-ui.svg?label=npm%20downloads)](https://www.npmjs.com/package/swagger-ui)
![docker registry](https://img.shields.io/badge/docker-docker.swagger.io%2Fswaggerapi%2Fswagger--ui-blue)
![monthly packagist installs](https://img.shields.io/packagist/dm/swagger-api/swagger-ui.svg?label=packagist%20installs)
[![gzip size](https://img.shields.io/bundlephobia/minzip/swagger-ui.svg?label=gzip%20size)](https://bundlephobia.com/package/swagger-ui)

## Introduction
[Swagger UI](https://swagger.io/tools/swagger-ui/) allows anyone — be it your development team or your end consumers — to visualize and interact with the API’s resources without having any of the implementation logic in place. It’s automatically generated from your OpenAPI (formerly known as Swagger) Specification, with the visual documentation making it easy for back end implementation and client side consumption.

## General
**👉🏼 Want to score an easy open-source contribution?** Check out our [Good first issue](https://github.com/swagger-api/swagger-ui/issues?q=is%3Aissue+is%3Aopen+label%3A%22Good+first+issue%22) label.

**🕰️ Looking for the older version of Swagger UI?** Refer to the [*2.x* branch](https://github.com/swagger-api/swagger-ui/tree/2.x).


This repository publishes three different NPM modules:

* [swagger-ui](https://www.npmjs.com/package/swagger-ui) is a traditional npm module intended for use in single-page applications that are capable of resolving dependencies (via Webpack, Browserify, etc.).
* [swagger-ui-dist](https://www.npmjs.com/package/swagger-ui-dist) is a dependency-free module that includes everything you need to serve Swagger UI in a server-side project, or a single-page application that can't resolve npm module dependencies.
* [swagger-ui-react](https://www.npmjs.com/package/swagger-ui-react) is Swagger UI packaged as a React component for use in React applications.

We strongly suggest that you use `swagger-ui` instead of `swagger-ui-dist` if you're building a single-page application, since `swagger-ui-dist` is significantly larger.

If you are looking for plain ol' HTML/JS/CSS, [download the latest release](https://github.com/swagger-api/swagger-ui/releases/latest) and copy the contents of the `/dist` folder to your server.


## Compatibility
The OpenAPI Specification has undergone 5 revisions since initial creation in 2010.  Compatibility between Swagger UI and the OpenAPI Specification is as follows:

| Swagger UI Version | Release Date | OpenAPI Spec compatibility                                  | Notes                                                                 |
|--------------------|--------------|-------------------------------------------------------------|-----------------------------------------------------------------------|
| 5.32.0             | 2026-02-27   | 2.0, 3.0.0, 3.0.1, 3.0.2, 3.0.3, 3.0.4, 3.1.0, 3.1.1, 3.1.2, 3.2.0 | [tag v5.32.0](https://github.com/swagger-api/swagger-ui/tree/v5.32.0) |
| 5.19.0             | 2025-02-17   | 2.0, 3.0.0, 3.0.1, 3.0.2, 3.0.3, 3.0.4, 3.1.0, 3.1.1, 3.1.2 | [tag v5.19.0](https://github.com/swagger-api/swagger-ui/tree/v5.19.0) |
| 5.0.0              | 2023-06-12   | 2.0, 3.0.0, 3.0.1, 3.0.2, 3.0.3, 3.1.0               | [tag v5.0.0](https://github.com/swagger-api/swagger-ui/tree/v5.0.0)   |
| 4.0.0              | 2021-11-03   | 2.0, 3.0.0, 3.0.1, 3.0.2, 3.0.3                      | [tag v4.0.0](https://github.com/swagger-api/swagger-ui/tree/v4.0.0)   |
| 3.18.3             | 2018-08-03   | 2.0, 3.0.0, 3.0.1, 3.0.2, 3.0.3                      | [tag v3.18.3](https://github.com/swagger-api/swagger-ui/tree/v3.18.3) |
| 3.0.21             | 2017-07-26   | 2.0                                                  | [tag v3.0.21](https://github.com/swagger-api/swagger-ui/tree/v3.0.21) |
| 2.2.10             | 2017-01-04   | 1.1, 1.2, 2.0                                        | [tag v2.2.10](https://github.com/swagger-api/swagger-ui/tree/v2.2.10) |
| 2.1.5              | 2016-07-20   | 1.1, 1.2, 2.0                                        | [tag v2.1.5](https://github.com/swagger-api/swagger-ui/tree/v2.1.5)   |
| 2.0.24             | 2014-09-12   | 1.1, 1.2                                             | [tag v2.0.24](https://github.com/swagger-api/swagger-ui/tree/v2.0.24) |
| 1.0.13             | 2013-03-08   | 1.1, 1.2                                             | [tag v1.0.13](https://github.com/swagger-api/swagger-ui/tree/v1.0.13) |
| 1.0.1              | 2011-10-11   | 1.0, 1.1                                             | [tag v1.0.1](https://github.com/swagger-api/swagger-ui/tree/v1.0.1)   |

## Anonymized analytics

SwaggerUI uses [Scarf](https://scarf.sh/) to collect [anonymized installation analytics](https://github.com/scarf-sh/scarf-js?tab=readme-ov-file#as-a-user-of-a-package-using-scarf-js-what-information-does-scarf-js-send-about-me). These analytics help support the maintainers of this library and ONLY run during installation. To [opt out](https://github.com/scarf-sh/scarf-js?tab=readme-ov-file#as-a-user-of-a-package-using-scarf-js-how-can-i-opt-out-of-analytics), you can set the `scarfSettings.enabled` field to `false` in your project's `package.json`:

```
// package.json
{
  // ...
  "scarfSettings": {
    "enabled": false
  }
  // ...
}
```

Alternatively, you can set the environment variable `SCARF_ANALYTICS` to `false` as part of the environment that installs your npm packages, e.g., `SCARF_ANALYTICS=false npm install`.

## Documentation

#### Usage
- [Installation](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/installation.md)
- [Configuration](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/configuration.md)
- [CORS](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/cors.md)
- [OAuth2](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/oauth2.md)
- [Deep Linking](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/deep-linking.md)
- [Limitations](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/limitations.md)
- [Version detection](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/usage/version-detection.md)

#### Customization
- [Overview](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/customization/overview.md)
- [Plugin API](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/customization/plugin-api.md)
- [Custom layout](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/customization/custom-layout.md)

#### Development
- [Setting up](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/development/setting-up.md)
- [Scripts](https://github.com/swagger-api/swagger-ui/blob/HEAD/docs/development/scripts.md)

#### Contributing
- [Contributing](https://github.com/swagger-api/.github/blob/HEAD/CONTRIBUTING.md)

##### Integration Tests

End-to-end tests use [Cypress](https://www.cypress.io/). Run the full suite locally with `npm run cy:ci`, which starts the required servers, runs Cypress headless, and shuts the servers down afterwards - be sure you aren't running a dev server on the same ports when testing!

To debug or run individual specs interactively, use `npm run cy:dev` to open the Cypress runner.

To run a single spec headless, start the servers in one terminal and run that spec in another:

```sh
npm run cy:start
# in a second terminal:
npm run cy:run -- --spec "test/e2e-cypress/e2e/features/deep-linking.cy.js"
```

### Browser support
Swagger UI works in the latest versions of Chrome, Safari, Firefox, and Edge.

### Known Issues

To help with the migration, here are the currently known issues with 3.X. This list will update regularly, and will not include features that were not implemented in previous versions.

- Only part of the parameters previously supported are available.
- The JSON Form Editor is not implemented.
- Support for `collectionFormat` is partial.
- l10n (translations) is not implemented.
- Relative path support for external files is not implemented.

## Security contact

Please disclose any security-related issues or vulnerabilities by emailing [security@swagger.io](mailto:security@swagger.io), instead of using the public issue tracker.

## License

SwaggerUI is licensed under [Apache 2.0 license](https://github.com/swagger-api/swagger-ui/blob/main/LICENSE).
SwaggerUI comes with an explicit [NOTICE](https://github.com/swagger-api/swagger-ui/blob/main/NOTICE) file
containing additional legal notices and information.


## Source license

```text

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [yyyy] [name of copyright owner]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

```
