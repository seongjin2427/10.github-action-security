# GitHub Actions Security & Permissions

---

## Security Problems

- Script Injection

- Malicious Third-Party Actions

- Permission Issues

- Etc.

---

## Script Injection

1. Issue 생성 이벤트에서 Issue의 제목을 참조하여 워크플로우에 활용한다고 할 때, Script Injection이 발생할 수 있습니다. - 

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
  - 
