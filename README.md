# TrocaFácil · WecTI

Aplicativo acadêmico Android para reutilização de materiais entre estudantes. MVP **Java + XML + SQLiteOpenHelper**, sem Kotlin, Compose, Firebase, backend ou comunicação entre aparelhos. Funciona sem internet depois de instalado.

O APK de debug pronto para instalar está incluído em **`apk/TrocaFacil-debug.apk`**, além da entrega separada do APK.

## Abrir e executar

1. Extraia o ZIP inteiro e abra a pasta **TrocaFacil** em **Android Studio → Open**.
2. Selecione o JDK incorporado do Android Studio em **Settings → Build Tools → Gradle → Gradle JDK**. Este projeto foi compilado com JDK 21.
3. No SDK Manager, instale **Android SDK Platform 35**, **Build Tools 35.0.0** e **Platform Tools**. Aceite as licenças e aguarde a sincronização Gradle.
4. O Android Studio cria `local.properties` com o caminho do SDK da sua máquina. Esse arquivo não faz parte do ZIP.
5. Selecione um celular/emulador **Android 8.0 (API 26) ou superior** e clique em **Run**.

A primeira sincronização/compilação pode precisar de internet para baixar o Gradle e o plugin Android. O aplicativo não solicita permissão de internet. Não é necessário cadastrar chaves ou criar um servidor.

### Pela linha de comando

PowerShell, dentro da pasta do projeto:

```powershell
$env:JAVA_HOME = 'C:\Program Files\Android\Android Studio\jbr'
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
.\gradlew.bat :app:assembleDebug :app:testDebugUnitTest :app:lintDebug --no-daemon --no-configuration-cache
```

Linux/macOS: configure `JAVA_HOME` e `ANDROID_HOME`, execute `chmod +x gradlew` e use `./gradlew` com as mesmas tarefas.

APK gerado: `app/build/outputs/apk/debug/app-debug.apk`. No Android Studio, também é possível usar **Build → Generate App Bundles or APKs → Generate APKs** (o nome do menu pode variar conforme a versão).

Para instalar com um dispositivo conectado e depuração USB autorizada:

```powershell
& "$env:ANDROID_HOME\platform-tools\adb.exe" install -r app\build\outputs\apk\debug\app-debug.apk
```

Também é possível copiar o APK para o celular e abri-lo, autorizando a instalação pelo gerenciador de arquivos. O APK de debug é para demonstração e testes.

## Contas de demonstração

Criadas automaticamente **somente na primeira criação do banco**. A senha de todas é **`Demo@123`**.

| Pessoa | E-mail | Itens iniciais |
| --- | --- | --- |
| Ana Souza | ana@aluno.unicid.edu.br | Livro de programação Java; kit de cadernos |
| Bruno Lima | bruno@aluno.unicid.edu.br | Calculadora científica; fone de ouvido |
| Carla Alves | carla@aluno.unicid.edu.br | Mochila universitária; Cálculo I |

Na tela de entrada, **Conhecer com a conta da Ana** preenche o acesso de demonstração. Para alternar contas, abra **Perfil → Sair / trocar de conta**. Todas as contas compartilham o mesmo banco neste aparelho. Fechar e reabrir mantém a sessão e os dados.

## Funcionalidades implementadas

- Cadastro com nome, e-mail institucional, curso, senha e confirmação. E-mail único, sem distinguir maiúsculas/minúsculas. Validação local de `.edu.br`, `.edu` e `unicid.com.br`; **não comprova matrícula, posse do e-mail ou identidade**.
- Login/logout e sessão local. Senhas com PBKDF2-HMAC-SHA256, 120 mil iterações e salt aleatório por senha, nunca texto puro.
- Publicação, consulta, edição e exclusão dos próprios anúncios. Título, descrição, categoria, condição, troca/doação, interesse desejado e uma imagem.
- Seleção de imagem pelo seletor de documentos do Android; cópia reduzida em armazenamento privado. Sem permissão ampla de galeria. As imagens iniciais são ilustrações demonstrativas incluídas no projeto.
- Busca por título, descrição e categoria, sem diferença de acentos/maiúsculas. Filtros de categoria e modalidade.
- Match por regras Java, com explicação de cada sugestão.
- Solicitações de troca com escolha do item oferecido; doação sem contrapartida. Ponto de encontro sugerido.
- Histórico de enviadas e recebidas, data, participantes, modalidade e status.
- Perfil editável, interesses, anúncios, avaliações e reputação calculada. Perfil público local de outros anunciantes.
- Avaliação de 1 a 5 estrelas e comentário opcional após a conclusão. Cada participante pode avaliar uma única vez por transação.

