# P6Spy P6LogQuery: elapsed time and logging

> Source: https://raw.githubusercontent.com/p6spy/p6spy/master/src/main/java/com/p6spy/engine/common/P6LogQuery.java
> Collected: 2026-10-13
> Published: Unknown
> Format: Official repository source from master, preserved verbatim below.

```java
/**
 * P6Spy
 *
 * Copyright (C) 2002 P6Spy
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.p6spy.engine.common;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.text.SimpleDateFormat;
import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.TimeUnit;
import java.util.regex.Pattern;

import com.p6spy.engine.logging.Category;
import com.p6spy.engine.logging.P6LogLoadableOptions;
import com.p6spy.engine.logging.P6LogOptions;
import com.p6spy.engine.spy.P6ModuleManager;
import com.p6spy.engine.spy.P6SpyOptions;
import com.p6spy.engine.spy.appender.FileLogger;
import com.p6spy.engine.spy.appender.FormattedLogger;
import com.p6spy.engine.spy.appender.MessageFormattingStrategy;
import com.p6spy.engine.spy.appender.P6Logger;
import com.p6spy.engine.spy.option.P6OptionChangedListener;

public class P6LogQuery implements P6OptionChangedListener {
  
  private static final Set<Category> CATEGORIES_IMPLICITLY_INCLUDED = new HashSet<Category>(
	      Arrays.asList(Category.ERROR, Category.OUTAGE /* we still want to have outage category enabled! */));
  /**
   * Options that cause re-init of {@link P6LogQuery}.
   */
  private static final Set<String> ON_CHANGE = new HashSet<String>(Arrays.asList(
    P6SpyOptions.APPENDER_INSTANCE, P6SpyOptions.LOGFILE, P6SpyOptions.LOG_MESSAGE_FORMAT_INSTANCE));

  protected static P6Logger logger;

  static {
    initialize();
  }

  public void optionChanged(final String key, final Object oldValue, final Object newValue) {
    if (ON_CHANGE.contains(key)) {
      initialize();
    }
  }

  public static synchronized void initialize() {
    final P6ModuleManager moduleManager = P6ModuleManager.getInstance();
    if (null == moduleManager) {
      // not initialized yet => can't proceed
      return;
    }
    
    final P6SpyOptions opts = moduleManager.getOptions(P6SpyOptions.class);
    logger = opts.getAppenderInstance();
    if (logger != null) {
      if (logger instanceof FileLogger) {
        final String logfile = opts.getLogfile();
        ((FileLogger) logger).setLogfile(logfile);
      }
      if (logger instanceof FormattedLogger) {
        final MessageFormattingStrategy strategy = opts.getLogMessageFormatInstance();
        if (strategy != null) {
          ((FormattedLogger) logger).setStrategy(strategy);
        }
      }
    }
  }

  protected static void doLog(long elapsed, Category category, String prepared, String sql) {
    doLog(-1, elapsed, category, prepared, sql, "");
  }

  // this is an internal method called by logElapsed
  protected static void doLogElapsed(int connectionId, long timeElapsedNanos, Category category, String prepared, String sql, String url) {
    doLog(connectionId, timeElapsedNanos, category, prepared, sql, url);
  }

	/**
	 * Writes log information provided.
	 * 
	 * @param connectionId
	 * @param elapsedNanos
	 * @param category
	 * @param prepared
	 * @param sql
   * @param url
	 */
	protected static void doLog(int connectionId, long elapsedNanos, Category category, String prepared, String sql, String url) {
	    // give it one more try if not initialized yet
	    if (logger == null) {
	      initialize();
	      if (logger == null) {
	    	  return;
	      }
	    }
    
      final String format = P6SpyOptions.getActiveInstance().getDateformat();
      final String stringNow;
      if (format == null) {
        stringNow = Long.toString(System.currentTimeMillis());
      } else {
        stringNow = new SimpleDateFormat(format).format(new java.util.Date()).trim();
      }

      logger.logSQL(connectionId, stringNow, TimeUnit.NANOSECONDS.toMillis(elapsedNanos), category, prepared, sql, url);

      final boolean stackTrace = P6SpyOptions.getActiveInstance().getStackTrace();
      if (stackTrace) {
	    final String stackTraceClass = P6SpyOptions.getActiveInstance().getStackTraceClass();
    	Exception e = new Exception();
        if (stackTraceClass != null) {
          StringWriter sw = new StringWriter();
          PrintWriter pw = new PrintWriter(sw);
          e.printStackTrace(pw);
          String stack = sw.toString();
          if (stack.indexOf(stackTraceClass) == -1) {
            e = null;
          }
        }
        if (e != null) {
          logger.logException(e);
        }
      }
  }

  static boolean isLoggable(String sql) {
    // empty batches and connection commits/rollbacks
    if (sql == null || sql.isEmpty()) {
      return true;
    }

    final P6LogLoadableOptions opts = P6LogOptions.getActiveInstance();
    
    if (!opts.getFilter()) {
      return true;
    }

    final Pattern sqlExpressionPattern = opts.getSQLExpressionPattern();
    final Pattern includeExcludePattern = opts.getIncludeExcludePattern();
    
    return (sqlExpressionPattern == null || sqlExpressionPattern.matcher(sql).matches())
        && (includeExcludePattern == null || includeExcludePattern.matcher(sql).matches());
  }

  static boolean isCategoryOk(Category category) {
    final P6LogLoadableOptions opts = P6LogOptions.getActiveInstance();
    if (null == opts) {
      return CATEGORIES_IMPLICITLY_INCLUDED.contains(category);
    }
    
    final Set<Category> excludeCategories = opts.getExcludeCategoriesSet();
    
    return logger != null && logger.isCategoryEnabled(category) 
    	&& (excludeCategories == null || !excludeCategories.contains(category));
  }
  
  // ----------------------------------------------------------------------------------------------------------
  // public accessor methods for logging and viewing query data
  // ----------------------------------------------------------------------------------------------------------

  public static void log(Category category, String prepared, String sql) {
    if (logger != null && isCategoryOk(category)) {
      doLog(-1, category, prepared, sql);
    }
  }

  public static void log(Category category, Loggable loggable) {
    if (logger != null && isCategoryOk(category) && isLoggable(loggable.getSql())) {
      doLog(-1, category, loggable.getSql(), loggable.getSqlWithValues());
    }
  }

  public static void logElapsed(int connectionId, long timeElapsedNanos, Category category, String prepared, String sql, String url) {
    if (logger != null && meetsThresholdRequirement(timeElapsedNanos) && isCategoryOk(category) && isLoggable(sql) ) {
      doLogElapsed(connectionId, timeElapsedNanos, category, prepared, sql, url == null ? "" : url);
    } else if (isDebugEnabled()) {
      debug("P6Spy intentionally did not log category: " + category + ", statement: " + sql + "  Reason: logger=" + logger + ", isLoggable="
          + isLoggable(sql) + ", isCategoryOk=" + isCategoryOk(category) + ", meetsTreshold=" + meetsThresholdRequirement(timeElapsedNanos));
    }
  }
  
  public static void logElapsed(int connectionId, long timeElapsedNanos, Category category, Loggable loggable) {
    // usually an expensive operation => cache where possible
    String sql = loggable.getSql();
    String url = loggable.getConnectionInformation().getUrl();
    if (logger != null && meetsThresholdRequirement(timeElapsedNanos) && isCategoryOk(category) && isLoggable(sql)) {
      doLogElapsed(connectionId, timeElapsedNanos, category, sql, loggable.getSqlWithValues(), url == null ? "" : url);
    } else if (isDebugEnabled()) {
      sql = loggable.getSqlWithValues();
      debug("P6Spy intentionally did not log category: " + category + ", statement: " + sql + "  Reason: logger=" + logger + ", isLoggable="
          + isLoggable(sql) + ", isCategoryOk=" + isCategoryOk(category) + ", meetsTreshold=" + meetsThresholdRequirement(timeElapsedNanos));
    }
  }

  //->JAW: new method that checks to see if this statement should be logged based
  //on whether on not it has taken greater than x amount of time.
  private static boolean meetsThresholdRequirement(long timeTaken) {
        final P6LogLoadableOptions opts = P6LogOptions.getActiveInstance();
        long executionThreshold = null != opts ? opts.getExecutionThreshold() : 0;
    
    return executionThreshold <= 0 || TimeUnit.NANOSECONDS.toMillis(timeTaken) > executionThreshold;
  }

  public static void info(String sql) {
    if (logger != null && isCategoryOk(Category.INFO)) {
      doLog(-1, Category.INFO, "", sql);
    }
  }

  public static boolean isDebugEnabled() {
    return isCategoryOk(Category.DEBUG);
  }

  public static void debug(String sql) {
    if (isDebugEnabled()) {
      if (logger != null) {
        doLog(-1, Category.DEBUG, "", sql);
      } else {
        System.err.println(sql);
      }
    }
  }

  public static void error(String sql) {
    System.err.println("Warning: " + sql);
    if (logger != null) {
      doLog(-1, Category.ERROR, "", sql);
    }
  }
  
  public static P6Logger getLogger() {
    return logger;
  }

}

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
