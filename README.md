# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

**Grupo:** StackHunters

| Integrante | RM | Turma |
|---|---|---|
| Matheus Morelli | 562765 | 2CCPH |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | 26 testes, 0 falhas |

---

## Parte 1 — Bugs encontrados

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | O atendimento criado pelo Builder ficava com o nome do pet `null`, mesmo informando o nome em `comPet()`. | `AtendimentoBuilder.java`, linhas ~23-27, método `comPet()`. O código fazia `petNome = petNome`, atribuindo o parâmetro a ele mesmo. | Alterado para `this.petNome = petNome`, armazenando corretamente o valor no atributo do Builder. | Builder, encapsulamento e uso do `this`. |
| bug02 | O Builder permitia construir atendimentos sem nome ou porte do pet. | `AtendimentoBuilder.java`, linhas ~42-55, método `construir()`. Não existia validação dos campos obrigatórios. | Foram adicionadas validações para nome e porte, lançando `IllegalArgumentException` quando ausentes ou vazios. | Builder, validação de dados e invariantes de objeto. |
| bug03 | Ao solicitar um atendimento do tipo `TOSA`, a Factory devolvia um objeto `Banho`. | `AtendimentoFactory.java`, método `criar()`. O `case "TOSA"` instanciava `new Banho(...)`. | O case foi corrigido para criar `new Tosa(...)`. | Factory e polimorfismo. |
| bug04 | A consulta veterinária era criada sem os dados recebidos e não inicializava corretamente o status e os atributos herdados. | `ConsultaVeterinaria.java`, linhas ~16-18. O construtor utilizava apenas `super()`. | Alterado para `super(protocolo, petNome, petPorte, tutorNome, dataHora)`. | Herança e chamada de construtor da superclasse. |
| bug05 | Chamadas consecutivas ao `GeradorProtocolo.getInstancia()` não garantiam a mesma instância do Singleton. | `GeradorProtocolo.java`, método `getInstancia()`. Uma nova instância era retornada sem ser armazenada em `instancia`. | A nova instância passou a ser atribuída a `instancia` antes do retorno. | Padrão Singleton e estado compartilhado. |
| bug06 | Um segundo atendimento para o mesmo pet e mesmo horário conseguia passar pela verificação de conflito em determinadas situações. | `AgendaService.java`, método `agendar()`. Strings e `LocalDateTime` eram comparados com `==`. | As comparações passaram a utilizar `.equals()`, verificando o conteúdo dos objetos. | Igualdade de objetos, `==` versus `.equals()`. |
| bug07 | Ao buscar um ID inexistente, o service retornava `null` em vez de propagar `AtendimentoNaoEncontradoException`. | `AgendaService.java`, método `buscarPorId()`. Um `catch (Exception)` capturava a exceção e retornava `null`. | O `try/catch` genérico foi removido e o `orElseThrow()` passou a propagar a exceção corretamente. | Tratamento de exceções e exceções customizadas. |
| bug08 | Os preços de banho por porte estavam incorretos: o porte pequeno recebia o maior valor e o grande recebia o menor. | `Banho.java`, método `calcularPreco()`. Os valores de PEQUENO e GRANDE estavam invertidos. | Os valores foram ajustados para R$ 60,00, R$ 80,00 e R$ 100,00 para PEQUENO, MEDIO e GRANDE. | Regras de negócio e polimorfismo. |
| bug09 | A Tosa retornava duração de 30 minutos, herdada de `Atendimento`, em vez dos 60 minutos definidos no contrato. | `Tosa.java`, método de duração. Existia `getDuracaoMinutos(String porte)`, que era uma sobrecarga e não uma sobrescrita. | O método foi alterado para `getDuracaoMinutos()` e recebeu `@Override`, retornando 60. | Sobrescrita, sobrecarga e polimorfismo. |
| bug10 | Era possível tentar agendar um atendimento com data e hora no passado, chegando inclusive à camada de persistência. | `AgendaService.java`, início do método `agendar()`. Não havia validação temporal antes do acesso ao repository. | Foi adicionada validação com `LocalDateTime.now()`, lançando `IllegalArgumentException` antes de qualquer consulta ao repository. | Validação de regra de negócio e separação de responsabilidades. |
| bug11 | Um atendimento já `CONCLUIDO` ou `CANCELADO` podia ser cancelado novamente. | `Atendimento.java`, método `cancelar()`. O método alterava diretamente o status para `CANCELADO` sem conferir o estado atual. | Foi adicionada validação permitindo o cancelamento apenas quando o status é `AGENDADO`, lançando `StatusInvalidoException` nos demais casos. | Máquina de estados, regras de negócio e exceções. |
| bug12 | O campo `id` da entidade JPA estava marcado apenas com `@Id`, sem estratégia automática de geração. | `Atendimento.java`, linhas ~14-16, atributo `id`. Não havia `@GeneratedValue`. | Foi adicionada `@GeneratedValue(strategy = GenerationType.IDENTITY)`. | JPA, persistência e geração de chave primária. |

