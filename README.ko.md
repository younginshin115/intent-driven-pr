# intent-driven-pr

[English](README.md) | [한국어](README.ko.md)

브랜치의 커밋과 diff를 바탕으로 **변경 의도와 의사결정을 담은 PR 설명**을 작성하는 스킬입니다. 10개 섹션으로 구성된 템플릿에 따라 PR 본문을 작성하고, `gh`로 PR을 생성하거나 수정합니다. **Claude Code, Cursor, Codex CLI**에서 사용할 수 있습니다.

## 배경

이 스킬은 [카카오페이 기술 블로그의 「PR을 더 느리게 만들기 위한 고민」](https://tech.kakaopay.com/post/kakaopayins-slow-pr-fast-dev/)에서 영감을 받았습니다. PR 설명을 충분히 고민해서 작성하면 리뷰가 빨라진다는 관점에서 출발했습니다.

**목표는 리뷰를 쉽게 만드는 것입니다.** "버그 수정" 한 줄로는 전달하기 어려운 두 가지 정보를 담습니다.

- **변경 의도**: 변경이 필요한 이유, 주요 결정과 제외한 대안, 검증 방법
- **리뷰 포인트**: 파일·영역별 위험도(🔴🟡🟢)와 검토가 필요한 이유

작성자가 알고 있는 맥락을 문서로 남겨 리뷰 과정에서 반복되는 질문과 답변을 줄입니다.

## 사전 준비

- [GitHub CLI](https://cli.github.com/) (`gh`) 설치 및 인증 (`gh auth login`)
- GitHub 원격 저장소가 연결된 Git 저장소

스킬이 `git`과 `gh`를 직접 사용하므로 Python이나 별도 런타임은 필요하지 않습니다.

## 설치

아래 표에서 사용하는 에이전트의 `--agent` 값을 확인한 뒤, 설치 명령 중 하나의 `<AGENT>`에 넣어 실행하세요. 선택한 에이전트에 스킬 파일(`skills/write-pr/`)이 설치됩니다.

| 도구 | `--agent` 값 | 스킬 설치 후 직접 호출 예시 |
|------|---------------|-----------|
| Claude Code | `claude-code` | `/write-pr [기준 브랜치 또는 PR 번호]` |
| Cursor 2.4+ | `cursor` | `/write-pr` |
| Codex CLI | `codex` | `$write-pr` |

**GitHub CLI로 설치:**

```bash
gh skill install younginshin115/intent-driven-pr write-pr --agent <AGENT> --scope user
```

**npx로 설치:**

```bash
npx skills add younginshin115/intent-driven-pr --agent <AGENT> --global
```

**Claude Code 플러그인으로 설치하는 방법:**

```text
/plugin marketplace add younginshin115/intent-driven-pr
/plugin install intent-driven-pr@intent-driven-pr
```

플러그인으로 설치한 경우 `/intent-driven-pr:write-pr 123`처럼 직접 호출할 수 있습니다.

## 사용법

PR을 생성하거나 수정할 브랜치에서 에이전트에게 자연어로 요청하세요.

```text
write-pr로 현재 브랜치의 PR을 만들어줘.
write-pr로 develop을 기준으로 PR을 만들어줘.
write-pr로 PR #123의 본문을 수정해줘.
```

에이전트가 요청 내용에 따라 스킬을 자동으로 선택할 수 있습니다. 직접 호출하려면 위의 도구별·설치 방식별 예시를 사용하세요.

스킬은 `git log`, `git diff`, 기존 PR에서 맥락을 수집합니다. 변경 규모에 맞게 필요한 섹션을 작성한 뒤 PR을 생성하거나 수정합니다.

**언어:** 대화 언어가 한국어면 섹션 제목과 본문을 한국어로, 그 외에는 영어로 작성합니다. 출력 언어를 따로 요청하면 해당 요청을 우선합니다.

## 생성되는 내용

한국어 PR 본문은 다음 섹션으로 구성됩니다. 해당하지 않는 섹션은 생략합니다.

1. 어떤 문제를 해결하나요?
2. 왜 필요한가요?
3. 어떻게 해결했나요?
4. 기존 동작에 미치는 영향
5. 엣지 케이스 및 실패 시나리오
6. 한계 및 트레이드오프
7. 검토한 대안과 선택 이유
8. 검증 및 테스트 결과
9. 리뷰 포인트 (파일/영역별 위험도 🔴🟡🟢)
10. 후속 작업 및 필수 조치

## 출처

이 스킬의 템플릿은 카카오페이 기술 블로그의 [「PR을 더 느리게 만들기 위한 고민」(2026)](https://tech.kakaopay.com/post/kakaopayins-slow-pr-fast-dev/)에 소개된 PR 템플릿을 바탕으로 구성했습니다.

원문의 8개 섹션에 '검증 및 테스트 결과'와 '후속 작업 및 필수 조치'를 추가하고, 영어·한국어 템플릿과 `gh` 자동화를 제공하는 스킬로 구성했습니다. "느린 PR이 빠른 개발을 만든다"는 철학에서 영감을 받았으며, 스킬 구현은 별도로 작성했습니다.

## 라이선스

[MIT](LICENSE) © Youngin Shin
