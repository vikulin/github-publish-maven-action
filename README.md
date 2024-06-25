# github-publish-maven-action
Build Java project with Gradle, extract metadata, and publish artifacts to a repository.

# Usage in a Workflow

You can use this action in your workflows. Here's an example of how to use it:

```
name: Build and Publish

on:
  workflow_dispatch:

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - name: Use Maven publish action
        uses: RiV-chain/github-publish-maven-action@main
        with:
          gh_pat: ${{ secrets.PAT }}
          artifact_repo: 'RiV-chain/artifact'
          gradle_file_path: './gradlew'
          java_version: '8'
          distribution: 'adopt'
```

1. Add following lines in ```build.gradle``` build script:

Groovy
```
publishing {
    repositories {
        mavenLocal()
    }
    publications {
        gpr(MavenPublication) {
            from(components.java)
        }
    }
}
```

Kotlin
```
android {
//...
// This is important for publishing Android libraries
    publishing {
        singleVariant("release") {
            withSourcesJar()
            withJavadocJar()
        }
    }
}

publishing {
    publications {
        create<MavenPublication>("gpr") {
            groupId = "org.rivchain"
            artifactId = "libsodium-android"
            version = "1.0.0"

            // Use the release component that we defined above
            afterEvaluate {
                from(components["release"])
            }
        }
    }
    repositories {
        mavenLocal()
    }
}
```

2. Register ```secret.PAT``` with permission for you Maven repo and replace the path it in ```artifact_repo``` parameter.

4. Add your repo URL in build.gradle. Your Maven repo will have path ```https://github.com/<username>/<repo>/raw/<branch>```. Example [repo](https://github.com/RiV-chain/artifact).

Groovy
```
    repositories {
        maven {
            url "https://github.com/RiV-chain/artifact/raw/main"
        }
    }
```

Kotlin
```
    repositories {
        maven {
                url = uri("https://github.com/RiV-chain/artifact/raw/main")
        }
    }
```

# Notes:
  * Permissions: Ensure your repository permissions allow this action to perform the required tasks, including checking out repositories and pushing changes.
  * Secrets: Ensure the ```${{ secrets.PAT }}``` secret is set in your Maven GitHub repository secrets.
  * Java ```distribution``` parameter receives possible values: https://github.com/actions/setup-java?tab=readme-ov-file#supported-distributions.
