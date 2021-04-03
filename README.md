# Gradle Libraries Plugin

Forked from: github/fkorotkov/gradle-libraries-plugin

This plugin allows to specify versions of external libraries in a centralized place and use them across the project. It's specifically useful for Gradle multi-projects.

This plugin also uses [`com.github.ben-manes.versions`](https://github.com/ben-manes/gradle-versions-plugin) plugin to automatically update libraries once new releases are available.

## Usage

After applying the [plugin](https://plugins.gradle.org/plugin/com.github.fkorotkov.libraries):

```groovy
plugins {
  id("com.github.fkorotkov.libraries") version "1.1"
}
```

Run `./gradlew syncLibraries` to create `$rootDir/dependencies.json` file with all currently known dependencies.

`dependencies.json` file format is pretty straightforward. It contains an alphabetically sorted list of dependencies in the following format:

```json
{
  "libraries": [
    {
      "group": "com.google.guava",
      "name": "guava",
      "version": "22.0"
    }
  ]
}  
```

Once `dependencies.json` file is in place by generating via `syncLibraries` task or just by manually creating it, all declared in `dependencies.json` libraries can be used in project files as showed below:

```groovy
dependencies {
  compile libraries['com.google.guava:guava']
}
```

## Using aliases

There is also an option to specify an alias for a library to override the default `<group>:<name>` key. Simply add `alias` property in `depepndencies.json`:

```json
{
  "libraries": [
    {
      "group": "com.google.guava",
      "name": "guava",
      "version": "22.0",
      "alias": "google-guava"
    }
  ]
}  
```

And now it can be used like this:

```groovy
dependencies {
  compile libraries['google-guava']
}
```

## Updating Libraries Automatically

It's not a secret that it's really hard keep libraries up to date especially when there are dozens and sometimes hundreds of them. For that purpose this plugin has `updateLibraries` task that will automatically check for available new versions of libraries. This functionality is based on an awesome [`com.github.ben-manes.versions`](https://github.com/ben-manes/gradle-versions-plugin) plugin(this plugin brings `com.github.ben-manes.versions` plugin on the classpath). See `com.github.ben-manes.versions` plugin docs for more details on how to customize `resolutionStrategy` to alert you only on updates you are interested.

Here is an example of a `resolutionStrategy` that rejects versions that contain `dev` or `eap`.

```groovy
updateLibraries {
  resolutionStrategy {
    componentSelection { rules ->
      rules.all { ComponentSelection selection ->
        boolean rejected = ['dev', 'eap'].any { qualifier ->
          selection.candidate.version ==~ /(?i).*[.-]${qualifier}[.\d-]*/
        }
        if (rejected) {
          selection.reject('dev version')
        }
      }
    }
  }
}
```   
