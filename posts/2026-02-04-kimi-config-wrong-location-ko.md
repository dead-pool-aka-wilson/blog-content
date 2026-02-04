---
title: "설정 변경이 적용되지 않는 이유: 템플릿과 활성 설정의 차이"
date: 2026-02-04
author: Raoul
categories: [debugging, lessons-learned]
tags: [config, opencode]
draft: false
summary: "개발자들이 흔히 겪는 실수: 저장소에 있는 템플릿 파일을 수정하면서 실제 애플리케이션이 사용하는 활성 설정 파일은 놓치는 경우에 대해 알아봅니다."
lang: ko
cover:
  image: "/covers/2026-02-04-kimi-config-wrong-location.png"
---

## 어떤 일이 있었나

OpenCode 환경에서 Kimi K2 모델 설정을 미세 조정하려고 했습니다. `openclaw-setup` 저장소로 이동하여 `configs/opencode/opencode.json` 파일을 찾아 모델 파라미터를 여러 번 수정했습니다.

파일을 저장하고 애플리케이션을 재시작했지만... 아무런 변화가 없었습니다. 더 과감하게 값을 바꿔보고, 심지어 JSON 문법을 일부러 틀리게 고쳐서 에러가 발생하는지 확인해 보기도 했습니다. 하지만 여전히 아무 일도 일어나지 않았습니다. 애플리케이션은 마치 제가 단 한 줄도 건드리지 않은 것처럼 평온하게 작동했습니다.

## 근본 원인

근본 원인은 전형적인 "엉뚱한 파일 수정"이었습니다.

현대적인 개발 워크플로우에서는 설정 파일을 별도의 "dotfiles"나 "setup" 저장소에 보관하는 경우가 많습니다. 이는 버전 관리나 여러 기기 간의 설정 공유에 매우 유용합니다. 하지만 대부분의 애플리케이션은 `~/Dev/my-setup/`과 같은 임의의 폴더에서 설정을 직접 읽어오지 않습니다.

OpenCode는 다른 많은 도구와 마찬가지로 XDG 기본 디렉토리 사양(XDG Base Directory Specification)을 따릅니다. 즉, 실제 활성 설정은 다음 경로에서 찾습니다.
`~/.config/opencode/opencode.json`

제가 수정하고 있던 `~/Dev/openclaw-setup/configs/opencode/opencode.json` 파일은 단지 제 설정 저장소에 보관된 **템플릿** 또는 **참조용 복사본**일 뿐이었습니다. 해당 파일을 `~/.config/` 디렉토리에 명시적으로 심볼릭 링크(symlink)를 걸어두지 않는 한, 애플리케이션은 그 파일의 존재조차 알 수 없습니다.

## 해결 방법

해결 방법은 간단했습니다. 애플리케이션이 실제로 읽어들이는 파일을 수정하는 것이었습니다.

```bash
# 잘못된 접근:
# vim ~/Dev/openclaw-setup/configs/opencode/opencode.json

# 올바른 접근:
vim ~/.config/opencode/opencode.json
```

설정 저장소를 실제로 유용하게 만들기 위해, 저장소의 변경 사항이 활성 설정에 자동으로 반영되도록 심볼릭 링크를 사용했습니다.

```bash
ln -sf ~/Dev/openclaw-setup/configs/opencode/opencode.json ~/.config/opencode/opencode.json
```

## 배운 점

1.  **활성 설정 경로를 확인하세요.** 설정 변경이 적용되지 않아 디버깅에 시간을 쏟기 전에, 애플리케이션이 정확히 어디에서 설정을 읽고 있는지 확인해야 합니다. 많은 도구가 `--verbose` 플래그나 `info` 명령어를 통해 이 경로를 보여줍니다.
2.  **템플릿 vs. 활성 설정.** 버전 관리용 파일(템플릿)과 런타임용 파일(활성 설정)을 명확히 구분하세요.
3.  **표준 경로.** macOS나 Linux에서는 항상 `~/.config/` 또는 `~/Library/Application Support/`를 먼저 확인하는 습관을 들이세요.
4.  **심볼릭 링크 활용.** 설정 파일을 Git 저장소에서 관리하고 싶다면, 심볼릭 링크를 사용하여 표준 경로와 연결하세요. 이렇게 하면 "진실의 원천(source of truth)"이 곧 "활성 원천"이 됩니다.

## 예방 조치

이러한 혼란을 방지하기 위해 다음과 같은 조치를 취했습니다.
-   **주석 추가:** 템플릿 파일 상단에 주석을 추가했습니다: `// TEMPLATE ONLY - ~/.config/opencode/opencode.json으로 복사하거나 심볼릭 링크를 거세요.`
-   **자동화:** `stow`와 같은 도구나 간단한 셋업 스크립트를 사용하여 심볼릭 링크를 자동으로 관리합니다.
-   **자기 점검:** 설정 변경이 작동하지 않을 때 가장 먼저 스스로에게 던져야 할 질문은 "내가 지금 맞는 파일을 고치고 있는가?"입니다.

작은 실수지만 큰 좌절감을 줄 수 있습니다. 파일 이름이 `config.json`이고 프로젝트 폴더에 있다고 해서, 그것이 반드시 사용 중인 파일이라고 가정하지 마세요.
