# AWS-SAMのおける環境変数の定義

1. template.yaml 内で、Parameterを定義することでCFnテンプレートの実行時に該当の変数が利用可能になる。

```yaml
Parameters:
  ENV:
    Type: String
    Description: The deployment environment
    AllowedValues:
      - test
      - production
    Default: test
```

template.yaml内での利用例は以下。

```yaml
FunctionName: !Sub "${ENV}-hello-world-function"
```

2. template.yaml 内のラムダ関数定義に以下を追加することで、Lambdaの実行ランタイム内でLinuxの環境変数として、exportされる

```yaml
Environment:
        Variables:
          ENV: !Ref ENV
```

従って、TypeScriptのコード内で、`process.env` で取得できるようになる。

```typescript
export const lambdaHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
    console.log(`${process.env.ENV}`);

    try {
        return {
            statusCode: 200,
            body: JSON.stringify({
                message: `${process.env.ENV}`,
            }),
        };
```

3. samlconfig内で以下のように定義することで、`--config-env=xxx` で変数の値を切り替えられるようになる。

```toml
[dev.global.parameters]
parameter_overrides = "ENV=dev"

[production.global.parameters]
parameter_overrides = "ENV=production"
```

# How to test

```bash
sam build --use-container
sam local invoke

sam local invoke --config-env=dev
sam local invoke --config-env=production
```



