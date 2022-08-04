# 常用Shell命令

## Gradle

- 查看Gradle依赖树(debugApk、debugCompile、releaseApk、releaseCompile、compile)

```groovy
./gradlew :app:dependencies
./gradlew :app:dependencies --configuration compile
chmod +x gradlew
```

- 查看项目根目录下所有Tasks

```groovy
./gradlew tasks
```

## SSH

```shell
ssh-add id_rsa
```