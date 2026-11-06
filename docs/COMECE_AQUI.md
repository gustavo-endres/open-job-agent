# 🚀 Comece aqui — guia para iniciantes

Este guia leva você do zero até rodar o **Open Job Agent** e criar a sua própria versão. Não é preciso experiência prévia com Python ou GitHub Actions.

> **Antes de tudo:** a demonstração usa apenas **vagas fictícias**. Ela não precisa de conta Google, chaves de API nem acesso a sites de vagas. O projeto **não envia candidaturas**.

## Sumário

1. [Ver a demo sem instalar nada](#1-ver-a-demo-sem-instalar-nada)
2. [Criar a sua cópia do projeto](#2-criar-a-sua-cópia-do-projeto)
3. [Rodar no seu computador](#3-rodar-no-seu-computador)
4. [Entender o resultado](#4-entender-o-resultado)
5. [Personalizar com segurança](#5-personalizar-com-segurança)
6. [Problemas comuns](#6-problemas-comuns)

## 1. Ver a demo sem instalar nada

1. Abra a aba **Actions** do repositório.
2. Na lateral esquerda, clique em **Open Job Agent (manual)**.
3. Abra uma execução com ✅ verde e clique no job **`run`**.
4. Expanda o passo **Demo (safe with zero secrets)**. Você verá uma linha parecida com esta:

```text
RESULT: {"deduplicated": 3, "errors": 0, "reported": 3, "review": 1, "seen": 3}
SYNC: skipped (local mode)
```

Os logs públicos mostram **apenas contagens**, nunca títulos de vagas ou dados pessoais. Isso é proposital.

## 2. Criar a sua cópia do projeto

Você tem duas opções:

| Opção | Como fazer | Quando usar |
|---|---|---|
| **Use this template** | Botão verde **Use this template → Create a new repository** | Você quer um projeto **seu**, independente, com histórico limpo. **Recomendado.** |
| **Fork** | Botão **Fork** no topo da página | Você quer acompanhar o projeto original ou sugerir melhorias via *pull request*. |

Depois de criar a cópia, para rodar a demo pelo GitHub:

1. Abra a aba **Actions** da **sua cópia**.
2. Se aparecer um aviso sobre workflows desativados, clique para habilitá-los.
3. Escolha **Open Job Agent (manual) → Run workflow**, mantenha `demo` e confirme.
4. Aguarde cerca de 1 minuto e abra a execução.

## 3. Rodar no seu computador

### 3.1 Instale o Python

Baixe o **Python 3.10 ou superior** em [python.org/downloads](https://www.python.org/downloads/). No Windows, marque a opção **"Add python.exe to PATH"** durante a instalação.

### 3.2 Baixe o projeto

- **Com Git:** `git clone https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git`
- **Sem Git:** na página da sua cópia, clique em **Code → Download ZIP** e extraia a pasta.

### 3.3 Instale, teste e rode a demo

Abra o **PowerShell dentro da pasta do projeto** (no Explorador de Arquivos, clique na barra de endereço, digite `powershell` e aperte Enter). Depois execute uma linha por vez:

```powershell
py --version
py -m pip install -e ".[dev]"
py -m pytest -q
py -m jobagent.cli demo
```

O que esperar de cada comando:

| Comando | Resultado esperado |
|---|---|
| `py --version` | `Python 3.10` ou superior |
| `py -m pip install -e ".[dev]"` | Instala o projeto e o `pytest` |
| `py -m pytest -q` | `27 passed` |
| `py -m jobagent.cli demo` | `RESULT: ... "errors": 0 ... "review": 1 ...` |

No Linux ou macOS, troque `py` por `python3`.

## 4. Entender o resultado

A demo gera o arquivo **`build/report.json`** (ignorado pelo Git). Cada vaga recebe:

| Campo | Significado |
|---|---|
| `score` | Compatibilidade heurística de 0 a 100. **Não é probabilidade de contratação.** |
| `status` | `REVIEW` (vale revisar), `LOW_MATCH` (pontuação abaixo do limite) ou `BLOCKED` (algum impeditivo) |
| `blockers` | Motivos que bloqueiam a vaga, por exemplo trabalho híbrido quando você quer remoto |
| `unknowns` | Informações que **você** precisa confirmar, como salário e tipo de contrato |
| `matched_skills` | Habilidades do seu perfil encontradas na descrição |
| `auto_apply` | Sempre `false`: o projeto nunca se candidata sozinho |

Na amostra incluída, o resultado é:

- **Automation & Integration Analyst** (remoto): `score` 100, status `REVIEW`
- **Integration Operations Specialist** (híbrido): `BLOCKED`, porque o perfil de exemplo prefere remoto
- **Customer Service Associate**: `BLOCKED`, porque o cargo está fora das áreas configuradas

## 5. Personalizar com segurança

### 5.1 Seu perfil

```powershell
New-Item -ItemType Directory -Path config/private -Force
Copy-Item config/profile.example.json config/private/profile.json
```

Edite **somente** `config/private/profile.json`:

| Campo | O que ajustar |
|---|---|
| `priority_terms` | Palavras dos cargos que você procura |
| `confirmed_skills` | Habilidades que você realmente tem |
| `preferred_workplace` | `remote` (bloqueia vagas híbridas e presenciais) ou `any` (aceita qualquer modelo) |
| `country_keywords` | Termos de localização aceitos |
| `minimum_monthly_brl` | Piso salarial por contrato (`CLT`, `PJ`) ou `null` |
| `review_score_threshold` | Nota mínima para uma vaga ir para revisão |

### 5.2 Suas fontes de vagas

```powershell
Copy-Item config/sources.example.json config/private/sources.json
```

Adicione apenas quadros **públicos** de empresas que usam Greenhouse, Lever ou Ashby, e cujo acesso seja permitido. Depois rode:

```powershell
py -m jobagent.cli scan
```

Detalhes e a integração opcional com Google Sheets estão em [Configuração privada](CONFIGURACAO_PRIVADA.md).

### 5.3 Suas regras de pontuação

As regras ficam em `jobagent/scoring.py`. Ao mudar alguma regra, adicione ou ajuste um teste em `tests/test_scoring.py` e rode `py -m pytest -q` novamente.

### ⚠️ Nunca publique

- Currículo, e-mail, telefone ou salário desejado
- Qualquer arquivo de `config/private/`, `.env` ou `build/`
- URLs de webhook, segredos HMAC ou tokens

Essas pastas já estão no `.gitignore`, mas **confira os arquivos antes de cada commit**.

## 6. Problemas comuns

| Problema | Solução |
|---|---|
| `py` não é reconhecido | Reinstale o Python marcando **"Add python.exe to PATH"**, ou tente `python` no lugar de `py` |
| `No module named jobagent` | Você não está na pasta do projeto. Use `cd` até a pasta que contém `pyproject.toml` |
| `ERROR: ...` ao rodar `scan` | Falta `config/private/profile.json` ou `sources.json`. Comece pela `demo` |
| Workflow não aparece em Actions | Na sua cópia, habilite os workflows na aba **Actions** |
| Não consigo rodar o workflow no repositório original | É normal: só o dono pode disparar workflows. Use a sua cópia |

---

**Próximos passos:** leia a [Arquitetura](ARQUITETURA.md) para entender as decisões técnicas e a página de [Segurança e publicação](SEGURANCA_E_PUBLICACAO.md) antes de configurar integrações reais.
