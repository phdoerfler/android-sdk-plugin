# Android SDK Plugin for SBT #

Current version is 0.2.12

## Description ##

This is a simple plugin for existing and newly created android projects.
It is tested and developed against sbt 0.11.2; I have not verified whether
other versions work.

The plugin supports normal android projects and projects that reference
library projects. 3rd party libraries can be included by placing them in
`libs` as in regular projects, or they can be added by using sbt's
`libraryDependencies` feature.

Features not support from the regular android build yet are compiling `NDK`
code. Although `NDK` libraries will be picked up from `libs` as in typical
ant builds.

### Differences from jberkel/android-plugin ###

Why create a new plugin for building android applications?  Because
`jberkel/android-plugin` seems to be pretty difficult to use, and enforces
an sbt-style project layout. Ease of configuration is a primary objective
of this plugin; configuring a project to use this plugin is ~4 lines of
configuration plus a standard android project layout, jberkel's requires
the installation of `conscript`, `g8` and cloning a template project which
has lots of autogenerated configuration. This is incompatible with the
built-in SDK configuration and doesn't load up into Eclipse easily either.

* android-sdk-plugin includes rudimentary device management support,
  jberkel's implementation may be more fully featured.
  * The commands `devices` and `device` are implemented. The former lists
    all connected devices. The latter command is for selecting a target
    device if there is more than one device. If there is more than one
    device, and no target is selected, all commands will execute against the
    first device in the list.
  * `android:install` and `android:run` are tasks that can be used to install
    and run the built apk on device respectively.
* This plugin uses the standard Android project layout as created by
  Eclipse and `android create project`. Additionally, it reads all the
  existing configuration out of the project's `.properties` files.
* `TR` for typed resources improves upon `TR` in android-plugin. It should be
  compatible with existing applications that use `TR` while also adding a
  type to `TypedLayout[A]`. An implicit conversion on `LayoutInflater` to
  `TypedLayoutInflater` allows calling
  `inflater.inflate(TR.layout.foo, container, optionalBoolean)` and receiving
  a properly typed resulting view object.
  * Import `TypedResource._` to get the implicit conversions
* No apklib support, someone enlighten me, why do I want this and/or how do
  I support it?

## Usage ##

1. Install sbt (https://github.com/harrah/xsbt)
2. Create a new android project using `android create project` or Eclipse
   * Instead of creating a new project, one can also do
     `android update project` to make sure everything is up-to-date.
3. Create a directory named `project` within your project and name it
   `plugins.sbt`, in it, add the following lines:

    ```
    resolvers += Resolver.url("scala-sbt releases", new URL(
      "http://scalasbt.artifactoryonline.com/scalasbt/sbt-plugin-releases/"))(
      Resolver.ivyStylePatterns)

    addSbtPlugin("com.hanhuy.sbt" % "android-sdk-plugin" % "0.2.12")
    ```

4. Create a file named `build.sbt` in the root of your project and add the
   following lines with a blank line between each:
   * `name := YOUR-PROJECT-NAME` (optional, but you'll get a stupid default
     if you don't set it)
   * `seq(androidBuildSettings: _*)`
5. Now you will be able to run SBT, some available commands in sbt are:
   * `compile`
     * Compiles all the sources in the project, java and scala
     * Compile output is automatically processed through proguard if there
       are any Scala sources, otherwise; it can be enabled manually.
   * `android:package-release`
      * Builds a release APK and signs it with a release key if configured
   * `android:package-debug`
      * Builds a debug APK and signs it using the debug key
   * `android:package`
     * Builds an APK for the project of the last type selected, by default
       `debug`
   * Any task can be repeated continuously whenever any source code changes
     by prefixing the command with a `~`. `~ android:package-debug`
     will continuously build a debug build any time one of the project's
     source files is modified.
6. If you want android-sdk-plugin to automatically sign release packages
   add the following lines to `local.properties` (or any file.properties of
   your choice that you do not check in to source control):
   * `key.alias: YOUR-KEY-ALIAS`
   * `key.store: /path/to/your/.keystore`
   * `key.store.password: YOUR-KEY-PASSWORD`
   * `key.store.type: pkcs12` (optional, defaults to `jks`)

## Advanced Usage ##

* Multi-project builds
  * See https://gist.github.com/1897936 for an example of how to setup the
    root project
  * The example config makes `package` in the app project depend on `package`
    in the library project.  `package` in the root project depends on
    `android:package` in the app project. So if `package` is called from the
    root, or if `android:package` is called from `appproject`, then the
    right thing happens
  * Another example of a multi-project build using an android project and
    common code that is not meant to target only android is available at
    https://gist.github.com/2296619
    * This example does the same as above, except allows loading an arbitrary
      Java or Scala project into the Android app.
* Configuring `android-sdk-plugin` by editing build.sbt
  * `import AndroidKeys._` at the top to make sure you can use the plugin's
    configuration options
  * Add configuration options according to the sbt style:
    * `useProguard in Android := true` to enable proguard
  * Configurable keys can be discovered by typing `android:<tab>` at the
    sbt shell
* Configuring proguard, some options are available
  * `proguardOptions in Android += Seq("-dontobfuscate", "-dontoptimize")` -
    will tell proguard not to obfuscute nor optimize code (any valid proguard
    option is usable here)
 * `proguardConfig in Android ...` can be used to replace the entire
   proguard config included with android-sdk-plugin
* I have found that Scala applications on android build faster if they're
  using scala 2.8.2. Set the scala version in `build.sbt` by entering
  `scalaVersion := "2.8.2"`
* Unit testing with robolectric, see my build.sbt for this configuration:
  * https://gist.github.com/2503441
  * To get rid of robolectric's warnings about not finding certain classes
    to shadow, change the project target to include google APIs
  * jberkel has written a Suite trait to be able to use robolectric with
    scalatest rather than junit, see https://gist.github.com/2662806

### TODO / Known Issues ###

* Implement the NDK build process
* Changes to `AndroidManifest.xml` will require the plugin to be reloaded.
  The manifest data is stored internally as read-only data and does not
  reload automatically when it is changed. The current workaround is to
  type `reload` manually everytime `AndroidManifest.xml` is updated.
* sbt `0.12` breaks compatibility with this plugin. Don't have a solution yet,
  the current workaround is to change `javacOptions <<= (....) { .... }` to
  `javacOptions <<= (....) map { .... }` and use a locally published plugin.

#### Thanks to ####

* pfurla, jberkel, mharrah, retronym and the other folks from #sbt and #scala
  for bearing with my questions and helping me learn sbt.
