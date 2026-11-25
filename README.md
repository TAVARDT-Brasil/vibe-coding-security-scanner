# Auditor de Segurança Vibe Coding e Arquitetura de IA

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-brasil.github.io/vibe-coding-security-scanner/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Security](https://img.shields.io/badge/Security-AI%20Architecture-blue.svg)](https://tavardt-brasil.github.io/vibe-coding-security-scanner/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guia corporativo de arquitetura defensiva, mitigação de vazamento de credenciais e ferramental CLI determinístico para inspeção de segurança em aplicações desenvolvidas com ferramentas de inteligência artificial (Vibe Coding).

***

## Panorama Executivo: A Realidade de Segurança do Vibe Coding

A transição acelerada para o desenvolvimento de software assistido por inteligência artificial (utilizando ferramentas como Cursor, Lovable, v0, Bolt, Windsurf, Replit, Claude Code e GitHub Copilot) permitiu que equipes de engenharia e fundadores solos criem aplicações web completas em tempo recorde. No entanto, geradores autônomos de código priorizam o funcionamento visual imediato em detrimento do isolamento perimetral, do princípio do menor privilégio e do saneamento de artefatos de compilação.

Quando desenvolvedores solicitam que modelos de IA criem recursos em alta velocidade, o código resultante herda com frequência vulnerabilidades arquiteturais graves:
* Configurações de depuração e desenvolvimento vazam para a compilação final em produção.
* Chaves mestras de banco de dados e tokens de IA são expostos em bundles estáticos baixados pelo navegador.
* Tabelas em bancos relacionais são criadas sem regras declarativas de autorização (Row Level Security).
* Cabeçalhos defensivos de proteção web são omitidos dos servidores de aplicação e proxies.

Este repositório estabelece um padrão de arquitetura defensiva corporativa, fornecendo modelos de configuração prontos para uso em frameworks modernos e um utilitário determinístico em Python (`audit_vibe_security.py`) para identificar e mitigar essas vulnerabilidades antes da publicação.

***

## Superfície Detalhada de Risco em Aplicações Criadas por IA

### 1. Vazamento de Sourcemaps em Produção (.js.map)
Ferramentas de IA costumam configurar empacotadores frontend (Vite, Webpack, Turbopack, Rollup) com configurações padrão que geram mapas de código completos junto com os scripts minificados. Ao serem implantados em servidores públicos ou redes de entrega de conteúdo (CDNs), os arquivos `.js.map` viabilizam a desobfuscação reversa do código:
* Reconstrução exata do código fonte original em TypeScript e JavaScript.
* Mapeamento completo dos modelos de dados internos, esquemas de banco e interfaces da API.
* Exposição de endpoints administrativos ocultos, rotas internas e comentários deixados durante o desenvolvimento.

### 2. Chaves de Provedores de IA e Tokens BaaS Compilados no Frontend
Como os motores de IA frequentemente ignoram os limites físicos entre o ambiente do servidor e o navegador do cliente, credenciais confidenciais são embutidas diretamente no código cliente. Quando variáveis de ambiente com prefixos como `NEXT_PUBLIC_` ou `VITE_` são inseridas em componentes frontend, seus valores literais são compilados nos arquivos JavaScript finais:
* Chaves de API de OpenAI, Anthropic e Google Gemini com consumo de faturamento irrestrito.
* Chaves `service_role` e tokens JWT do Supabase que ignoram completamente as permissões de usuário.
* Credenciais de acesso à nuvem AWS, chaves de API do Google Cloud e webhooks de pagamento.

### 3. Ausência de Row Level Security (RLS) em Bancos de Dados Relacionais
A cultura do Vibe Coding estimula a comunicação direta entre o navegador e o banco de dados via SDKs modernos. Embora muito ágil, as IAs rotineiramente criam tabelas PostgreSQL sem executar o comando `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`. Sem políticas obrigatórias de RLS:
* Qualquer visitante que inspecione a URL do projeto no navegador pode realizar consultas REST e GraphQL diretas.
* Bancos inteiros de clientes, cadastros de usuários, dados pessoais (LGPD) e transações financeiras podem ser extraídos por requisições anônimas.

### 4. Ausência de Cabeçalhos HTTP de Segurança Perimetral
Aplicações criadas por IA raramente configuram cabeçalhos de proteção HTTP por padrão, deixando o sistema vulnerável a ataques no lado do cliente:
* `Content-Security-Policy` (CSP): Fundamental para impedir a injeção e execução de scripts maliciosos de terceiros.
* `Strict-Transport-Security` (HSTS): Garante a utilização exclusiva de transporte criptografado HTTPS.
* `X-Frame-Options` e `X-Content-Type-Options`: Bloqueiam enquadramentos indevidos (Clickjacking) e ataques baseados em interpretação de tipo MIME.

### 5. Rotas Expostas de Arquivos de Ambiente (.env)
Servidores web mal configurados deixam rotas como `/.env`, `/.env.local` e `/.env.production` acessíveis publicamente via HTTP, permitindo a extração instantânea de credenciais corporativas com uma única requisição.

***

## Ferramenta Oficial de Auditoria CLI

Aplicações executadas em navegadores sofrem limitações rigorosas de isolamento (sandbox) e restrições de CORS que impedem scripts web de inspecionar cabeçalhos de outros domínios ou ler respostas de forma irrestrita. Para fornecer uma auditoria técnica reprodutível e com zero telemetria externa, a TAVARDT desenvolveu um utilitário em Python para execução local via terminal ou em esteiras de integração contínua (CI/CD).

### Fluxo de Inspeção Determinística

O utilitário (`audit_vibe_security.py`) executa um ciclo defensivo em cinco etapas:
1. **Auditoria de Perímetro Raiz:** Conecta via TLS, valida o código de resposta HTTP e verifica a presença dos cabeçalhos defensivos recomendados.
2. **Descoberta de Scripts no DOM:** Analisa o HTML para mapear dinamicamente todos os arquivos de script carregados na aplicação.
3. **Verificação de Sourcemaps:** Testa a existência de arquivos `.js.map` correspondentes para cada script identificado.
4. **Inspeção de Segredos em Bundles:** Aplica expressões regulares de alta precisão no código baixado para identificar chaves ativas de IA, tokens administrativos e credenciais de nuvem.
5. **Varredura de Rotas de Ambiente:** Testa se arquivos críticos de ambiente (`/.env`, `/.env.local`, `/.env.production`) estão bloqueados ou expostos.

### Instruções de Execução

Baixe e execute o utilitário determinístico diretamente contra a aplicação alvo:

```bash
# 1. Obter o script oficial do repositorio
curl -sO https://raw.githubusercontent.com/TAVARDT-Brasil/vibe-coding-security-scanner/main/audit_vibe_security.py

# 2. Executar auditoria direta contra a aplicacao
python audit_vibe_security.py https://sua-aplicacao.com
```

***

## Modelos de Mitigação Defensiva em Produção

O repositório fornece configurações homologadas para proteger a infraestrutura da aplicação:

### Configuração Vite (`vite.config.ts`)
Desativa completamente os sourcemaps de produção e sanitiza artefatos:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    sourcemap: false,
    chunkSizeWarningLimit: 1000,
    rollupOptions: {
      output: {
        manualChunks: undefined
      }
    }
  }
});
```

### Configuração Next.js (`next.config.js`)
Desativa mapas de código no navegador e injeta cabeçalhos de segurança corporativos:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  productionBrowserSourceMaps: false,
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' }
        ]
      }
    ];
  }
};

module.exports = nextConfig;
```

