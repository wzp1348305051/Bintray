# Bintray
AndroidStudio中上传库文件步骤：
1.在工程目录下的local.properties文件中加入有关bintray账号信息，如下：
bintray.user=bintray账号用户名
bintray.apikey=bintray账号API KEY
bintray.gpg.password=bintray账号下设置的GPG证书口令

2.在工程目录下的gradle.properties文件中加入有关需要上传的包信息，如下：
# 包组织名
PROJ_GROUP=wengzhipeng
# 包ID
PROJ_ARTIFACTID=bintray
# 包版本号
PROJ_VERSION=1.0.0.0
# 最后gradle引用的形式就是$PROJ_GROUP:$PROJ_ARTIFACTID:$PROJ_VERSION

# 包名
PROJ_NAME=BintrayDemo
# 包项目主页地址
PROJ_WEBSITEURL=http://xxx
# 包问题跟踪地址
PROJ_ISSUETRACKERURL=http://xxx
# 包VCS地址
PROJ_VCSURL=http://xxx
# 包描述
PROJ_DESCRIPTION=xxx

# 包证书信息
PROJ_LICENCE_NAME=The Apache software License, Version 2.0
PROJ_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
PROJ_LICENCE_DESCRIPTION=Apache License

# 开发者信息
DEVELOPER_ID=wengzhipeng
DEVELOPER_NAME=wengzhipeng
DEVELOPER_EMAIL=wzp1348305051@gmail.com

3.在需要发布的模块目录下创建bintray.gradle文件，文件内容如下：
// 添加maven插件,用于自动生成pom文件等
apply plugin: 'com.github.dcendents.android-maven'
// 添加bintray插件
apply plugin: 'com.jfrog.bintray'

// 库的包名
group = PROJ_GROUP
// 库的版本
version = PROJ_VERSION

// 定义POM并打包AAR
install {
    repositories.mavenInstaller {
        pom {
            project {
                packaging 'aar'
                // 库的简单描述
                description PROJ_DESCRIPTION

                // 添加自己的描述信息
                name PROJ_NAME
                url PROJ_WEBSITEURL

                // 设置证书信息
                licenses {
                    license {
                        name PROJ_LICENCE_NAME
                        url PROJ_LICENCE_URL
                        distribution PROJ_LICENCE_DESCRIPTION
                    }
                }

                // 开发者信息
                developers {
                    developer {
                        id DEVELOPER_ID
                        name DEVELOPER_NAME
                        email DEVELOPER_EMAIL
                    }
                }

                scm {
                    connection PROJ_VCSURL
                    developerConnection PROJ_VCSURL
                    url PROJ_WEBSITEURL
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs //source指定了源码位置
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    failOnError = false
}

// 这里dependsOn意为仅当javadoc完成后才开始本task
task javadocJar(type: Jar, dependsOn: javadoc) {
    from javadoc.destinationDir
    classifier = 'javadoc'
}

artifacts {
    archives javadocJar
    archives sourcesJar
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")

    configurations = ['archives']
    pkg {
        repo = "android" // 此处对应Bintray上的Repositories名称
        name = PROJ_NAME
        websiteUrl = PROJ_WEBSITEURL
        vcsUrl = PROJ_VCSURL
        licenses = ["Apache-2.0"]
        publish = true
        version {
            desc = PROJ_DESCRIPTION
            gpg {
                sign = true // 是否用GPG签名文件
                passphrase = properties.getProperty("bintray.gpg.password") // GPG密钥口令
            }
        }
    }
}

4.在需要发布的模块目录下的build.gradle最后加入apply from: 'bintray.gradle'

5.在AndroidStudio的Terminal窗口中输入gradle install编译库文件

6.在AndroidStudio的Terminal窗口中输入gradle bintrayUpload上传库文件至Bintray

7.在Bintray上进入刚刚上传的包，点击Add to JCenter发布到JCenter
