# SITE OFFLINE — dudaline.com

## Site

`https://dudaline.com/` (VIP Line + Dudalu Uniformes)

## Motivo técnico

Site retirado temporariamente de produção por decisão administrativa relacionada ao serviço de hospedagem.

## Data da suspensão

- Preparação, backup e ponto de preservação Git: **2026-10-27**.
- Aplicação da camada 503 na Hostinger: **PENDENTE — ação manual** (ver "Como retirar do ar"). Registre aqui a data em que for aplicada: `____-__-__`.

> Status em 2026-10-27, no fim do trabalho automatizado: o site **continua ONLINE** (HTTP 200). A retirada exige acesso ao hPanel, que não estava disponível nesta sessão.

---

## Estado anterior (o que estava publicado)

| Item | Valor |
|---|---|
| Tecnologia | Site estático: HTML + Tailwind CSS compilado (`assets/css/tailwind.min.css`) + Alpine.js via jsDelivr. Sem backend, sem CMS, sem PHP. |
| Páginas | `index.html`, `vipline.html`, `dudalu.html`, `misterio.html`, `robots.txt`, `sitemap.xml`, `.htaccess` |
| Repositório | `https://github.com/pauloblum1-boop/site-dudalu-vipiline` (público). A raiz do repositório é a pasta `dist/`. |
| Branch de produção | `main` |
| Commit publicado | `9e73dda` — "feat: require correct mystery guess" (2026-07-16). Confirmado: o hash Git de todos os HTML/CSS/JS servidos é idêntico ao do commit. |
| Tag de preservação | `site-before-offline-2026` → `9e73dda` |
| Branch de preservação | `archive/site-before-offline-2026` (= `9e73dda` + este README) |
| Build | `npm install && npm run build` na pasta-mãe (Tailwind CLI 3.4 + `sharp` para WebP). Node instalado localmente; `package-lock.json` presente. |
| Hospedagem | Hostinger (hPanel), servidor LiteSpeed, com Hostinger CDN (`Server: hcdn`) na frente. |
| Domínio | Registrado na Hostinger. Criado em 2026-04-29, **expira em 2027-04-29**. |
| SSL | Let's Encrypt gerenciado pela Hostinger (validade observada em 15/09: até 2026-11-25, renovação automática). |

### Fluxo real de publicação

```
pasta local dist/  ──(manual)──>  Hostinger public_html  ──>  Hostinger CDN  ──>  dudaline.com
        │
        └── git push ──> GitHub (somente versionamento)
```

- **Sem GitHub Actions** (não há `.github/workflows` no repositório).
- **Script local** `npm run deploy` (`scripts/hostinger-deploy.mjs`): faz build, zipa `dist/` e envia pela API da Hostinger. Precisa de `HOSTINGER_API_TOKEN` em `.env.local`, que **não existe** nesta máquina. Por isso não há publicação automática por esse caminho.
- **Hostinger Git Deployment (GitHub → Hostinger): EXISTE** (confirmado no hPanel em 2026-10-27). Repositório `https://github.com/pauloblum1-boop/site-dudalu-vipiline.git`, ramo `main`, caminho `/` (= `public_html`). Ou seja, `public_html` é um clone do repositório. O "Implantar" manual, ou um push em `main` com a implantação automática ligada (webhook no GitHub), sobrescreve `public_html`, **incluindo o `.htaccess`**, e desfaz a suspensão.
- As imagens no servidor **não são byte-idênticas** às do Git: o servidor tem versões redimensionadas (ex.: `escolas/sb1.jpg` com 1600×2133 no servidor e 3024×4032 no Git). O Git guarda os originais em resolução maior. A cópia exata do servidor foi baixada (ver Backup).

### Banco de dados, CMS, variáveis de ambiente

- Banco de dados: **não existe** (site estático).
- CMS: **não existe**.
- Variáveis de ambiente: apenas para os scripts de deploy/DNS. O modelo está em `.env.example`, sem segredos. Nenhum `.env` real foi encontrado.
- Uploads e conteúdo gerado no servidor: nenhum identificado. A listagem de diretório do servidor é bloqueada (`Options -Indexes`), então só o backup do hPanel confirma 100%.
- Cron jobs e webhooks: não identificados localmente. Conferir no hPanel.

### DNS (consultado em 2026-10-27 via 8.8.8.8). NÃO ALTERAR.

| Registro | Valor |
|---|---|
| NS | `aster.dns-parking.com`, `helios.dns-parking.com` (Hostinger) |
| A `@` | `91.108.127.153`, `147.79.105.44` (Hostinger CDN) |
| AAAA `@` | `2a02:4780:17:5cee:d395:c8f7:89e0:9081`, `2a02:4780:18:6c0c:cb80:18a2:3d5d:a36a` |
| CNAME `www` | `www.dudaline.com.cdn.hstgr.net` |
| **MX** | `5 mx1.hostinger.com`, `10 mx2.hostinger.com` (**e-mail ativo na Hostinger**) |
| **TXT SPF** | `v=spf1 include:_spf.mail.hostinger.com ~all` |
| **DKIM** | `hostingermail-a/b/c._domainkey` → CNAME `hostingermail-*.dkim.mail.hostinger.com` |
| **DMARC** | `_dmarc` TXT `v=DMARC1; p=none` |