### Endurecimento de Row Level Security no Supabase (`supabase-rls.sql`)
Isola o acesso aos registros e revoga permissões anônimas de extração em massa:

```sql
/* 1. Ativacao obrigatoria de Row Level Security em todas as tabelas */
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.transactions ENABLE ROW LEVEL SECURITY;

/* 2. Revogacao de privilegios irrestritos da role anonima */
REVOKE ALL ON ALL TABLES IN SCHEMA public FROM anon;

/* 3. Criacao de politica estrita de leitura associada a identidade do usuario */
CREATE POLICY "Usuarios visualizam apenas seus proprios registros" 
ON public.profiles FOR SELECT 
TO authenticated 
USING (auth.uid() = id);
```

### Regra de Proteção Perimetral no Cloudflare WAF
Regra de borda para descarte imediato de varreduras automatizadas:

```text
(http.request.uri.path contains ".js.map") or 
(http.request.uri.path contains ".env") or 
(http.request.uri.path contains ".git")
Acao: Block (Descartar conexao imediatamente)
```

***

## Perguntas Frequentes de Arquitetura (FAQ Técnico)

### Quais os principais riscos de segurança originados pelo Vibe Coding?
Geradores de código baseados em IA estruturam soluções com base em padrões de probabilidade funcional, e não em modelagem formal de ameaças. Com frequência, eles posicionam credenciais secretas no cliente, esquecem de ativar Row Level Security no banco de dados e mantêm opções de depuração ativas em produção.

### Por que a publicação de sourcemaps (.js.map) em produção é prejudicial?
Os sourcemaps contêm o mapeamento inverso exato do código minificado para o código fonte original. Publicar esses arquivos permite que invasores compreendam a lógica interna de negócios, descubram parâmetros de validação e mapeiem rotas administrativas sem qualquer dificuldade técnica.

### Como consumir APIs de inteligência artificial de forma segura no frontend?
Aplicações cliente jamais devem chamar provedores de IA diretamente com chaves mestras. Todas as interações devem ser intermediadas por rotas de backend protegidas ou funções de borda (serverless edge functions) que autentiquem o usuário, apliquem limitação de taxa (rate limiting) e mantenham as chaves em variáveis de ambiente restritas ao servidor.

### A ferramenta determinística em Python causa instabilidade na aplicação auditada?
Não. O utilitário realiza exclusivamente diagnósticos passivos e não destrutivos de leitura. Ele emite requisições HTTP GET e HEAD convencionais para inspecionar arquivos públicos e cabeçalhos, sem alterar registros ou sobrecarregar a infraestrutura de destino.

***

## Conformidade Regulatória e Padrões Corporativos

Arquitetura defensiva e controles técnicos alinhados à Lei Geral de Proteção de Dados (LGPD, Lei nº 13.709/2018), às diretrizes de segurança da ANPD e aos controles de integridade de software do CIS Controls v8.

***

## Repositório & Documentação Oficial

* **Página Oficial:** [https://tavardt-brasil.github.io/vibe-coding-security-scanner/](https://tavardt-brasil.github.io/vibe-coding-security-scanner/)
* **Portal TAVARDT:** [Portal Institucional](https://ag.tavardt.com/br/)
* **Ferramentas:** [Central de Ferramentas](https://tools.tavardt.com/br/)

***

## Links Úteis

* **Portal Corporativo:** https://ag.tavardt.com/br/
* **Suíte de Diagnósticos:** https://tools.tavardt.com/br/
* **Contato Técnico:** https://ag.tavardt.com/br/contato/

***

### Licença
Distribuído sob a Licença MIT. Consulte o arquivo [LICENSE](LICENSE) para obter detalhes completos.

***

© 2026 TAVARDT. Todos os direitos reservados.
