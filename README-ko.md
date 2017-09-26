# OpenWhisk - 첫 번째 액션, 트리거 그리고 룰

*다른 언어로 보기: [English](README.md).*

OpenWhisk 액션, 트리거 및 룰에 대한 단순 데모로서, [Build a cloud native app with Apache OpenWhisk webinar](https://developer.ibm.com/tv/build-a-cloud-native-app-with-apache-openwhisk/)용으로 생성되었습니다. 먼저, [Bluemix 계정과 최신 버젼의 OpenWhisk 명령행 도구를 설치해야 합니다.](/docs/OPENWHISK-ko.md)

![High level diagram](demo-1.png)

## 액션: `handler.js`

OpenWhisk 에서 _action_ 으로 불리는 이 JavaScript 함수는 `params`를 매개변수로 받고 OpenWhisk 로그나 IBM Bluemix 모니터링 콘솔에서 받을 수 있는 정보를 출력합니다. 또한 현재 날짜를 JSON 객체로 리턴합니다.

먼저, 터미널 윈도를 열어 OpenWhisk 활성화 로그에 대한 폴링을 시작합니다. 액션에서 `console.log` 구문은 여기에 로그를 남기고 다음 명령으로 스티리밍 할 수 있습니다:

```bash
wsk activation poll
```

또 다른 터미널 윈도에서, 다음 명령으로 OpenWhisk에 파일을 업로드 합니다:

```bash
wsk action create handler handler.js
```

이제 이를 수동으로 호출하십시오. 그러면 JSON 메시지를 현재 윈도에 나타내고 다른 윈도에 활성화가 로깅됩니다.

```bash
wsk action invoke --blocking handler
```

## 트리거: `every-20-seconds`

이 트리거는 내장 알람 패키지를 사용하며, 생성될 때 `cron` 매개변수를 통해 cron 구문으로 지정되어 매 20초 마다 이벤트를 발생합니다. `maxTriggers` 매개 변수는 기본 값인 10,000회 호출보다 5분동안 최대 15회까지 발생하도록 제한합니다.

다음 명령으로 트리거를 생성합니다:

```bash
wsk trigger create every-20-seconds \
    --feed  /whisk.system/alarms/alarm \
    --param cron '*/20 * * * * *' \
    --param maxTriggers 15
```

## 룰: `invoke-periodically`

이 룰은 `every-20-seconds` 트리거가 `handler.js` 액션에 어떻게 선언적으로 매핑되는지 보여줍니다. 이름이 추상적으로 지어져 있기에 매 분마다 호출되는것 같은 다른 트리거를 사용하고자 할 때 여전히 논리적인 이름을 사용할 수 있다는 것을 주목해 주십시오.

다음 명령으로 룰을 생성합니다:

```bash
wsk rule create \
    invoke-periodically \
    every-20-seconds \
    handler
```

이 시점에서 다른 윈도에서 활성화 로그를 확인하여 해당 액션이 트리거에 의해 잘 실행되고 있는지 확인 해 볼 수 있습니다.

## 설치 지침

OpenWhisk 개발자는 개발하는 애플리케이션에 대해 반복되는 액션, 트리거 그리고 룰의 생성을 종종 자동화 합니다. 많은 예제 앱에서 관습적으로 `deploy.sh` 스크립트를 사용하고, `local.env` 파일과 함께 사용하여 환경 변수를 외부화합니다.

스크립트로 현재 구성에 대한 설정, 삭제 및 확인을 할 수 있습니다:

```bash
./deploy.sh --install
./deploy.sh --uninstall
./deploy.sh --env # Not used in this demo
```

> **참고**: `deploy.sh`은 향후 [`wskdeploy`](https://github.com/openwhisk/openwhisk-wskdeploy)로 교체될 예정입니다. `wskdeploy`는 선언된 트리거, 액션 및 규칙을 OpenWhisk에 배포하기 위해 manifest를 사용합니다.

## 문제 해결
가장 먼제 OpenWhisk 활성화 로그에서 오류를 확인 하십시오. 명령창에서 `wsk activation poll`을 이용하여 로그 메시지를 확인하거나 [Bluemix의 모니터링 콘솔](https://console.ng.bluemix.net/openwhisk/dashboard)에서 시각적으로 상세정보를 확인해 보십시오.

오류가 즉각적으로 분명하지 않다면, [최신 버젼의 `wsk` CLI](https://console.ng.bluemix.net/openwhisk/learn/cli)가 설치되어 있는지 확인하십시오. 만약 이전 것이라면 다운로드하고 업데이트 하십시오.
```bash
wsk property get --cliversion
```

## 라이센스
[Apache 2.0](LICENSE.txt)