> Atenção: os IPs `195.35.41.169` / `2a02:4780:13:1294:0:b15:ce90:d` em `.env.example` e `DEPLOYMENT.md` estão **desatualizados** (de antes da ativação do CDN). Não rode `npm run domain:on` com eles sem revisar.

---

## Método escolhido

**Opção C: camada temporária HTTP 503 no `.htaccess` do servidor.** Motivos:

- A Hostinger **não oferece botão "pausar"** para sites estáticos (o modo manutenção do hPanel é só para WordPress).
- A opção "Alterar domínio / domínio temporário" do hPanel **remove contas de e-mail e subdomínios** do domínio. **PROIBIDA.**
- Alterar DNS (Opção D / `npm run domain:off`) é menos previsível com o CDN e arrisca o e-mail. **Descartada.**
- A camada 503 não apaga nada: são só um bloco de texto no `.htaccess` e um arquivo extra. Remover os dois devolve o site ao estado exato de antes.

O bloco preserva `/.well-known/` para a renovação do SSL continuar funcionando, e envia `Retry-After` e `noindex`.

---

## Como retirar do ar (ação manual no hPanel)

Arquivos prontos em `.offline-backup-2026-10-27/offline-layer/`:
`site-offline.html` e `htaccess-offline-block.txt`.

1. Acesse `https://hpanel.hostinger.com` → **Sites** → `dudaline.com` → **Gerenciar** (Manage).
2. **Antes de tudo, desligue republicação automática:** menu **Avançado → GIT → Gerenciar repositórios** → `⋮` da linha `site-dudalu-vipiline` (`main`, `/`) → **Implantação automática**. Anote a URL de webhook exibida: `______`. Em seguida, no GitHub, abra **Settings → Webhooks** (`https://github.com/pauloblum1-boop/site-dudalu-vipiline/settings/hooks`). Se houver webhook com essa URL, **desmarque "Active"** e salve. **Não** exclua o webhook. Nesse menu `⋮`, **não clique em "Remover"** (desconecta o repositório) **nem em "Implantar"** (republica `main` e desfaz a suspensão). Registre aqui: webhook existia? `___` Desativado em `____-__-__`.
3. **Backup do servidor:** menu **Arquivos → Backups** → gere e baixe o backup dos arquivos (`public_html`). Guarde o arquivo junto com esta pasta.
4. **Arquivos → Gerenciador de Arquivos** → `public_html`:
   1. Baixe uma cópia do `.htaccess` atual (clique direito → Download) e salve como `.htaccess.original-2026` no computador.
   2. Faça upload de `site-offline.html` para `public_html/`.
   3. Edite `.htaccess` e cole o conteúdo de `htaccess-offline-block.txt` **no topo do arquivo**, antes de tudo. Não apague nada do restante. Salve.
5. **Desempenho → CDN → Limpar cache** (Flush cache). Sem isso, o CDN pode continuar servindo páginas antigas.
6. Teste (seção "Verificação").

**NÃO clicar:** "Excluir site", "Alterar domínio"/"domínio temporário", "Redefinir conta", excluir arquivos de `public_html`, alterar nameservers, tocar em registros MX/TXT/CNAME de e-mail na Zona DNS, cancelar o domínio, desligar a renovação automática do domínio.

Se o `.htaccess` editado causar erro 500: abra o gerenciador de arquivos, remova o bloco colado e salve. O site volta ao normal na hora.

## Verificação

```bash
for u in https://dudaline.com/ https://www.dudaline.com/ http://dudaline.com/ http://www.dudaline.com/ https://dudaline.com/vipline.html https://dudaline.com/dudalu.html https://dudaline.com/misterio.html https://dudaline.com/assets/img/logo-vip.png; do
  printf '%s -> ' "$u"; curl -s -o /dev/null -w '%{http_code}\n' "$u"
done
```

O esperado é `503` em todos. Confirme também no navegador, em janela anônima, que aparece "Site temporariamente indisponível".

E-mail: envie e receba uma mensagem de/para um endereço `@dudaline.com` depois da mudança. Os registros MX/SPF/DKIM/DMARC acima não devem mudar (`nslookup -type=MX dudaline.com 8.8.8.8`).

---

## Arquivos alterados

**Nenhum arquivo do projeto foi alterado.** `dist/`, `src/`, `scripts/`, `package.json` e `.htaccess` do repositório estão intactos, e as mudanças locais não commitadas continuam no working tree.

Criados (fora de `dist/`, não publicados):

- `SITE_OFFLINE_README.md` (este arquivo)
- `.offline-backup-2026-10-27/live-snapshot/`: cópia dos 98 arquivos públicos servidos em 2026-10-27, como estavam no servidor
- `.offline-backup-2026-10-27/local-uncommitted/`: `uncommitted-changes.patch` + `copa2/4/5.png` (trabalho local não commitado, preservado)
- `.offline-backup-2026-10-27/offline-layer/`: `site-offline.html`, `htaccess-offline-block.txt`