---

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory.java` | Os parâmetros do método `criar()` tinham nomes de apenas uma letra (`p`, `t`, `n`, `po`, `tu`, `d`), dificultando a leitura e manutenção do código. | Os parâmetros foram renomeados para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. |
| clean02 | `AgendaService.java` | O repository era injetado diretamente no atributo com `@Autowired`, deixando a dependência menos explícita e o campo mutável. | Foi aplicada injeção por construtor e o atributo passou a ser `final`. |
| clean03 | `AtendimentoController.java` | O service também era injetado diretamente no campo com `@Autowired`. | O `AgendaService` passou a ser recebido pelo construtor e armazenado em um atributo `final`. |
| clean04 | `GeradorProtocolo.java` | O Singleton escrevia diretamente no console em seu construtor, adicionando efeito colateral desnecessário. | O `System.out.println("GeradorProtocolo criado!")` foi removido. |
| clean05 | `AgendaService.java` | A camada de serviço imprimia um recibo diretamente com `System.out.println`, misturando regra de negócio com saída de console. | A impressão direta foi removida, mantendo o método responsável apenas pelo agendamento e retorno do atendimento salvo. |
| clean06 | `AtendimentoController.java` | Existia código morto para uma funcionalidade futura de fidelidade que não era utilizada pelo sistema. | O método privado `calcularDescontoFidelidade()` e seus comentários de funcionalidade futura foram removidos. |

---

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCalcularPrecoDoBanhoConformePorte()` | Banho deve custar R$ 60,00 para PEQUENO, R$ 80,00 para MEDIO e R$ 100,00 para GRANDE. | **Vermelho.** Revelou o bug08, pois os preços de PEQUENO e GRANDE estavam invertidos. |
| teste02 | `TosaTest.deveDurar60Minutos()` | Todo atendimento de Tosa deve possuir duração de 60 minutos. | **Vermelho.** Revelou o bug09, pois o método da classe Tosa era uma sobrecarga e a duração herdada era 30 minutos. |
| teste03 | `ConsultaVeterinariaTest.deveManterPrecoFixoQuandoPorteDoPetMudar()` | Consulta veterinária deve custar R$ 150,00 independentemente do porte do pet. | **Verde de primeira.** A regra de preço fixo já estava implementada corretamente. |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoNoPassadoSemConsultarBanco()` | Data/hora no passado deve gerar `IllegalArgumentException` e o repository não deve ser consultado. | **Vermelho.** Revelou o bug10, pois não existia validação de data no service. |
| teste05 | `AgendaServiceTest.deveCancelarAtendimentoAgendado()` | Um atendimento com status `AGENDADO` deve poder mudar para `CANCELADO`. | **Verde de primeira.** O caminho válido de cancelamento já funcionava corretamente. |
| teste06 | `AgendaServiceTest.deveRecusarCancelamentoQuandoStatusNaoForAgendado()` | Atendimentos `CONCLUIDO` ou `CANCELADO` devem rejeitar nova tentativa de cancelamento com `StatusInvalidoException`. | **Vermelho.** Revelou o bug11, pois `cancelar()` alterava qualquer status diretamente para `CANCELADO`. |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)

A suíte de testes funcionou como a especificação executável do PetFiap, porque cada falha mostrava uma diferença concreta entre o comportamento esperado e o comportamento real.  
Por exemplo, quando o Builder deveria manter o nome `Rex` e o resultado era `null`, fomos diretamente ao método `comPet()` e encontramos `petNome = petNome` em vez de `this.petNome = petNome`.  
Também utilizamos falhas de Factory, Singleton, conflito de horário e busca por ID para localizar a causa antes de alterar o código.  
Depois de cada correção, a suíte inteira era executada novamente para garantir que a mudança não criasse regressões em funcionalidades que já estavam funcionando.  
Em comparação com testes manuais usando `curl`, os testes JUnit são repetíveis, rápidos e verificam automaticamente tanto o retorno quanto interações com dependências.  
Isso permitiu validar o sistema sem iniciar o Spring, sem banco Oracle e sem repetir manualmente dezenas de requisições.

### 2. Mock e injeção de dependência (Aulas 13 a 15)

No `AgendaServiceTest`, o `@Mock` cria uma implementação falsa do `AtendimentoRepository`, controlada pelo Mockito.  
O `@InjectMocks` usa esse mock para montar o `AgendaService`, fazendo nos testes o papel de fornecer sua dependência.  
Na aplicação real, esse trabalho é feito pelo container do Spring, que encontra o `AtendimentoRepository` e o injeta no `AgendaService`.  
Durante o Clean Code, alteramos o service para injeção por construtor, deixando `repository` como `final`; como existe apenas um construtor, o Spring consegue utilizá-lo sem necessidade de `@Autowired` explícito.  
O teste não precisa de Oracle porque operações como `findById()`, `findByPetNome()` e `save()` são simuladas pelo mock.  
Também não é necessário subir o contexto Spring, fazendo com que a suíte seja mais rápida e verdadeiramente unitária.

### 3. `==` vs `.equals()` (Aula 7)

O bug de conflito de horário acontecia porque `AgendaService` utilizava `==` para comparar `String` e `LocalDateTime`.  
Em Java, `==` verifica se duas variáveis apontam para a mesma referência na memória, e não se os objetos possuem o mesmo conteúdo.  
Uma String literal como `"Rex"` pode aparentemente funcionar com `==` por causa do String Pool, em que literais iguais podem compartilhar a mesma referência, mas não é uma comparação segura de conteúdo.  
O problema ficou evidente com `LocalDateTime`, pois dois objetos diferentes podem representar exatamente o mesmo instante e ainda assim falhar na comparação com `==`.  
O teste criou propositalmente outro objeto `LocalDateTime` com o mesmo valor para reproduzir esse cenário.  
A correção substituiu as comparações por `.equals()`, fazendo o conflito depender dos valores do pet e horário e não das referências dos objetos.

### 4. Sobrescrita vs sobrecarga (Aula 7)

Na classe `Atendimento`, o método de duração possui a assinatura `getDuracaoMinutos()` e retorna 30 minutos por padrão.  
A classe `Tosa` possuía um método `getDuracaoMinutos(String porte)`, que parecia relacionado, mas possuía uma assinatura diferente.  
Isso caracteriza sobrecarga (`overload`): foi criado outro método, sem substituir o método herdado da superclasse.  
Por isso, quando o sistema chamava `getDuracaoMinutos()` em uma Tosa, o método herdado continuava sendo utilizado e devolvia 30 minutos.  
A correção implementou exatamente `getDuracaoMinutos()` na classe `Tosa`, retornando 60 minutos.  
Ao adicionar `@Override`, o próprio compilador passa a avisar caso a assinatura não corresponda a um método da superclasse, ajudando a impedir esse tipo de bug.

### 5. Singleton manual vs bean do Spring (Aula 14)

O `GeradorProtocolo` utiliza um Singleton manual porque precisa manter uma única instância responsável pela sequência global dos protocolos.  
O bug estava em `getInstancia()`: quando `instancia` era `null`, o código criava e retornava `new GeradorProtocolo()`, mas não guardava esse objeto no atributo estático.  
Com isso, chamadas seguintes podiam gerar novas instâncias e perder o estado do contador.  
A correção passou a fazer `instancia = new GeradorProtocolo()` e depois retornar sempre essa referência.  
O `AgendaService` não precisa implementar esse controle manual porque é anotado com `@Service` e seu ciclo de vida é administrado pelo container do Spring, cujo escopo padrão cria um único bean por contexto.  
Isso evita o mesmo erro de instanciação manual, embora o fato de um objeto ser Singleton não torne automaticamente todos os seus estados thread-safe.

### 6. Cobertura de testes: onde parar? (Aula 15)

Vale a pena manter também os testes que ficaram verdes de primeira, porque eles documentam regras importantes e impedem que uma alteração futura quebre um comportamento que hoje está correto.  
Neste projeto, o preço fixo da consulta e o cancelamento de um atendimento `AGENDADO` já funcionavam, mas agora essas regras possuem proteção automatizada contra regressões.  
Os quatro testes que ficaram vermelhos também mostraram que cobertura não serve apenas para medir quantidade de linhas executadas, mas para encontrar comportamentos incorretos.  
Em um projeto real com prazo limitado, eu priorizaria regras críticas de negócio, caminhos de erro e pontos onde uma falha poderia causar maior impacto.  
Depois garantiria os principais caminhos felizes e casos de borda relevantes.  
Buscar 100% de cobertura apenas como número não seria prioridade; testes úteis, determinísticos e ligados aos riscos reais do sistema são mais importantes.

---

## Parte 5 — Espaço livre (opcional)

A principal dificuldade do checkpoint foi perceber que nem todos os bugs apareciam nos testes originais. Foi necessário comparar o contrato com a suíte existente, criar os seis testes adicionais e também revisar o código manualmente. Alguns bugs estavam em cascata, então uma correção permitia que outro problema aparecesse com mais clareza. A estratégia de realizar alterações pequenas e um commit por correção facilitou a identificação das causas e evitou misturar mudanças diferentes no histórico do Git.