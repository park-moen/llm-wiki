# P6Spy official spy.properties

> Source: https://raw.githubusercontent.com/p6spy/p6spy/master/src/main/assembly/individualFiles/spy.properties
> Collected: 2026-10-13
> Published: Unknown
> Format: Official repository source from master, preserved verbatim below.

```properties
###
# #%L
# P6Spy
# %%
# Copyright (C) 2002 - 2017 P6Spy
# %%
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# #L%
###

#################################################################
# P6Spy Options File                                            #
# See documentation for detailed instructions                   #
# http://p6spy.github.io/p6spy/2.0/configandusage.html          #
#################################################################

#################################################################
# MODULES                                                       #
#                                                               #
# Module list adapts the modular functionality of P6Spy.        #
# Only modules listed are active.                               #
# (default is com.p6spy.engine.logging.P6LogFactory and         #
# com.p6spy.engine.spy.P6SpyFactory)                            #
# Please note that the core module (P6SpyFactory) can't be      #
# deactivated.                                                  #
# Unlike the other properties, activation of the changes on     #
# this one requires reload.                                     #
#################################################################
#modulelist=com.p6spy.engine.spy.P6SpyFactory,com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory

################################################################
# CORE (P6SPY) PROPERTIES                                      #
################################################################

# A comma separated list of JDBC drivers to load and register.
# (default is empty)
#
# Note: This is normally only needed when using P6Spy in an
# application server environment with a JNDI data source or when
# using a JDBC driver that does not implement the JDBC 4.0 API
# (specifically automatic registration).
#driverlist=

# for flushing per statement
# (default is false)
#autoflush=false

# sets the date format using Java's SimpleDateFormat routine.
# In case property is not set, milliseconds since 1.1.1970 (unix time) is used (default is empty)
#dateformat=

# prints a stack trace for every statement logged
#stacktrace=false
# if stacktrace=true, specifies the stack trace to print
#stacktraceclass=

# determines if property file should be reloaded
# Please note: reload means forgetting all the previously set
# settings (even those set during runtime - via JMX)
# and starting with the clean table
# (default is false)
#reloadproperties=false

# determines how often should be reloaded in seconds
# (default is 60)
#reloadpropertiesinterval=60

# specifies the appender to use for logging
# Please note: reload means forgetting all the previously set
# settings (even those set during runtime - via JMX)
# and starting with the clean table
# (only the properties read from the configuration file)
# (default is com.p6spy.engine.spy.appender.FileLogger)
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
#appender=com.p6spy.engine.spy.appender.StdoutLogger
#appender=com.p6spy.engine.spy.appender.FileLogger

# name of logfile to use, note Windows users should make sure to use forward slashes in their pathname (e:/test/spy.log)
# (used for com.p6spy.engine.spy.appender.FileLogger only)
# (default is spy.log)
#logfile=spy.log

# append to the p6spy log file. if this is set to false the
# log file is truncated every time. (file logger only)
# (default is true)
#append=true

# class to use for formatting log messages (default is: com.p6spy.engine.spy.appender.SingleLineFormat)
#logMessageFormat=com.p6spy.engine.spy.appender.SingleLineFormat

# Custom log message format used ONLY IF logMessageFormat is set to com.p6spy.engine.spy.appender.CustomLineFormat
# default is %(currentTime)|%(executionTime)|%(category)|connection%(connectionId)|%(sqlSingleLine)
# Available placeholders are:
#   %(connectionId)            the id of the connection
#   %(currentTime)             the current time expressing in milliseconds
#   %(executionTime)           the time in milliseconds that the operation took to complete
#   %(category)                the category of the operation
#   %(effectiveSql)            the SQL statement as submitted to the driver
#   %(effectiveSqlSingleLine)  the SQL statement as submitted to the driver, with all new lines removed
#   %(sql)                     the SQL statement with all bind variables replaced with actual values
#   %(sqlSingleLine)           the SQL statement with all bind variables replaced with actual values, with all new lines removed
#   %(url)                     the database url where the sql statement executed
#customLogMessageFormat=%(currentTime)|%(executionTime)|%(category)|connection%(connectionId)|%(sqlSingleLine)

# format that is used for logging of the java.util.Date implementations (has to be compatible with java.text.SimpleDateFormat)
# (default is yyyy-MM-dd'T'HH:mm:ss.SSSZ)
#databaseDialectDateFormat=yyyy-MM-dd'T'HH:mm:ss.SSSZ

# format that is used for logging of the java.sql.Timestamp implementations (has to be compatible with java.text.SimpleDateFormat)
# (default is yyyy-MM-dd'T'HH:mm:ss.SSSZ)
#databaseDialectTimestampFormat=yyyy-MM-dd'T'HH:mm:ss.SSSZ

# format that is used for logging booleans, possible values: boolean, numeric
# (default is boolean)
# databaseDialectBooleanFormat=boolean

# Specifies the format for logging binary data. Not applicable if excludebinary is true.
# (default is com.p6spy.engine.logging.format.HexEncodedBinaryFormat)
#databaseDialectBinaryFormat=com.p6spy.engine.logging.format.PostgreSQLBinaryFormat
#databaseDialectBinaryFormat=com.p6spy.engine.logging.format.MySQLBinaryFormat
#databaseDialectBinaryFormat=com.p6spy.engine.logging.format.HexEncodedBinaryFormat

# whether to expose options via JMX or not
# (default is true)
#jmx=true

# if exposing options via jmx (see option: jmx), what should be the prefix used?
# jmx naming pattern constructed is: com.p6spy(.<jmxPrefix>)?:name=<optionsClassName>
# please note, if there is already such a name in use it would be unregistered first (the last registered wins)
# (default is none)
#jmxPrefix=

# if set to true, the execution time will be measured in nanoseconds as opposed to milliseconds
# (default is false)
#useNanoTime=false

#################################################################
# DataSource replacement                                        #
#                                                               #
# Replace the real DataSource class in your application server  #
# configuration with the name com.p6spy.engine.spy.P6DataSource #
# (that provides also connection pooling and xa support).       #
# then add the JNDI name and class name of the real             #
# DataSource here                                               #
#                                                               #
# Values set in this item cannot be reloaded using the          #
# reloadproperties variable. Once it is loaded, it remains      #
# in memory until the application is restarted.                 #
#                                                               #
#################################################################
#realdatasource=/RealMySqlDS
#realdatasourceclass=com.mysql.jdbc.jdbc2.optional.MysqlDataSource

#################################################################
# DataSource properties                                         #
#                                                               #
# If you are using the DataSource support to intercept calls    #
# to a DataSource that requires properties for proper setup,    #
# define those properties here. Use name value pairs, separate  #
# the name and value with a semicolon, and separate the         #
# pairs with commas.                                            #
#                                                               #
# The example shown here is for mysql                           #
#                                                               #
#################################################################
#realdatasourceproperties=port;3306,serverName;myhost,databaseName;jbossdb,foo;bar

#################################################################
# JNDI DataSource lookup                                        #
#                                                               #
# If you are using the DataSource support outside of an app     #
# server, you will probably need to define the JNDI Context     #
# environment.                                                  #
#                                                               #
# If the P6Spy code will be executing inside an app server then #
# do not use these properties, and the DataSource lookup will   #
# use the naming context defined by the app server.             #
#                                                               #
# The two standard elements of the naming environment are       #
# jndicontextfactory and jndicontextproviderurl. If you need    #
# additional elements, use the jndicontextcustom property.      #
# You can define multiple properties in jndicontextcustom,      #
# in name value pairs. Separate the name and value with a       #
# semicolon, and separate the pairs with commas.                #
#                                                               #
# The example shown here is for a standalone program running on #
# a machine that is also running JBoss, so the JNDI context     #
# is configured for JBoss (3.0.4).                              #
#                                                               #
# (by default all these are empty)                              #
#################################################################
#jndicontextfactory=org.jnp.interfaces.NamingContextFactory
#jndicontextproviderurl=localhost:1099
#jndicontextcustom=java.naming.factory.url.pkgs;org.jboss.naming:org.jnp.interfaces

#jndicontextfactory=com.ibm.websphere.naming.WsnInitialContextFactory
#jndicontextproviderurl=iiop://localhost:900

################################################################
# P6 LOGGING SPECIFIC PROPERTIES                               #
################################################################

# filter what is logged
# please note this is a precondition for usage of: include/exclude/sqlexpression
# (default is false)
#filter=false

# comma separated list of strings to include
# please note that special characters escaping (used in java) has to be done for the provided regular expression
# (default is empty)
#include=
# comma separated list of strings to exclude
# (default is empty)
#exclude=

# sql expression to evaluate if using regex
# please note that special characters escaping (used in java) has to be done for the provided regular expression
# (default is empty)
#sqlexpression=

#list of categories to exclude: error, info, batch, debug, statement,
#commit, rollback, result and resultset are valid values
# (default is info,debug,result,resultset,batch)
#excludecategories=info,debug,result,resultset,batch

#whether the binary values (passed to DB or retrieved ones) should be logged with placeholder: [binary] or not.
# (default is false)
#excludebinary=false

# Execution threshold applies to the standard logging of P6Spy.
# While the standard logging logs out every statement
# regardless of its execution time, this feature puts a time
# condition on that logging. Only statements that have taken
# longer than the time specified (in milliseconds) will be
# logged. This way it is possible to see only statements that
# have exceeded some high water mark.
# This time is reloadable.
#
# executionThreshold=integer time (milliseconds)
# (default is 0)
#executionThreshold=

################################################################
# P6 OUTAGE SPECIFIC PROPERTIES                                #
################################################################
# Outage Detection
#
# This feature detects long-running statements that may be indicative of
# a database outage problem. If this feature is turned on, it will log any
# statement that surpasses the configurable time boundary during its execution.
# When this feature is enabled, no other statements are logged except the long
# running statements. The interval property is the boundary time set in seconds.
# For example, if this is set to 2, then any statement requiring at least 2
# seconds will be logged. Note that the same statement will continue to be logged
# for as long as it executes. So if the interval is set to 2, and the query takes
# 11 seconds, it will be logged 5 times (at the 2, 4, 6, 8, 10 second intervals).
#
# outagedetection=true|false
# outagedetectioninterval=integer time (seconds)
#
# (default is false)
#outagedetection=false
# (default is 60)
#outagedetectioninterval=30

```

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
      boilerplate notice, with the fields enclosed by brackets "{}"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright 2016 P6Spy

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
