# jgitver-maven-plugin [![Build Status](https://travis-ci.org/jgitver/jgitver-maven-plugin.svg?branch=master)](https://travis-ci.org/jgitver/jgitver-maven-plugin)

> *__IntelliJ IDEA users__*: due to [IDEA-155733](https://youtrack.jetbrains.com/issue/IDEA-155733) update your IDE to version >= `2016.1.3` and use jgtiver-maven-plugin >= `0.1.1`  
>  
> If you cannot update due to old perpetual licenses in use for example, as a workaround, you can move the plugin into a profile as described [here](#maven-dependency-management-is-broken-under-intellij-idea).

This plugin allows to define the pom version of your project using the information from your git history. 
It calculates the version, a little bit like `git describe` would do but in a more efficient way for maven projects:

- new commits have upper version than previous commit (in the way maven/semver interpret versions)
- version calculation is based on git tags
- git lightweight tags allow for intermediate version controlling between releases
    - allow to define what is the _next_ version pattern to use
- minimal setup via maven extension

Here is an illustration of the capabilities of the plugin

![Example](src/doc/images/jgitver-maven-like.gif?raw=true "Example")

## Usage

### pure extension

Using the module as pure maven extension, allows a minimal setup inside your pom.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    ...
    <version>0</version>    <!-- your project version becomes irrelevant -->
    ...
    <build>
        <extensions>
            <extension>
                <groupId>fr.brouillard.oss</groupId>
                <artifactId>jgitver-maven-plugin</artifactId>
                <version>X.Y.Z</version>
            </extension>
        </extensions>
    </build>
</project>
```

Used like that _i.e._ as a pure extension, [jgitver](https://github.com/jgitver/jgitver) will be used with the following parameters:

- __mavenLike__: `true` 
- __nonQualifierBranches__: `master` 

### plugin extension with configuration

The plugin can also be defined as a plugin extension so that you have the possibility to define your own configuration, and thus bypass the default one:

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    ...
    <version>0</version>    <!-- your project version becomes irrelevant -->
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>fr.brouillard.oss</groupId>
                <artifactId>jgitver-maven-plugin</artifactId>
                <version>X.Y.Z</version>
                <extensions>true</extensions>
                <configuration>
                    <mavenLike>true/false</mavenLike>
                    <autoIncrementPatch>true/false</autoIncrementPatch>
                    <useCommitDistance>true/false</useCommitDistance>
                    <useGitCommitId>true/false</useGitCommitId>
                    <gitCommitIdLength>integer</gitCommitIdLength>  <!-- between [8,40] -->
                    <nonQualifierBranches>master</nonQualifierBranches> <!-- comma separated, example "master,integration" -->
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Please consult [jgitver](https://github.com/jgitver/jgitver#configuration-modes--strategies) documentation to fully understand what the parameters do.

## Example

If you want to give it a try you can use the following script that will setup a demo project. 

Then play around with it doing:

- `mvn validate`
- `mvn install`
- `git checkout XXXX`

```
cd /d
rm -rf /d/demo-jgitver-maven-plugin
mkdir -p /d/demo-jgitver-maven-plugin
cd /d/demo-jgitver-maven-plugin
git init
cat > pom.xml << EOF
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>fr.brouillard.oss.demo</groupId>
    <artifactId>demo-jgitver-maven-plugin</artifactId>
    <version>0</version>
    <packaging>pom</packaging>
    <build>
        <extensions>
            <extension>
                <groupId>fr.brouillard.oss</groupId>
                <artifactId>jgitver-maven-plugin</artifactId>
                <version>[0.0.3,)</version>
            </extension>
        </extensions>
    </build>
</project>
EOF
echo A > content
git add pom.xml
git add content
git commit -m "initial commit"
echo B > content && git add -u && git commit -m "added B data"
git tag 1.0 -m "release 1.0"
echo C > content && git add -u && git commit -m "added C data"
git checkout -b cool-feature
echo D > content && git add -u && git commit -m "added D data"
git checkout master
echo E > content && git add -u && git commit -m "added E data"
mvn validate
```

## Build & release

### Normal build

- `mvn -Prun-its clean install`

### Release

- `mvn -Poss clean install`: this will simulate a full build for oss delivery (javadoc, source attachement, GPG signature, ...)
- `git tag -a -s -m "release X.Y.Z, additionnal reason" X.Y.Z`: tag the current HEAD with the given tag name. The tag is signed by the author of the release. Adapt with gpg key of maintainer.
    - Matthieu Brouillard command:  `git tag -a -s -u 2AB5F258 -m "release X.Y.Z, additionnal reason" X.Y.Z`
    - Matthieu Brouillard [public key](https://sks-keyservers.net/pks/lookup?op=get&search=0x8139E8632AB5F258)
- `mvn -Poss,release -DskipTests deploy`
- `git push --follow-tags origin master`

> DISCLAIMER  
> This plugin has been highly inspired by the work of [Brian Demers](https://github.com/bdemers) in his [maven-external-version](https://github.com/bdemers/maven-external-version/) plugin.  
> I rewrote such a plugin mainly to simplify usage compared to a [maven-external-version](https://github.com/bdemers/maven-external-version/) extension (which I wrote also as [maven-external-version-jgitver](https://github.com/jgitver/maven-external-version-jgitver)).  
> Such a simplification leads to:
> - usage as pure extension without configuration 
> - benefit from a direct configuration on the plugin allowing for example IDE completion & -D property usage 

## Known issues

### Maven dependency management is broken under Intellij IDEA

due to [IDEA-155733](https://youtrack.jetbrains.com/issue/IDEA-155733), Intellij IDEA versions lower than `2016.1.3` _(this includes of course all 14.X & 15.X versions)_ do not handle correctly the plugin. This results in having the IDEA-maven integration being broken:
- no update of dependencies
- other issues, ...
 
If you can, upgrade to Intellij IDEA >= `2016.1.3`, [IDEA-155733](https://youtrack.jetbrains.com/issue/IDEA-155733) has been resolved in this version.

If you cannot upgrade, the solution is to disable the plugin usage in Intellij by moving it inside a maven profile. Then you have the choice to either deactivate the profile in Intellij or have the profile deactivated by default (thus not working in Intellij) and manually activating it on your CI/build system.

Here is an example configuration that deactivates the plugin:
```
...
<profiles>
    ...
    <profile>
        <id>jgitver</id>
        <activation>
            <activeByDefault>false</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                    <groupId>fr.brouillard.oss</groupId>
                    <artifactId>jgitver-maven-plugin</artifactId>
                    <version>0.1.1</version>
                    <extensions>true</extensions>
                </plugin>
            </plugins>
        </build>
    </profile>
    ...
</profiles>
```

Then on your build system, you would just have to activate the profile to have the magic happen again:  
`mvn install -Pjgitver`
