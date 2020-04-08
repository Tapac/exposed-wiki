# How to contibute to an Exposed project
## What to contibute
When you had faced some problem or absent functionallity in Exposed feel free to search through [opened issues](https://github.com/JetBrains/Exposed/issues) to find a similar one. If there is no such issue you could try to resolve it by yourself and provide us with Pull Request (see below). 

If you want to help the project and don't how you could start with looking for issues taged with [good-first-issue](https://github.com/JetBrains/Exposed/issues?q=is%3Aopen+is%3Aissue+label%3Agood-first-issue) tag. Select one, assign it on you and start to work on Pull Request.

Please do not keep an issue assigned on you if you do not plan to fix it as someone else could take it.

## How to contibute
1. Fork [Exposed repository](https://github.com/JetBrains/Exposed). (Read more about forks in [github documentation](https://help.github.com/en/github/getting-started-with-github/fork-a-repo))
2. Make apropriate changes in your repository copy.
3. Run embedded tests by executing `./gradlew tests` inside the root folder of your local copy.
4. Run docker tests for every supported database (see below) or skip if you will check them on TeamCity CI.
5. [Create a Pull Request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork) and check the PR id.
6. Open [Exposed](https://teamcity.jetbrains.com/project.html?projectId=Exposed) section on https://teamcity.jetbrains.com/ and filter it by your PR id
7. Check that all configurations (except Build, Deploy and Oracle which broken at the moment) are passed and green after your changes.
8. Submit Pull Request on review

## How to run docker-based tests
There are many databases which could not be tested without running the real instances. To run tests against them you have to install [Docker](https://docs.docker.com/get-docker/) on your system. Please note that it also will require a huge amount of free disc space to store database images.

When you have Docker installed:
1. Open [gradle.properties](https://github.com/JetBrains/Exposed/blob/master/gradle.properties) file locally
2. Change `dialect` parameter from `none` to one of:
-`mysql` - run against the latest MySQL 5.7
-`mysql8` - run against the latest MySQL 8.0
-`mariadb` - run against the latest MariaDB
-`sqlserver` - run against the latest SQLServer 2017
-`oracle` - run against the latest Oracle 18 XE
3. Run `./gradlew exposedDialectTestWithDocker` inside the root folder.
4. Await tests to finish (it could take time to download database images on the first run)
5. Repeat with another database dialect if needed.

# How to improve wiki pages

Due to github wiki limitations (and also JetBrains repository restrictions) the prefered way to contribute to wiki is to:
1. Clone https://github.com/Tapac/exposed-wiki repository (it's a mirror of the https://github.com/JetBrains/Exposed/wiki repository)
2. Make changes and submit Pull Request. 
3. Changes will appear after PR review.