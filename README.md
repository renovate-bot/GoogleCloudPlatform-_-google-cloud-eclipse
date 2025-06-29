[![stable](http://badges.github.io/stability-badges/dist/stable.svg)](http://github.com/badges/stability-badges)

**Deprecation Note**: Cloud Tools for Eclipse is only compatible with the App Engine Java 8 runtime, which will reach the [end of support](https://cloud.google.com/appengine/docs/standard/lifecycle/runtime-lifecycle#end_of_support)
on January 31, 2024. After this date, your existing Java 8 applications will continue
to run and receive traffic. However, you cannot deploy new or update existing applications
that use runtimes [after their end of support date](https://cloud.google.com/appengine/docs/standard/lifecycle/support-schedule#java).
We recommend that you [migrate to the latest supported version of Java](https://cloud.google.com/appengine/migration-center/standard/migrate-to-second-gen/java-differences).

This project provides an Eclipse plugin for building, debugging, and deploying Google Cloud Platform applications.

[End user documentation and installation instructions can be found on cloud.google.com.](https://cloud.google.com/eclipse/docs/?utm_source=github&utm_medium=google-cloud-eclipse&utm_campaign=ToolsforEclipse)

_TL;DR_: `mvn package` should
generate a p2-accessible repository in `gcp-repo/target/repository`.

# Development

This project is built using _Maven Tycho_, a set of extensions to
Maven for building Eclipse bundles and features.

## Requirements

1. The [Google Cloud SDK](https://cloud.google.com/sdk/); install this somewhere on your file system and add it to your path.

2. Install the App Engine Java components:

   `$ gcloud components install app-engine-java`

3. JDK 8

4. The [Eclipse IDE](https://www.eclipse.org/downloads/eclipse-packages/).
   It's easiest to use the _Eclipse IDE for Java EE Developers_ package. You must use
   Eclipse 4.8 - 4.21 (versions since 4.22 are currently not supported).  We use _target platforms_ to support building
   for earlier versions of Eclipse.  You also need the following:

   1. The [M2Eclipse plugin](http://www.eclipse.org/m2e/) (also called m2e) is
      required to import the projects into Eclipse.  M2Eclipse is included in
      [several packages](https://www.eclipse.org/downloads/compare.php?release=photon),
      such as the _Eclipse IDE for Java EE Developers_ package.

   2. The [m2e connector for maven-dependency-plugin](https://github.com/ianbrandt/m2e-maven-dependency-plugin)
      should be installed from `http://ianbrandt.github.io/m2e-maven-dependency-plugin/`.
      This connector should be prompted for by M2Eclipse.  If not, use
      _Preferences > Maven > Discovery > Open Catalog_ and search
      for _Dependency_ and install.

   3. The [Google Java Format plugin for Eclipse](https://github.com/google/google-java-format/).
      Download the [latest version](https://github.com/google/google-java-format/releases/)
      and place the jar into your Eclipse installation's `dropins/` directory
      (on MacOS this may be in `Eclipse.app/Contents/Eclipse/dropins/`).

5. Maven 3.5.0 or later.  Although m2eclipse is bundled with its own Maven install,
   Maven is necessary to test command-line builds.

6. git (optional: you can use EGit from within Eclipse instead)

7. Clone the project to a local directory using `git clone
   https://github.com/GoogleCloudPlatform/google-cloud-eclipse.git`.

## Running Tests

If you're using Goobuntu, first point JAVA_HOME at a stock OpenJDK, not at the Google JDK:

```sh
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

If all you want is to run tests, just do `mvn verify`.

However, the integration tests will interfere with your display, disrupt your using your computer, and likely fail if you touch the screen while the integration tests are running. A nice solution on Unix-like systems is to do `xvfb-run mvn verify`. (Install [Xvfb](https://linux.die.net/man/1/xvfb) if necessary.) `xvfb-run` will use a virtual display, and you and the tests will not be disrupted.

## Configuring Maven/Tycho Builds

The plugin is built using Maven/Tycho and targeted to Java 8.

The tests need to find the Google Cloud SDK.  You can either:

  1. Place the _SDK_`/bin` directory on your `PATH`
  2. Set `GOOGLE_CLOUD_SDK_HOME` to point to your SDK

### Changing the Eclipse Platform compilation and testing target

By default, the build is targeted against Eclipse `2018-09` (4.9).
You can explicitly set the `eclipse.target` property to `2018-12` (4.10).
```
$ mvn -Declipse.target=2018-12 package
```

### Adding a new bundle/fragment

We normally put production code into a bundle and tests as a fragment hosted
by that bundle, put under the `plugins/` directory.
For now we have been committing both the `pom.xml` and Eclipse's
`.project`, `.classpath`, and `.settings/` files.  We have a master set 
of project settings in [`eclipse/settings`](eclipse/settings/); see the
[`README.md`](eclipse/settings/README.md) for more details.

Our CI process is configured to run our tests with JaCoCo, which requires
some additional configuration to add new bundles and fragments
in `build/jacoco/`.


## Import into Eclipse

We pull in some dependencies directly from Maven-style repositories,
such as Maven Central and the Sonatype staging repository, which isn't
directly supported within Eclipse.  We have a few hoops to jump
through to set up a working development environment.

### Assemble the IDE Target Platform

The Eclipse IDE and Tycho both use a _Target Platform_ to manage
the dependencies for the source bundles and features under development.
Although Tycho can pull dependencies directly from Maven-style
repositories (like [Maven Central](https://search.maven.org)), Eclipse
cannot.  So we use Tycho to cobble together
a target platform suitable for the Eclipse IDE with the following command.
```
$ (cd eclipse; mvn -Pide-target-platform package)        # may want -Declipse.target=XXX
```
This command creates a local copy of the
target platform, including any Maven dependencies, into
[`eclipse/ide-target-platform/target/repository`](eclipse/ide-target-platform/target/repository).
You will use this repository to create a target platform within the IDE,
as described below.

The Eclipse version used for the target platform is affected by the
`eclipse.target` property, described below.

You must regenerate the target platform and reconfigure the IDE's
target platform whenever dependencies are updated.

### Steps to import into the Eclipse IDE


1. Setup JDK 8 in Eclipse (this may already be set up by Eclipse's JRE/JDK auto-discovery)

   1. Select `Window/Preferences` (on Mac `Eclipse/Preferences`).

   2. Under `Java/Installed JREs` click `Add`.

   3. Select Standard VM and click `Next`.

   4. Select the folder that contains the JDK 8 installation by clicking
      `Directory`.

   5. Click `Finish`.

   6. Select `Java/Installed JREs/Execution Environments` page.

   7. Click on `JavaSE-1.8` in the list on the left under `Execution
      Environments:`.

   8. The JDK just added should show up in the list on the right along with other
      installed JDKs/JREs. Set the checkbox next to the JDK 8 added in the
      previous steps to mark it as compatible with the `JavaSE-1.8` execution
      environment.

   9. Click `OK`.

2. Set up the Target Platform: you will need to repeat this process whenever
   items are changed in the target platform, such as a new release of the
   `appengine-plugins-core`.

   1. As described above, you must first build the target platform with Maven:

      `$ (cd eclipse; mvn -Pide-target-platform package)`

   2. Open the `Preferences` dialog, go to `Plug-in Development` > `Target Platform`.

   3. Click `Add...` > `Nothing` to create a new Target Platform.

   4. Name it `GCP IDE Target Platform`.

   5. Select `Add` > `Software Site`.

   6. Select the `Add...` button (found beside the `Work with:` field) and then select `Local`
      to find a local repository. Navigate to `.../eclipse/ide-target-platform/target/repository`,
      and click `OK`.

   7. Once the main content populates, check the `Uncategorized` item to pull in all items. Click `Finish`.

   8. Click `Finish` to complete the new target platform definition.

   9. Select your new target platform (instead of Running Platform) in the `Target Platform` preferences.

   10. Click `Apply and Finish` to load this new target platform.

   11. Eclipse will load the target.

3. Import the projects.

   1. Select `File/Import...` menu in Eclipse.

   2. Select `Existing Maven Projects` from the list.

   3. Click `Browse...` and select the directory that
      contains the project.

   4. Under `Projects:` the `pom.xml` files representing modules should be
      displayed. Make sure that all of them are selected _except_ 
      `verify_feature_completeness` and the sub-directories under 
      `eclipse`. You probably need to explicitly unselect these
      projects. Click `Finish`.

      - The subprojects under the `eclipse` directory define target platforms for the Tycho build. It's easier to edit the files from the `eclipse-setup` project.

   5. Maven may prompt you to install several additional plugin connector plugins from
      [Tycho](https://eclipse.org/tycho/) if they are not already installed. Click
      `Finish` to install them. If Eclipse prompts you to install any other
      plugins, do so.

   6. Restart Eclipse when prompted.

4. Check the imported projects.

   - There should be no errors in the `Markers` or `Problems` views in Eclipse.
      However you may see several low-priority warnings.

      - You may see Maven-related errors like _"plugin execution not
        covered by lifecycle configuration"_.
        If so, right-click on the problem and select
        _Quick Fix_ > _Discover new m2e connectors_
	and follow the process to install the recommended plugin
	connectors.

5. Create and initialize a launch configuration:

   1. Right-click the `gcloud-eclipse-tools.launch` file under the
      `google-cloud-eclipse` module in the `Package Explorer`.

   2. Select `Run As` > `Run Configurations...`

   3. Set variables required for launch:

      1. Go to the second tab for `Arguments`

      2. Click the `Variables...` button for `VM argument:`

      3. Click the `Edit variables...` button

      4. Click `New...`

      5. Set the name to `oauth_id`, and the value to the value you want to use
        (description optional)

      6. Click `OK`, the variable will appear in the list

      7. Repeat steps 6-8 but use `oauth_secret` as the name and use the
         corresponding value

      8. Click `OK` to close the edit variables dialog

      9. Click `Cancel` to close the variable selection dialog

      10. Click `Apply` to apply the changes to the run config

   4. From the `Run` menu, select `Run History > gcloud-eclipse-tools`. A new instance of Eclipse launches with the plugin installed.


# Updating Target Platforms

### Updating the `.target` files

We use _Target Platform_ files (`.target`) to collect the dependencies used
for the build.  These targets specify exact versions of the bundles and
features being built against. We currently maintain three target platforms,
targeting the latest version of the current, previous, and next releases.
This is currently:

  - Eclipse 2018-09 (4.9): [`eclipse/eclipse-2018-09/gcp-eclipse-2018-09.target`](eclipse/eclipse-2018-09/gcp-eclipse-2018-09.target)
  - Eclipse 2018-12 (4.10): [`eclipse/eclipse-2018-12/gcp-eclipse-2018-12.target`](eclipse/eclipse-2018-12/gcp-eclipse-2018-12.target)

These `.target` files are generated and *should not be manually updated*.
Updating `.target` files directly becomes a chore once it has more than a
couple of dependencies.  We instead generate these `.target`s from
_Target Platform Definition_ `.tpd` files.
The `.tpd` files use a simple DSL to specify the bundles and features,
and the location of the repositories containing them.
The `.tpd` files are processed using the [TPD Editor](https://github.com/eclipse-cbi/targetplatform-dsl)
which resolves the specified dependencies and creates a `.target`.
The process is:

   1. Install the TPD Editor, if necessary
      - Use _Help > Install New Software_ and specify `https://download.eclipse.org/cbi/tpd/3.0.0-SNAPSHOT/`
        as the location.
      - Restart Eclipse when prompted
   2. Open the `.tpd` file in Eclipse.
   3. Make any necessary changes and save.
      - Note that the TPDs specify artifacts using their _p2 identifiers_.
        Bundles are specified using their OSGi Bundle Symbolic Name (e.g.,
        `org.eclipse.core.runtime`).
        Features are specified using their Feature ID suffixed with `.feature.group`
        (e.g., `org.eclipse.rcp.feature.group`).
   4. Select the file in the Package Explorer, right-click, and choose _Create Target Definition File_
      to update the corresponding .target file.

Both the `.tpd` and `.target` files should be committed.

### Updating Dependencies

The IDE Target Platform needs to be rebuilt at the command line
and reimported into Eclipse when dependency versions are changed:

1. `(cd eclipse; mvn -Pide-target-platform package)`
2. Preferences > Plug-in Development > Target Platforms
3. Select your target ("GCP IDE Target Platform") and click Edit
4. Select the location and click Reload to cause any cached info to be discarded.
5. Click Edit and then select Uncategorized.
6. Finish / OK until done.

### Updating the Eclipse IDE Target Platforms

The IDE Target Platform, defined in `eclipse/ide-target-platform`,
may need to be updated when dependencies are added or removed.  The
contents are defined in the `category.xml` file, which specifies
the list of features and bundles that should be included.  This
file can be edited using the Category editor in Eclipse.  Ideally
the version should be specified as `"0.0.0"` to indicate that the
current version found should be used.  Unlike the `.tpd` file,
the identifiers are not p2 identifiers, and so features do not
require the `.feature.group` suffix.


## Other Miscellaneous Dev Tasks

### Updating IDE settings

See [`eclipse/settings/`](eclipse/settings/README.md) for details.

### Configuring Maven/Tycho Toolchains for new JDK releases

Now that OpenJDK is moving to a 6-month release cycle...

We use Tycho's support for Maven Toolchains to ensure that new
language features do not creep into the code.  Tycho's support is
automatically enabled in the build when compiling with a newer JDK
than our minimium supported platform.  When using such a JDK, currently
anything later than JDK 8, we configure the Tycho compiler plugin to
use the [`useJDK=BREE`](https://eclipse.org/tycho/sitedocs/tycho-compiler-plugin/compile-mojo.html)
setting to ensure bundles are *compiled* with a JDK that matches
the bundle's `Bundle-RequiredExecutionEnvironment`.  However we leave
`tycho-surefire` to run the tests using the current toolchain
(the default for
[`useJDK=SYSTEM`](https://eclipse.org/tycho/sitedocs/tycho-surefire/tycho-surefire-plugin/test-mojo.html#useJDK))
so as to catch any non-backwards compatible changes.

These settings require configuring
[Maven's toolchains](https://maven.apache.org/guides/mini/guide-using-toolchains.html)
to point to appropriate JRE installations.  Tycho further requires
that a toolchain defines an `id` for the specified _Execution
Environment_ identifier.  For example, a `~/.m2/toolchains.xml` to
configure Maven on macOS for 8, and 11 toolchains might be:

```xml
<?xml version="1.0" encoding="UTF8"?>
<toolchains>
  <toolchain>
    <type>jdk</type>
    <provides>
      <id>JavaSE-11</id>
      <version>11</version>
      <vendor>openjdk</vendor>
    </provides>
    <configuration>
      <jdkHome>/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home</jdkHome>
    </configuration>
  </toolchain>
  <toolchain>
    <type>jdk</type>
    <provides>
      <id>JavaSE-1.8</id>
      <version>1.8</version>
      <vendor>oracle</vendor>
    </provides>
    <configuration>
      <jdkHome>/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre</jdkHome>
    </configuration>
  </toolchain>
</toolchains>
```

Note that _jdkHome_ for `JavaSE-1.8` specifies the
`jre/` directory: Tycho sets the default boot classpath to
_jdkHome_`/lib/*`, _jdkHome_`/lib/ext/*`, and _jdkHome_`/lib/endorsed/*`.
For many JDKs, including Oracle's JDK and the OpenJDK *prior to Java 9*, those
directories are actually found in the `jre/` directory.  Compilation
errors such as `java.lang.String` not found and `java.lang.Exception`
not found indicate a misconfigured _jdkHome_.  With the introduction of
_Java modules_ with Java 9, there is no longer a separate JRE distribution.
