## 全局配置
Android工程中有一个项目根目录的build.gradle,每个Module也都有一个私有的build.gradle。全局配置对应项目根目录下的build.gradle，私有配置
则是Module下的build.gradle

- 设定UTF-8
```gradle
allProjects{
    repositories {
       jcenter()
       mavenCentral()
    }

    tasks.withType(JavaCompile){
      options.encoding = "UTF-8"
    }
}
```

- 依赖Google仓库
```gradle
allProjects{
    repositories {
       google()
       jcenter()
    }

}
```
打印地址：
```gradle
allProjects{
    repositories {
       google()
       jcenter()
    }
    repositories.each{
        println it.getUrl()
    }
}
```

- 支持Groovy
```gradle
allProjects{
}

apply plugin: 'groovy'

dependencies{
    compile localGroovy()
}
```

- 定义全局变量
  - 定义全局targetSdkVersion
    在progect根目录下的build.gradle中定义全局变量
    ```gradle
    ext {
        minSdkVersion = 16
        targetSdkVersion = 24
    }
    buildscript {
        
    }
    ```
    然后可以在子module中的build.gradle中通过rootProject.ext.minSdkVersion来引用这些变量
    > 另一种方式
      ```gradle
      buildscript{
        ext.kotlin_version = '1.1.3-2'
      }
      ```
      然后再module中可以直接使用这些变量，implementation "*****:$kotlin_version" 
      这里是双引号用法，单引号需要字符拼接的方式实现 
    
  - 在gradle.properties中定义全局变量
    ```properties
    isFusion = false
    ```
    使用
    ```gradle
    if(isFusion.toBoolean()){
        apply plugin: 'com.android.application'
    }else{
        apply plugin: 'com.android.library'
    }
    ```

- 配置Lint选项
Lint默认会做严格检查，遇到报错会终止构建过程，有时候需要关闭严格检查
```gradle
android{
    lintOptions{
        disable 'InvalidPackage'
        checkReleaseBuilds false
        abortOnError false
    }
}

## 多模块项目可以添加全局过滤配置
task lintCheck(){
    getAllTasks(true).each{
        def lintTasks = it.value.findAll { it.name.contains("lint") }
        lintTasks.each {
            it.enabled = false
        }
    }
}
```

## 操作Task
- 更改输出的APK的名字
```gradle
static def releaseTime(){
    return new Date().format("yyyy-MM-dd",TimeZone.getTimeZone("UTC"))
}
android {
    buildTypes{
        
    }
    android.applicationVariants.all { variant ->
        variant.outputs.all { output ->
            def outputFile = output.outputFile
            if(outputFile != null && outputFile.name.endWith('.apk')){
                def fileName = outputFile.name.replace("app","${defaultConfig.applicationId}_${defaultConfig.versionName}_${releaseTime()}")
                outputFileName = fileName
            }
        } 
    }
}
```
- 更改AAR输出的位置
```gradle
static def releaseTime(){
    return new Date().format("yyyy-MM-dd",TimeZone.getTimeZone("UTC"))
}
android {
    buildTypes{
        
    }
    android.libraryVariants.all {
        variant ->
        variant.outputs.all { output ->
            if(output.outputFile != null && output.outputFile.name.endWidth('.aar') && output.outputFile.size() != 0){
                copy {
                    from output.outputFile
                    into "${rootDir}/libs/"
                }
            }
        } 
    }
}
```
- 跳过AndroidTest
```gradle
tasks.whenTaskAdded {task -> 
    if(task.name.contains('AndroidTest')){
        task.enabled = false
    }
}
```
每一个task都有输入输出，当一个task的输入与前一次执行时的输入完全一致，gradle会认为该Task是可以被优化的，gradle会直接复用上次的输出，即增量构建
为了更好地观察输入和输出信息，可以定义一个查看输入和输出详细信息的Task
```gradle
gradle.taskGraph.afterTask { task -> 
    StringBuffer taskDetails = new StringBuffer()
    taskDetails << """"--------\nname:$task.name"""
    taskDetails << "\nInputs:\n"
    task.iputs.files.each{ inp -> 
        taskDetails << " ${inp}\n"    
    }
    taskDetails << "Outputs:\n"
    task.outputs.files.each{ out -> 
        taskDetails << " ${out.absolutePath}\n"
    }
    println taskDetails
}
```

- 找出耗时的Task
先定义一个监听类
```java
```
然后在app的build.gradle中注册监听
```gradle
project.gradle.addListener(new BuildTimeListener())
```

- 抽离Task脚本
一个成熟的项目必然有庞大的build.gradle,脚本多了，自然就想要抽离出去，可以建立一个xxx.gradle的文件存放这些脚本，引用者只需要通过apply from xxx.gradle进行以来即可


## 动态化
- 动态设置buildConfig
测试开发阶段，开发人员并不会修改老的版本号，每次提交测试额时候很难行为包的打包时间节点
为了方便测试和开发人员能够迅速定位当前包的节点，可以在App详情页面或调试界面的logo下发增加一个commit值

