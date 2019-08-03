Flutter和Android原生有两种混合开发模式，一种是原生项目下集成moudle，一种是将Flutter项目打包为aar放入原生项目中。
### 问题：
以 aar 的模式集成好处是减少对原生的入侵，对于二者的开发人员，能有效的隔离。弊端是对于 flutter 依赖的第三方插件，需要手动的从 flutter SDK 中取出第三方插件的 aar，复制到原生项目下依赖，过程比较繁琐。<br/>
本文的脚本基于网上的代码，填了几个坑，做了一些改造来降低与项目的耦合度。只需要引入 2 个文件，便可以一键打包依赖插件的aar，并上传到Maven仓库。

### 思路：
1. 我们的 flutter 项目在 pubspec.yaml 中声明依赖了第三方插件并且执行过 “ flutter package get ” 命令后，Flutter 会下载第三方插件项目并存到 Flutter 的 SDK 目录中 ，我们的 Flutter 项目根路径中有一个 .flutter-plugins 文件，该文件以 key-value 方式记录了 flutter 项目依赖的插件，和插件项目的位置路径。
2. 我们拿到 .flutter-plugins 文件，就可以遍历 依赖的第三方插件 的项目。
3. 遍历插件项目的时候，我们可以给遍历到的插件项目应用新的 gradle 脚本文件，新的脚本文件中添加了 uploadArchives 系列代码，uploadArchives 是 gradle 的语法，能够将打包的 产出上传到 maven 仓库。
4. 在给每个遍历的插件项目应用新的 gradle 文件后，我们还需要依次调用改插件项目的 assembleDebug ，uploadArchives 命令，这两条命令分别执行打 aar 包和上传 aar 包的操作。

## 思路总结：
1. 我们通过 .flutter-plugins 文件，遍历依赖的第三方插件项目。
2. 遍历每个插件项目的时候，依次执行下面操作：
     A. 给每个插件项目依赖我们的 gradle 文件（android_flutter_maven）。
     B. 执行 插件项目的 assembleDebug 命令构建 aar 。
     C. 执行插件项目的 uploadArchives 上传aar。
3. 以上的操作均使用脚本实现。

### 使用方法：
1. 在 Flutter 项目的根目录下添加下面两个文件：aar_plugin_script.sh 和 android_flutter_maven.gradle。
2. 命令行 cd 进入该 Flutter 项目的根目录。
3. 执行命令，使脚本具有执行权限： chmod +x ./aar_plugin_script.sh.sh
4. 按格式执行脚本，例如： aar_plugin_script.sh 0.0.1 com.koo.flutter

### 使用注意：
1. 脚本是 shell 文件，只适用于 mac 系统。
2. 执行脚本时，版本号为必填参数，报名为可选参数。脚本文件中可以配置默认包名。
3. 使用时，需要在 android_flutter_maven.gradle 文件中配置上传 本地maven 还是 远端maven ，以及远端 maven 的账号密码。
4. 本地Maven默认地址为： flutter项目/.android/maven-local/。
5. 执行脚本的时候，如果 FlutterSDK 是在用户目录下导致脚本无权限修改，不要使用 sudo，
  会造成权限错乱，建议将 Flutter SDK 目录移动出来并且修改 Android 的FlutterSDK位置及系统的环境变量。

