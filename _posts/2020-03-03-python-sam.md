---
layout: post
title:  "AWS에서 SAM 서비스를 이용해 파이썬 서버리스 프로젝트를 만드는 방법"
author: jeffrey
image: assets/images/thumbnail/art-background-black-633409.jpg
featured: true
excerpt: SAM cli 도구를 이용하여  CloudFormation으로 배포해 스택을 생성하는 방법에 대한 포스팅
---

## 개요

AWS의 SAM 문서를 살펴보면 다음과 같은 문장이 있다.

> AWS SAM templates are an extension of AWS CloudFormation templates. That is, any resource that you can declare in an AWS CloudFormation template you can also declare in an AWS SAM template.

즉, SAM, Serverless Application Model은 AWS의 CloudFormation의 확장 기능인 셈이다. 원래 CloudFormation 서비스가 IaC를 위해 나온 서비스임을 생각하면, SAM은 그 중에서도 서버리스 환경에 집중해서 기능을 간소화한 서비스이다.

이 서비스를 이용하는 방법은 크게 두가지가 있다. 
하나는 AWS에서 운영하는 SAR(Serverless Application Repository)라는 서버리스 앱 배포 서버를 이용하는 것이다. 여기에서 다른 사람이 만든 서버리스 앱을 가져와서 쓸 수도 있고, 내가 만든 서버리스 앱을 배포할 수도 있다. 그리고 private application으로 관리하면 접근권한을 조절해서 팀원들간에만 관리하도록 할 수도 있다. 
다른 하나는 SAM cli 도구를 이용해서 직접 CloudFormation으로 배포해 스택을 생성하는 것이다. 이 글에서는 두번째 방법인 cli를 이용한 배포에 대해 다룰 것이다.

이 서비스는 서버리스 앱에 대한 기반을 정의하는 문서가 필요하다. yaml, json 두가지 포맷을 지원하며 기본 이름은 template.yaml 혹은 template.json이다.

