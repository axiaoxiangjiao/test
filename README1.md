# IaC代码本地测试

```
git clone git@gitlab.onwalk.net:root/iac-pipeline.git
cd iac-pipeline
git branch dev
```
以 iam job为例，测试分支提交代码后，按照如下参考步骤，进行IaC Code进行单元测试
```
export VAULT_URL=<https://vault-url:port>
export VAULT_TOKEN=<vault-token>
export VAULT_PATH=<path-name>
export VAULT_SECRET_COUNT=<secret-name-1>
export VAULT_SECRET_S3=<secret-name-2>

make -C iam init      #验证iam目录配置初始化
make -C iam apply     #验证iam资源创建
make -C iam destroy   #验证iam资源删除 
```

# IaC流水线的测试与使用

```
cd iac-pipeline
git push -uf origin dev

```
1. 代码提交后，会自动触发CICD流水线对测试分支进行测试
2. 登陆Gitlab创建PR，项目维护者可以审批是否合并到主分支
3. 审批通过后合并到主分支， 会触发主分支CICD流水线


# CICD-Pipeline设计说明

* 目录结构
```
.gitlab-ci.yml                    Gitlab CICD pipeline文件
common-variable.yaml              存放全局变量
modules/                          存放所有公共模块目录
network/                          network-pipeline-jobs模版
...                               其他pipeline-jobs模版
Makefile                          全局Makefile，调用其他jobs模版目录的Makefile
README.md                         参考说明
```

* 变量定义

|           |    存储位置                 | 参考说明                                 |
|-----------|---------------------------- | -----------------------------------------|
|敏感信息   | vault-sever                 | 存储账户AK/SK等需要严格保密的信息        |
|非敏感信息 |.gitlab-ci.yml    variables  | 存储 CICD 变量 比如是选择 dev用户  还是选择 prod 用户| 
|非敏感信息 | iam/input-variable.yaml     | 存储 iam jobs专属的input变量|
|非敏感信息 | netork/input-variable.yaml  | 存储 network jobs专属的input变量|
|非敏感信息 | 其他pipeline jobs 依次类推  |

*  示例

以iam-pipeline-jobs为例，iam 目录各个文件说明

1. iam/Makefile 定义4个target
```
init 初始化pipeline jobs需要的的参数和配置文件;
apply 创建资源，依赖init;
output 输出结果;
destroy 删除资源
```
2. iam/init.py init脚本，从Gitlab CICD环境变量, vault server读取输入的变量，生成pipeline jobs需要的的配置文件
3. iam/template/temp-provider.tf  模版文件，被init.py调用, 生成  iam/provider.tf
4. iam/template/temp-aws-config   模版文件，被init.py调用, 生成 iam/.aws/conf
5. iam/template/temp-aws-credentials 模版文件,被init.py调用, 生成 iam/.aws/credential
6. iam/input-variable.yaml        存储 iam-pipeline-jobs专属的input变量文件
7. iam/main.tf                    terraform主文件
8. iam/variables.tf               terraform文件,定义变量引用
9. iam/output.tf                  terraform文件,定义变量输出
