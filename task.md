S1-01: Repositório e processo (fácil)

A pasta ordemcerta/ já existe e contém docs/PRD.md. Ela será a raiz do repositório.

1. Inicialize o Git ali e crie o .gitignore com tudo o que o PRD lista. O mais importante.env, que não pode entrar
2. Faça o primeiro commit direto na main, com o .gitignore, um README mínimo e o PRD. Essa é a única vez que você comita direto na main.
3. Crie o repositório público no GitHub e faça o push.
4. Configure o ruleset da main em Settings → Rules → Rulesets: exigir pull request, bloquear force push e bloquear exclusão da branch. A exigência de status checks fica para depois do S1-11: o GitHub só deixa selecionar um check depois que ele rodou pelo menos uma vez.
5. Crie os labels, a milestone "Sprint 1", o quadro no Projects (colunas da seção 12.5) e uma issue para cada tarefa S1-xx.
6. Crie os templates tarefa.md e pull_request_template.md com as seções que o PRD pede.

Documentação: no GitHub Docs, procure "About rulesets" e "Planning and tracking with Projects".

S1-02: Ambiente e dependências (fácil)

1. Crie o venv com Python 3.12 e instale as dependências da seção 9.1 nas versões exatas.
2. Monte os arquivos. O base.txt lista só as dependências de produção com as transitivas fixadas (asgiref e sqlparse). O dev.txt começa referenciando o base.txt e acrescenta as ferramentas de desenvolvimento.
   - Cuidado: não jogue o pip freeze inteiro num arquivo só, como no projeto de receitas. A ideia é cada arquivo conter só o que pertence a ele.
3. No pyproject.toml, crie duas seções:
   - Ruff: os grupos de regras citados, a exclusão das pastas de migrations e o tamanho de linha.                                                                              coverage: de onde medir e próprios testes, settings,manage.py, wsgi/asgi).
4. O grupo DJ (flake8-django) vai cobrar coisas como "todo model precisa de __str__". Isso é ótimo, porque já casa com o PRD.

Documentação: Ruff, página "Configuring Ruff" (docs.astral.sh/ruff); coverage.py, página "Configuration reference".

S1-03: Projeto e settings por ambiente (médio)

1. Use o startproject com o nome config, gerando os arquivos na própria raiz (não numa subpasta).
2. Não rode migrate ainda. O runserver vai reclamar de migrations pendentes. Ignore por enquanto.
3. Transforme o settings.py em um pacote settings/ com base, dev, test e prod. Cada arquivo de ambiente importa tudo do base e sobrescreve só o que muda.
4. Armadilhas desta tarefa:
   - BASE_DIR: ao mover o arquivo um nível para dentro, ele passa a apontar para o lugar errado, porque agora precisa subir um diretório a mais. Esse é o erro número um dessa refatoração.
   - Variáveis de ambiente chegam sempre como texto. "False" como string é verdadeiro em Python. Você precisa converter DJANGO_DEBUG para booleano de forma explícita. Já DJANGO_ALLOWED_HOSTS vem como texto separado por vírgulas e precisa virar lista.
   - Comportamento sem chave secreta (CA1.2): em produção, a ausência da chave deve levantar ImproperlyConfigured (de django.core.exceptions). Em desenvolvimento, o dev.py pode ter um valor padrão inseguomo tal. O test.py tambémprecisa de uma chave própria, porque no CI não existe .env.
   - Pontos de entrada: manage.py aponta por padrão para config.settings.dev; wsgi.py e asgi.py, para config.settings.prod. O valor padrão só vale se a variável DJANGO_SETTINGS_MODULE não estiver definida.
5. Ainda nesta tarefa:
   - LANGUAGE_CODE e TIME_ZONE;
   - TEMPLATES com DIRS apontando para a pasta templates/ da raiz;
   - as configurações de static e media;
   - MESSAGE_TAGS com nomes semânticos;
   - o .env.example com todas as variáveis da seção 6.4 e valores de exemplo, nunca reais.
6. O python-dotenv deve carregar o .env no base.py, antes da leitura das variáveis.

Documentação: "Settings" (docs.djangoproject.com/en/5.2/topics/settings/) e "Deployment checklist" (/en/5.2/howto/deployment/checklist/).

S1-04: App core e model abstrato (fácil)

1. Crie a app core e escreva nela o TimeStampedModel.
   - O que o torna abstrato é uma opção na classe Meta interna. Sem ela, o Django criaria uma tabela vazia inútil.
   - Use verbose_name em português nos dois campos.
