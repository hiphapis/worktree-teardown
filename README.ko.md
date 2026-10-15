> [!IMPORTANT]
> 이 저장소는 [hiphapis/claude-plugins](https://github.com/hiphapis/claude-plugins/tree/main/plugins/worktree-teardown)로 이전되었고 보관(archive) 처리되었습니다. 설치: `/plugin marketplace add hiphapis/claude-plugins` → `/plugin install worktree-teardown@hiphapis`

# worktree-teardown

[English](README.md)

git 워크트리를 **지우기 전에** 감사해서, 지워도 아무것도 함께 사라지지 않게 하는 Claude Code 스킬입니다.

워크트리를 지워서 커밋이 날아가는 일은 드뭅니다. 변경이 남은 워크트리는 git 이 삭제를 거부하기 때문입니다. 실제로 잃는 것은 다른 곳에 있습니다.

- **gitignore 된 자산**: 에이전트 인계 노트, 계획, 생성한 이미지·영상, 그 워크트리에서만 고친 `.env`
- **빠뜨린 마무리**: 이슈가 아직 "진행 중"이고, 워크스페이스 상태도 그대로이고, 문서도 갱신하지 않음
- **낡은 기록**: 실제로는 병합했는데 노트에는 "미병합"으로 남아 있거나, 그 반대

스킬은 감사 항목 11종을 훑어 항목마다 등급을 매기고, 승인받은 마무리를 수행한 뒤 다시 감사합니다. 그다음에야 삭제 명령을 **보여줍니다**. 삭제는 스킬이 직접 하지 않습니다.

## 설치

플러그인으로 설치(권장):

```
/plugin marketplace add hiphapis/worktree-teardown
/plugin install worktree-teardown@worktree-teardown
```

또는 스킬 폴더를 직접 복사:

```bash
git clone https://github.com/hiphapis/worktree-teardown
cp -r worktree-teardown/skills/worktree-teardown ~/.claude/skills/
```

## 사용

워크트리 안(또는 리포 어디서든)에서 이렇게 물어보면 됩니다.

> 이 워크트리 지워도 돼?
> can I delete this worktree?

`/worktree-teardown:worktree-teardown` 으로 직접 부를 수도 있습니다. 다른 워크트리를 감사하려면 경로나 브랜치를 알려주세요.

## 감사 항목

| # | 항목 | 삭제를 막는 조건 |
| --- | --- | --- |
| 1 | 커밋 안 한 tracked 변경 | 하나라도 있음 |
| 2 | untracked 파일(ignore 아님) | 하나라도 있음 |
| 3 | 푸시 안 한 커밋 | 원격에 사본이 없음 |
| 4 | 병합 안 한 커밋(`git cherry` 로 squash 병합 판별) | 병합·보존·폐기 중 선택 |
| 5 | stash(고아 stash 포함) | 이 브랜치 소유 stash가 있는데 브랜치를 폐기함 |
| 6 | 재생성 불가 ignored 자산(블랙리스트, `.env` 는 `diff` 로 판정) | 보존 대상이 있음 |
| 7 | 이슈 또는 PR 상태(GitHub) | 프로젝트 라벨 기준 |
| 8 | Orca 워크스페이스 상태(이슈와 교차 판정) | 이슈는 닫혔는데 상태가 `completed` 아님 |
| 9 | 자식 워크트리(Orca) | 하나라도 있음 |
| 10 | 정본 문서 드리프트: 이번 작업이 없앤 사실이 문서에 남아 있음 | 정본 경로에서 발견 |
| 11 | 기록 정합(판단 항목, 자동으로 막지 않음) | — |

등급은 🔴 BLOCKED · 🟡 확인 필요 · ✓ 정상 · **N/A** 네 가지입니다. N/A는 프로젝트가 쓰지 않는 도구(Orca 없음, GitHub 아닌 원격)라는 뜻이고 삭제를 막지 않습니다. 쓰는 도구인데 호출이 실패하면 🟡 입니다. 확인하지 못한 항목을 정상으로 치지 않습니다.

종료 상태: `SAFE`(삭제 명령 제시) · `BLOCKED(n)` · `HANDOFF`(마무리를 끝낸 뒤 병합으로 넘김) · `ABORT`.

## 요구 사항

- `git` (필수)
- 로그인한 `gh`: 이슈 점검용. 없으면 해당 항목이 N/A(GitHub 아님) 또는 🟡(GitHub인데 호출 실패)가 됩니다.
- Orca (선택): Orca가 관리하는 워크트리면 사용하고, 아니면 일반 `git worktree` 로 동작합니다.
- `superpowers:finishing-a-development-branch` (선택): 설치돼 있으면 병합 인계에 사용합니다.

## 프로젝트 정책 (선택)

라벨, 완료 절차, 어떤 문서가 정본이고 어떤 문서가 역사 기록인지는 프로젝트마다 다릅니다. 리포에 `.claude/worktree-teardown.md` 를 두면 스킬이 그 내용을 따릅니다([예시](examples/worktree-teardown.md) · [형식](skills/worktree-teardown/references/project-config.md)). 이 파일이 없으면 `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md` 에서 작업 수명주기 규칙을 찾고, 그것도 없으면 기본값을 씁니다.

## 안전 계약

워크트리를 삭제하지 않음 · 대신 커밋하지 않음 · merge·rebase·push 하지 않음 · 파일을 삭제·이동하지 않음(보존은 복사로만) · 승인 없이 이슈를 닫지 않고, `not planned` 종료는 승인이 있어도 직접 하지 않음 · 실패한 점검을 정상으로 보고하지 않음.

전체 규칙은 [`skills/worktree-teardown/references/audit-rules.md`](skills/worktree-teardown/references/audit-rules.md) 에 있습니다.

## 라이선스

MIT