만약 SAM에 대한 메뉴얼 문서를 보고싶다면, [AWS의 공식 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)와 [AWS SAM github페이지](https://github.com/awslabs/serverless-application-model)를 참고하는 것이 좋다.

## 설치

SAM 서비스를 사용하기 위해서는  Pypi 서버에 aws-sam-cli라는 패키지가 있으니 간단히 pip 도구를 이용해 설치할 수 있다.

```bash
pip install --user aws-sam-cli
```

이렇게 설치하면 터미널 환경에서 다음과 같은 명령어를 이용해 설치된 버전을 확인할 수 있다.

```bash
sam --version
SAM CLI, version 0.41.0
```

그리고 `sam --help`라는 명령어를 입력하면 사용 가능한 명령어 목록을 볼 수 있다. 또한 각 명령어에 --help 옵션을 붙이면 명령어에 대한 사용법도 확인이 가능하다.

## 사용 전 준비

SAM을 사용하기 위해서는 배포하려는 사용자의 계정 토큰과 몇가지 IAM 권한이 필요하다.
계정 토큰은 ~/.aws/config에 ini 포맷으로 저장해두면 되고, awscli의 config 명령으로도 넣을 수 있다.
다음은 이 사용자 계정 토큰의 IAM권한들이 필요하다.

* S3 조작에 대한 권한

    Lambda에 업로드할 코드 파일을 저장할 S3 공간에 대한 파일 업로드 권한이 필요하다.
    > "s3:PutObject"

* CloudFormation 구성에 대한 권한

    CloudFormation에 스택을 구성하고, 변경하기 때문에 필요한 권한이다.

    > "cloudformation:DescribeStackEvents"
    > "cloudformation:DescribeStackSet"
    > "cloudformation:CreateStack"
    > "cloudformation:GetTemplate"
    > "cloudformation:UpdateStack"
    > "cloudformation:DescribeChangeSet"
    > "cloudformation:ExecuteChangeSet"
    > "cloudformation:CancelUpdateStack"
    > "cloudformation:CreateChangeSet"
    > "cloudformation:DescribeStacks"
    > "cloudformation:ContinueUpdateRollback"

* IAM 권한

    각 스택에 필요한 권한을 확인하고, 추가하고, 제거하기 위한 권한이다.

    > "iam:ListPolicies",
    > "iam:DetachRolePolicy",
    > "iam:DeleteRolePolicy",
    > "iam:CreateRole",
    > "iam:DeleteRole",
    > "iam:AttachRolePolicy",
    > "iam:PutRolePolicy"

이외에도 만들려는 각 AWS 서비스의 생성 권한이 필요하다. (예를 들어 람다가 있다면 람다 생성, 람다 람다 권한, 람다 레이어 등등의 관련 권한이 필요하다.)

## 프로젝트 작성

SAM 프로젝트는 직접 파이썬 프로젝트 폴더를 만들어서 나중에 template.yaml파일을 추가해도 되고, SAM cli의 명령을 이용해도 된다.

```bash
sam init --name "PROJECT_NAME" --runtime python3.7 # 파이썬은 2.7, 3.6, 3.7, 3.8 버전을 지원한다.
```

이 명령어를 입력하면 간단하게 아래 폴더트리가 생성된다.

```bash
.
+-- template.yaml # 어플리케이션 구성을 정의하는 문서
+-- event.json # 로컬환경에서 람다를 테스트할 때 event 파라미터로 입력되는 값
+-- hello_world # 프로그램 코드
|   +-- __init__.py
|   +-- app.py # 람다 핸들러 함수가 들어있는 파일
+-- tests
|   +-- unit
|   |   +-- __init__.py
|   |   +-- test_handler.py # 람다 핸들러 함수를 테스트하기위한 유닛테스트 코드
```

이제 여기에 template.yaml 파일을 수정하면 된다.

### template.yaml

템플릿 파일은 AWS에 구축될 서버리스 앱의 구조를 작성하는 문서이다. yaml과 json 포맷을 지원하지만 json보다 작성이 편하기 때문에 yaml로 작성하는것을 추천한다.
구조를 전부 설명하기엔 너무 길어지고 불필요한 내용도 많기 때문에 아래 문서들을 참고하면서 작성하면 그럭저럭 작성할 수 있다.

* github의 template 스펙 문서: [https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md)

* AWS의 SAM 문서: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html)

* AWS의 CloudFormation 문서: [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)

