---
layout: post
title:  "떡볶이맵 제작기 #2 - 에러 모니터링"
subtitle: "홈서버가 죽었을때 슬랙으로 모니터링하기"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - golang
    - slack
    - monitoring
    - serverless
date:   2021-09-26
multilingual: flase
---

# 문제점
떡볶이맵의 백엔드 서버는 홈서버에서 운영되고 있기 때문에 해당 서버의 헬스체크는 홈서버 내에서 이루어질 수 없었고, 이로 인해 정전 등의 이유로 컴퓨터가 꺼지는 등의 장애 상황을 신속하게 파악하기 힘들었습니다.

# 해결방안
이러한 문제점을 해결하기 위해 외부에서 홈서버의 건강상태를 체크해주는 API를 만들어야 했습니다.
떡볶이맵 서버와 상관없는 홈서버 자체의 API를 만들어서 헬스체크를 할 수도 있었지만, 좀더 다양한 상황에 유연하게 대처가 가능하도록 떡볶이맵 클라이언트에서 api 요청시에 실패하는 경우에 모니터링을 받는 식으로 구축을 하고 싶었습니다.

두가지 방식이 생각이 났는데, sentry를 frontend에 연동하는 방법이 있고, 다른 하나는 api 요청 실패시 serverless api를 통해 슬랙으로 노티를 받는 방법이 있었습니다.

## sentry vs slack
sentry를 사용하는건 간단하지만 실시간으로 noti를 받으려면  team plan이상을 사용해야 했습니다.
반면에 slack은 api를 무료로 지원하고있고, 마크다운 양식을 지원하여 원하는 형태로 가공하여 메시지를 받아볼 수 있어서 유지보수에 용이하다고 판단되었습니다.

<figure>
<img src="https://user-images.githubusercontent.com/34048253/134799434-89184247-bb34-454c-ac85-a9cffcda5c2f.png" />
<figcaption>Third party integrations 기능은 free plan에서는 사용이 불가능하다...</figcaption>
</figure>

## aws lambda
slack api를 frontend에서 직접 요청하는것은 api token의 노출 등의 위험이 있기 때문에 클라이언트에서는 aws lambda로 api를 호출하고, lambda에서 slack으로 알림을 전송하는 식으로 구현하고자 했습니다.

홈서버에 장애가 발생한 상황에서도, aws lambda를 통해 slack으로 장애관련 noti를 받을 수 있게 됩니다.
<img src="https://user-images.githubusercontent.com/34048253/134799280-8708fa31-c456-48e6-b9d2-9d7ccd7845ec.png" width=600 />

## 코드 작성
알림을 받을 채널이 당장은 slack밖에 없었지만, 인터페이스를 분리하여 추후에 메일 등을 통해서도 알림을 받을 수 있도록 처리했습니다.

### client
클라이언트는 에러객체에 담겨있는 정보를 서버로 전송합니다.
이를위해 axios post와 axios get을 한번 감싸주었습니다.

```typescript
const parseAxiosError = (error: AxiosError): any => {
  return {
    message: error.message,
    name: error.name,
    stack: error.stack,
    config: error.config,
    code: error.code,
    ...(error.request && { request: error.request }),
    ...(error.response?.status && { responseStatus: error.response?.status }),
    ...(error.response?.data && { responseData: error.response?.data }),
    raw: error.toJSON(),
  };
};

export async function post<T>(
  url: string,
  data?: T,
  config?: AxiosRequestConfig,
): Promise<AxiosResponse> {
  return axios.post(url, data, { ...config, timeout }).catch((error: AxiosError) => {
    return axios.post(env.api.errorHelper, {
      serviceName,
      types: notificationTypes,
      description: JSON.stringify(parseAxiosError(error)),
    });
  });
}
```

### noti api
golang으로 작성한 api는 클라이언트로부터 받은 메시지를 다양한 채널로 전송합니다.
```golang
type Slack struct{}

func (c *Slack) Send(message string, description string) {
	var jsonStr = []byte(fmt.Sprintf("{'text': '%s" + "\n>```%s```'}", message, description))
	fmt.Println(bytes.NewBuffer(jsonStr))
	fmt.Println(os.Getenv("SLACK_WEBHOOK_URL"))
	res, err := http.Post(os.Getenv("SLACK_WEBHOOK_URL"), "application/json", bytes.NewBuffer(jsonStr))
	if err != nil {
		fmt.Println(err)
	} else {
		if res.StatusCode != 200 {
			fmt.Println(fmt.Sprintf("[%s] 에러 메시지 전송에 실패했습니다: %s", strconv.Itoa(res.StatusCode), err))
			fmt.Println(fmt.Sprintf("%s", res.Body))
		} else {
			fmt.Println(fmt.Sprintf("[%s] 에러 메시지가 전송되었습니다: %s", strconv.Itoa(res.StatusCode), message))
		}
	}
}

type Body struct {
	ServiceName string   `json:"serviceName"`
	Types       []string `json:"types"`
	Description string   `json:"description"`
}

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	loadEnv(".env")
	body := Body{}
	err := json.Unmarshal([]byte(string(request.Body)), &body)
	if err != nil {
		return events.APIGatewayProxyResponse{StatusCode: 400, Body: "json parse error"}, err
	}
	clients := GetClients(body.Types)
	message := fmt.Sprintf("[%s] 문제가 발생했습니다", body.ServiceName)
	description := fmt.Sprintf("%s", body.Description)
	for _, c := range clients {
		c.Send(message, description)
	}
	return events.APIGatewayProxyResponse{StatusCode: 200, Body: "OK"}, nil
}
```

[serverless-error-helper](https://github.com/siner308/serverless-error-helper)에서 전체 코드를 확인 할 수 있습니다.

# 결과
timeout을 1ms로 수정하여 테스트해본 결과 아래 스크린샷처럼 슬랙을 통해 에러메시지와 당시 상황을 자세하게 확인할 수 있게 되었습니다.
![스크린샷 2021-09-26 오후 5 20 22](https://user-images.githubusercontent.com/34048253/134799787-d7fe9d54-361d-4803-9aa3-6064ca736e8b.png)

