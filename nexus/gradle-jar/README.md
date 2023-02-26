# Gradle JAR

## Init

```sh
gradle init --type=java-application --test-framework=junit-jupiter --dsl=groovy
```

## Publish to Nexus Repository

- [build.gradle](app/build.gradle)

```groovy
plugins {
    id 'maven-publish'
}

jar {
    archiveFileName = "app.jar"
    manifest {
        // Main 클래스를 지정하지 않으면 아래와 같은 에러가 발생한다.
        // no main manifest attribute, in app.jar
        attributes 'Main-Class': 'maven.pom.App'
    }
}

publishing {
    publications {
        markruler(MavenPublication) {
            artifact jar // jar (task): org.gradle.api.tasks.bundling.Jar
            LocalDateTime now = LocalDateTime.now()
            DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyMMddHHmmss")
            version = now.format(pattern)
        }
    }
    repositories {
        maven {
            name = "local-nexus"
            url = uri("http://localhost:8081/repository/maven-releases/")
            allowInsecureProtocol = true
            credentials {
                username = System.getenv("NEXUS_USERNAME") ?: "admin"
                password = System.getenv("NEXUS_PASSWORD") ?: "admin"
            }
        }
    }
}
```

```sh
# JAVA_HOME=$HOME/.sdkman/candidates/java/17.0.6-zulu \
./gradlew publish
```

Nexus에 배포된 JAR를 다운로드한다.

```shell
curl -v -u admin:admin -L http://localhost:8081/repository/maven-releases/com%2Fmarkruler%2Fapp%2F230226195126%2Fapp-230226195126.jar -o app.jar
```

실행하면 아래와 같이 Hello World!가 출력된다.

```shell
java -jar app.jar
# Hello World!
```
