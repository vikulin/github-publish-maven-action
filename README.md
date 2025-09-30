# github-publish-maven-action

This GitHub Action repurposes a GitHub repository to act as a Maven repository, allowing you to store and serve Java/Android artifacts directly from GitHub. Build Java project with Gradle, extract metadata, and publish artifacts to a your GitHub repository with no additional registrations and efforts.

This GitHub Action allows you to:

- Checkout your project repository.
- Set up the specified JDK version.
- Extract the latest Git commit message and hash.
- Build the project or specific Gradle modules.
- Publish artifacts to the local Maven repository.
- Copy artifacts and POM files to a specified GitHub repository.
- Generate checksums (`.sha1` and `.md5`) for JAR, AAR, and POM files.
- Commit and push artifacts to a separate repository.

---

## Usage in a Workflow

You can use this action in your workflows. Here's an example:

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
              gradle_modules: ':app,:mylib'
              java_version: '21'
              distribution: 'adopt'

---

## Gradle Configuration

### 1. Add publishing to `build.gradle`

**Groovy:**

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

**Kotlin (Android library):**

    android {
        // ...
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

### 2. Register Secret

Register `secret.PAT` with permissions for your Maven repository and replace the path in the `artifact_repo` parameter.

### 3. Add Your Maven Repo URL

Your Maven repository URL will be:

    https://github.com/<username>/<repo>/raw/<branch>

**Groovy:**

    repositories {
        maven {
            url "https://github.com/RiV-chain/artifact/raw/main"
        }
    }

**Kotlin:**

    repositories {
        maven {
            url = uri("https://github.com/RiV-chain/artifact/raw/main")
        }
    }

---

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `gh_pat` | Personal Access Token for GitHub Maven artifact repository | Yes | - |
| `artifact_repo` | GitHub repository to publish artifacts | Yes | - |
| `gradle_file_path` | Path to the Gradle file | Yes | `./gradlew` |
| `java_version` | Java version to set up | Yes | - |
| `distribution` | Java distribution to set up | Yes | - |
| `gradle_modules` | Comma-separated list of Gradle modules (e.g., `:app,:mylib`). Empty means root module. | No | `""` |

---

## How it Works

1. **Checkout Repository** – Clones your source repository.
2. **Set up JDK** – Installs the specified Java distribution and version.
3. **Extract Git Metadata** – Retrieves the latest commit message and hash.
4. **Install Utilities** – Installs `xmlstarlet` and `tree` for processing POM files.
5. **Build with Gradle** – Builds the root project or specified modules and publishes them to the local Maven repository.
6. **Checkout Artifact Repository** – Clones the repository where artifacts will be stored.
7. **Process Artifacts** – Copies Maven artifacts (`.jar`, `.aar`, `.pom`) from local Maven cache to the artifact repository, generates checksums, and organizes them in Maven structure.
8. **Commit & Push** – Commits the processed artifacts to the artifact repository and pushes the changes.

---

## Notes

- Permissions: Ensure your repository permissions allow this action to check out repositories and push changes.
- Secrets: Ensure the `secrets.PAT` secret is set in your Maven GitHub repository secrets.
- Java `distribution` parameter accepts values from [GitHub Setup-Java Supported Distributions](https://github.com/actions/setup-java#supported-distributions).
- If no modules are specified, the action builds the root Gradle project.
- Checksums are automatically generated for JAR, AAR, and POM files.

---

## License

This project is licensed under the GPL-3.0 license. See the LICENSE file for details.
