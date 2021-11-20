---
title: 在Azure Pipeline上编译React Native Android APP
tags:
  - DevOps
  - React Native
  - Pipeline
categories: DevOps
description: 使用Azure DevOps中的Pipeline来编译基于RN的Android APP
abbrlink: bf91
date: 2021-11-13 22:04:16
keywords:
---

本文介绍在Azure Pipeline上搭建Android 编译打包过程以及中间遇到的问题，最终将打包好的APP发布到微软APP Center中

![image-20211114074340599](https://oss.smart-lifestyle.cn/file/3adoj.png)



## 流水线

流水线的Pipeline配置如下（实际使用的是Pipeline的经典模式，即图形化模式，下面配置跟实际稍微有区别）

```yaml
pool:
  name: Azure Pipelines
#Your build pipeline references an undefined variable named ‘envconfig.secureFilePath’. Create or edit the build pipeline for this YAML file, define the variable on the Variables tab. See https://go.microsoft.com/fwlink/?linkid=865972
#Your build pipeline references an undefined variable named ‘keystorefile.secureFilePath’. Create or edit the build pipeline for this YAML file, define the variable on the Variables tab. See https://go.microsoft.com/fwlink/?linkid=865972
#Your build pipeline references an undefined variable named ‘envconfig.secureFilePath’. Create or edit the build pipeline for this YAML file, define the variable on the Variables tab. See https://go.microsoft.com/fwlink/?linkid=865972

steps:
- task: DownloadSecureFile@1
  displayName: 'Download keystore file'
  inputs:
    secureFile: xxx.xxx.xxx.keystore

- task: DownloadSecureFile@1
  displayName: 'Download envconfig'
  inputs:
    secureFile: .env.prod

- script: |
   echo "link env file"
   ln -s $(envconfig.secureFilePath) ./ 
   echo "link keystore"
   ln -s $(keystorefile.secureFilePath) ./android/app
  workingDirectory: ./
  displayName: 'link env file and keystore file'

- task: NodeTool@0
  displayName: 'Use Node 14'
  inputs:
    versionSpec: 14

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-installer-task.YarnInstaller@3
  displayName: 'Use Yarn 1.22'
  inputs:
    versionSpec: 1.22

- task: Cache@2
  displayName: Cache
  inputs:
    key: 'yarn | "$(Agent.OS)" | yarn.lock'
    path: './node_modules'
    cacheHitVar: 'yarn | "$(Agent.OS)"'

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn add jetifier'
  inputs:
    projectDirectory: ./
    arguments: 'add jetifier'

- script: 'npx jetify'
  workingDirectory: ./
  displayName: 'npx jetify'

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn  install'
  inputs:
    projectDirectory: ./
    arguments: install

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn build'
  inputs:
    projectDirectory: ./
    arguments: 'sh:assemble:android:prod'

- task: CopyFiles@2
  displayName: 'Copy Files to: $(build.artifactstagingdirectory)'
  inputs:
    SourceFolder: '$(system.defaultworkingdirectory)'
    Contents: '**/*.apk'
    TargetFolder: '$(build.artifactstagingdirectory)'
  condition: succeededOrFailed()

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  inputs:
    PathtoPublish: '$(build.artifactstagingdirectory)'
  condition: succeededOrFailed()

- task: AppCenterTest@1
  displayName: 'Test with Visual Studio App Center'
  inputs:
    appFile: '**/*.apk'
    prepareTests: false
    runTests: false
  enabled: false
  condition: succeededOrFailed()

- task: AppCenterDistribute@2
  displayName: 'Deploy **/*.apk to Visual Studio App Center'
  inputs:
    serverEndpoint: 'service name'
    appSlug: 'org/appname'
    appFile: '**/*.apk'
    symbolsIncludeParentDirectory: false
    releaseNotesInput: 'new release'

```

### ### 下载机密文件和配置文件

```yaml
- task: DownloadSecureFile@1
  displayName: 'Download keystore file'
  name: keystorefile
  inputs:
    secureFile: xxx.xxx.xxx.keystore

- task: DownloadSecureFile@1
  displayName: 'Download envconfig'
  name: envconfig
  inputs:
    secureFile: .env.prod
```

Azure DevOps Pipeline中提供了安全文件的下载，我们在环境中使用的keystore文件以及env文件是不能直接提交到代码中，第一是为了安全，如果泄漏后果很严重，第二是方便多个环境（开发、测试、生产）的配置

在实际使用中，通过`keystorefile.secureFilePath`和`envconfig.secureFilePath`来使用这两个文件

```yaml
- script: |
   echo "link env file"
   ln -s $(envconfig.secureFilePath) ./ 
   echo "link keystore"
   ln -s $(keystorefile.secureFilePath) ./android/app
  workingDirectory: ./
  displayName: 'link env file and keystore file'
```

### 安装指定版本的node、yarn

```yaml
- task: NodeTool@0
  displayName: 'Use Node 14'
  inputs:
    versionSpec: 14

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-installer-task.YarnInstaller@3
  displayName: 'Use Yarn 1.22'
  inputs:
    versionSpec: 1.22
```

### 安装依赖包

```yaml
- task: Cache@2
  displayName: Cache
  inputs:
    key: 'yarn | "$(Agent.OS)" | yarn.lock'
    path: './node_modules'
    cacheHitVar: 'yarn | "$(Agent.OS)"'

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn add jetifier'
  inputs:
    projectDirectory: ./
    arguments: 'add jetifier'

- script: 'npx jetify'
  workingDirectory: ./
  displayName: 'npx jetify'

- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn  install'
  inputs:
    projectDirectory: ./
    arguments: install
```

### 编译apk

这里实际执行sh:assemble:android:prod命令为：`cd android/ && export ENVFILE=.env.beta && ./gradlew assembleBetaRelease && cd ..`

```yaml
- task: geeklearningio.gl-vsts-tasks-yarn.yarn-task.Yarn@3
  displayName: 'Yarn build'
  inputs:
    projectDirectory: ./
    arguments: 'sh:assemble:android:prod'
```

### 分发到appcenter

```yaml
- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  inputs:
    PathtoPublish: '$(build.artifactstagingdirectory)'
  condition: succeededOrFailed()

- task: AppCenterTest@1
  displayName: 'Test with Visual Studio App Center'
  inputs:
    appFile: '**/*.apk'
    prepareTests: false
    runTests: false
  enabled: false
  condition: succeededOrFailed()

- task: AppCenterDistribute@2
  displayName: 'Deploy **/*.apk to Visual Studio App Center'
  inputs:
    serverEndpoint: 'service name'
    appSlug: 'org/appname'
    appFile: '**/*.apk'
    symbolsIncludeParentDirectory: false
    releaseNotesInput: 'new release'
```

## 问题总结

大部分问题在于引用环境变量，引用机密文件上，导致在编译时找不到一些内容而出错

### 问题1 `android.support.annotation does not exist`

安装jetifier

`yarn add jetifier`

`npx jetify`



### 问题2 缓存node依赖

为了避免每次build时都重新下载node依赖，azure pipeline提供了管道缓存，查看[参考地址](https://docs.microsoft.com/zh-cn/azure/devops/pipelines/release/caching?view=azure-devops)

但是由于时间问题没有调通

```yaml
- task: Cache@2
  displayName: Cache
  inputs:
    key: 'yarn | "$(Agent.OS)" | yarn.lock'
    path: './node_modules'
    cacheHitVar: 'yarn | "$(Agent.OS)"'
```

### gradle file(project.env.get(filename))

这里的filename只能是文件名，而且gradle会自动加上完整的android/app/filename，如果这里写的是其他路径的文件会找不到（即使写的绝对路径），因此通过软连接的形式，将文件链接到android/app文件夹下

```yaml
- script: |
   echo "link env file"
   ln -s $(envconfig.secureFilePath) ./ 
   echo "link keystore"
   ln -s $(keystorefile.secureFilePath) ./android/app
  workingDirectory: ./
  displayName: 'link env file and keystore file'
```





## 参考

[1] [react native build error: package android.support.annotation does not exist - Stack Overflow](https://stackoverflow.com/questions/56667264/react-native-build-error-package-android-support-annotation-does-not-exist)

[2] [react-native-pipeline/azure-pipelines-android.yml at master · staff0rd/react-native-pipeline (github.com)](https://github.com/staff0rd/react-native-pipeline/blob/master/azure-pipelines-android.yml)

[3] [Continuous deployment of React Native app with Azure DevOps - LogRocket Blog](https://blog.logrocket.com/continuous-deployment-of-react-native-app-with-azure-devops/)

[4] [管道缓存 - Azure Pipelines | Microsoft Docs](https://docs.microsoft.com/zh-cn/azure/devops/pipelines/release/caching?view=azure-devops)

[5] [Continuous Integration for React Native with Azure Pipelines | by Liam Andrew | Medium](https://medium.com/@liam.e.andrew/continuous-integration-for-react-native-with-azure-pipelines-245d90948f6a)

[6] [下载安全文件任务 - Azure Pipelines | Microsoft Docs](https://docs.microsoft.com/zh-cn/azure/devops/pipelines/tasks/utility/download-secure-file?view=azure-devops)

[7] [gradle.properties doesn't process project.env.get("_keystore_file") · Issue #314 · luggit/react-native-config (github.com)](https://github.com/luggit/react-native-config/issues/314)



