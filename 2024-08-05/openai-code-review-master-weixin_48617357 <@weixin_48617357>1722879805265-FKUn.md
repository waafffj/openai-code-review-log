# 小傅哥项目： OpenAi 代码评审.
### 😀代码评分：90
#### 😀代码逻辑与目的：
代码逻辑主要是在GitHub Actions中定义了一个工作流程，用于构建和运行一个名为OpenAiCodeReview的Java程序。该程序用于代码审查，并配置了与微信和ChatGLM API的集成。
#### 🤔问题点：
1. `mvn dependency:copy` 命令中的输出目录拼写错误，应为 `outputDirectory` 而不是 `outputDirdectory`。
2. 在代码审查步骤中，环境变量设置使用了 `GITHUB_REVIEW_LOG_URI` 和 `GITHUB_TOKEN`，但没有在步骤中看到这些秘密的引用。
3. 在Java代码中，配置信息直接硬编码在类中，这不利于配置管理和安全性。
#### 🎯修改建议：
1. 修正 `mvn dependency:copy` 命令中的输出目录拼写错误。
2. 在环境变量设置步骤中添加获取和设置秘密的步骤。
3. 使用配置文件或配置管理服务来管理配置信息，避免硬编码。
#### 💻修改后的代码：
```yaml
diff --git a/.github/workflows/main-maven-jar.yml b/.github/workflows/main-maven-jar.yml
index a3c0b6f..c89319e 100644
--- a/.github/workflows/main-maven-jar.yml
+++ b/.github/workflows/main-maven-jar.yml
@@ -3,23 +3,31 @@ name: Build and Run OpenAiCodeReview By Main Maven Jar
 on:
   push:
     branches:
-      - *
+      - master
   pull_request:
     branches:
-      - *
+      - master
 
 jobs:
   build:
     runs-on: ubuntu-latest
     steps:
       - name: Checkout repository
         uses: actions/checkout@v2
         with:
           fetch-depth: 2
       - name: Set up JDK 11
         uses: actions/setup-java@v2
         with:
@@ -30,7 +38,7 @@ jobs:
         run: mvn clean install
       - name: Copy openai-code-review-sdk JAR
         run: mvn dependency:copy -Dartifact=plus.gaga.middleware:openai-code-review-sdk:1.0 -DoutputDirectory=./libs
       - name: Get repository name
         id: repo-name
         run: echo "REPO_NAME=${GITHUB_REPOSITORY##*/}" >> $GITHUB_ENV
@@ -39,7 +47,7 @@ jobs:
         run: echo "COMMIT_AUTHOR=$(git log -1 --pretty=format:'%an <%ae>')" >> $GITHUB_ENV
       - name: Get commit message
         id: commit-message
         run: echo "COMMIT_MESSAGE=$(git log -1 --pretty=format:'%s')" >> $GITHUB_ENV
@@ -46,7 +54,6 @@ jobs:
       - name: Print repository, branch name, commit author, and commit message
         run: |
           echo "Repository name is ${{ env.REPO_NAME }}"
           echo "Branch name is ${{ env.BRANCH_NAME }}"
-          echo "Commit author is ${{ env.COMMIT_AUTHOR }}"
-          echo "Commit message is ${{ env.COMMIT_MESSAGE }}"
+          echo "Commit author is ${{ secrets.COMMIT_AUTHOR }}"
+          echo "Commit message is ${{ secrets.COMMIT_MESSAGE }}"
 
       - name: Run Code Review
         run: java -jar ./libs/openai-code-review-sdk-1.0.jar
         env:
           GITHUB_REVIEW_LOG_URI: ${{ secrets.CODE_REVIEW_LOG_URI }}
           GITHUB_TOKEN: ${{ secrets.CODE_TOKEN }}
           COMMIT_PROJECT: ${{ env.REPO_NAME }}
           COMMIT_BRANCH: ${{ env.BRANCH_NAME }}
           COMMIT_AUTHOR: ${{ secrets.COMMIT_AUTHOR }}
           COMMIT_MESSAGE: ${{ secrets.COMMIT_MESSAGE }}
           WEIXIN_APPID: ${{ secrets.WEIXIN_APPID }}
           WEIXIN_SECRET: ${{ secrets.WEIXIN_SECRET }}
           WEIXIN_TOUSER: ${{ secrets.WEIXIN_TOUSER }}
           WEIXIN_TEMPLATE_ID: ${{ secrets.WEIXIN_TEMPLATE_ID }}
           CHATGLM_APIHOST: ${{ secrets.CHATGLM_APIHOST }}
           CHATGLM_APIKEYSECRET: ${{ secrets.CHATGLM_APIKEYSECRET }}
diff --git a/openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/OpenAiCodeReview.java b/openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/OpenAiCodeReview.java
index a203fad..e94d128 100644
--- a/openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk