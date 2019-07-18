
### jekins搭建步骤
### 项目配置
1. 将jerkins用户加入到需要打包的工程内

2. 进入 Jenkins，注册登录、点击New Item

3. 输入项目名、选择 Freestyle project (自由风格)、ok

4. 在 Source Code Management (源码管理) 中选择 Git

5. 输入 Repository URL 使用SSH方式

6. Credentials 选择 jerkins-ssh-key

7. Branch 中填写需要打包的分支

8. 在 Build Triggers 中勾选 Poll SCM 输入 */1 * * * * (表示每分钟轮询一次是否有commit提交)

9. 在 Build 中点击 Add build step 选择需要的打包方式 (Execute shell 或 Invoke Gradle script)，打包基本流程如下：

getPRType 判断打包类型

读取项目版本(添加类型后缀)

打包脚本

判断打包是否成功

tag 打 tag (按需)

getPRCommit 获取邮件内容，sendEmail 发送邮件

最后记得保存

工具脚本说明


发送邮件
脚本路径：/Users/Shared/Jenkins/bin/sendEmail
脚本说明：发送邮件(默认使用Jenkins@51dojo.com邮箱发送邮件)

脚本用法：

# sendEmail 收件人(多账号使用,分割) 抄送(多账号使用,分割) 主题 内容 [附件]
/Users/Shared/Jenkins/bin/sendEmail "$to" "$cc" "$subject" "$email_content" "$attachment"
获取提交类型
脚本路径：/Users/Shared/Jenkins/bin/getPRType

脚本说明：获取到PR约定的提交类型，返回的结果可能是alpha、rc、空字符、其他

脚本用法：

# getPRType commit_id [pr_type]
/Users/Shared/Jenkins/bin/getPRCommit $GIT_COMMIT $pr_type
获取提交内容
脚本路径：/Users/Shared/Jenkins/bin/getPRCommit

脚本说明：获取到PR约定的提交内容，返回格式为HTML

脚本用法：

# getPRType commit_id [pr_type]
/Users/Shared/Jenkins/bin/getPRCommit $GIT_COMMIT $pr_type
打标签
脚本路径：/Users/Shared/Jenkins/bin/tag
脚本说明：打 tag 并上传

脚本用法：

# tag tag
/Users/Shared/Jenkins/bin/tag "moxie-client-1.0.0"
PR格式约定
dev -> testing

type:alpha/rc （目前只支持alpha/rc这两种提测类型）
## 提测日期

xxxx/xx/xx

## 相关需求

1. [xxxxx](http://chandao.51dojo.com/xxxx)

## 测试方式及注意点

1. xxx
2. yyy
testing -> master

## 发版日期：

xxxx/xx/xx

## 影响业务（可选）

1. xxx
2. xxx

## 发版内容

1. xxx
2. yyy

## 发版人

- xxx/10000000000



# 防止上次打包遗留的文件产生影响
git add .
git reset --hard

# 发送错误邮件
sendError(){
email_content="<h2>$1</h2><a href='http://192.168.1.1:8080/job/${JOB_BASE_NAME}/${BUILD_NUMBER}/console'>点击查看日志</a>"
to="zhangman@51dojo.com,fangzhou@51dojo.com"
cc=""
subject="[$1][${JOB_BASE_NAME}] ${BUILD_NUMBER}"
/Users/Shared/Jenkins/bin/sendEmail "$to" "$cc" "$subject" "$email_content"
}

npm i

pr_type=`/Users/Shared/Jenkins/bin/getPRType $GIT_COMMIT`


if [[ $pr_type != "alpha" && $pr_type != "rc" ]]; then
	# sendError "错误的提测类型"
    # echo "错误的提测类型"
    exit 1
else
	base_version=`cat ./src/scripts/version.js | cut -d "'" -f 2`
	time_version=`date +"%Y%m%d%H%M%S"`
 	temp_version="$base_version-$pr_type.$time_version"
    echo "export default '${temp_version}'" > ./src/scripts/version.js
    {
    	npm run pub-test
        # todo 打tag 发邮件
        if [[ $pr_type == "alpha" ]]; then
            commit_content=`/Users/Shared/Jenkins/bin/getPRCommit $GIT_COMMIT $pr_type`
            email_content="${commit_content}"
            to="ceshi@51dojo.com"
            cc="zhangman@51dojo.com,fangzhou@51dojo.com"
            subject="[提测申请][${JOB_BASE_NAME}]"
            /Users/Shared/Jenkins/bin/sendEmail "$to" "$cc" "$subject" "$email_content"
        fi

    } || {
    	sendError "打包失败"
        echo "打包失败"
        exit 1
    }


### 发送邮件脚本
# sendEmail 收件人 抄送 主题 内容 附件
if [[ "$5" = "" && "$6" = "" ]];then
 echo $4 | mutt -c "$2" -s "$3" -- $1
elif [[ "$6" = "" ]];then
 echo $4 | mutt -c "$2" -s "$3" -a "$5" -- $1
else
 echo $4 | mutt -c "$2" -s "$3" -a "$5" -a "$6" -- $1
fi
fi

### 获取提交信息
# getPRType commit_id pr_type
if [[ $2 = "" ]]; then
 git show $1 | sed 's/^[ ]*//g' | sed -n '8,1000000p' | md2html
else
 git show $1 | sed 's/^[ ]*//g' | sed -n '9,1000000p' | md2html
fi


# getPRType commit_id
key=`git show $1 | sed 's/^[ ]*//g' | sed -n '8,8p' | cut -d ":" -f 1`
value=`git show $1 | sed 's/^[ ]*//g' | sed -n '8,8p' | cut -d ":" -f 2`

if [[ $key = 'type' ]]; then
 echo $value
else
 echo ""
fi

