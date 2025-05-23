# Agora REST Client for Go

<p>
<img alt="GitHub License" src="https://img.shields.io/github/license/AgoraIO-Community/agora-rest-client-go">
<a href="https://pkg.go.dev/github.com/AgoraIO-Community/agora-rest-client-go"><img src="https://pkg.go.dev/badge/github.com/AgoraIO-Community/agora-rest-client-go.svg" alt="Go Reference"></a>
<a href="https://github.com/AgoraIO-Community/agora-rest-client-go/actions/workflows/go.yml"><img src="https://github.com/AgoraIO-Community/agora-rest-client-go/actions/workflows/go.yml/badge.svg" alt="Go Actions"></a>
<a href="https://github.com/AgoraIO-Community/agora-rest-client-go/actions/workflows/gitee-sync.yml"><img alt="gitee-sync" src="https://github.com/AgoraIO-Community/agora-rest-client-go/actions/workflows/gitee-sync.yml/badge.svg?branch=main"></a>
<img alt="GitHub go.mod Go version" src="https://img.shields.io/github/go-mod/go-version/AgoraIO-Community/agora-rest-client-go">
<img alt="GitHub" src="https://img.shields.io/github/v/release/AgoraIO-Community/agora-rest-client-go">
<img alt="GitHub Issues or Pull Requests" src="https://img.shields.io/github/issues-pr/AgoraIO-Community/agora-rest-client-go">
</p>

[English](./README.md) | 简体中文

`agora-rest-client-go`是用 Go 语言编写的一个开源项目，专门为 Agora REST API 设计。它包含了 Agora 官方提供的 REST API 接口的包装和内部实现，可以帮助开发者更加方便的集成服务端 Agora REST API。

> [!IMPORTANT]
> 该 SDK 经过一些测试以确保其基本功能正常运作。然而，由于软件开发的复杂性，我们无法保证它是完全没有缺陷的，我们鼓励社区的开发者和用户积极参与，共同改进这个项目。

## 特性

-   封装了 Agora REST API 的请求和响应处理，简化与 Agora REST API 的通信流程
-   当遇到 DNS 解析失败、网络错误或者请求超时等问题的时候，提供了自动切换最佳域名的能力，以保障请求 REST API 服务的可用性
-   提供了易于使用的 API，可轻松地实现调用 Agora REST API 的常见功能，如开启云录制、停止云录制等
-   基于 Go 语言，具有高效性、并发性和可扩展性

## 支持的服务

-   [云端录制 Cloud Recording](./services/cloudrecording/README.md)
-   [云端转码 Cloud Transcoder](./services/cloudtranscoder/README.md)
-   [对话式 AI 引擎 Conversational AI Engine](./services/convoai/README_ZH.md)

## 环境准备

