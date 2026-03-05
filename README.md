# Prática 3: Integração Contínua e Commits Atômicos vs. Monolíticos

Este repositório contém o código-base para a terceira etapa da prática de versionamento. O objetivo é contrastar a cultura DevOps de pequenos commits frequentes com a prática de grandes entregas espaçadas, evidenciando o impacto na rastreabilidade de bugs e na resolução de conflitos.

## 1. Código-Base Inicial

O repositório possui a branch `main` com o arquivo `analisador_dados.py` contendo a seguinte estrutura básica:

```python
class AnalisadorDados:
    def __init__(self):
        self.dados = []

    def carregar_dados(self, dados_brutos):
        self.dados = dados_brutos

    def processar(self):
        pass

```

## 2. Preparação do Ambiente Local

Divida-se em dois grupos (A e B). Ambos devem clonar o repositório e criar suas respectivas branches a partir da `main`.

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DA_PASTA>

```

## 3. Execução das Tarefas

O Grupo A simulará um fluxo ágil com commits atômicos. O Grupo B simulará um desenvolvimento prolongado com um único commit final ("Big Bang").

### Grupo A: Abordagem Ágil (Commits Atômicos)

**Objetivo:** Implementar as etapas de processamento dividindo as responsabilidades em commits lógicos e isolados.

**Passo 1:** Criar a branch de trabalho.

```bash
git checkout -b feat/processamento-agil

```

**Passo 2 (Commit 1 - Funcionalidade Base):** Substitua todo o código de `analisador_dados.py` pelo bloco abaixo e faça o commit.

```python
class AnalisadorDados:
    def __init__(self):
        self.dados = []

    def carregar_dados(self, dados_brutos):
        self.dados = dados_brutos

    def processar(self):
        soma = sum(self.dados)
        return soma

```

```bash
git add analisador_dados.py
git commit -m "feat: adiciona calculo de soma no processamento principal"

```

**Passo 3 (Commit 2 - Introdução de Regra de Negócio com Bug):** Substitua todo o código de `analisador_dados.py` pelo bloco abaixo. Um bug intencional foi inserido na limpeza de dados.

```python
class AnalisadorDados:
    def __init__(self):
        self.dados = []

    def carregar_dados(self, dados_brutos):
        self.dados = dados_brutos

    def limpar_dados(self):
        # Bug intencional: remove números negativos em vez de apenas nulos
        self.dados = [d for d in self.dados if d and d > 0]

    def processar(self):
        self.limpar_dados()
        soma = sum(self.dados)
        return soma

```

```bash
git add analisador_dados.py
git commit -m "feat: implementa limpeza de dados nulos antes do calculo"

```

**Passo 4 (Commit 3 - Refatoração):** Substitua todo o código de `analisador_dados.py` pelo bloco abaixo para adicionar o relatório.

```python
class AnalisadorDados:
    def __init__(self):
        self.dados = []

    def carregar_dados(self, dados_brutos):
        self.dados = dados_brutos

    def limpar_dados(self):
        # Bug intencional: remove números negativos em vez de apenas nulos
        self.dados = [d for d in self.dados if d and d > 0]

    def processar(self):
        self.limpar_dados()
        soma = sum(self.dados)
        return soma

    def exibir_relatorio(self):
        resultado = self.processar()
        print(f"--- Relatório de Processamento ---")
        print(f"Total calculado: {resultado}")

```

```bash
git add analisador_dados.py
git commit -m "feat: cria formatacao de saida para o relatorio final"

```

**Passo 5:** Envie para o remoto e abra o Pull Request para a `main` (aprovar e realizar o merge imediatamente).

```bash
git push origin feat/processamento-agil

```

### Grupo B: Abordagem Monolítica (Commit "Big Bang")

**Objetivo:** Implementar múltiplas funcionalidades complexas (log, conversão de tipos, ordenação) trabalhando por muito tempo sem integrar com a branch principal.

**Passo 1:** Certifique-se de estar no commit inicial da `main` (antes das alterações do Grupo A) e crie a branch.

```bash
git checkout main
git pull origin main # Apenas se ainda não tiver feito o merge do Grupo A
git checkout <HASH_DO_COMMIT_INICIAL> # Opcional, para garantir o ponto de partida
git checkout -b feat/processamento-completo

```

**Passo 2:** Substitua todo o código do arquivo `analisador_dados.py` pelo bloco abaixo. Isso representa dias de trabalho modificando a estrutura da classe inteira de uma vez.

```python
import logging

logging.basicConfig(level=logging.INFO)

class AnalisadorDados:
    def __init__(self, ignorar_erros=False):
        self._dados_internos = []
        self.ignorar_erros = ignorar_erros
        logging.info("Analisador inicializado.")

    def carregar_dados(self, array_dados):
        if not isinstance(array_dados, list):
            raise ValueError("Os dados devem ser uma lista")
        self._dados_internos = array_dados
        logging.info(f"{len(array_dados)} registros carregados.")

    def ordenar_dados(self):
        self._dados_internos.sort()

    def converter_textos(self):
        temp = []
        for d in self._dados_internos:
            try:
                temp.append(float(d))
            except (ValueError, TypeError):
                if not self.ignorar_erros:
                    raise
        self._dados_internos = temp

    def processar(self):
        self.converter_textos()
        self.ordenar_dados()
        total = sum(self._dados_internos)
        logging.info(f"Processamento concluído. Total: {total}")
        return total

```

**Passo 3:** Realize um único commit contendo todas as alterações estruturais e envie para o remoto.

```bash
git add analisador_dados.py
git commit -m "Implementa funcionalidades de log, conversao, ordenacao e refatora a classe toda"
git push origin feat/processamento-completo

```

## 4. Análise de Impacto (O Problema)

### Cenário 1: Reversão Rápida (Rollback)

A equipe de QA identificou que o sistema está falhando ao processar números negativos. A falha está no código do **Grupo A**.

* **Ação:** O Grupo A deve utilizar o histórico (`git log`) para identificar que o bug foi introduzido no Commit 2 ("implementa limpeza de dados nulos...").
* **Resolução:** Como o commit foi atômico, basta isolar e reverter apenas ele, sem perder a funcionalidade de cálculo (Commit 1) e o relatório (Commit 3).

```bash
# O grupo A deve encontrar o hash do commit 2 com git log e executar:
git revert <HASH_DO_COMMIT_2>
git push origin feat/processamento-agil

```

* *Reflexão: E se o Grupo B tivesse introduzido o erro? Reverter o commit único significaria desfazer semanas de trabalho válidas apenas para corrigir uma linha.*

### Cenário 2: Integration Hell

O **Grupo B** agora tentará abrir um Pull Request da sua branch `feat/processamento-completo` para a `main`.

* **Ação:** Observe a interface do GitHub ao tentar criar o Pull Request.
* **Resultado:** Ocorrerá um conflito estrutural massivo. Como o Grupo B não realizou atualizações incrementais e alterou partes fundamentais da classe (mudando `self.dados` para `self._dados_internos`, alterando o `__init__`, etc.) que o Grupo A já havia modificado, o arquivo estará bloqueado para merge automático. A resolução exigirá reescrever a classe quase do zero, mesclando as regras de negócio de ambos os grupos manualmente.
