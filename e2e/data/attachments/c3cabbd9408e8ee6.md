# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: autenticacao/login.spec.ts >> Autenticação via Authentik >> login com usuário multi-vínculo redireciona para seleção de condomínio
- Location: tests/autenticacao/login.spec.ts:5:7

# Error details

```
Test timeout of 30000ms exceeded.
```

```
Error: page.waitForURL: Test timeout of 30000ms exceeded.
=========================== logs ===========================
waiting for navigation until "load"
  navigated to "http://localhost:4200/login"
============================================================
```

# Page snapshot

```yaml
- generic [ref=e4]:
  - banner [ref=e5]:
    - img [ref=e7]
    - generic [ref=e11]: Integra Condo
  - generic [ref=e12]:
    - heading "Bem-vindo(a) ao Integra Condo!" [level=1] [ref=e13]
    - paragraph [ref=e14]: Você será redirecionado com segurança para o Authentik, nosso provedor de identidade, para concluir o login.
  - generic [ref=e16]:
    - paragraph [ref=e17]: Continue com sua conta Integra Condo
    - link "Entrar no Sistema" [ref=e18] [cursor=pointer]:
      - /url: /oauth2/authorization/authentik
  - generic [ref=e19]:
    - generic [ref=e20]: © 2026
    - generic [ref=e21]: A3S It Solutions LTDA
```

# Test source

```ts
  1  | import { test, expect } from '@playwright/test';
  2  | import { epic, feature, story, severity, Severity, tag } from 'allure-js-commons';
  3  | 
  4  | test.describe('Autenticação via Authentik', () => {
  5  |   test('login com usuário multi-vínculo redireciona para seleção de condomínio', async ({ page }) => {
  6  |     await epic('Autenticação');
  7  |     await feature('Login via Authentik (BFF-Session)');
  8  |     await story('Login com múltiplos vínculos ativos leva à seleção de condomínio');
  9  |     await severity(Severity.CRITICAL);
  10 |     await tag('smoke');
  11 | 
  12 |     const senha = process.env.AUTHENTIK_QA_ADMIN_PASSWORD;
  13 |     if (!senha) {
  14 |       throw new Error(
  15 |         'AUTHENTIK_QA_ADMIN_PASSWORD não definido. Configure no .env local ou como secret de CI.',
  16 |       );
  17 |     }
  18 | 
  19 |     await test.step('Acessar a tela de login do shell', async () => {
  20 |       await page.goto('/login');
  21 |       await expect(page.getByRole('link', { name: 'Entrar no Sistema' })).toBeVisible();
  22 |     });
  23 | 
  24 |     await test.step('Clicar em "Entrar no Sistema" e ser redirecionado ao Authentik', async () => {
  25 |       await page.getByRole('link', { name: 'Entrar no Sistema' }).click();
> 26 |       await page.waitForURL(/localhost:9000\/.*flow/);
     |                  ^ Error: page.waitForURL: Test timeout of 30000ms exceeded.
  27 |     });
  28 | 
  29 |     await test.step('Preencher usuário (qa-admin) no formulário nativo do Authentik', async () => {
  30 |       // Seletor baseado no stage "identification" padrão do Authentik (campo uidField).
  31 |       // VALIDAR em runtime — a UI do Authentik é de terceiro, fora deste monorepo, e pode
  32 |       // divergir conforme customização do flow.
  33 |       await page.locator('input[name="uidField"]').fill('qa-admin');
  34 |       await page.getByRole('button', { name: /continuar|log ?in|next/i }).click();
  35 |     });
  36 | 
  37 |     await test.step('Preencher senha e confirmar', async () => {
  38 |       await page.locator('input[name="password"]').fill(senha);
  39 |       await page.getByRole('button', { name: /continuar|log ?in|entrar/i }).click();
  40 |     });
  41 | 
  42 |     await test.step('Validar redirect pós-login para a seleção de condomínio', async () => {
  43 |       // qa-admin tem 4 vínculos ativos (SINDICO, CONDOMINO, COLABORADOR_INTERNO, PRESTADOR_EXTERNO)
  44 |       // no mesmo condomínio — o guard condominioSelecionadoGuard sempre redireciona para esta tela
  45 |       // em vez de ir direto a /sindico.
  46 |       await page.waitForURL(/\/selecionar-condominio$/);
  47 |       await expect(page.locator('.selecionar-condominio-page__title')).toHaveText(
  48 |         'Escolha seu condomínio',
  49 |       );
  50 |       await expect(page.locator('.selecionar-condominio-page__logada-como')).toContainText(
  51 |         'Logada como',
  52 |       );
  53 |     });
  54 | 
  55 |     await test.step('Validar que os vínculos ativos do qa-admin aparecem na lista', async () => {
  56 |       await expect(page.locator('.selecionar-condominio-page__item')).toHaveCount(4);
  57 |     });
  58 |   });
  59 | });
  60 | 
```