<!--
   Copyright 2015 Benjamin Gandon

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->

JULI-dedicated jcl-over-slf4j library
=====================================

This project is a solution for using [SLF4J](http://www.slf4j.org/) and
[Logback](http://logback.qos.ch/) as [Tomcat](http://tomcat.apache.org/)
internal logging system.

This library is a re-packaged bundle of [jcl-over-slf4j](http://www.slf4j.org/legacy.html#jclOverSLF4J),
[SLF4J](http://www.slf4j.org/) and [Logback](http://logback.qos.ch/),
highly inspirated by [tomcat-slf4j-logback](https://github.com/grgrzybek/tomcat-slf4j-logback).
This was done in the context of documenting the [juli-to-slf4j](https://github.com/bgandon/juli-to-slf4j)
bridge and its [alternatives](https://github.com/bgandon/juli-to-slf4j/blob/master/ALTERNATIVES.md).

It builds an additional library that plugs into the
[tomcat-extras-juli](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.apache.tomcat.extras%22%20AND%20a%3A%22tomcat-extras-juli%22)
“extra” component to provide SLF4J logging with the Logback backend, keeping
the _tomcat-extras-juli_ library separate, and unmodified.
(Contrarily to [tomcat-slf4j-logback](https://github.com/grgrzybek/tomcat-slf4j-logback),
which retains only a portion of the default _tomcat-juli_ in order to build a
smaller library.)

This lib provides:

1. A JULI-dedicated bridge from the full JCL implementation of
   _tomcat-extras-juli_ and a package-renamed SLF4J API.

2. Package-renamed SLF4J & Logback libraries, so that they might not conflict
   with any SLF4J & Logback that web applications may ship with.

Web applications that ship with their own SLF4J & Logback libraries will
properly create separate logging contexts. This is the
[simple approach](http://logback.qos.ch/manual/loggingSeparation.html#easy).
The configuration in `conf/logback-catalina.xml` shall use the package-renamed
code, prepending `com.apache.juli.` to all Logback class names.
[JNDI](http://logback.qos.ch/manual/loggingSeparation.html#ContextJNDISelector)
cannot be setup since Logback intializes before the Tomcat JNDI system is
ready.


Caveats
-------

The `tomcat-extras-juli-over-slf4j.jar` must appear before the
`tomcat-extras-juli.jar` on the `$CLASSPATH` as detailed below. Indeed, the
default `org.apache.juli.logging.LogFactory` is not suppressed from the
original `tomcat-extras-juli.jar`. It is just masked by the one provided by
`tomcat-extras-juli-over-slf4j.jar`. Any reason for a non-guraanteed classpath
order would result in a non-working system.

Running Tomcat with a security manager requires some more setup in
`conf/catalina.policy`.


Setup
-----

1. Build the lib

```
mvn clean package
```

2. Copy the resulting `target/tomcat-extras-juli-over-slf4j-1.7.12.jar` to the
   `$CATALINA_BASE/bin/` directory.

3. Add the Bash code below to your `$CATALINA_BASE/bin/setenv.sh` script.

```bash
# Logging configuration
# Because the juli config needs to be set early at bootstrap time

# Deactivate standard JULI config (the correct way by Bugzilla #45585)
LOGGING_CONFIG="-Dnop"

# Add bridge to the class path
CLASSPATH="${CLASSPATH:+$CLASSPATH:}$CATALINA_BASE"/bin/tomcat-extras-juli-over-slf4j-1.7.12.jar

# Activate diagnostics in LogFactory of 'tomcat-extras-juli'
# Actually non are output because the original JULI LogFactory is masked
# by the one provided by 'juli-jcl-over-slf4j' above
CATALINA_OPTS="$CATALINA_OPTS -Dorg.apache.juli.logging.diagnostics.dest=STDOUT"

# Setup config file
CATALINA_OPTS="$CATALINA_OPTS -Djuli.logback.configurationFile=$CATALINA_BASE/conf/logback-catalina.xml"
```

4. Remove the default `$CATALINA_BASE/conf/logging.properties` because it's no
   use keeping it anymore.

5. Restart Tomcat.


Author and Licenses
-------------------

Copyright © 2015, Benjamin Gandon

The JULI-JCL-over-SLF4J library is released under the terms of the
[Apache 2.0 license](LICENSE.txt), excepted some specific sub-components, as
detailed in [LICENSES.md](LICENSES.md).


<!--
# Local Variables:
# indent-tabs-mode: nil
# End:
-->
