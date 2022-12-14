# LambdaにDeploy
### 認証情報を入力
`./aws/credentials`にデプロイ権限のあるユーザーのidとシークレットを入力

```toml
[default]
aws_access_key_id = **********
aws_secret_access_key = **********
```

必要な権限は以下
Lambda_FullAccessとかで行けたはずです。
```bash
"iam:CreateRole"
"iam:AttachRolePolicy"
"iam:PassRole"
"iam:UpdateAssumeRolePolicy"
"lambda:GetFunction"
"lambda:CreateFunction"
"lambda:UpdateFunctionCode"
```

ユーザーや認証情報を用意するのが面倒な方は、こちらで一時的に用意したものを利用ください


## Build
以下のコマンドでリリースbuild

```bash
$ cargo lambda build --release
```

## Deploy

以下のコマンドでdeploy

`--iam-role`オプションがない場合、自動で`AWSLambdaBasicExecutionRole`を作成してデプロイしてくれるとのこと

roleを指定するようエラーが出たら、エラーメッセージ内にあるArn（`--iam-role arn:aws:iam::******:role/*********`）か、該当ポリシーを持つroleのArnをコマンドに追加して実行

（私のawsアカウントでは`--iam-role arn:aws:iam::205985056391:role/lambdaBasicExec`）

```bash
$ cargo lambda deploy -r ap-northeast-1
```
# 　
ちなみに、`cargo lambda deploy --help`をで見れるoption一覧はこんな感じ
```bash
Deploy Lambda functions to AWS

Usage: cargo lambda deploy [OPTIONS] [NAME]

Arguments:
  [NAME]  Name of the function or extension to deploy

Options:
  -p, --profile <PROFILE>                          AWS configuration profile to use for authorization
  -r, --region <REGION>                            AWS region to deploy, if there is no default
  -a, --alias <ALIAS>                              AWS Lambda alias to associate the function to
      --retry-attempts <RETRY_ATTEMPTS>            Number of attempts to try failed operations [default: 1]
      --memory <MEMORY>                            Memory allocated for the function
      --enable-function-url                        Enable function URL for this function
      --disable-function-url                       Disable function URL for this function
      --timeout <TIMEOUT>                          How long the function can be running for, in seconds
      --env-var <ENV_VAR>                          Option to add one or many environment variables, allows multiple repetitions Use
                                                   VAR_KEY=VAR_VALUE as format
      --env-file <ENV_FILE>                        Read environment variables from a file
      --tracing <TRACING>                          Tracing mode with X-Ray
      --role <ROLE>                                IAM Role associated with the function
      --layer <LAYER>                              Lambda Layer ARN to associate the deployed function with
  -l, --lambda-dir <LAMBDA_DIR>                    Directory where the lambda binaries are located
      --manifest-path <PATH>                       Path to Cargo.toml [default: Cargo.toml]
      --binary-name <BINARY_NAME>                  Name of the binary to deploy if it doesn't match the name that you want to deploy it with
      --binary-path <BINARY_PATH>                  Local path of the binary to deploy if it doesn't match the target path generated by
                                                   cargo-lambda-build
      --s3-bucket <S3_BUCKET>                      S3 bucket to upload the code to
      --extension                                  Whether the code that you're building is a Lambda Extension
      --compatible-runtimes <COMPATIBLE_RUNTIMES>  Comma separated list with compatible runtimes for the Lambda Extension
                                                   (--compatible_runtimes=provided.al2,nodejs16.x) List of allowed runtimes can be found in
                                                   the AWS documentation:
                                                   https://docs.aws.amazon.com/lambda/latest/dg/API_CreateFunction.html#SSS-CreateFunction-request-Runtime [default:
                                                   provided.al2]
      --output-format <OUTPUT_FORMAT>              Format to render the output (text, or json) [default: Text]
  -h, --help                                       Print help information
```