# GO-OPENAI-PROXY

基于 Go 实现 OpenAI API 调用代理
支持实时Streaming模式

### 编译打包：

```bash
./build.sh
```

### 部署测试
使用openai golang sdk进行流模式测试：

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"github.com/sashabaranov/go-openai"
	"io"
)

func main() {
	openaiAPIURLv1 := "your proxy server"
	openaiKey := "api key"

	config := openai.DefaultConfig(openaiKey)
	config.BaseURL = openaiAPIURLv1

	c := openai.NewClientWithConfig(config)
	ctx := context.Background()

	req := openai.ChatCompletionRequest{
		Model:     openai.GPT3Dot5Turbo,
		MaxTokens: 3000,
		Messages: []openai.ChatCompletionMessage{
			{
				Role:    openai.ChatMessageRoleUser,
				Content: "write a typescript code to implement a resume viewer based on react and antd",
			},
		},
		Stream: true,
	}
	stream, err := c.CreateChatCompletionStream(ctx, req)
	if err != nil {
		fmt.Printf("ChatCompletionStream error: %v\n", err)
		return
	}
	defer stream.Close()

	fmt.Printf("Stream response: ")
	for {
		response, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			fmt.Println("\nStream finished")
			return
		}

		if err != nil {
			fmt.Printf("\nStream error: %v\n", err)
			return
		}

		fmt.Printf(response.Choices[0].Delta.Content)
	}
}
```