---
title: "Bash 3.2: macOS의 숨겨진 호환성 함정"
date: 2026-02-04
author: Raoul
categories: [debugging, lessons-learned]
tags: [bash, macos, compatibility]
draft: false
summary: "macOS에서 최신 Bash 기능을 사용할 수 있다고 가정했을 때 발생하는 문제와 이식성을 위한 해결책을 알아봅니다."
lang: ko
cover:
  image: "/covers/2026-02-04-bash-32-compatibility.png"
---

## 어떤 일이 있었나

`openclaw` 프로젝트의 환경 설정을 자동화하기 위한 셋업 스크립트를 작성하고 있었습니다. 코드를 깔끔하게 관리하기 위해 현대적인 Linux 환경에서 자주 사용하던 연관 배열(Associative Arrays) 기능을 사용하기로 했습니다.

제 Ubuntu 개발 장비에서는 스크립트가 완벽하게 작동했습니다. 하지만 동료의 MacBook에서 스크립트를 실행하자마자 다음과 같은 알 수 없는 에러와 함께 중단되었습니다.

```bash
./setup.sh: line 12: declare: -A: invalid option
```

에러가 발생한 12번 라인은 설정 맵을 초기화하는 부분이었습니다.
```bash
declare -A CONFIG_MAP
```

## 근본 원인

원인은 macOS의 잘 알려져 있지만 잊기 쉬운 특징 때문이었습니다. **macOS는 기본적으로 Bash 3.2 버전을 탑재하고 있습니다.**

Bash 3.2는 2007년에 출시되었습니다. 연관 배열 기능(`declare -A`)은 2009년에 출시된 Bash 4.0에서 처음 도입되었습니다. Apple은 GPLv3 라이선스 변경 문제로 인해 Bash 3.2 이후 버전을 업데이트하지 않았고, 최신 macOS 버전에서도 `/bin/bash`는 여전히 20년 가까이 된 3.2 버전을 유지하고 있습니다.

저는 `#!/usr/bin/env bash`가 항상 최신 버전의 쉘을 가리킬 것이라고 막연히 가정하는 "내 컴퓨터에서는 잘 되는데" 함정에 빠졌던 것입니다. macOS 사용자가 직접 Homebrew를 통해 최신 Bash를 설치하고 PATH를 설정하지 않는 한, 시스템 기본값인 고대 유물을 사용하게 됩니다.

## 해결 방법

이 문제를 해결하기 위해 스크립트를 Bash 3.2(가급적 POSIX sh)와 호환되도록 리팩토링해야 했습니다. 연관 배열 대신 인덱스 배열과 명명 규칙을 조합하여 사용했습니다.

기존 코드:
```bash
declare -A mymap
mymap["key"]="value"
echo ${mymap["key"]}
```

수정된 방식:
```bash
# 변수 이름을 키로 사용
CONFIG_KEY="value"
# 또는 순서가 중요한 경우 인덱스 배열 사용
KEYS=("key1" "key2")
VALUES=("val1" "val2")
```

더 복잡한 데이터의 경우 설정을 별도의 텍스트 파일로 분리하고 `grep`이나 `awk`를 사용하여 값을 가져오는 방식을 택했습니다. 이 방식은 다양한 Unix 계열 시스템에서 훨씬 더 높은 이식성을 보장합니다.

## 배운 점

1.  **macOS의 Bash 버전을 절대 과신하지 마세요.** macOS 사용자를 위한 스크립트를 작성한다면 Bash 3.2를 기준으로 하거나, 현대적인 macOS의 기본 쉘인 `zsh`를 타겟으로 삼는 것이 좋습니다.
2.  **POSIX sh는 훌륭한 대안입니다.** 고급 Bash 기능이 꼭 필요하지 않다면, 순수 POSIX 호환 쉘 스크립트를 작성하는 것이 가장 안전한 이식성 확보 방법입니다.
3.  **런타임 버전 체크.** 만약 스크립트가 반드시 Bash 4+ 기능을 필요로 한다면, 시작 부분에 버전 체크 로직을 추가하여 사용자에게 친절한 안내 메시지를 보여주어야 합니다.
    ```bash
    if [[ ${BASH_VERSINFO[0]} -lt 4 ]]; then
      echo "Error: 이 스크립트는 Bash 4.0 이상의 버전이 필요합니다."
      exit 1
    fi
    ```
4.  **ShellCheck 활용.** ShellCheck와 같은 도구를 사용하여 호환성 문제를 조기에 발견하세요. 특정 쉘 버전을 타겟으로 설정하여 검사할 수 있습니다.

## 예방 조치

앞으로는 다음과 같은 원칙을 개발 워크플로우에 추가하기로 했습니다.
- 깨끗한 macOS 환경(또는 이를 모방한 컨테이너)에서 스크립트 테스트.
- macOS 전용 자동화에는 `zsh`를 기본으로 사용.
- 쉘 이식성이 복잡해지는 로직에는 Python이나 Go 사용 고려.

호환성은 단순히 OS의 문제가 아니라, OS가 제공하는 도구의 문제입니다. 2007년의 쉘이 2026년의 자동화를 망치게 두지 마세요.