2. Crie o PageTitleMixin: um mixin para CBVs com um atributo de classe para o título e um get_context_data que o coloca no contexto como page_title.
   - Lembre de sempre chamar o super() dentro do get_context_data. É isso que permite
     combinar mixins.
3. Crie o BusinessRuleError em exceptions.py, por enquanto só a classe base.
4. O core não terá migrations com tabelas, e isso é normal.

Documentação: "Models → Abstract base classes" (/en/5.2/topics/db/models/#abstract-base-classes).

S1-05: Custom User Model (difícil, a tarefa mais importante da sprint)

1. Crie a app accounts.
2. O model: o User herda de AbstractUser (de django.contrib.auth.models) e do seu TimeStampedModel.
   - Para remover o username, você atribui nulo ao campo na própria classe.
   - Redefina o email como único.
   - Ajuste USERNAME_FIELD e REQUIRED_FIELDS. A lista de obrigatórios não pode conter o próprio campo de login, senão o Django acusa erro.
   - Adicione phone sem validador por enquanto. O validador chega na S2.
3. O manager: como o UserManager padrão do Django exige username, você precisa do seu próprio, herdando de BaseUserManager.
   - O padrão da documentação é ter um método interno que cria e salva o usuário, mais create_user e create_superuser chamando esse método com os valores certos de is_staff e is_superuser.
   - O create_user deve levantar ValueError sem e-mail e usar o normalize_email que o BaseUserManager já oferece. Ele padroniza o domínio do e-mail, o que importa para o CA1.4.
   - Use o set_password para a
   - O create_superuser deve recusar is_staff ou is_superuser falsos.
4. O admin, onde quase todo mundo trava: o UserAdmin do Django menciona username em vários lugares: ordering, list_display, search_fields, fieldsets e add_fieldsets. Você precisa sobrescrever todos, senão o check do Django acusa erro.
   - Os formulários padrão do admin (UserCreationForm e UserChangeForm) também apontam para o model auth.User com o campo username. Crie subclasses deles cujo Meta aponte para o seu model e use o e-mail, e informe-as ao admin pelos atributos add_form e form.
5. Settings: AUTH_USER_MODEL no base.py.
6. roles.py: o enum Role com TextChoices.
7. Só agora rode o makemigrations e depois o migrate. Se você rodou migrate antes por engano, apague o db.sqlite3 e comece de novo. Esse é exatamente o motivo da ordem.
8. Teste manual: o createsuperuser deve pedir e-mail, nome e sobrenome, e o login no admin deve funcionar com e-mail.

Documentação: "Customizing authentication", seções "Substituting a custom User model" e "A full example" (/en/5.2/topics/auth/customizing/). O exemplo completo de lá é a sua referência principal. Leia, entenda cada parte e adapte; não copie.

S1-06: Esqueleto das demais apps (fácil)

Para cada uma das 6 apps:

1. Use o startapp.
2. Ajuste o verbose_name em português no AppConfig.
3. Registre a app em INSTALLED_
4. Troque o tests.py por uma pasta tests/ com __init__.py. Sem esse arquivo, o test runner não encontra os testes.
5. Crie o urls.py com app_name, o que dá o namespace da app.
6. Ligue as URLs no config/urls.py com include, usando os prefixos da seção 7.

Organize o INSTALLED_APPS em três blocos (Django, terceiros, apps do projeto). Ajuda muito a leitura.

Documentação: "URL dispatcher → URL namespaces" (/en/5.2/topics/http/urls/#url-namespaces).

S1-07: Views iniciais (fácil)

São três TemplateView:

- HomeView: simples, com título.
- StyleguideView: responde 404 (Http404) quando o DEBUG está desligado. O melhor lugar para essa checagem é o dispatch, porque ele roda antes de qualquer método HTTP.
- UnderConstructionView: uma única classe reaproveitada em 6 rotas. O que muda entre elas é o section_name (e o título). Repare que o as_view aceita atributos da classe como argumentos nomeados, e existe também o extra_context.

Os nomes das rotas já são os definitivos (customers:list e os demais), para os links do menu não quebrarem quando as views reais chegarem na S2.

Templates provisórios: o Frontentão crie templates mínimos noscaminhos do contrato, só exibindo as variáveis, para testar.

Documentação: "Base views → TemplateView" (/en/5.2/ref/class-based-views/base/).

S1-08: Context processor de navegação (médio)

Um context processor é uma função comum: recebe o request e devolve um dicionário, que se mistura ao contexto de todos os templates. Você a registra na lista de context_processors dentro de TEMPLATES.

1. Monte a lista nav_items com o formato exato do contrato (seção 14.0), usando reverse para as URLs.
2. Para saber qual item está ativo, compare o nome da rota atual com o de cada item. O request.resolver_match traz o namespace e o nome da rota.
3. Armadilha: em uma página 404 não existe rota resolvida, e o resolver_match vem nulo. O processor roda mesmo assim, porque o template de 404 também usa contexto, e não pode quebrar nesse caso.
4. O user_role fica None por enquanto.

Documentação: "Templates API → Writing your own context processors" (/en/5.2/ref/templates/api/).

S1-09: Logging (fácil)

1. Monte o dicionário LOGGING n
   - um formatter no formato do PRD;
   - um handler de console;
   - o logger django;
   - um logger para cada app, com o nível vindo da variável de ambiente.
2. Mantenha disable_existing_loggers desligado. Ligado, ele silencia loggers que já existiam.
3. No test.py, deixe o logging silencioso para não poluir a saída dos testes.

Documentação: "Logging" (/en/5.2/topics/logging/), especialmente os exemplos de configuração.

S1-10: Docker (médio)

1. Dockerfile:
   - imagem python:3.12-slim;
   - as duas variáveis de ambiente do PRD (entenda o que cada uma faz);
   - copiar primeiro só os arquivos de requirements e instalar, e só depois copiar o código. Essa ordem aproveita o cache de camadas: mudar o código não reinstala as dependências.
   - criar e usar um usuário não-root.
   - No WSL: se o UID do usuário do container for diferente do seu, arquivos criados pelo container (como o db.sqlite3 e as migrations) podem ficar com dono errado. Pesquise como alinhar o UID.
2. docker-compose.yml: serviço web com o runserver escutando em todas as interfaces (sem isso, o servidor fica inacessível de fora do container), o bind mount do código, env_file
   e a porta.
3. Makefile: alvos que chamam os comandos que você já usa.

Documentação: Docker Docs, "Dockerfile reference" e "Compose file reference".

S1-11: CI (médio)

1. Crie um workflow em .github/workflows/ disparado em push e pull request, com dois jobs:
   - lint: Ruff check e format check.
   - test:
     - checagem de migrations pendentes;
     - testes rodando com as settings de teste (pelo coverage);
     - relatório de cobertura com o mínimo de 70%.
2. Use a action oficial de setup do Python, com cache de pip.
3. O CI não tem .env, então as settings de teste precisam funcionar sem ele (você resolveu isso no S1-03).
4. Depois do primeiro run verde, volte ao ruleset e marque lint e test como checks obrigatórios.

Documentação: GitHub Docs, "Building and testing Python".

S1-12: Testes (médio, junto com cada tarefa)

1. Manager: usuário sem e-mail falha; e-mail normalizado; superusuário com as flags certas; e-mail duplicado gera IntegrityError.
2. Datas automáticas: testadas
   - created_at é preenchido.
   - updated_at muda ao salvar de novo. Pense em como garantir que o tempo "passou" de forma confiável dentro de um teste.
3. Rotas: status 200 e assertTemplateUsed.
4. Styleguide: 404 com override_settings.
5. Context processor: o item ativo correto e o caso sem rota resolvida.
6. Fato útil: o test runner do Django força DEBUG=False durante os testes, independentemente das settings.

Documentação: "Testing tools" (/en/5.2/topics/testing/tools/).

S1-13: README v1 (fácil)

Escreva para alguém que nunca viu o projeto: o que é, os pré-requisitos, como rodar com venv (passo a passo, incluindo copiar o .env.example), como rodar com Docker, a tabela de variáveis, como rodar testes e lint, e um link para o PRD. O critério para saber se ficou bom: um clone novo, seguindo só o README, precisa funcionar.

Sobre o Frontend

O Especialista Frontend precisa que a estrutura exista: a pasta templates/, as rotas nomeadas e o context processor. Quando o S1-08 estiver na main, me avise. Ele então entrega o Tailwind, o base.html, os componentes e as páginas de erro numa branch ui/s1-base, e os seus templates provisórios são substituídos.
mo pedir ajuda e entregarTravou? Diga a tarefa e o nív- 1: o conceito;- 2: onde procurar;- 3: passo a passo em texto.- Eu só subo de nível se vocêEntrega: quando terminar a sp uma tarefa isolada, mande oslinks dos PRs ou avise. O repdev/ordemcerta e consigo lerdireto. O Code Reviewer faz a

Bom trabalho. Comece pelo S1-01





