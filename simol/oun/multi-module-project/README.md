# 🔴 oun 프로젝트 셋업

## 🟠 mutli module project setup

### 🟢 프로젝트 생성

spring initializr 에서 프로젝트를 생성한다.

### 🟢 multi module project 생성

tool에 맞춰서 spring project를 추가로 생성해주면 된다. spring initializr 에서 프로젝트를 생성할 수도 있고 intellij의 경우는 module로 추가할 수도 있다. vscode나 cursor의 경우 프로젝트 루트 경로에서 java project를 추가해줄 수 있다.

## 🟠 프로젝트 생성 후 작업

프로젝트를 생성한 이후 해야할 작업이 있다.

1. 루트 프로젝트 settings.gradle 수정
1. 루트 프로젝트 build.gradle 수정
1. 루트 프로젝트 src 제거 (필수는 아니지만 헷갈리니까 제거하면 좋다.)
1. 추가한 프로젝트 build.gradle 수정
1. 추가한 프로젝트 사용하지 않는 파일들 제거

### 🟢 루트 프로젝트 settings.gradle 수정

```groovy
pluginManagement {
	repositories {
		maven { url 'https://repo.spring.io/milestone' }
		gradlePluginPortal()
	}
}
rootProject.name = 'oun'
```

기존 설정에서

```groovy
pluginManagement {
	repositories {
		maven { url 'https://repo.spring.io/milestone' }
		gradlePluginPortal()
	}
}
rootProject.name = 'oun'


include 'oun-api'
include 'oun-common'
include 'oun-user'
```

아래와 같이 include만 추가해주면 된다.

### 🟢 루트 프로젝트 build.gradle 수정

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.5.0-M1'
	id 'io.spring.dependency-management' version '1.1.7'
}

bootJar {
    enabled = false
}

jar {
    enabled = true
}

tasks.named('test') {
	useJUnitPlatform()
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

allprojects {
	repositories {
		mavenCentral()
		maven { url 'https://repo.spring.io/milestone' }
	}
}

subprojects {

	group = 'com.simol'
	version = '0.0.1-SNAPSHOT'

    tasks.withType(org.springframework.boot.gradle.tasks.bundling.BootJar) {
        enabled = false
    }

	java {
		toolchain {
			languageVersion = JavaLanguageVersion.of(21)
		}
	}

    apply plugin: 'java-library'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

	dependencies {
		compileOnly 'org.projectlombok:lombok'
		annotationProcessor 'org.projectlombok:lombok'
		implementation 'org.springframework.boot:spring-boot-starter'
		testImplementation 'org.springframework.boot:spring-boot-starter-test'
		testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
		
		// swagger
	    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.5.0'
	}
}

project(':oun-api') {
	dependencies {
		implementation project(':oun-common')
        implementation project(':oun-user')
	}
}
```

### 🟢 루트 프로젝트 src 제거

루트 프로젝트 src 제거 (필수는 아니지만 헷갈리니까 제거하면 좋다.)

### 🟢 추가한 프로젝트 build.gradle 수정

api 프로젝트의 build.gradle
```groovy
// api
tasks.withType(org.springframework.boot.gradle.tasks.bundling.BootJar) {
    enabled = true
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation project(':oun-common')
    implementation project(':oun-user')
}
```

common 프로젝트의 build.gradle
```groovy
// common

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

user 프로젝트의 build.gradle
```groovy
// user
```

### 🟢 추가한 프로젝트 사용하지 않는 파일들 제거

내 환경의 경우 cursor로 작업을 진행하고 있어 생성되는 파일중 제거할 리스트는 다음과 같다.

1. .vscode
2. .gitignore
3. .gitattributes
4. gradlew.bat
5. HELP.md
6. settings.gradle(중요!)

위에 파일들은 사용하지 않으므로 제거해주면 된다. 그중에서도 settings.gradle은 중요하다. 이 파일은 꼭 제거해주어야 spring이 정상적으로 multi module project로 진행해준다.

### 🟢 정상 모듈 확인

```bash
./gradlew clean build
```

위 명령어를 실행하면 정상적으로 모듈이 빌드되는 것을 확인할 수 있다.

그 후 개발툴에서 api를 실행하면 정상적으로 api 서버가 실행되면 된다.

```
내 환경의 경우 cursor에서는 리로드가 되어야 정상적으로 실행이 되어서 cursor 툴을 완전히 종료했다가 다시 실행해주었더니 정상적으로 인식되어 실행되었다.
```