* SAM cli 릴리즈 노트: [https://github.com/awslabs/aws-sam-cli/releases](https://github.com/awslabs/aws-sam-cli/releases)

CloudFormation문서를 함께 봐야 하는 이유는 SAM 서비스가 CloudFormation에서 왔기 때문에 SAM 문서에는 없지만 지원하는 서비스가 있기 때문이다.

## 동작

SAM 프로젝트는 보통 다음 절차로 배포가 된다

(코드검사) -> 빌드 -> 패키징 -> 배포

여기서 코드검사는 SAM cli의 기능이 아직 완전하지 않아서 template 파일에 있는 오류를 다 잡아주지 못한다. 배포 도중에 에러가 나서 잡아야 하는 경우도 많아지니 현재는 생략해도 관계 없는 단계이다.(0.41.0 기준)

### 빌드

빌드는 build 명령어로 실행하는데, 아마존 리눅스의 환경에 맞춰서 패키지를 빌드한다. 이 때 패키지 대상 폴더에 있는 requirements.txt 파일에 있는 패키지들을 함께 다운로드하도록 되어있다.

명령어의 여러 옵션이 있지만 대개 이정도 옵션이면 충분하다. 나머지는 [AWS의 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html) 참고

```bash
sam build \
--profile "PROFILE_NAME" \
--template "TEMPLATE_FILE" \
--build-dir "BUILD_PATH" \
--parameter-overrides ParamKey1=ParamValue1,ParamKey2=ParamValue2
```

### 패키징

빌드 후 에 빌드폴더에 template.yaml파일이 함께 복사되는데, 그 파일을 CloudFormation에서 사용되는 template 파일 포맷으로 변환된다. 물론 완전히 CloudFormation에서 쓰이는 서비스 코드로 바뀌는 것은 아니다.

이렇게 변환 후에 코드를 압축해서 S3 버킷에 업로드를 한다. 이후에 할 배포 작업은 여기에 업로드한 압축된 코드들을 배포하게된다.

최소한 이정도 옵션은 사용해야 한다. 그 외에 명령어 설명은 [AWS의 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-package.html)를 참고하면 된다.

```bash
sam package \
--profile "PROFILE_NAME" \
--template-file "BUILD_PATH_TEMPLATE" \
--output-template-file "PACKAGED_TEMPLATE_PATH \
--s3-bucket "UPLOAD_S3_BUCKET"
```

### 배포

패키징이 끝난 프로젝트를 배포할 때 이 명령어를 사용한다. 여기에서 배포되는 과정은 CloudFormation 대시보드에서도 확인할 수 있으며 터미널에도 과정이 로그형태로 출력된다. 이 작업이 끝나고나면 람다를 포함한 모든 서비스가 만들어진 뒤에 스택이라는 묶음으로 관리된다.

자세한 배포 명령어에 대한 [AWS 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)

```bash
sam deploy \
--profile "PROFILE_NAME" \
--template-file "PACKAGED_TEMPLATE_FILE_PATH" \
--capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND --stack-name "STACK_NAME" \
--parameter-overrides ParamKey1=ParamValue1,ParamKey2=ParamValue2
```

## 관리

배포 이후에 프로젝트는 알아서 잘 돌겠지만, 업데이트도 해야하고 다양하게 관리를 해야한다. 코드가 수정되거나 스택 구조가 변경되면 소스코드와 template 파일들을 변경한 뒤에 위에있는 명령어를 동일하게 실행하면 CloudFormation이 알아서 잘 업데이트 해준다. 스택 구조도 깔끔하게 변경되니 손쉽게 기능을 추가하거나 제거할 수 있는데, S3나 DB같은 경우는 잘못하면 데이터가 같이 날라가버릴 수 있으니 조심해야 한다.

코드 최초 배포는 물론이고 업데이트 전에 테스트를 해야 하는데, 이 때 람다 환경에서 동작하는 결과를 확인하고 싶다면 SAM cli 명령어 안에 Docker를 이용해서 로컬에서 테스트를 실행할 수 있다.

```bash
sam local invoke \
--profile "PROFILE_NAME" \
--template "TEMPLATE_FILE" \
--event "TEST_INPUT(예: event.json)" \
--env-vars "TEST_ENV_FILE" \
--log-file "TEST_RESULT_LOG" \
--debug \
"${TEST_TARGET}"
```

여기서 --event에는 json 포맷으로 된 값을 넣으면 lambda handler 함수에서 event 파라미터로 값이 전달된다.

또한 --env_vars도 마찬가지로 json 포맷으로 된 환경변수 값을 넣으면 실행될 때 전달된다.

```json
{
    "LambdaFunction": {
        "ENV1": "VALUE1",
        "ENV2": "VALUE2",
        "ENV3": "VALUE3",
        "ENV4": "VALUE4",
    }
}
```

이렇게 실행하면 결과가 CloudWatch의 로그 스트림과 같은 포맷으로 결과가 파일로 저장된다.

## 결론

SAM이 개발되기 전에도 IaC를 위한 도구가 많았다. 거기다 요즘처럼 멀티 클라우드 환경까지도 구축, 운영되는 환경에서는 SAM을 쓸 수 없는 상황도 있다.

그러나 AWS 서비스에서 서버리스 환경을 구축한다면 SAM으로 개발하는게 가장 간편하다고 생각된다. 시스템 스펙을 기술하는 template 파일의 구조도 단순하고 AWS 서비스의 이름을 사용하고있기 때문에 찾기도 쉽기 때문이다. 아직 SAM cli는 정식버전이 없고 계속 업데이트가 되고있으니 사용하기 전에 한번쯤은 릴리즈 노트를 참고해서 어떤 기능이 추가되었는지 확인하는것이 좋을 것 같다.
