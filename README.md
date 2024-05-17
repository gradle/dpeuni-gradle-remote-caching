# DPE University Training

<p align="left">
<img width="10%" height="10%" src="https://user-images.githubusercontent.com/120980/174325546-8558160b-7f16-42cb-af0f-511849f22ebc.png">
</p>

## Gradle Remote Caching Exercise

This is a hands-on exercise to go along with the
[Incremental Builds and Build Caching](https://dpeuniversity.gradle.com/app/catalog)
training module. In this exercise you will go over the following:

* Enable and use remote caching
* Compare build scans to identify cause of cache misses

---
## Prerequisites

* Finished going through the relevant sections in the training course
* Completed the incremental build and local caching exercise

---
## Develocity Authentication

We will use the DPE University Develocity instance as the remote cache.
If you haven't already done so, you can authenticate with the Develocity service by running:

```shell
./gradlew provisionGradleEnterpriseAccessKey
```

The output of the task will indicate a browser window will come up from which you can complete the authentication:

<p align="center">
<img width="75%" height="75%" src="https://github.com/gradle/build-tool-training-exercises/assets/120980/ccafa270-dbab-4c66-ba12-caabcd10399c">
</p>

Once the browser window comes up you can enter a title for the access key that will be created or go with the suggested title:

<p align="center">
<img width="75%" height="75%" src="https://github.com/gradle/build-tool-training-exercises/assets/120980/1aeef46a-2fb6-472a-8d87-82af31b20799">
</p>

Once confirmed you will see the following message and you can close the browser window and return to the editor:

<p align="center">
<img width="75%" height="75%" src="https://github.com/gradle/build-tool-training-exercises/assets/120980/1711c9db-814c-4df1-9d18-42fe5d1b82f8">
</p>

---
### Enable Remote Cache

1. Edit the `gradle.properties` file and add `org.gradle.caching=true` to it. The
   contents of the file now look like:

```properties
org.gradle.console=verbose
org.gradle.caching=true
```

2. Open the `settings.gradle.kts` file. Notice the `com.gradle.enterprise` plugin applied and the `gradleEnterprise` configuration:

```kotlin
plugins {
    // Apply the foojay-resolver plugin to allow automatic download of JDKs
    id("org.gradle.toolchains.foojay-resolver-convention") version "0.8.0"
    id("com.gradle.enterprise") version "3.16.2"
}

gradleEnterprise {
    server = "https://dpeuniversity-develocity.gradle.com"
    buildScan {
        capture {
            isTaskInputFiles = true
        }
    }
}
```

3. In the `settings.gradle.kts` file add a `buildCache` configuration which disables the local cache and uses the `gradleEnterprise` configuration for the remote cache:

```kotlin
buildCache {
    local {
        isEnabled = false
    }
    remote(gradleEnterprise.buildCache) {
        isEnabled = true
        isPush = true
    }
}
```

4. Now run a clean followed by the tests. We will see no task output was fetched from the remote cache.

```diff
$ ./gradlew :app:clean :app:test
# Task :app:clean
# Task :app:generateLocalUniqueValue UP-TO-DATE
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
- Task :app:compileTestJava
# Task :app:processTestResources NO-SOURCE
- Task :app:testClasses
- Task :app:test
```

5. Now run the clean and tests again and you will see the remote cache being used.
   Also pass the `--scan` flag which will generate a build scan that we can explore.

```diff
$ ./gradlew :app:clean :app:test --scan
# Task :app:clean
# Task :app:generateLocalUniqueValue UP-TO-DATE
! Task :app:compileJava FROM-CACHE
# Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE
! Task :app:compileTestJava FROM-CACHE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
! Task :app:test FROM-CACHE
```

6. Open the build scan, go to the Timeline (in the left menu) and expand the
   compileJava task and inspect the cache details.

<p align="center">
<img width="60%" height="60%" src="https://user-images.githubusercontent.com/120980/177324633-23517ec0-5392-40e9-8cfa-a4d5eb593b93.png">
</p>

7. Now edit the string in `app/src/main/java/com/gradle/lab/App.java` to
   something different and run the clean and tests with the `--scan` flag.

```diff
$ ./gradlew :app:clean :app:test --scan
# Task :app:clean
> Task :app:generateLocalUniqueValue UP-TO-DATE
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
! Task :app:compileTestJava FROM-CACHE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
- Task :app:test
```

8. You can see the cache misses. Now open the build scan, and in the top right
   click on the "Build Scans" link to view all the scans.

<p align="center">
<img width="35%" height="35%" src="https://user-images.githubusercontent.com/120980/177330766-ff55fbda-2f0d-45a4-98b1-645d1776a9c4.png">
</p>

9. Select your two recent scans to compare and click on the "Compare" button on
   the bottom right.

<p align="center">
<img width="60%" height="60%" src="https://user-images.githubusercontent.com/120980/177334045-e52b140b-a989-4b03-933e-8685eaa09398.png">
</p>

10. Expand the file properties to see which files were different in the inputs
    which caused the cache miss.

<p align="center">
<img width="60%" height="60%" src="https://user-images.githubusercontent.com/120980/177334631-5c94bbc3-5279-4fee-b202-ab4dc0599e17.png">
</p>
