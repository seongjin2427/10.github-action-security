# GitHub Actions Security & Permissions

---

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
