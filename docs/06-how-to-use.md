# How to Use

## Gradle

For use with Gradle see the JFrog documentation [here](https://jfrog.com/help/r/jfrog-integrations-documentation/working-with-gradle).

Below is an example:

![gradle](./assets/gradle.png)

When using `Generate Settings` ensure you select `[your team]-all-maven` in
`Plugin Resolver` and in `Libs Resolver`. In `Libs Publisher` select
`[your team]-maven` and then, hit the `Generate Settings` button.

You will have two more buttons to `Download Snippet` and `Download gradle.properties`.

Find your local gradle directory, usually under your user home dir you will find
a `.gradle` directory. If you don’t have gradle directory, create one.

Copy the `gradle.properties` file you’ve downloaded from Artifactory into the
`.gradle` directory.

In your projects `build.gradle` scripts, you may need to set:

This part needs to go on the top of the `build.gradle` script, if you already
have any of the sections below, already exist, just add the missing information.

```js
buildscript {
  repositories {
    maven {
      url 'https://directline.jfrog.io/artifactory/[your team]-all-maven'
      credentials {
        username = "${artifactory_user}"
        password = "${artifactory_password}"
      }
  }
}
dependencies {
  classpath "org.jfrog.buildinfo:build-info-extractor-gradle:4+"
  }
}
apply plugin: "com.jfrog.artifactory"
apply plugin: 'maven-publish' // this line is only needed if you need to publish to artifactory
```

This is the repositories section outside or the `buildscript` section (they may
be different in some situations, but for DLG’s setup, they should be duplicated).

```js
repositories {
 maven {
  url 'https://directline.jfrog.io/artifactory/[your team]-all-maven'
    credentials {
    username = "${artifactory_user}"
    password = "${artifactory_password}"
  }
 }
}
```

This is section below will publish your artifacts to JFrog:

```js
artifactory {
  contextUrl = "${artifactory_contextUrl}"
  publish {
    repository {
    repoKey = '[your team]-maven'
    username = "${artifactory_user}"
    password = "${artifactory_password}"
    maven = true
    }
    defaults {
      publications ('mavenJava')
    }
  }
  resolve {
    repository {
    repoKey = '[your team]-all-maven'
    username = "${artifactory_user}"
    password = "${artifactory_password}"
    maven = true
    }
  }
}
publishing {
  publications {
    mavenJava(MavenPublication) {
    from components.java
    }
  }
}
```

In places where you can’t use the `gradle.properties` file, you can pass the
properties to gradle in the following form:

```zsh
gradle -Partifactory_user=${username} -Partifactory_password=${pwd}
-Partifactory_contextUrl=https://directline.jfrog.io/artifactory clean test
build artifactoryPublish
```

Where pwd is the Encrypted Password that you can find in your Artifactory
profile.

## PyPi/pip

Select the appropriate `[your team]-pypi` repository and then, click on the
“Set Me Up” button on the top right corner.

![pypi](./assets/pypi.png)

Follow the instructions above and you will be able to publish/download python
packages.

### Installing packages with pip from JFrog

To avoid typing credentials every time (and to keep them out of history),
your auth credentials should be stored in a `~/.netrc` file (set permissions to
0600, i.e. only accessible by your own user), in the format below:

```python
machine directline.jfrog.io
  login <JFrog_username>
  password <identity_token>
```

- `<JFrog_username>` is replaced by your JFrog username (for user accounts this
  is a lower case RACF ID, for automation accounts this is however it appears in
  the JFrog platform)
- `<identity_token>` is replaced by your generated [identity token](https://dlg.service-now.com/sp?id=kb_article_view&table=kb_knowledge&sys_kb_id=c2c5e49a1ba246509963b99f1d4bcbf0&searchTerm=Jfrog)
  .

For example:

```python
machine directline.jfrog.io
  login foxg
  password 12345abcdef
```

Pip can now be executed as:

```zsh
pip install \
  --requirement requirements.txt \
  --index-url https://directline.jfrog.io/artifactory/api/pypi/darwin-pypi/simple
  --extra-index-url https://directline.jfrog.io/artifactory/api/pypi/pypi/simple
```

### Using a pip configuration file instead of a command line option

For a given project, you probably need pip to download packages from jfrog every
time. So create a pip config file for this project's python virtual environment
(venv) called `${​​​​​​​VIRTUAL_ENV}​​​​​​​​​​​​​​/pip.conf`.

```zsh
[global]
index-url = https://directline.jfrog.io/artifactory/api/pypi/darwin-pypi/simple
extra-index-url = https://directline.jfrog.io/artifactory/api/pypi/pypi/simple
```

The `[global]` here specifies which pip commands to use this section of config
for; use `[install]` if you only want to specify the index-url for pip install
to use. With this in place, pip no longer requires a specified index-url:
pip install --requirement requirements.txt

To use across all your projects, create a directory ~/.pip/ (i.e. in your home
directory), and move the pip.conf file there instead. It will now be used by
pip across all virtual environments, without having to set up a new pip.conf for
each one.

## Upload third party artifacts with Maven

You can deploy new artifacts to Artifactory using the web interface or the
[bulk upload](https://github.com/Direct-Line-Group/se-shared-scripts/blob/develop/jfrog/bulk_upload.sh)
script - instructions for the script can be found at it github. Here, we will
cover the web interface.

Once logged, navigate on the left menu to Application → Artifactory → Artifacts
and on the list of repositories, select dlg-ext-lib. Click on the deploy button
on the top-right corner.

![maven](./assets/maven.png)

Select single for a single file or multi for multiple files, and in target path
make sure to enter a path, based on the group id, artifact id and version.

eg: Suppose my dependency looks like this in maven:

```xml
<dependency>
 <groupId>com.oracle</groupId>
 <artifactId>ojdbc7</artifactId>
 <version>12.1.0.2</version>
</dependency>
```

Replace the `.` in group id with `/` and join then in this manner
`<group id separated by /s>/<artifact id>/<version>` e.g. com/oracle/ojdbc7/12.1.0.2

Hit deploy to upload the artifacts.

Some third party do not offer a `pom.xml` along with the jar files. You can use
the maven command to generate one. eg:

```zsh
mvn install:install-file -Dfile=ojdbc7-12.1.0.2.jar -DgroupId=com.oracle -DartifactId=ojdbc7 -Dpackaging=jar -Dversion=12.1.0.2
```
