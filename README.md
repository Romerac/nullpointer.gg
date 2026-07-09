# Nullpointer.gg

Site de compartilhamento de promoções de jogos digitais — Next.js 14 (App Router), TypeScript, Tailwind CSS e Supabase.

## Stack

- **Next.js 14** (App Router) + **React 18** + **TypeScript**
- **Tailwind CSS** — tema dark customizado (veja `tailwind.config.ts`)
- **Supabase** — Postgres, Auth (somente admins) e Storage (imagens dos jogos)
- **Framer Motion** — animações discretas
- **Sonner** — toasts
- **lucide-react** — ícones (armazenados como string no banco, resolvidos em runtime por `DynamicIcon`)

## Estrutura do projeto

```
src/
  app/
    page.tsx                 # Home (grid de jogos, filtros, busca)
    layout.tsx                # Layout raiz + metadata/SEO
    not-found.tsx              # 404 customizada
    sitemap.ts / robots.ts     # SEO dinâmico
    admin/
      login/page.tsx           # Login (fora do grupo protegido)
      (protected)/
        layout.tsx              # AdminSidebar + guarda de layout
        page.tsx                 # Dashboard
        games/page.tsx           # CRUD de jogos
        categories/page.tsx      # CRUD de categorias (lojas)
        social/page.tsx          # CRUD de redes sociais
        settings/page.tsx        # Conta do admin
  components/
    layout/                   # Header, Sidebar
    games/                    # GameCard, GameGrid, Skeleton, EmptyState
    filters/                  # PriceFilterPopover, SortDropdown
    search/                   # SearchBar
    admin/                    # AdminSidebar, GameForm
    ui/                       # Button, DynamicIcon
  hooks/                      # useGames, useStores, useSocialLinks, useDebounce
  services/                   # Camada de acesso a dados (Supabase)
  lib/supabase/               # Clientes browser/servidor
  types/database.ts           # Tipos compartilhados
supabase/schema.sql           # Schema completo + RLS + triggers + seed
middleware.ts                 # Protege /admin/* exigindo sessão autenticada
```

## Configurando o Supabase

1. Crie um projeto em [supabase.com](https://supabase.com).
2. Abra o **SQL Editor** e execute todo o conteúdo de `supabase/schema.sql`. Isso cria:
   - `stores`, `social_links`, `games`, `price_history`
   - Políticas de RLS (leitura pública, escrita restrita a usuários autenticados)
   - Trigger que registra automaticamente o histórico de preço a cada alteração e calcula `historic_low`
   - Bucket de Storage `game-images` (público para leitura, restrito para escrita)
   - Categorias e redes sociais iniciais (seed)
3. Em **Authentication → Users**, crie manualmente o(s) usuário(s) administrador(es) — o cadastro público de admins foi propositalmente deixado fora do site.
4. Copie `.env.example` para `.env.local` e preencha com a URL e a chave anônima do seu projeto.

## Rodando localmente

```bash
npm install
cp .env.example .env.local   # preencha com suas credenciais do Supabase
npm run dev
```

Acesse `http://localhost:3000` para o site público e `http://localhost:3000/admin` para o painel administrativo.

## Deploy na Vercel

1. Suba o projeto para um repositório Git.
2. Importe o repositório na Vercel.
3. Configure as variáveis de ambiente (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `NEXT_PUBLIC_SITE_URL`) no painel do projeto.
4. Deploy. O `middleware.ts` e as rotas dinâmicas (`sitemap.ts`, `robots.ts`) funcionam nativamente no runtime da Vercel.

## Funcionalidades implementadas

- Header fixo com logo, busca instantânea (debounced) e botão de admin
- Sidebar com categorias, redes sociais e filtro de preço (popover), 100% gerenciáveis pelo painel
- Grid responsivo (4 colunas desktop / 2 tablet / 1 mobile) com infinite scroll, skeleton loading e empty state
- Cards com selos de Oferta Relâmpago, Menor Preço Histórico e Destaque, desconto calculado automaticamente
- Ordenação (mais recentes, maior desconto, menor/maior preço, nome A-Z/Z-A)
- Painel administrativo protegido por Supabase Auth (middleware redireciona não autenticados)
- CRUD completo de jogos (com upload de imagem para o Storage), categorias (com reordenação por drag-and-drop) e redes sociais
- **Histórico de preços** (`price_history`): toda alteração de preço gera um registro automaticamente via trigger no banco, e o selo de "menor preço histórico" é recalculado sozinho. A tabela já está pronta para alimentar, no futuro: gráfico de evolução de preço, "há X dias custava R$ Y", alertas de queda de preço e rankings semanais/mensais — sem precisar mexer na arquitetura atual.
- SEO: metadata dinâmica, Open Graph, sitemap.xml e robots.txt gerados automaticamente
- Acessibilidade: labels ARIA, foco visível, navegação por teclado, `prefers-reduced-motion` respeitado

## Próximos passos sugeridos

- Componente de gráfico (ex. Recharts) consumindo `services/priceHistoryService.ts` para exibir a evolução do preço na página do jogo
- Página de detalhe do jogo (`/jogo/[slug]`) — o sitemap já está preparado para essa rota
- Alertas de queda de preço por e-mail (Supabase Edge Functions + cron)
- Testes automatizados (Playwright para E2E, Vitest para unidades)

## Notas de qualidade

- Tipagem TypeScript estrita (`strict: true`, `noUncheckedIndexedAccess`)
- Camada de serviços isolada da UI — trocar Supabase por outro backend no futuro exige mexer só em `src/services`
- RLS no Postgres é a linha de defesa real (não apenas o middleware do Next.js), então mesmo chamadas diretas à API do Supabase respeitam as mesmas regras
- Substitua `public/favicon.ico.txt` por um favicon real antes do deploy
