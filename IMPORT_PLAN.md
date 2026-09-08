# Однократный импорт в main

Предлагаемый процесс запускается только при изменении временной ветки. Он собирает 30 частей, проверяет SHA-256, распаковывает Git-объекты, проверяет, что main не изменился, создаёт коммит с точным проверенным деревом и отправляет его в main обычным push без force. Ветки не удаляются. Используется временный GITHUB_TOKEN с правом записи содержимого этого репозитория. В окончательном проекте workflow отсутствует.

Ниже только текст для проверки. Этот файл не является исполняемым GitHub workflow.

```yaml
name: Import complete website
on:
  push:
    branches: [codex-source-transfer]
permissions:
  contents: write
jobs:
  import:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Restore verified source into main
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        shell: bash
        run: |
          set -euo pipefail
          auth="$(printf 'x-access-token:%s' "$GH_TOKEN" | base64 -w0)"
          echo "::add-mask::$auth"
          git -c http.extraHeader="AUTHORIZATION: basic $auth" clone --no-checkout "https://github.com/${GITHUB_REPOSITORY}.git" repo
          cd repo
          git checkout --detach "$GITHUB_SHA"
          cat .source-transfer/part-*.bin > "$RUNNER_TEMP/source.pack"
          echo "cc5c1b3449d15e1fb036fdf94d10bbc6997b9b4649d5a964fa6ff499e69f7293  $RUNNER_TEMP/source.pack" | sha256sum --check
          git index-pack --stdin < "$RUNNER_TEMP/source.pack"
          git cat-file -e 8c2a873d4835a04b383b57dec259c1e739b6c287
          git -c http.extraHeader="AUTHORIZATION: basic $auth" fetch origin main
          test "$(git rev-parse origin/main)" = "38b50c7ffeb815eeffdb37a3ca51b1073432d44d"
          git checkout -B main origin/main
          git read-tree --reset -u 79ccff433034d5e26291c4531c386ce25d7df0ca
          git config user.name "west1kexe"
          git config user.email "326056554+west1kexe@users.noreply.github.com"
          git commit -m "Add complete virtual CaseBattle website, admin and local assets"
          test "$(git rev-parse HEAD^{tree})" = "79ccff433034d5e26291c4531c386ce25d7df0ca"
          git -c http.extraHeader="AUTHORIZATION: basic $auth" push origin HEAD:refs/heads/main

```
