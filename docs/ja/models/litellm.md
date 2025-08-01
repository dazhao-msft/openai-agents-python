---
search:
  exclude: true
---
# LiteLLM 経由で任意モデルを使用

!!! note

    LiteLLM インテグレーションはベータ版です。特に小規模なモデルプロバイダーでは問題が発生する可能性があります。問題があれば [GitHub Issues](https://github.com/openai/openai-agents-python/issues) で報告してください。迅速に対応いたします。

[LiteLLM](https://docs.litellm.ai/docs/) は、1 つのインターフェースで 100 以上のモデルを扱えるライブラリです。Agents SDK に LiteLLM インテグレーションを追加し、任意の AI モデルを利用できるようにしました。

## セットアップ

`litellm` が利用可能であることを確認してください。オプションの `litellm` 依存グループをインストールすることで対応できます。

```bash
pip install "openai-agents[litellm]"
```

インストールが完了したら、どのエージェントでも [`LitellmModel`][agents.extensions.models.litellm_model.LitellmModel] を使用できます。

## 例

以下は完全に動作するコード例です。実行するとモデル名と API キーの入力を求められます。例として次のように入力できます。

-   モデルに `openai/gpt-4.1`、API キーに OpenAI のキー
-   モデルに `anthropic/claude-3-5-sonnet-20240620`、API キーに Anthropic のキー
-   そのほか

LiteLLM がサポートするモデルの一覧は、[litellm providers docs](https://docs.litellm.ai/docs/providers) を参照してください。

```python
from __future__ import annotations

import asyncio

from agents import Agent, Runner, function_tool, set_tracing_disabled
from agents.extensions.models.litellm_model import LitellmModel

@function_tool
def get_weather(city: str):
    print(f"[debug] getting weather for {city}")
    return f"The weather in {city} is sunny."


async def main(model: str, api_key: str):
    agent = Agent(
        name="Assistant",
        instructions="You only respond in haikus.",
        model=LitellmModel(model=model, api_key=api_key),
        tools=[get_weather],
    )

    result = await Runner.run(agent, "What's the weather in Tokyo?")
    print(result.final_output)


if __name__ == "__main__":
    # First try to get model/api key from args
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument("--model", type=str, required=False)
    parser.add_argument("--api-key", type=str, required=False)
    args = parser.parse_args()

    model = args.model
    if not model:
        model = input("Enter a model name for Litellm: ")

    api_key = args.api_key
    if not api_key:
        api_key = input("Enter an API key for Litellm: ")

    asyncio.run(main(model, api_key))
```