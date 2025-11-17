# CI Teste (ci-teste)

Repositório de exemplo para testar GitHub Actions com um pequeno projeto Go.

Este README descreve tudo que é necessário para rodar o workflow do GitHub Actions presente em `.github/workflows/ci.yaml` e como executar os testes/localmente.

## Conteúdo do repositório
- `math.go` — aplicação simples que imprime o resultado de `soma`.
- `math_teste.go` — teste unitário para a função `soma`.
- `go.mod` — arquivo de módulo Go.
- `.github/workflows/ci.yaml` — workflow do GitHub Actions (executa checkout, setup do Go e roda testes e aplicação).

## Objetivo
Rodar testes automatizados via GitHub Actions toda vez que houver push para o repositório e permitir execução local dos mesmos comandos.

---

## Pré-requisitos (local)
- Go instalado na sua máquina (versão compatível com a usada no `go.mod` ou no workflow).
  - Verifique com:
    ```sh
    go version
    ```
- Git e uma conta GitHub para puxar/push do repositório.

Se `go` não for encontrado:
- Instale o Go: https://go.dev/dl/
- No Windows, verifique se `C:\Go\bin` foi adicionado ao PATH.

---

## Configuração local (passo a passo)
1. Clone o repositório:
   ```sh
   git clone https://github.com/pedrovieira249/ci-teste.git
   cd ci-teste
   ```

2. Verifique o `go.mod` existente:
   - Ele deve conter uma linha `go X.Y` (ex.: `go 1.16`, `go 1.25`).
   - Se você preferir usar a versão do workflow atual (`1.16`), ajuste:
     ```sh
     # editar go.mod manualmente ou:
     go mod edit -go=1.16
     ```
   - Se preferir usar uma versão mais nova, atualize o workflow (`.github/workflows/ci.yaml`) para usar a mesma versão (ex.: `1.25`).

3. Baixe dependências (se houver) e organize o módulo:
   ```sh
   go mod tidy
   ```

---

## Rodando os testes localmente
Na raiz do projeto:
```sh
go test ./...
```
ou, para o pacote atual:
```sh
go test
```
Saída esperada: testes passam sem erros.

---

## Rodando a aplicação localmente
```sh
go run math.go
```
Deve imprimir o resultado da soma definida em `math.go`.

---

## Sobre o GitHub Actions (workflow)
Arquivo: `.github/workflows/ci.yaml`

O workflow atual faz:
- Checkout do repositório (actions/checkout@v2)
- Setup do Go (actions/setup-go@v2) com `go-version: 1.16`
- Roda `go test`
- Roda `go run math.go`

Para que o CI passe, é importante que o `go.mod` esteja presente e correto (campo `go` compatível com a versão do Go utilizada).

### Como disparar o CI
- Faça commit e push para a branch `main` (ou qualquer branch, conforme trigger do workflow).
- Vá para a aba "Actions" do repositório no GitHub e acompanhe a execução.

---

## Erros comuns e como resolver

1. "bash: go: command not found"
   - Significa que o Go não está instalado ou não está no PATH. Instale o Go e reinicie o terminal.

2. "go: cannot find main module, but found .git/config ... to create a module there, run: go mod init"
   - Crie um módulo na raiz do projeto:
     ```sh
     go mod init github.com/pedrovieira249/ci-teste
     ```
   - Commit e push do `go.mod`.

3. "invalid go version '1.25.4': must match format 1.23"
   - A linha `go` em `go.mod` aceita apenas `MAJOR.MINOR` (ex.: `go 1.25`), sem o terceiro número. Corrija para:
     ```
     go 1.25
     ```
   - Exemplo de edição:
     ```sh
     go mod edit -go=1.25
     git add go.mod
     git commit -m "Fix go.mod version format"
     git push
     ```

4. Workflow com múltiplos `uses` no mesmo step
   - Cada `uses` deve estar em um step separado. Exemplo correto (já presente no repositório):
     ```yaml
     - uses: actions/checkout@v2
     - uses: actions/setup-go@v2
       with:
         go-version: 1.16
     ```

5. Versões inconsistentes entre `go.mod` e o `go-version` do workflow
   - Mantenha-os consistentes. Ou:
     - Atualize `go.mod` para `go X.Y` correspondente à versão usada no workflow.
     - Ou atualize `.github/workflows/ci.yaml` para usar a versão que você deseja (por exemplo `1.25`).

---

## Sugestões / Melhores práticas
- Harmonize o `go` em `go.mod` com o `go-version` do workflow.
- Use `go test ./...` no CI para garantir execução de todos os pacotes.
- Adicione `- name: Cache Go modules` no workflow para acelerar builds (opcional).
- Em vez de `go run math.go`, prefira compilar (`go build`) e, se necessário, executar o binário nos passos de integração.

Exemplo opcional de step para rodar todos os testes:
```yaml
- name: Run tests
  run: go test ./...
```

---

## Exemplo de solução rápida para o repositório atual
- Se quiser que o workflow use Go 1.25 e o `go.mod` também:
  1. Atualize `.github/workflows/ci.yaml`:
     ```yaml
     - uses: actions/setup-go@v2
       with:
         go-version: '1.25'
     ```
  2. Ajuste `go.mod` para:
     ```
     go 1.25
     ```
  3. Commit e push.

- Ou, se preferir manter `go.mod` com `go 1.16`, altere `go.mod` para `go 1.16`.

---

## Contato
Se precisar que eu atualize o workflow para combinar com a versão do `go.mod` desejada (ou ajustar `go.mod` para combinar com o workflow), posso criar o arquivo corrigido e mostrar exatamente as mudanças. Basta dizer qual versão do Go você quer usar no CI (ex.: `1.16`, `1.20`, `1.25`).