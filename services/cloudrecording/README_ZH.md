# 云端录制服务

[English](./README.md) | 简体中文

## 服务简介
云端录制是声网为音视频通话和直播研发的录制组件，提供 RESTful API 供开发者实现录制功能，并将录制文件存至第三方云存储。云端录制有稳定可靠、简单易用、成本可控、方案灵活、支持私有化部署等优势，是在线教育、视频会议、金融监管、客户服务场景的理想录制方案。

## 环境准备

- 获取声网App ID -------- [声网Agora - 文档中心 - 如何获取 App ID](https://docs.agora.io/cn/Agora%20Platform/get_appid_token?platform=All%20Platforms#%E8%8E%B7%E5%8F%96-app-id)

  > - 点击创建应用
  >
  >   ![](../../assets/imges/CN/create_app_1.png)
  >
  > - 选择你要创建的应用类型
  >
  >   ![](../../assets/imges/CN/create_app_2.png)

- 获取App 证书 ----- [声网Agora - 文档中心 - 获取 App 证书](https://docs.agora.io/cn/Agora%20Platform/get_appid_token?platform=All%20Platforms#%E8%8E%B7%E5%8F%96-app-%E8%AF%81%E4%B9%A6)

  > 在声网控制台的项目管理页面，找到你的项目，点击配置。
  > ![](../../assets/imges/CN/config_app.png)
  > 点击主要证书下面的复制图标，即可获取项目的 App 证书。
  > ![](../../assets/imges/CN/copy_app_cert.png)

- 开启云录制服务
  > ![](../../assets/imges/CN/config_app.png)
  > ![](../../assets/imges/CN/config_service.png)  
  > ![](../../assets/imges/CN/open_cloud_recording.png)

## API 接口调用示例
### 获取云端录制资源
> 在开始云端录制之前，你需要调用 acquire 方法获取一个 Resource ID。一个 Resource ID 只能用于一次云端录制服务。

需要设置的参数有：
- appId: 声网的项目 AppID
- username: 声网的Basic Auth认证的用户名
- password: 声网的Basic Auth认证的密码
- cname: 频道名
- uid: 用户 UID
- 更多 clientRequest中的参数见[Acquire](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/post-v1-apps-appid-cloud_recording-acquire)接口文档

通过调用`Acquire`方法来实现获取云端录制资源
```go
    appId := "xxxx"
    username := "xxxx"
    password := "xxxx"
    credential := auth.NewBasicAuthCredential(username, password)
	config := &agora.Config{
		AppID:      appId,
		Credential: credential,
        DomainArea: domain.CN,
		Logger:     agoraLogger.NewDefaultLogger(agoraLogger.DebugLevel),
	}

	cloudRecordingClient, err := cloudrecording.NewClient(config)
	if err != nil {
		log.Println(err)
		return
	}

	resp, err := cloudRecordingClient.Acquire(context.TODO(), &cloudRecordingAPI.AcquireReqBody{
		Cname: "12321",
		Uid:   "43434",
		ClientRequest: &cloudRecordingAPI.AcquireClientRequest{
			Scene:               0,
		},
	})
	if err != nil {
		log.Fatal(err)
	}
	if resp.IsSuccess() {
		log.Printf("resourceId:%s", resp.SuccessRes.ResourceId)
	} else {
		log.Printf("resp:%+v", resp)
	}
```

### 开始云端录制
> 通过 acquire 方法获取云端录制资源后，调用 start 方法开始云端录制。

需要设置的参数有：
- cname: 频道名
- uid: 用户 UID
- token：用户 UID 对应的token
- resourceId: 云端录制资源ID
- mode: 云端录制模式
- 更多 clientRequest中的参数见[Start](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/post-v1-apps-appid-cloud_recording-resourceid-resourceid-mode-mode-start)接口文档

通过调用`Start`方法来实现开始云端录制
```go
	startResp, err := cloudRecordingClient.Start(ctx, resp.SuccessRes.ResourceId, mode, &cloudRecordingAPI.StartReqBody{
		Cname: cname,
		Uid:   uid,
		ClientRequest: &cloudRecordingAPI.StartClientRequest{
			Token: token,
			RecordingConfig: &cloudRecordingAPI.RecordingConfig{
				ChannelType:  1,
				StreamTypes:  2,
				AudioProfile: 2,
				MaxIdleTime:  30,
				TranscodingConfig: &cloudRecordingAPI.TranscodingConfig{
					Width:            640,
					Height:           260,
					FPS:              15,
					BitRate:          500,
					MixedVideoLayout: 0,
					BackgroundColor:  "#000000",
				},
				SubscribeAudioUIDs: []string{
					"22",
					"456",
				},
				SubscribeVideoUIDs: []string{
					"22",
					"456",
				},
			},
			RecordingFileConfig: &cloudRecordingAPI.RecordingFileConfig{
				AvFileType: []string{
					"hls",
				},
			},
			StorageConfig: &cloudRecordingAPI.StorageConfig{
				Vendor:    2,
				Region:    3,
				Bucket:    "xxx",
				AccessKey: "xxxx",
				SecretKey: "xxx",
				FileNamePrefix: []string{
					"xx1",
					"xx2",
				},
			},
		},
	})
	if err != nil {
		log.Fatalln(err)
	}
	if startResp.IsSuccess() {
		log.Printf("success:%+v", &startResp.SuccessResponse)
	} else {
		log.Printf("failed:%+v", &startResp.ErrResponse)
	}
```

### 停止云端录制
> 开始录制后，你可以调用 stop 方法离开频道，停止录制。录制停止后如需再次录制，必须再调用 acquire 方法请求一个新的 Resource ID。

需要设置的参数有：
- cname: 频道名
- uid: 用户ID
- resourceId: 云端录制资源ID
- sid: 会话ID
- mode: 云端录制模式
- 更多 clientRequest中的参数见[Stop](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/post-v1-apps-appid-cloud_recording-resourceid-resourceid-sid-sid-mode-mode-stop)接口文档

因为Stop 接口返回的不是一个固定的结构体，所以需要根据返回的serverResponseMode来判断具体的返回类型

通过调用`Stop`方法来实现停止云端录制
```go
    stopResp, err := cloudRecordingClient.Stop(ctx, resourceId, sid, mode, &cloudRecordingAPI.cloudRecordingAPI{
		Cname: cname,
		Uid:   uid,
		ClientRequest: &cloudRecordingAPI.StopClientRequest{
			AsyncStop: true,
		},
	})
	if err != nil {
		log.Fatalln(err)
	}
	if stopResp.IsSuccess() {
		log.Printf("stop success:%+v", &stopResp.SuccessResponse)
	} else {
		log.Fatalf("stop failed:%+v", &stopResp.ErrResponse)
	}
```

### 查询云端录制状态
> 开始录制后，你可以调用 query 方法查询录制状态。

需要设置的参数有：
- cname: 频道名
- uid: 用户ID
- resourceId: 云端录制资源ID
- sid: 会话ID
- mode: 云端录制模式
- 更多 clientRequest中的参数见[Query](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/get-v1-apps-appid-cloud_recording-resourceid-resourceid-sid-sid-mode-mode-query)接口文档

因为 Query 接口返回的不是一个固定的结构体，所以需要根据返回的serverResponseMode来判断具体的返回类型

通过调用`Query`方法来实现查询云端录制状态
```go
	queryResp, err := cloudRecordingClient.Query(ctx, resourceId, sid, mode)
	if err != nil {
		log.Fatalln(err)
	}
	if queryResp.IsSuccess() {
		log.Printf("query success:%+v", queryResp.SuccessResponse)
	} else {
		log.Printf("query failed:%+v", queryResp.ErrResponse)
		return
	}

	var queryServerResponse interface{}

	querySuccess := queryResp.SuccessResponse
	switch querySuccess.GetServerResponseMode() {
	case cloudRecordingAPI.QueryServerResponseUnknownMode:
		log.Fatalln("unknown mode")
	case cloudRecordingAPI.QueryIndividualRecordingServerResponseMode:
		log.Printf("serverResponseMode:%d", cloudRecordingAPI.QueryIndividualRecordingServerResponseMode)
		queryServerResponse = querySuccess.GetIndividualRecordingServerResponse()
	case cloudRecordingAPI.QueryIndividualVideoScreenshotServerResponseMode:
		log.Printf("serverResponseMode:%d", cloudRecordingAPI.QueryIndividualVideoScreenshotServerResponseMode)
		queryServerResponse = querySuccess.GetIndividualVideoScreenshotServerResponse()
	case cloudRecordingAPI.QueryMixRecordingHlsServerResponseMode:
		log.Printf("serverResponseMode:%d", cloudRecordingAPI.QueryMixRecordingHlsServerResponseMode)
		queryServerResponse = querySuccess.GetMixRecordingHLSServerResponse()
	case cloudRecordingAPI.QueryMixRecordingHlsAndMp4ServerResponseMode:
		log.Printf("serverResponseMode:%d", cloudRecordingAPI.QueryMixRecordingHlsAndMp4ServerResponseMode)
		queryServerResponse = querySuccess.GetMixRecordingHLSAndMP4ServerResponse()
	case cloudRecordingAPI.QueryWebRecordingServerResponseMode:
		log.Printf("serverResponseMode:%d", cloudRecordingAPI.QueryWebRecordingServerResponseMode)
		queryServerResponse = querySuccess.GetWebRecording2CDNServerResponse()
	}

	log.Printf("queryServerResponse:%+v", queryServerResponse)
```

### 更新云端录制设置
> 开始录制后，你可以调用 update 方法更新如下录制配置：
> * 对单流录制和合流录制，更新订阅名单。
> * 对页面录制，设置暂停/恢复页面录制，或更新页面录制转推到 CDN 的推流地址（URL）。

需要设置的参数有：
- cname: 频道名
- uid: 用户 UID
- resourceId: 云端录制资源ID
- sid: 会话ID
- mode: 云端录制模式
- 更多 clientRequest中的参数见[Update](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/post-v1-apps-appid-cloud_recording-resourceid-resourceid-sid-sid-mode-mode-update)接口文档

通过调用`Update`方法来实现更新云端录制设置
```go
	updateResp, err := cloudRecordingClient.Update(ctx, resourceId, sid, mode, &cloudRecordingAPI.UpdateReqBody{
		Cname: cname,
		Uid:   uid,
		ClientRequest: &cloudRecordingAPI.UpdateClientRequest{
			StreamSubscribe: &cloudRecordingAPI.UpdateStreamSubscribe{
				AudioUidList: &cloudRecordingAPI.UpdateAudioUIDList{
					SubscribeAudioUIDs: []string{
						"999",
					},
				},
				VideoUidList: &cloudRecordingAPI.UpdateVideoUIDList{
					SubscribeVideoUIDs: []string{
						"999",
					},
				},
			},
		},
	})

	if err != nil {
		log.Fatalln(err)
	}
	if updateResp.IsSuccess() {
		log.Printf("update success:%+v", updateResp.SuccessResponse)
	} else {
		log.Printf("update failed:%+v", updateResp.ErrResponse)
		return
	}
```

### 更新云端录制合流布局
> 开始录制后，你可以调用 updateLayout 方法更新合流布局。
> 每次调用该方法都会覆盖原来的布局设置。例如，在开始录制时设置了 backgroundColor 为 "#FF0000"（红色），如果调用 updateLayout 方法更新合流布局时如果不再设置 backgroundColor 字段，背景色就会变为黑色（默认值）。

需要设置的参数有：
- cname: 频道名
- uid: 用户 UID
- resourceId: 云端录制资源ID
- sid: 会话ID
- mode: 云端录制模式
- 更多 clientRequest中的参数见[UpdateLayout](https://doc.shengwang.cn/doc/cloud-recording/restful/cloud-recording/operations/post-v1-apps-appid-cloud_recording-resourceid-resourceid-sid-sid-mode-mode-updateLayout)接口文档

通过调用`UpdateLayout`方法来实现更新云端录制合流布局
```go
	updateLayoutResp, err := cloudRecordingClient.UpdateLayout(ctx, resourceId, sid, mode, &cloudRecordingAPI.UpdateLayoutReqBody{
		Cname: cname,
		Uid:   uid,
		ClientRequest: &cloudRecordingAPI.UpdateLayoutClientRequest{
			MixedVideoLayout: 3,
			BackgroundColor:  "#FF0000",
			LayoutConfig: []cloudRecordingAPI.UpdateLayoutConfig{
				{
					UID:        "22",
					XAxis:      0.1,
					YAxis:      0.1,
					Width:      0.1,
					Height:     0.1,
					Alpha:      1,
					RenderMode: 1,
				},
				{
					UID:        "2",
					XAxis:      0.2,
					YAxis:      0.2,
					Width:      0.1,
					Height:     0.1,
					Alpha:      1,
					RenderMode: 1,
				},
			},
		},
	})
	if err != nil {
		log.Fatalln(err)
	}
	if updateLayoutResp.IsSuccess() {
		log.Printf("updateLayout success:%+v", updateLayoutResp.SuccessResponse)
	} else {
		log.Printf("updateLayout failed:%+v", updateLayoutResp.ErrResponse)
		return
	}
```

## 错误码和响应状态码处理
具体的业务响应码请参考[业务响应码](https://doc.shengwang.cn/doc/cloud-recording/restful/response-code)文档