Gradle提供了一个buildCofigField的DSL，在编译的时候可以直接设置其中的一些参数，从而在项目中利用这些参数进行逻辑判断。
```gradle
anroid {
    defaultConfig {
        buildConfigField "String", "BaseUrl", '"http://www.****.com"'
        buildConfigField "String", "LAST_COMMIT", "\"" + revision() + "\""
        resValue "String", "build_host", hostName() //产生res中的资源
    }
}

def revision(){
    def code = new ByteArrayInputStream()
    exec {
        //执行 git rev-parse --short HEAD
        commandLine 'git', 'rev-parse', '--short', 'HEAD'
        standardOutput = code
    }
    return code.toString().substring(0,code.size() - 1)
}

def hostName(){
    return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
}
```
> resValue 生成的build_host会在build/generated/res/resValue/.../geerate.xml中看到，在代码中可以通过getString()拿到

- 填充Manifest中的值
子开发三方库的时候可能根据引用者额不同另一不同的Manifest的值，但Manifest本身是一个写死的xml文件，不具备逻辑能力，灵活性差
比如开发一个第三方登录分享的库，在库的manifest中国必须配置一个腾讯的ID，但这个ID必须交个接入者定义，同时产生了可变和不变的矛盾
为了解决这个问题，希望将变化的部分抽离出去，让Manifest中的变化元素变成变量
```xml
<category android:name="${id}" />
```
上述代码用${id}来做占位，让使用者可以在编译时动态设置id的值
```gradle
    productFlavors {
        google{}
        baidu{}
    }
    productFlavors.all { flavor -> 
        manifestPlaceholders.put("id",name)
    }
```

- 让buildType支持继承
如果想新增一个buildType，又想继承之前配置的一些参数，则可以用init.with()
```gradle
buildTypes {

        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfigs.release
            buildConfigField "String", "BaseUrl", getProps("base_release")
            buildConfigField "String", "FlowUrl", getProps("flow_release")
            buildConfigField "String", "MiFiUrl", getProps("miFi_release")
            buildConfigField "String", "ShareUrl", getProps("share_release")
            buildConfigField "String", "HelpUrl", getProps("help_release")
            buildConfigField "String", "IotUrl", getProps("iot_release")
            buildConfigField "String", "PrivacyClauseUrl", getProps("privacy_clause_release")
            buildConfigField "String", "UserAgreementUrl", getProps("user_agreement_release")
            buildConfigField "String", "UserUnRegisterAgreementUrl", getProps("user_unregister_agreement_release")
            buildConfigField "String", "UmengAppKey", getProps("app_key_release")
            buildConfigField "boolean", "WalleSwitch", getProps("walle_enable")
            buildConfigField "boolean", "isDebug", "false"
            resValue "String", "build_host", hostName()
        }
        debug.initWith(buildTypes.release) //继承
        debug {
            //覆盖
        }
    }
```

- 让flavor支持继承
在开发的过程中，有开发版本，提交测试版本，内部测试版本，预发版本，正式版本等多个版本，为了标识这些版本，需要用到flavor的高级用法
Flavor是一个多维度的定义，下面举个实际的四维例子
```gradle
flavorDimensions 'isfree', 'channel', 'sex'
productFlavors {
        free {
            dimension 'isfree'
        }
        paid {
            dimension 'isfree'
        }
        google {
            dimension 'channel'
        }
        xiaomi {
            dimension 'channel'
        }
        male {
            dimension 'sex'
            applicationId 'com.test.id'
        }
        female {
            dimension 'sex'
        }
}
```
我们可以用父flavor做通用配置，子flavor做差异化配置，比如现在有多个内部测试渠道，但对于开发者来货内部测试的代码几乎是一样的，所以只需要IS_FOR_TEST的变量标识即可。
而flavor的配置直接实现了flavor的继承。
```gradle
flavorDimensions "innerTest", "channel"
productFlavors {
    innerTest {
        dimension "innertest"
        buildConfigFiel "boolean","IS_FOR_TEST","true"   
    }

    forJackTest {
        dimension "channel"
    }
    forTonyTest {
        dimension "channel"
    }
}
```
在java代码中，可以很容易的进行二维判断，代码也比较直观
```java
if(BuildConfig.IS_FOR_TEST){
    switch(BuildConfig.FLAVOR_channel){
        case "forJackTest":

            break;
        case "forTonyTest":
            break;
    }
}
```
> 每增加一个维度，Flavor的个数会呈指数级增加，可以通过ignore去掉不需要的Flavor。
```gradle
    variantFilter { variant -> 
        def names = variant.flavors*.name
        if (names.contains("minApi21") && nmes.contains("demo")){
            setIgnore(true)
        }
        if (variant.buildType.name.equal("debug")){
            variant.setIgnore(true)
        }
    }
```

- 内侧版本用特定的icon
不同的Flavor在目录结构中可以映射不同的文件夹，可以子src中建立以Flavor命名的包，然后在里面针对Flavor做私有的操作。
比如在main同级目录建立一个 innerTest 的文件夹，将特定的icon放在该目录下的res 文件夹下的 图片目录下。
如果要建立开发或测试环境的应用，可以继承原本应用，然后做差异化处理