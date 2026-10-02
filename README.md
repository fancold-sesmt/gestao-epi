# Fluxo de EPI — Fancold

Sistema de solicitação, validação, separação e entrega de EPI, rodando em
**GitHub Pages** (as páginas) + **Firebase** (login e banco de dados).

```
index.html            o sistema
firebase-config.js    a chave do seu projeto Firebase  <- você preenche
firestore.rules       as regras de segurança do banco  <- você cola no console
importar.html         carga inicial dos dados (roda uma vez)
dados-iniciais.json   o que já existia no sistema antigo
.nojekyll             faz o GitHub Pages servir tudo sem processar
```

---

## 1. Criar o projeto no Firebase

1. Abra <https://console.firebase.google.com> e clique em **Criar projeto**.
   Nome sugerido: `fancold-epi`. Pode desativar o Google Analytics.
2. Dentro do projeto, menu **Criação > Authentication > Vamos começar**.
   Em **Sign-in method**, habilite **E-mail/senha** e salve.
3. Menu **Criação > Firestore Database > Criar banco de dados**.
   Escolha **Iniciar no modo de produção** e a região `southamerica-east1` (São Paulo).
4. Clique na engrenagem > **Configurações do projeto** > role até **Seus apps** >
   ícone `</>` (Web). Dê um apelido (`epi-web`) e **registre o app**.
   A tela mostra um bloco `const firebaseConfig = { ... }`.
5. Copie os valores para dentro do `firebase-config.js` deste repositório.

> Esses valores não são senha. Eles só dizem qual é o projeto. Quem fecha a porta
> são as regras do passo 3 e o login do passo 2.

## 2. Colar as regras de segurança

No console: **Firestore Database > Regras**. Apague o que estiver lá, cole o
conteúdo de `firestore.rules` e clique em **Publicar**.

As regras dizem, em resumo:
- ninguém sem login lê ou grava nada;
- quem tem login mas não tem perfil no sistema também não acessa nada;
- usuário desativado perde o acesso na hora;
- só quem tem `papel: "admin"` cria, edita ou apaga usuários.

## 3. Criar o seu acesso de administrador

Como as regras só deixam um admin criar usuários, o primeiro tem que ser feito à mão.

1. **Authentication > Users > Adicionar usuário**: coloque o seu e-mail e uma senha.
   Copie o **UID** que aparece na lista (uma sequência de letras e números).
2. **Firestore Database > Iniciar coleção**:
   - ID da coleção: `usuarios`
   - ID do documento: **cole o UID do passo anterior**
   - campos:

   | Campo   | Tipo    | Valor                      |
   |---------|---------|----------------------------|
   | `nome`  | string  | Gustavo Alves de Arruda    |
   | `email` | string  | o mesmo e-mail do passo 1  |
   | `papel` | string  | `admin`                    |
   | `ativo` | boolean | `true`                     |

3. Salve. A partir daqui todos os outros usuários você cadastra pela aba
   **Usuários** do próprio sistema, sem voltar ao console.

## 4. Publicar no GitHub

1. Crie um repositório novo em <https://github.com/new>. Sugestão: `fluxo-epi`.
   Pode ser **privado** — o GitHub Pages de repositório privado exige conta paga,
   então para publicar grátis deixe **público**. Os dados continuam protegidos:
   eles não estão no repositório, estão no Firestore atrás de login.
2. Envie os arquivos desta pasta para a raiz do repositório
   (botão **Add file > Upload files**, arraste tudo, **Commit changes**).
3. No repositório: **Settings > Pages**.
   Em *Source* escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`. Salve.
4. Espere um minuto. O endereço aparece no topo da mesma tela:
   `https://SEU-USUARIO.github.io/fluxo-epi/`
5. Volte ao Firebase em **Authentication > Settings > Domínios autorizados** e
   adicione `SEU-USUARIO.github.io`. Sem isso o login é recusado.

## 5. Trazer os dados que já existiam

Abra `https://SEU-USUARIO.github.io/fluxo-epi/importar.html`, entre com o seu
e-mail de admin e clique em importar. Ele grava as 15 solicitações, os 53
colaboradores, os 11 movimentos de estoque e as 2 fotos de evidência.

Depois disso o `importar.html` não é mais necessário — se quiser, apague o
arquivo do repositório.

## 6. Cadastrar a equipe

Entre no sistema, aba **Usuários**. Para cada pessoa: nome, e-mail, papel e uma
senha inicial. Passe o e-mail e a senha para ela; na primeira entrada ela pode
trocar a senha no bloco **Minha senha**, na mesma aba.

Papéis e o que cada um enxerga:

| Papel | Abas |
|---|---|
| Administrador | todas, incluindo Usuários |
| Segurança do Trabalho | Solicitação, Controle, Dashboard, Pendências, Colaboradores, Estoque, Importar |
| Almoxarifado ADM | Controle, Dashboard, Colaboradores, Estoque, Importar |
| Almoxarifado Operacional | Controle, Dashboard, Estoque |
| Supervisor | Solicitação (e as fichas que ele mesmo enviou) |

---

## O que mudou em relação à versão que rodava no Claude

- **Login de verdade.** A senha agora é verificada pelo servidor do Firebase, não
  pelo navegador. Ninguém consegue ler a senha de ninguém, nem olhando o código.
- **Entrada por e-mail**, não mais por `nome.sobrenome` — é assim que o Firebase
  Auth identifica a conta, e é o que permite o "Esqueci minha senha".
- **Esqueci minha senha** manda um link por e-mail. Na aba Usuários, o botão
  antes chamado "Nova senha" agora dispara esse mesmo link.
- **O e-mail não pode ser editado** depois de criado: ele é a identidade da conta.
  Para trocar, cadastre outro usuário e desative o antigo.
- **Excluir um usuário** remove o perfil e corta o acesso, mas o login continua
  existindo em Authentication. Para apagar de vez, remova também lá no console.

## Custo

Tudo isso cabe no plano gratuito (Spark) do Firebase: 50 mil leituras e 20 mil
gravações por dia no Firestore, e autenticação por e-mail/senha sem limite
prático. O GitHub Pages é gratuito para repositório público.
