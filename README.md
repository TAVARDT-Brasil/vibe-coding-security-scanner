# Vibe Coding Security Scanner & Auditor de Aplicações Geradas por IA | TAVARDT Brasil

[![Licence MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Vibe Coding Security](https://img.shields.io/badge/Vibe%20Coding-Security%20Hardened-00d26a.svg)](README.md)
[![Language](https://img.shields.io/badge/Language-Português-green.svg)](README.md)
[![Engineering](https://img.shields.io/badge/TAVARDT-Cybersecurity-5b3cf5.svg)](https://ag.tavardt.com/br/ciberseguranca/)

Scanner de código aberto para auditoria de segurança em aplicações desenvolvidas via vibe coding (Cursor, Lovable, Bolt, v0). Detecta chaves expostas no frontend, falhas de RLS no Supabase e vazamento de sourcemaps.

## Aplicação Web Interativa

Acesse a ferramenta online e audite em tempo real:

- [https://tavardt-brasil.github.io/vibe-coding-security-scanner/](https://tavardt-brasil.github.io/vibe-coding-security-scanner/)

---

## 1. Visão Geral do Scanner de Segurança Vibe Coding

O ecossistema de desenvolvimento assistido por IA (*vibe coding*) permite criar e publicar aplicações completas em minutos com ferramentas como **Cursor, Lovable.dev, Bolt.new, v0 e Replit**.

No entanto, a ausência de revisão de arquitetura de segurança frequentemente expõe segredos críticos, bancos de dados abertos para o público e credenciais de produção no frontend.

Este repositório fornece uma ferramenta leve em Python para auditar **a superfície pública de qualquer aplicação web no ar**, apontando falhas reais antes que invasores descubram.

---

## 2. Vulnerabilidades e Vetores Auditados

| Vetor de Risco | Como Acontece no Vibe Coding | Impacto Técnico |
| :--- | :--- | :--- |
| **Chaves de API no Frontend** | Criadores colocam chaves da OpenAI (`sk-...`), Stripe ou AWS no `.env` do cliente ou direto no código React/Vue. | Qualquer visitante extrai a chave no DevTools e esgota sua cota de crédito. |
| **Supabase RLS Desativado** | O criador cria tabelas no Supabase sem ativar o *Row Level Security* (RLS). | Como a `anon_key` é pública, qualquer um pode baixar ou apagar a tabela inteira via REST. |
| **Sourcemaps em Produção** | O bundler (Vite/Next.js) é publicado com arquivos `.js.map` ativos. | O código-fonte original completo do projeto fica exposto para engenharia reversa. |
| **Arquivos Críticos Abertos** | Deploys mal configurados deixam rotas como `/.env` ou `/.git/HEAD` públicas. | Vazamento total de credenciais de infraestrutura e segredos de ambiente. |
| **Ausência de Headers** | Falta de políticas CSP, HSTS e X-Frame-Options. | Vulnerabilidade a Clickjacking, ataques Man-in-the-Middle e injeção XSS. |

---

## 3. Guia de Instalação e Execução

### Instalação Rápida

```bash
git clone https://github.com/TAVARDT-Brasil/vibe-coding-security-scanner.git
cd vibe-coding-security-scanner
```

### Executar Auditoria em uma URL

```bash
python scanner.py https://sua-aplicacao.vercel.app
```

### Opções Disponíveis

```bash
# Modo com relatório detalhado em JSON
python scanner.py https://sua-aplicacao.vercel.app --json

# Ignorar verificação de sourcemaps
python scanner.py https://sua-aplicacao.vercel.app --no-sourcemaps
```

---

## 4. Blindagem Profissional e Suporte Técnico

Se o diagnóstico apontou vazamento de chaves privadas, falhas de RLS em seu banco de dados Supabase/Firebase ou necessidade de arquitetura perimetral segura:

* **Portal Corporativo:** [TAVARDT Brasil](https://ag.tavardt.com/br/ciberseguranca/)
* **Atendimento Técnico Direto:** [Entrar em Contato com a Engenharia](https://ag.tavardt.com/br/contato/)

---

## Licença

Distribuído sob a licença de código aberto MIT. Consulte o arquivo [`LICENSE`](LICENSE) para mais detalhes.  
Copyright (c) 2026 TAVARDT Brasil.
