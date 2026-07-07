##Introduction

In this image, qwen2.5 gets a value in the rocm-smi and transfers to the llama3.3
and llama3.3 gets the value and transfers back to the qwen2.5. I think this image may run well.

# robot_dialogue_relay — 빌드 & 사용법

이 샌드박스에는 Docker/GPU가 없어서 이미지를 직접 빌드·검증하지 못했습니다.
아래 순서대로 선생님 환경(호스트 또는 radeon_robot_lab 컨테이너)에서 진행해주세요.

## 1. 빌드 컨텍스트 옮기기

압축을 풀면 `relay_docker/` 폴더 안에 `Dockerfile`, `gpu_telemetry.py`,
`grounding_check.py`, `relay_dialogue.py`가 있습니다.

```bash
tar -xzf relay_docker.tar.gz
cd relay_docker
```

## 2. 이미지 빌드 (이름은 원하시는 대로 바꾸셔도 됩니다)

```bash
docker build -t kgh07/robot_dialogue_relay:v1.0 .

## 3. 실행

호스트에서 이미 돌고 있는 llama-server(Qwen, 포트 2242) / Llama3.3(포트 11434)
서버에 접근하려면, 컨테이너 안에서 호스트 IP로 접속해야 합니다.

```bash
# 호스트 IP 확인 (docker0 브릿지)
ip addr show docker0 | grep inet
```

그 IP(예: 172.17.0.1)를 넣어서 실행:

```bash
docker run --rm -it \
  --device=/dev/kfd --device=/dev/dri \
  --group-add 44 --group-add 992 \
  -e QWEN_URL="http://172.17.0.1:2242/v1/chat/completions" \
  -e LLAMA33_URL="http://172.17.0.1:11434/v1/chat/completions" \
  kgh07/robot_dialogue_relay:v1.0 \
  --device 0 --threshold 45.0
```

(`--group-add`의 44/992는 선생님 머신에서 `getent group video`,
`getent group render`로 다시 확인해서 맞는 숫자로 바꿔주세요.)

## 4. 반복 실행 + 결과 저장

`relay_dialogue.py`는 `-n`, `--save` 옵션을 지원합니다. 기본 CMD를 덮어써서 실행:

```bash
docker run --rm -it \
  --device=/dev/kfd --device=/dev/dri \
  --group-add 44 --group-add 992 \
  -e QWEN_URL="http://172.17.0.1:2242/v1/chat/completions" \
  -e LLAMA33_URL="http://172.17.0.1:11434/v1/chat/completions" \
  -v $(pwd)/results:/app/results \
  kgh07/robot_dialogue_relay:v1.0 \
  --device 0 --threshold 45.0 -n 20 --save /app/results/relay_results.jsonl
```

## Good Luck!!