## Fluxo para apresentar

1. Entre como Ana. Em **Match**, a calculadora do Bruno recebe **100 pontos**: Ana procura calculadora e Bruno procura livros de programação.
2. Abra a calculadora → **Propor uma troca** → selecione o livro da Ana → mantenha ou edite o encontro na biblioteca → **Enviar**.
3. No histórico, a solicitação aparece como **Pendente**.
4. Saia pelo perfil e entre como Bruno. Abra **Histórico → Aceitar → Confirmar**. Os dois itens ficam reservados e deixam de aparecer na busca.
5. Após simular a entrega, Bruno usa **Concluir entrega → Confirmar**. A conclusão é definitiva neste MVP.
6. Bruno avalia a Ana. Depois entre como Ana e avalie Bruno. Abra o perfil de Bruno para visualizar a reputação recebida.
7. Para demonstrar doação, Ana solicita a mochila de Carla; Carla aceita e conclui pelo seu histórico. Nenhum item é oferecido em troca.
8. Para demonstrar cancelamento, crie outra solicitação; o solicitante pode cancelar enquanto pendente. Quando aceita, qualquer participante pode cancelar e os itens voltam a ficar disponíveis.

O banco de demonstração não é recriado ao abrir o aplicativo. Para reiniciar uma apresentação do zero, use **Configurações do Android → Apps → TrocaFácil → Armazenamento → Limpar dados**. Isso apaga contas, imagens e transações locais.

## Regras das solicitações

| Estado atual | Quem pode agir | Próximo estado |
| --- | --- | --- |
| Pendente | Dono do item solicitado | Aceita ou Recusada |
| Pendente | Solicitante | Cancelada |
| Aceita | Dono do item solicitado | Concluída |
| Aceita | Qualquer um dos dois participantes | Cancelada |
| Recusada, Concluída ou Cancelada | Ninguém | Estado final |

- Não é possível solicitar o próprio anúncio, oferecer item de outra pessoa ou oferecer um item de doação.
- Uma mesma pessoa não pode ter duas solicitações abertas para o mesmo item.
- Aceitar reserva ambos os itens em uma transação SQL. Solicitações pendentes concorrentes envolvendo qualquer um desses itens são canceladas automaticamente. Itens oferecidos por essas solicitações canceladas permanecem disponíveis.
- Cancelar uma solicitação aceita libera os itens. As solicitações concorrentes canceladas não são reabertas automaticamente.
- Não é possível editar/excluir anúncio com solicitação pendente ou aceita. Resolva as solicitações no histórico primeiro. Itens concluídos ficam no histórico/perfil e não podem ser anunciados de novo pelo mesmo registro.
- Exclusão é lógica (`EXCLUIDO`), preservando vínculos de solicitações anteriores.

## Banco de dados

Arquivo privado `trocafacil.db`, versão 1, gerenciado por `DatabaseHelper`. Chaves estrangeiras ativadas em `onConfigure()`.

| Tabela | Principais campos e relações |
| --- | --- |
| `users` | id, name, email único, course, password_hash, interests |
| `products` | id, owner_id → users, title, description, category, condition, mode, wanted, image, status |
| `requests` | id, requester_id → users, target_id → products, offered_id → products (opcional), status, meeting, created_at |
| `reviews` | id, request_id → requests, author_id/recipient_id → users, stars, comment; par request_id/author_id único |

Consultas e escritas usam parâmetros/ContentValues. Reservas, conclusão, cancelamento e avaliações usam transações para manter consistência. Restrições `CHECK`, `UNIQUE` e chaves estrangeiras complementam as validações Java. Sessão em `SharedPreferences`, guardando apenas o ID do usuário. Fotos são arquivos privados; o banco guarda o nome, não o bitmap.

## Como o match funciona

`MatchRules` remove acentos, converte para minúsculas e compara palavras com quatro ou mais caracteres. Remove `s` final como tratamento simples de plural. O texto consultado reúne título, categoria e descrição do produto.

Para cada item disponível de outro aluno, compara os anúncios disponíveis para troca do usuário:

- **100 pontos:** o interesse do meu anúncio combina com o item do outro e o interesse do outro combina com o meu.
- **60 pontos:** somente o interesse do meu anúncio combina com o item do outro.
- **30 pontos:** o item combina com os interesses do meu perfil, inclusive possíveis doações.

Mantém a melhor pontuação por anúncio e ordena do maior para o menor. São pontos de uma regra didática, **não porcentagens de probabilidade ou IA**. Categorias participam como texto; sinônimos e linguagem natural não são compreendidos. Uma sugestão não aceita nem envia proposta automaticamente.