## Infraestrutura alterada

- **GitHub:** só adições. Tag `site-before-offline-2026` e branch `archive/site-before-offline-2026`. `main` não foi tocada.
- **Hostinger:** nada alterado até 2026-10-27 (pendente, manual).
- **GitHub Actions:** não existem.
- **DNS:** nada alterado.

## O que foi preservado

Código ✔ · Assets (Git + cópia do servidor) ✔ · GitHub e histórico ✔ · Banco: não existe · Domínio ✔ · DNS ✔ · E-mail ✔ · Configurações (`.htaccess`, `tailwind.config.js`, `package-lock.json`, scripts) ✔

## Riscos ao cancelar a hospedagem

- **E-mail:** verifique em hPanel → **E-mails** se a caixa `@dudaline.com` é o e-mail gratuito **incluído no plano de hospedagem**. Se for, **cancelar a hospedagem pode encerrar o e-mail**. Nesse caso, contrate um plano de e-mail separado antes ou mantenha a hospedagem.
- **Arquivos:** com a hospedagem expirada, a Hostinger apaga `public_html` depois de um prazo. O backup do passo 3 e o Git cobrem isso.
- **Domínio:** expira em **2027-04-29**. Mantenha a renovação automática se o domínio for continuar com o cliente.
- Se a hospedagem for cancelada, a camada 503 some junto. O site continua fora do ar, mas sem a página neutra.

---

# COMO REATIVAR DUDALINE.COM

### Caso 1: hospedagem ainda ativa (só a camada 503 aplicada)

1. hPanel → Sites → `dudaline.com` → Gerenciador de Arquivos → `public_html`.
2. Edite `.htaccess` e apague tudo entre `# BEGIN SITE-OFFLINE-2026` e `# END SITE-OFFLINE-2026` (inclusive). Ou restaure o `.htaccess.original-2026` salvo.
3. Apague `public_html/site-offline.html`.
4. Desempenho → CDN → Limpar cache.
5. Se o webhook foi desativado no passo 2, reative-o: GitHub → Settings → Webhooks → marcar "Active". Opcionalmente, hPanel → Avançado → GIT → `⋮` → **Implantar**, para sincronizar `public_html` com `main`.
6. Rode os testes abaixo.

### Caso 2: hospedagem nova ou restaurada do zero

1. Contrate/reative a hospedagem na Hostinger e adicione o site `dudaline.com` (domínio já registrado na Hostinger).
2. Obtenha o código:
   ```bash
   git clone https://github.com/pauloblum1-boop/site-dudalu-vipiline.git dist
   cd dist && git checkout site-before-offline-2026   # versão exata que estava no ar
   ```
   A pasta-mãe (`package.json`, `src/`, `tailwind.config.js`, `scripts/`) **não está no GitHub**. Ela está nesta pasta local e nos pacotes `site-completo-entrega.zip` / `SITE_TRANSFER_PACKAGE.zip`. Só é necessária para recompilar o CSS. O `dist/` já está pronto para publicar.
3. (Opcional) Rebuild: na pasta-mãe, `npm install && npm run build`.
4. Envie todo o conteúdo de `dist/` (inclusive `.htaccess`, **sem** a pasta `.git`) para `public_html` pelo Gerenciador de Arquivos (upload de zip + extrair), ou configure `.env.local` com `HOSTINGER_API_TOKEN`/`HOSTINGER_USERNAME` e rode `npm run deploy`.
5. Para usar as imagens exatamente como estavam no servidor, use `.offline-backup-2026-10-27/live-snapshot/` ou o backup do hPanel.
6. DNS: só se a hospedagem nova tiver outro IP, ajuste **apenas** A/AAAA de `@` e CNAME de `www` na Zona DNS. **Nunca** mexa em MX/SPF/DKIM/DMARC.
7. SSL: hPanel → Segurança → SSL → instalar/ativar Let's Encrypt para `dudaline.com` e `www`.
8. CDN: ativar (opcional) e limpar cache.

### Testes após reativar

- [ ] `https://dudaline.com/` → 200
- [ ] `https://www.dudaline.com/` → 301 → `https://dudaline.com/`
- [ ] `http://dudaline.com/` e `http://www.dudaline.com/` → 301 → HTTPS
- [ ] Certificado válido (cadeado no navegador)
- [ ] `vipline.html`, `dudalu.html`, `misterio.html` → 200
- [ ] Imagens e CSS carregam (sem 404 no console do navegador)
- [ ] Botões de WhatsApp (`wa.me/5548996969795`) e tour 360° funcionando
- [ ] Formulário de cupom em `misterio.html` (sem backend: valida no navegador e abre o WhatsApp com a mensagem)
- [ ] Layout no celular
- [ ] `robots.txt` e `sitemap.xml` acessíveis
- [ ] `npm run health` (na pasta-mãe) sem falhas
- [ ] E-mail `@dudaline.com` enviando e recebendo
- [ ] Analytics: o site não usa ferramenta de analytics no código. Nada a reativar.
