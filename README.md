# NogyangSpigot-Fork-Example
How to make NogyangSpigot fork with Paperweight-Patcher
# Submodule way:
View a code right up Like Paperweight-examples Submodule Branch:
1. add submodules.
2. setup Your fork.
```
import io.papermc.paperweight.util.Git
import io.papermc.paperweight.util.constants.PAPERCLIP_CONFIG

plugins {
    java
    `maven-publish`

    // Nothing special about this, just keep it up to date
    id("com.github.johnrengelman.shadow") version "8.1.1" apply false

    // In general, keep this version in sync with upstream. Sometimes a newer version than upstream might work, but an older version is extremely likely to break.
    id("io.papermc.paperweight.patcher") version "1.7.1"
}

val paperMavenPublicUrl = "https://repo.papermc.io/repository/maven-public/"

repositories {
    mavenCentral()
    maven(paperMavenPublicUrl) {
        content { onlyForConfigurations(configurations.paperclip.name) }
    }
}

dependencies {
    remapper("net.fabricmc:tiny-remapper:0.8.10:fat") // Must be kept in sync with upstream
    decompiler("net.minecraftforge:forgeflower:2.0.627.2") // Must be kept in sync with upstream
    paperclip("io.papermc:paperclip:3.0.3") // You probably want this to be kept in sync with upstream
}

allprojects {
    apply(plugin = "java")
    apply(plugin = "maven-publish")

    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(21)
        }
    }
}

subprojects {
    tasks.withType<JavaCompile> {
        options.encoding = Charsets.UTF_8.name()
        options.release = 17
    }
    tasks.withType<Javadoc> {
        options.encoding = Charsets.UTF_8.name()
    }
    tasks.withType<ProcessResources> {
        filteringCharset = Charsets.UTF_8.name()
    }

    repositories {
        mavenCentral()
        maven(paperMavenPublicUrl)
    }
}

val paperDir = layout.projectDirectory.dir("work/NogyangSpigot")
val initSubmodules by tasks.registering {
    outputs.upToDateWhen { false }
    doLast {
        Git(layout.projectDirectory)("submodule", "update", "--init").executeOut()
    }
}

paperweight {
    serverProject = project(":NogyangSpigot-server")

    remapRepo = paperMavenPublicUrl
    decompileRepo = paperMavenPublicUrl

    upstreams {
        register("paper") {
            upstreamDataTask {
                dependsOn(initSubmodules)
                projectDir = paperDir
            }


            patchTasks {
                register("api") {
                    upstreamDir = paperDir.dir("NogyangSpigot-API")
                    patchDir = layout.projectDirectory.dir("patches/api")
                    outputDir = layout.projectDirectory.dir("YourFork-api")
                }

                register("MojangApi") {
                    isBareDirectory = true
                    upstreamDir = paperDir.dir("Paper-MojangAPI")
                    patchDir = layout.projectDirectory.dir("patches/mojangApi")
                    outputDir = layout.projectDirectory.dir("NogyangSpigot-MojangAPI")
                }

                register("generatedApi") {
                    isBareDirectory = true
                    upstreamDir = paperDir.dir("paper-api-generator/generated")
                    patchDir = layout.projectDirectory.dir("patches/generatedApi")
                    outputDir = layout.projectDirectory.dir("paper-api-generator/generated")
                }

                register("server") {
                    upstreamDir.set(paperDir.dir("Paper-Server"))
                    patchDir.set(layout.projectDirectory.dir("patches/server"))
                    outputDir.set(layout.projectDirectory.dir("NogyangSpigot-server"))
                    importMcDev.set(true)
                }

                }
            }
        }
    }

//
// Everything below here is optional if you don't care about publishing API or dev bundles to your repository
//

tasks.generateDevelopmentBundle {
    apiCoordinates = "kr.ms.nogyang.nogyangspigot:NogyangSpigot-api"
    mojangApiCoordinates = "kr.ms.nogyang.nogyangspigot:NogyangSpigot-mojangapi"
    libraryRepositories = listOf(
        "https://repo.maven.apache.org/maven2/",
        paperMavenPublicUrl,
        // "https://my.repo/", // This should be a repo hosting your API (in this example, 'com.example.paperfork:forktest-api')
    )
}

allprojects {
    // Publishing API:
    // ./gradlew :ForkTest-API:publish[ToMavenLocal]
    publishing {
        repositories {
            maven {
                name = "myRepoSnapshots"
                url = uri("https://my.repo/")
                // See Gradle docs for how to provide credentials to PasswordCredentials
                // https://docs.gradle.org/current/samples/sample_publishing_credentials.html
                credentials(PasswordCredentials::class)
            }
        }
    }
}

publishing {
    // Publishing dev bundle:
    // ./gradlew publishDevBundlePublicationTo(MavenLocal|MyRepoSnapshotsRepository) -PpublishDevBundle
    if (project.hasProperty("publishDevBundle")) {
        publications.create<MavenPublication>("devBundle") {
            artifact(tasks.generateDevelopmentBundle) {
                artifactId = "dev-bundle"
            }
        }
    }
```
3. add .gitmodules
```
[submodule "work/Paper"]
	path = work/NogyangSpigot
	url = https://github.com/GAME-CLI-SRV-DEV/NogyangSpigot
        branch = master
```
4. setup editorconfig, gitignore
```
.editorconfig
[*.java]
charset=utf-8
end_of_line=lf
insert_final_newline=true
indent_style=space
indent_size=4
.gitignore
.gradle/
build/

# Eclipse stuff
.classpath
.project
.settings/

# VSCode stuff
.vscode/

# netbeans
nbproject/
nbactions.xml

# we use maven!
build.xml

# maven
target/
dependency-reduced-pom.xml

# vim
.*.sw[a-p]

# various other potential build files
build/
bin/
dist/
manifest.mf

# Mac filesystem dust
.DS_Store/
.DS_Store

# intellij
*.iml
*.ipr
*.iws
.idea/
out/

# Linux temp files
*~

# other stuff
run/


NogyangSpigot-server
NogyangSpigot-api
NogyangSpigot-mojangapi
paper-api-generator

!gradle/wrapper/gradle-wrapper.jar
.gitattributes
* text=auto

*.sh text eol=lf
gradlew text eol=lf
*.bat text eol=crlf

*.jar binary
```
5. make a neccesary patches(Build-Changes) & Initialize the fork with gradle wrapper
6. type gradlew applypatches
7. type Gradlew CreatereobfPaperclipjar, this will build it.
