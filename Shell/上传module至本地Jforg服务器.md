# 上传module至本地Jforg服务器

-  Android Studio 只支持上传library到JFrog上 
-  上传JFrog步骤
  1.配置项目Gradle文件 
  2.配置library的Gradle文件  

```xml
    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:2.2.3'
            classpath "org.jfrog.buildinfo:build-info-extractor-gradle:3.1.1" //上传插件

            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }
    apply plugin: 'com.jfrog.artifactory'
    apply plugin: 'maven-publish'

    def packageName = 'com.jeroenmols.awesomelibrary'
    def libraryVersion = '1.0.0'

    publishing {
        publications {
            aar(MavenPublication) {
                groupId packageName
                version = libraryVersion
                artifactId project.getName()

                // Tell maven to prepare the generated "* .aar" file for publishing
                artifact("$buildDir/outputs/aar/${project.getName()}-release.aar")
            }
        }
    }

    /**
* 配置Jfrog Artifactory
*/
artifactory {
   contextUrl = 'http://localhost:8081/artifactory'
   publish {
       repository {
           // The Artifactory repository key to publish to
           repoKey = 'libs-release-local'

           username = "admin"
           password = "password"
       }
       defaults {
           // Tell the Artifactory Plugin which artifacts should be published to Artifactory.
           publications('aar')
           publishArtifacts = true

           // Properties to be attached to the published artifacts.
           properties = ['qa.level': 'basic', 'dev.team': 'core']
           // Publish generated POM files to Artifactory (true by default)
           publishPom = true
       }
   }
}
```

-  执行命令上传
  `gradlew assembleRelease artifactoryPublish` 