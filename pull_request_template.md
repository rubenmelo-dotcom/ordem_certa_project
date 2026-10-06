- **O que foi feito**;
  S1-01: Repositório e processo (fácil)

A pasta ordemcerta/ já existe e contém docs/PRD.md. Ela será a raiz do repositório.

1. Inicialize o Git ali e crie o .gitignore com tudo o que o PRD lista. O mais importante.env, que não pode entrar
2. Faça o primeiro commit direto na main, com o .gitignore, um README mínimo e o PRD. Essa é a única vez que você comita direto na main.
3. Crie o repositório público no GitHub e faça o push.
4. Configure o ruleset da main em Settings → Rules → Rulesets: exigir pull request, bloquear force push e bloquear exclusão da branch. A exigência de status checks fica para depois do S1-11: o GitHub só deixa selecionar um check depois que ele rodou pelo menos uma vez.
5. Crie os labels, a milestone "Sprint 1", o quadro no Projects (colunas da seção 12.5) e uma issue para cada tarefa S1-xx.
6. Crie os templates tarefa.md e pull_request_template.md com as seções que o PRD pede.

Documentação: no GitHub Docs, procure "About rulesets" e "Planning and tracking with Projects".

  - **Por quê / issue relacionada** (`Closes #12`);
    
    issue 01-01
    
    Nesta sprint quase não há funcionalidade de negócio, e isso é proposital. Vamos construir o alicerce: tudo o que, se ficar errado agora, vai doer para consertar depois. O exemplo clássico é o Custom User Model.pois que o banco já tem tabelas.
  - **Como testar** (passo a passo);
  - **Checklist do DoD**;
  - **Prints** (quando houver tela).
