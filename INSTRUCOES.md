# Sistema de Cotação Piratruck — instalação (Hostinger + Firebase)

O sistema é um único arquivo (`index.html`). O login e o histórico compartilhado usam o seu Firebase; a hospedagem é na Hostinger. Siga na ordem.

## 1. Configurar o Firebase (uns 10 minutos)

Acesse https://console.firebase.google.com e abra o seu projeto (ou crie um, ex.: `piratruck-cotacao`).

### 1.1 Ativar o login por e-mail e senha
1. Menu **Criação** (Build) → **Authentication** → **Vem começar** / **Sign-in method**.
2. Ative o provedor **E-mail/senha** e salve.
3. Na aba **Users**, clique em **Adicionar usuário** e cadastre cada consultor com e-mail e senha (é você quem define a senha inicial de cada um).

### 1.2 Criar o banco (Firestore)
1. Menu **Criação** → **Firestore Database** → **Criar banco de dados**.
2. Escolha o modo **produção** e a região `southamerica-east1` (São Paulo).
3. Na aba **Regras**, apague o conteúdo e cole o que está no arquivo `firestore.rules` deste pacote. Clique em **Publicar**. (Essas regras deixam ler e gravar cotações apenas quem estiver logado.)

### 1.3 Pegar a configuração do app
1. Engrenagem ⚙ → **Configurações do projeto** → seção **Seus apps**.
2. Clique em **</>** (Adicionar app da Web), dê um nome (ex.: `cotacao`) e registre.
3. Copie o objeto `firebaseConfig` mostrado (apiKey, authDomain, projectId...).
4. Abra o `index.html` num editor de texto, procure `FIREBASE_CONFIG` (logo no começo do script, está marcado com "COLE AQUI") e substitua os valores pelos do seu projeto.

### 1.4 Autorizar o seu domínio
Em **Authentication** → **Settings** → **Domínios autorizados**, adicione o domínio onde o sistema vai ficar (ex.: `cotacao.piratruck.com.br` ou o domínio da Hostinger). Sem isso o login é recusado.

## 2. Publicar na Hostinger

**Hospedagem de site (hPanel):**
1. hPanel → **Gerenciador de arquivos** → pasta `public_html` (ou crie um subdomínio, ex.: `cotacao.seudominio.com.br`, e use a pasta dele).
2. Envie o `index.html` já com o FIREBASE_CONFIG preenchido.
3. Acesse pelo navegador — use sempre **https://**.

**Ou no VPS (onde rodam os bots):** coloque o `index.html` na pasta servida pelo seu nginx/apache (ex.: `/var/www/html/cotacao/`) e aponte um subdomínio para ela, com certificado (certbot).

## 3. Como funciona no dia a dia

- Cada consultor entra com o e-mail e a senha cadastrados no Firebase. O nome do campo "Consultor" é preenchido automaticamente a partir do login (pode ser editado).
- **Salvar cotação** grava no histórico compartilhado; **Imprimir / PDF** salva e abre a impressão do navegador (aí é só escolher "Salvar como PDF").
- **Histórico** mostra as cotações de todos, com filtro por consultor, cliente e período; dá para reabrir, reimprimir e excluir.
- Cada cotação guarda também o e-mail de quem salvou (campo `usuario` no banco).

## Dicas

- Para trocar a senha de alguém: Firebase → Authentication → Users → ⋮ → Redefinir senha.
- Para atualizar a lista de preços: me mande a planilha nova no Claude que eu gero um novo `index.html` — basta substituir o arquivo na Hostinger (a configuração do Firebase eu mantenho se você me enviar o config ou o arquivo em uso).