-   [Go 1.18 或以上版本](https://go.dev/)
-   在声网 [Console 平台](https://console.shengwang.cn/)申请的 App ID 和 App Certificate
-   在声网 [Console 平台](https://console.shengwang.cn/)的 Basic Auth 认证信息
-   在声网 [Console 平台](https://console.shengwang.cn/)开启相关的服务能力

## 安装

使用以下命令从 GitHub 安装依赖：

```shell
go get -u github.com/AgoraIO-Community/agora-rest-client-go
```

## 使用示例

以调用云录制服务为例：

```go
package main

import (
	"context"
	"log"
	"time"

	"github.com/AgoraIO-Community/agora-rest-client-go/agora/auth"
	"github.com/AgoraIO-Community/agora-rest-client-go/agora/domain"
	agoraLogger "github.com/AgoraIO-Community/agora-rest-client-go/agora/log"
	"github.com/AgoraIO-Community/agora-rest-client-go/services/cloudrecording"
	cloudRecordingAPI "github.com/AgoraIO-Community/agora-rest-client-go/services/cloudrecording/api"
	"github.com/AgoraIO-Community/agora-rest-client-go/services/cloudrecording/req"
)

const (
	appId    = "<your appId>"
	cname    = "<your cname>"
	uid      = "<your uid>"
	username = "<the username of basic auth credential>"
	password = "<the password of basic auth credential>"
	token    = "<your token>"
)

var storageConfig = &cloudRecordingAPI.StorageConfig{
	Vendor:    0,
	Region:    0,
	Bucket:    "",
	AccessKey: "",
	SecretKey: "",
	FileNamePrefix: []string{
		"recordings",
	},
}

func main() {
	// Initialize Cloud Recording Config
	config := &cloudrecording.Config{
		AppID:      appId,
		Credential: auth.NewBasicAuthCredential(username, password),
		// Specify the region where the server is located. Options include CN, EU, AP, US.
		// The client will automatically switch to use the best domain based on the configured region.
		DomainArea: domain.CN,
		// Specify the log output level. Options include DebugLevel, InfoLevel, WarningLevel, ErrLevel.
		// To disable log output, set logger to DiscardLogger.
		Logger: agoraLogger.NewDefaultLogger(agoraLogger.DebugLevel),
	}

	// Initialize the cloud recording service client
	cloudRecordingClient, err := cloudrecording.NewClient(config)
	if err != nil {
		log.Fatal(err)
	}

	// Call the Acquire API of the cloud recording service client
	acquireResp, err := cloudRecordingClient.MixRecording().
		Acquire(context.TODO(), cname, uid, &req.AcquireMixRecodingClientRequest{})
		// Handle non-business errors
	if err != nil {
		log.Fatal(err)
	}

	// Handle business response
	if acquireResp.IsSuccess() {
		log.Printf("acquire success:%+v\n", acquireResp)
	} else {
		log.Fatalf("acquire failed:%+v\n", acquireResp)
	}

	// Call the Start API of the cloud recording service client
	resourceId := acquireResp.SuccessRes.ResourceId
	startResp, err := cloudRecordingClient.MixRecording().
		Start(context.TODO(), resourceId, cname, uid, &req.StartMixRecordingClientRequest{
			Token: token,
			RecordingConfig: &cloudRecordingAPI.RecordingConfig{
				ChannelType:  1,
				StreamTypes:  2,
				MaxIdleTime:  30,
				AudioProfile: 2,
				TranscodingConfig: &cloudRecordingAPI.TranscodingConfig{
					Width:            640,
					Height:           640,
					FPS:              15,
					BitRate:          800,
					MixedVideoLayout: 0,
					BackgroundColor:  "#000000",
				},
				SubscribeAudioUIDs: []string{
					"#allstream#",
				},
				SubscribeVideoUIDs: []string{
					"#allstream#",
				},
			},
			RecordingFileConfig: &cloudRecordingAPI.RecordingFileConfig{
				AvFileType: []string{
					"hls",
					"mp4",
				},
			},
			StorageConfig: storageConfig,
		})
		// Handle non-business errors
	if err != nil {
		log.Fatal(err)
	}

	// Handle business response
	if startResp.IsSuccess() {
		log.Printf("start success:%+v\n", startResp)
	} else {
		log.Fatalf("start failed:%+v\n", startResp)
	}

	sid := startResp.SuccessResponse.Sid
	// Query
	for i := 0; i < 6; i++ {
		queryResp, err := cloudRecordingClient.MixRecording().
			QueryHLSAndMP4(context.TODO(), resourceId, sid)
			// Handle non-business errors
		if err != nil {
			log.Fatal(err)
		}

		// Handle business response
		if queryResp.IsSuccess() {
			log.Printf("query success:%+v\n", queryResp)
		} else {
			log.Printf("query failed:%+v\n", queryResp)
		}
		time.Sleep(time.Second * 10)
	}

	// Call the Stop API of the cloud recording service client
	stopResp, err := cloudRecordingClient.MixRecording().
		Stop(context.TODO(), resourceId, sid, cname, uid, true)
		// Handle non-business errors
	if err != nil {
		log.Fatal(err)
	}

	// Handle business response
	if stopResp.IsSuccess() {
		log.Printf("stop success:%+v\n", stopResp)
	} else {
		log.Printf("stop failed:%+v\n", stopResp)
	}
}


```

更多的示例可在[Example](./examples) 查看

## 集成遇到困难，该如何联系声网获取协助

> 方案 1：如果您已经在使用声网服务或者在对接中，可以直接联系对接的销售或服务
>
> 方案 2：发送邮件给 [support@agora.io](mailto:support@agora.io) 咨询
>
> 方案 3：扫码加入我们的微信交流群提问
>
> <img src="https://download.agora.io/demo/release/SDHY_QA.jpg" width="360" height="360">

---

## 贡献

本项目欢迎并接受贡献。如果您在使用中遇到问题或有改进建议，请提出 issue 或向我们提交 Pull Request。

# SemVer 版本规范

本项目使用语义化版本号规范 (SemVer) 来管理版本。格式为 MAJOR.MINOR.PATCH。

-   MAJOR 版本号表示不向后兼容的重大更改。
-   MINOR 版本号表示向后兼容的新功能或增强。
-   PATCH 版本号表示向后兼容的错误修复和维护。
    有关详细信息，请参阅 [语义化版本](https://semver.org/lang/zh-CN/) 规范。

## 参考

-   [声网 API 文档](https://doc.shengwang.cn/)

## 许可证

该项目使用 MIT 许可证，详细信息请参阅 LICENSE 文件。