## Organização

```text
app/src/main/
  java/br/edu/wecti/trocafacil/
    activities/  Telas e ações do usuário
    models/      User, Product, TradeRequest, Match
    database/    DatabaseHelper e regras de persistência
    adapters/    Apresentação dos cartões de produtos
    utils/       Senha, validação, sessão, imagens e match
  res/layout/    Interfaces XML
  res/drawable*/ Ícones, fundos e imagens demonstrativas
app/src/test/        Testes Java (JUnit)
app/src/androidTest/ Testes de integração com SQLite real no Android
docs/                Guia acadêmico, requisitos e validação
gradle/wrapper/      Gradle Wrapper completo
```

## Testes

```powershell
.\gradlew.bat :app:testDebugUnitTest :app:lintDebug --no-daemon --no-configuration-cache
# Com um emulador iniciado ou aparelho autorizado:
.\gradlew.bat :app:connectedDebugAndroidTest --no-daemon --no-configuration-cache
```

Os testes instrumentados usam um banco **`tests.db` isolado**, apagado entre os casos, sem alterar o banco normal do aplicativo. O runner de testes é o runner nativo do Android; não há framework externo de UI. Relatórios automáticos: `app/build/reports/tests/testDebugUnitTest/`, `app/build/reports/androidTests/connected/` e `app/build/reports/lint-results-debug.html`.

Consulte `docs/VALIDACAO.md` para os resultados realmente obtidos nesta entrega e `docs/GUIA_ACADEMICO.md` para explicar o código em sala.

## Referências e identidade visual

- Conversa **Registrar documentação WecTI**, recuperada para esta implementação: requisitos detalhados do MVP Java/XML/SQLite.
- `docs/referencias/Banner-WecTI.pdf`: contexto, identidade e protótipos. Foram preservados o azul principal, amarelo de destaque, fundo claro, cartões arredondados, tipografia simples, navegação inferior e organização das telas.
- Atualização aprovada no chat: logo em PNG transparente na entrada, no cabeçalho e no ícone do Android. A entrada usa o roxo `#2627B5`, amostrado do botão Entrar do [Figma de referência](https://www.figma.com/design/J8GTZstwFKR08DsF84w1zx/layout-troca-f%C3%A1cil?node-id=0-1). O arquivo está em `app/src/main/res/drawable-nodpi/logo_trocafacil_transparente.png`.
- O vídeo de demonstração entregue anteriormente foi gravado antes dessa atualização de logo e da entrada roxa; os fluxos funcionais continuam os mesmos.
- `docs/referencias/Documento (16).pdf`: roteiro original, problema, solução e sequência de apresentação.
- O arquivo `WecTI_TrocaFacil_Documentacao_Completa.pdf` aparece citado na conversa, mas seu arquivo não estava disponível. Os dois PDFs originais que o compõem foram recuperados e analisados integralmente; estão incluídos sem alteração.
- [SQLiteOpenHelper — Android](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper)
- [Seletor de documentos e arquivos — Android](https://developer.android.com/training/data-storage/shared/documents-files)
- [Compatibilidade do Android Gradle Plugin 8.11](https://developer.android.com/build/releases/agp-8-11-0-release-notes)

## Limitações e próximos recursos

MVP de apresentação acadêmica em um único dispositivo, com listas pequenas. Não sincroniza dados, não envia mensagens ou notificações reais, não confirma e-mail/matrícula, não implementa recuperação de senha ou moderação. Não existe negociação financeira, WhatsApp, pagamentos, IA, gamificação, ranking, troca coletiva ou modo urgente. Os pontos de encontro são texto sugerido, não locais verificados pela instituição. Essas ideias do banner/roteiro ficam como evolução futura.

Cada anúncio tem uma foto. As imagens são reduzidas e armazenadas localmente; arquivos de fotos substituídas ou de formulários abandonados podem permanecer até limpar os dados/desinstalar. Não há exportação/backup, criptografia integral do banco ou migração entre aparelhos; o backup automático do app foi desativado. A proteção de senha não transforma esta demonstração em serviço de autenticação de produção. O aplicativo coleta somente os dados informados localmente.

Tecnologias fixadas: Android Gradle Plugin 8.11.1, Gradle 8.14.3, código-fonte Java 11, compile/target SDK 35, min SDK 26 e JUnit 4.13.2 para testes locais. As APIs antigas do runner nativo e algumas APIs tradicionais de Activity são usadas deliberadamente para manter a proposta introdutória.
