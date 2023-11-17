# GitHub Actions Security & Permissions

## Security Problems

- Script Injection

- Malicious Third-Party Actions

- Permission Issues

- Etc.

---

## Script Injection

1. Issue 생성 이벤트에서 Issue의 제목을 참조하여 워크플로우에 활용한다고 할 때, Script Injection이 발생할 수 있습니다. - [`91c6c956`](https://github.com/seongjin2427/10.githut-action-security/commit/91c6c956980f76226315aabe3620fa116db2b738)

- Process
  - `script-injection.yml`
    - ```yml
      name: Label Issues (Script Injection Example)
      on:
        # Issue가 열릴 때, 워크플로우가 동작합니다.
        issues:
          types:
            - opened
      jobs:
        assign-label:
          runs-on: ubuntu-latest
          steps:
            - name: Assign label
              # github 컨텍스트에서 issue의 title을 변수에 할당하고
              # 그 변수(issue_title)이 "bug"인지에 따라 출력이 달라집니다.
              run: |
                issue_title="${{ github.event.issue.title }}"
                if [[ "$issue_title" == *"bug"* ]]; then
                echo "Issue is about a bug!"
                else
                echo "Issue is not about a bug"
                fi
  - Issue 제목을 `a; echo Got your secrets`으로 작성하여 생성합니다.

- Result
  - Issue 제목만으로도 GitHub Action에서 의도하지 않았던 쉘 명령얼르 실행할 수 있습니다.
  - `Run issue_title="a";` 실행 후, `Got your secrets`를 출력합니다.'

2. 이러한 방식의 Script Injection을 막기 위해서, 컨텍스트의 참조 값을 별도의 환경변수로 할당하고, 러너의 터미널에서는 해당 환경변수만을 사용하여 실행합니다. - [`b0534ea8`](https://github.com/seongjin2427/10.githut-action-security/commit/b0534ea8378108220447827a3cc0ed8afbd8e37c)

- Process
  - `script-injection.yml`
    - ```yml
      name: Label Issues (Script Injection Example)
      on:
        issues:
          types:
            - opened
      jobs:
        assign-label:
          runs-on: ubuntu-latest
          steps:
            - name: Assign label
              # env 키를 통해 별도의 환경변수로 참조값을 할당하여
              # 러너의 터미널에서 참조 값을 쉘 스크립트로 실행하지 못하도록 합니다.
              env:
                TITLE: ${{ github.event.issue.title }}
              run: |
                if [[ "$TITLE" == *"bug"* ]]; then
                echo "Issue is about a bug!"
                else
                echo "Issue is not about a bug"
                fi

- Result
  - Issue의 제목을 통해 Script Injection을 시도하더라도 동작하지 않습니다.

<br>

---
## Permissions

1. `permissions` 키를 지정하여, 추가 레이어를 통해 특정 이벤트에 대한 한정된 권한을 부여함으로써 보안을 더 강화 할 수 있습니다. - [`af4367cb`](https://github.com/seongjin2427/10.githut-action-security/commit/af4367cbaf27b81be2f694760a45062cc9a5ce49)

- Process
  - `label-issues-real.yml`
    - ```yml
      name: Label Issues (Permissions Example)
      on:
        issues:
          types:
            - opened
      jobs:
        assign-label:
        # Action이나 Job 단위로 지정할 수 있습니다.
          permissions: 
            # 이슈에 대해서 쓰기만/읽기만/모두 불가 를 지정할 수 있습니다.
            # 여기에 작성된 이벤트에 대한 권한만 부여됩니다.
            issues: write
          runs-on: ubuntu-latest
          steps:
            - name: Assign label
              if: contains(github.event.issue.title, 'bug')
              run: |
                curl -X POST \
                --url https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.issue.number }}/labels \
                -H 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
                -H 'content-type: application/json' \
                -d '{
                    "labels": ["bug"]
                  }' \
                --fail

- Result
  - 작동에는 큰 차이가 없지만, 좀 더 안전한 보안을 가지게 됩니다.

2. `permissions`를 실제 Step에서 실행하는 것과 다르게 지정하여 일부러 실패해 봅시다. - [`d571268b`](https://github.com/seongjin2427/10.githut-action-security/commit/d571268b38e2b8d800170721d1a58690b93356c3)

- `${{ secrets.GITHUB_TOKEN ]}`
  - `secrets`으로 `GITHUB_TOKEN`를 지정하지 않았지만 참조가 됩니다.
  - GitHub에서 `permissions`에 해당하는 권한의 토큰을 자동으로 생성하여 `secrets.GITHUB_TOKEN`에 할당합나디.

- Process
  - `workflows/label-issues-real.yml`
    - ```yml
      ...
        permissions:
          issues: read
        ...

  - 다시 "bug" 단어를 포함해 Issue를 Open합니다.

- Result
  - `Assign label` Step의 `run` 명령어들은 문제가 없으나, 주어진 `permissions` - `issues: read` 권한에 맞지 않는 Step으로 워크플로우는 중단됩니다.

<br>

---