## 文件代码 (包含步骤注释)
**aar_plugin_script.sh ：**
 ```
 #!/usr/bin/env bash
 ######################################################################
 # 使用方式: aar_plugin_script version groupId
 # 如： ./androidFlutter 0.0.1 com.xxx
 # version 为必传参数， groupId 为可选参数
 ######################################################################
 # 定义默认groupId
 readonly DEF_GROUP=‘com.koo’
 # 读取传入参数
 debugMode=‘release’
 fVersion=$1 # 第一个参数:版本号,flutter module的版本和所有的插件的版本保持一致.
 fGroupId=$2 # 第二个参数:包名,所有library的groupId都一样，默认是com.koo.fultter.
 ###### 0. 检查参数
 if [ ! -n “${fVersion}” ] ;then
 Echo “>>> 版本参数不能为空!”
 exit
 fi
 if [ ! -n “${fGroupId}” ]
 then
 fGroupId=$DEF_GROUP
 fi
 ###### 1. 获取/更新包
 echo “>>> 执行 flutter packages get”
 flutter packages get
 ###### 2. 遍历 .flutter-plugin 文件
 echo “>>> 遍历 .flutter-plugin 文件”
 for line in $(cat .flutter-plugins)
 do
 plugin_name=${line%%=*}
 plugin_path=${line##*=}
 ###### 3. 向依赖的项目中添加 apply from: “./android_flutter_maven.gradle”
 echo -e “\n”
 echo “>>> 向[“${plugin_name}”]项目中插入 apply from: “./android_flutter_maven.gradle””
 cd .android
 cp ../android_flutter_maven.gradle ${plugin_path}’/android’
 pushd .
 cd ${plugin_path}’/android’
 if [ “$firstLine” != ‘apply from: “./android_flutter_maven.gradle”’ ]
 then
 # mac上插入内容要换行才行
 sed -I ‘’ -e ‘1i \
 apply from: “./android_flutter_maven.gradle”’ build.gradle
 else
 echo “>>> flutter_maven 文件已经存在 “
 fi
 popd
 ###### 4. 构建依赖项目的AAR
 if [ “$debugMode” == “debug” ]
 then
 echo “>>> 构建[“${plugin_name}”]插件的AAR文件(DEBUG)”
 ./gradlew :${plugin_name}:clean :${plugin_name}:assembleDebug -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
 else
 echo “>>> 构建[“${plugin_name}”]插件的AAR文件(RELEASE)”
 ./gradlew :${plugin_name}:clean :${plugin_name}:assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
 fi
 ###### 5. 上传依赖项目的AAR
 echo “>>> 上传[“${plugin_name}”]插件”
 ./gradlew :${plugin_name}:uploadArchives -PfGroupId=${fGroupId} -PfArtifactId=${plugin_name} -PfVersion=${fVersion}
 cd ../
 done
 Echo “>>> 任务完成!”
 ```

**android_flutter_maven.gradle ：**
```
apply plugin: ‘maven’
final def localMaven = true //true: 发布到本地maven仓库， false： 发布到maven私服
final def artGroupId = project.group
final def artVersion = project.version
final def artifactId = project.hasProperty(‘fArtifactId’) && project.fArtifactId ? project.fArtifactId : null
final def isFlutterModule = project.hasProperty(‘fModule’) && project.fModule ? project.fModule : false
if(artifactId == null || artVersion == null) {
return
}
// 因为只要执行./gradlew xxx等命令，rootProject和subProject的build.gradle都要执行一次，
// 所以这里要判断当前module和是否和正在处理的module一样，不是相同module就不处理，
// 但是因为不同业务的flutter module名称都是flutter并且artifactId和module名称又不相同，所以要通过isFlutterModule参数区分
if(! [project.name](http://project.name/) .equals(artifactId) && (! [project.name](http://project.name/) .equals(“flutter”) || !isFlutterModule)) {
return
}
// 很重要，用来覆盖各个自依赖中的group和version
// ./gradlew clean assembleRelease -PfGroupId=${fGroupId} -PfArtifactId=${fArtifactId} -PfVersion=${fVersion}
// 执行了这个命令，根目录的build.gradle就会执行，这时候把参数设置到project.rootProject.ext，后面各个subProject就可以拿到
artGroupId = project.hasProperty(‘fGroupId’) && project.fGroupId ? project.fGroupId : null
artVersion = project.hasProperty(‘fVersion’) && project.fVersion ? project.fVersion : null
project.rootProject.ext {
gArtGroupId = artGroupId
gArtVersion = artVersion
}
uploadArchives {
repositories {
mavenDeployer {
if(localMaven) {
repository(url: uri(project.rootProject.projectDir.absolutePath + ‘/maven-local’))
} else {
repository(url: ‘ [http://172.172.177.240:8081/nexus/content/repositories/snapshots](http://172.172.177.240:8081/nexus/content/repositories/snapshots) ‘) {
authentication(userName: deployment, password: 123456)
}
}
pom.groupId = artGroupId
pom.artifactId = artifactId
pom.version = artVersion
pom.project {
licenses {
license {
name ‘The Apache Software License, artVersion 2.0’
url ‘ [http://www.apache.org/licenses/LICENSE-2.0.txt](http://www.apache.org/licenses/LICENSE-2.0.txt) ‘
}
}
}
}
}
}
```

