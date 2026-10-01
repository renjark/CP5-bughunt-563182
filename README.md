# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

**Grupo:** ___ (preencher)

| Integrante | RM | Turma |
|---|---|---|
| | | |
| | | |
| | | |
| | | |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 (+1 achado extra, ver nota abaixo da Parte 1) |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | 26 testes (20 entregues + 6 novos), 0 falhas |

---

## Parte 1 — Bugs encontrados

> Numeração = ordem em que os bugs foram encontrados/corrigidos (mesma dos commits `fix: bugNN`).
> Os bugs 01–07 apareceram nos 9 testes vermelhos; os bugs 08–11 só apareceram ao escrever os
> testes novos; o bug 12 só aparece lendo o código (code review).

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | `GeradorProtocoloTest`: `assertSame` falhou (duas instâncias diferentes) e `expected: <2> but was: <1>` — todo protocolo saía como 1 | `GeradorProtocolo.getInstancia()` (l. 18-19): fazia `return new GeradorProtocolo()` e nunca guardava em `instancia`, então cada chamada criava um gerador novo com contador zerado | `instancia = new GeradorProtocolo();` e depois `return instancia;` | Padrão Singleton (Aula 14) |
| bug02 | `AtendimentoFactoryTest.deveCriarTosaQuandoTipoForTosa`: esperava `Tosa`, veio `Banho` | `AtendimentoFactory.criar` (l. 17): `case "TOSA" -> new Banho(...)` (copiar/colar) | `case "TOSA" -> new Tosa(...)` | Padrão Factory (Aula 14), polimorfismo |
| bug03 | `AtendimentoBuilderTest.deveMontarAtendimentoCompleto`: `expected: <Rex> but was: <null>` | `AtendimentoBuilder.comPet` (l. 24): `petNome = petNome;` atribuía o parâmetro a ele mesmo; o atributo ficava `null` (faltava `this.`) | `this.petNome = petNome;` | `this`, escopo/sombreamento de variáveis (Aula 3-4), Builder |
| bug04 | `AtendimentoFactoryTest.devePreencherOsDadosDoPetNaConsulta`: nome do pet `null` (e o status também ficava `null`) | `ConsultaVeterinaria` (construtor, l. 17): chamava `super()` vazio em vez de `super(protocolo, petNome, ...)`, descartando os dados e pulando `status = "AGENDADO"` | `super(protocolo, petNome, petPorte, tutorNome, dataHora);` | Herança e construtores com `super(...)` (Aula 6-7) |
| bug05 | `deveRecusarMontagemSemNomeDoPet` e `...SemPorte`: `Expected IllegalArgumentException to be thrown, but nothing was thrown` | `AtendimentoBuilder.construir` (l. 39-43): não validava nada (o comentário jogava a responsabilidade para o controller); o objeto nascia inválido | `construir()` lança `IllegalArgumentException` se nome ou porte forem `null`/em branco | Builder: objeto só nasce válido; exceções (Aula 11) |
| bug06 | `AgendaServiceTest.deveRecusarAgendamentoComHorarioJaOcupado`: nenhuma `HorarioOcupadoException`; o `save` era chamado com horário duplicado | `AgendaService.agendar` (l. 23): `a.getPetNome() == novo.getPetNome() && a.getDataHora() == novo.getDataHora()` compara referências, não valores | `Objects.equals(...)` nos dois campos (null-safe) | `==` vs `equals()` (Aula 7) |
| bug07 | `deveLancarExcecaoQuandoAtendimentoNaoExiste`: nenhuma exceção; o método devolvia `null` (e `concluir/cancelar` dariam `NullPointerException`; o controller nunca devolveria 404) | `AgendaService.buscarPorId` (l. 37-42): `try { ...orElseThrow(...) } catch (Exception e) { return null; }` engolia a exceção | Removido o `try/catch`; a `AtendimentoNaoEncontradoException` chega a quem chamou | Exceções unchecked, catch genérico (Aula 11) |
| bug08 | (teste01, vermelho) `expected: <60.0> but was: <100.0>` — banho de porte PEQUENO cobrando 100 | `Banho.calcularPreco` (l. 27-32): valores invertidos (PEQUENO 100 / GRANDE 60); contrato é 60/80/100 | PEQUENO 60, MEDIO 80, GRANDE 100 | Polimorfismo / regra de negócio no model (Aula 7, 14) |
| bug09 | (teste02, vermelho) `expected: <60> but was: <30>` — duração da tosa caía no padrão da superclasse | `Tosa` (l. 40): `public int getDuracaoMinutos(String porte)` é **sobrecarga** (assinatura nova), não sobrescreve `getDuracaoMinutos()` | `@Override public int getDuracaoMinutos() { return 60; }` | Sobrescrita vs sobrecarga, `@Override` (Aula 7) |
| bug10 | (teste03, vermelho) `Expected StatusInvalidoException to be thrown, but nothing was thrown` — atendimento concluído podia ser cancelado | `Atendimento.cancelar` (l. 63-65): atribuía `CANCELADO` sem verificar o status (diferente de `concluir()`) | `cancelar()` só aceita `AGENDADO`; senão lança `StatusInvalidoException` | Encapsulamento das regras de status no model, exceções (Aula 11) |
| bug11 | (teste04, vermelho) `Expected IllegalArgumentException ... nothing was thrown` — agendava no passado e ainda consultava o repository | `AgendaService.agendar` (l. 20): nenhuma validação da data/hora antes de usar o repository | Primeira linha do método: `dataHora == null` ou `isBefore(now)` → `IllegalArgumentException` (antes de consultar o banco) | Validação de entrada, exceções (Aula 11), regra de negócio no service |
| bug12 | Nenhum teste vermelho: só aparece no code review. O comentário da classe diz "Thread-safe", mas não é | `GeradorProtocolo` (l. 17 e 24): `getInstancia()` com *check-then-act* sem sincronização e `contador++` não atômico — duas requisições simultâneas podem criar dois geradores ou receber o mesmo protocolo | `getInstancia()` e `proximo()` passam a ser `synchronized` | Singleton thread-safe, concorrência (Aula 14) |

**Achado extra (bug13, fora da contagem de 12):** `Atendimento` tinha `@Id private Long id;` sem `@GeneratedValue`. Com a API rodando contra o Oracle, o `repository.save(...)` de um atendimento novo falharia (o Hibernate espera que o id seja atribuído manualmente), ou seja, `POST /api/atendimentos` não funcionaria. Corrigido com `@GeneratedValue(strategy = GenerationType.IDENTITY)` em um commit próprio (`fix: bug13`). Conceito: JPA/Spring Data (Aula 13).

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory.criar(int p, String t, String n, String po, String tu, LocalDateTime d)` | Nomes que revelam a intenção: parâmetros de uma letra (e `po`/`tu` ambíguos) | Renomeados para `protocolo, tipo, petNome, petPorte, tutorNome, dataHora` |
| clean02 | `AgendaService.agendar` (`System.out.println("Recibo...")`) e construtor do `GeradorProtocolo` (`println("GeradorProtocolo criado!")`) | Log de produção não deve ser `System.out`; prints de debug são ruído | Logger SLF4J (`log.info` com placeholders) no service; print do singleton removido |
| clean03 | `"AGENDADO"`, `"CONCLUIDO"`, `"CANCELADO"` espalhados em `Atendimento` e `AgendaService`; `"BANHO"/"TOSA"/"CONSULTA"` repetidos na factory | Strings mágicas / duplicação (DRY): um erro de digitação compila e quebra a regra | Constantes `STATUS_AGENDADO/CONCLUIDO/CANCELADO` em `Atendimento`; a factory usa `Banho.TIPO`, `Tosa.TIPO`, `ConsultaVeterinaria.TIPO` |
| clean04 | `AtendimentoController.calcularDescontoFidelidade` (privado, nunca chamado) + bloco de comentário "Fidelidade (futuro)" | Código morto / comentário que descreve algo que não existe (e a fórmula nem batia com o comentário) | Método e comentário removidos (a ideia pode virar issue, não código morto) |
| clean05 | `@Autowired private ...` em `AgendaService` e `AtendimentoController` | Dependências escondidas, campo mutável, difícil de testar sem container | Injeção por construtor com campos `final` (o `@InjectMocks` do Mockito continua funcionando) |
| clean06 | `ResponseEntity.status(201)` e `status(409)` no controller | Números mágicos: o leitor precisa lembrar o que 201/409 significam | `HttpStatus.CREATED` e `HttpStatus.CONFLICT` |

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCustar100ReaisQuandoPorteForGrande` | Banho de porte GRANDE custa R$ 100 | **Vermelho** → revelou o bug08 (preço invertido) |
| teste02 | `TosaTest.deveDurar60Minutos` | Tosa dura 60 min | **Vermelho** → revelou o bug09 (sobrecarga em vez de sobrescrita) |
| teste03 | `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoJaConcluido` | `cancelar()` em CONCLUIDO recusa com `StatusInvalidoException` e não salva | **Vermelho** → revelou o bug10 |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoQuandoDataHoraNoPassado` | Data/hora no passado: `IllegalArgumentException` e o banco nem é consultado | **Vermelho** → revelou o bug11 |
| teste05 | `ConsultaVeterinariaTest.deveCobrar150ReaisQuandoPorteForGrande` | Consulta custa R$ 150 fixo, mesmo para porte GRANDE | **Verde** de cara (regra já estava correta) |
| teste06 | `AgendaServiceTest.deveAgendarQuandoMesmoHorarioMasAtendimentoAnteriorCancelado` | Só atendimento AGENDADO bloqueia o horário; CANCELADO não gera conflito | **Verde** de cara (regra já estava correta) |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)
Cada mensagem de falha apontou para um arquivo. `expected: <Rex> but was: <null>` (`AtendimentoBuilderTest`) mostrou que o nome do pet se perdia no Builder e nos levou direto ao `comPet`, onde `petNome = petNome;` não tinha `this.`. `expected: <2> but was: <1>` no `GeradorProtocoloTest` mostrou que o contador reiniciava a cada chamada, e o `assertSame` falhando confirmou: `getInstancia()` criava um `new GeradorProtocolo()` sem guardar. `Expected IllegalArgumentException to be thrown, but nothing was thrown` entregou a ausência de validação no `construir()`. Rodamos a suíte depois de cada correção: no começo 9 vermelhos, depois da factory/builder/consulta restaram 4 e, depois do `==` e do `catch`, zerou. Com curl teríamos de subir a API, ter Oracle no ar e disparar requisições à mão; os testes rodam em segundos sem banco, cobrem os caminhos de erro (409, 404, 400) com a mesma facilidade do caminho feliz, e continuam valendo como rede de segurança: depois dos 12 bugs, qualquer regressão acende a barra vermelha.

### 2. Mock e injeção de dependência (Aulas 13 a 15)
Em produção, o container do Spring cria o `AtendimentoRepository` real (um proxy do Spring Data ligado ao Oracle) e o injeta no `AgendaService`; hoje isso acontece pelo construtor (clean05), antes era via `@Autowired` no campo. No `AgendaServiceTest`, quem faz esse papel é o Mockito: o `@Mock` cria um repository falso e o `@InjectMocks` o entrega ao `AgendaService`, também pelo construtor. Como o service só depende da *interface* `AtendimentoRepository`, ele não sabe se do outro lado há o Oracle ou um objeto de mentira. Por isso o teste roda sem subir o Spring e sem rede: `when(repository.findByPetNome("Rex")).thenReturn(List.of(existente))` basta para montar o cenário, e `verify(repository, never()).save(any())` prova que nada foi gravado quando há conflito. Isso isola a regra de negócio do `AgendaService` de falhas de infraestrutura.

### 3. `==` vs `.equals()` (Aula 7)
Em `AgendaService.agendar` o código comparava `a.getPetNome() == novo.getPetNome() && a.getDataHora() == novo.getDataHora()`. O operador `==` compara referências (se é o mesmo objeto na memória), não conteúdo. O teste cria a data da nova tentativa com `LocalDateTime.parse(existente.getDataHora().toString())`: valor idêntico, objeto diferente, então `==` dava `false` e o duplicado passava. Com `String` a coisa "funciona por sorte" para literais como `"Rex"`: o compilador os guarda no *string pool*, e dois literais iguais apontam para o mesmo objeto. Mas um nome vindo do `@RequestParam` de uma requisição HTTP ou lido do banco é uma `String` nova, e aí o `==` falha. A correção usa `Objects.equals(a, b)`, que compara o valor e ainda tolera `null` sem `NullPointerException`.

### 4. Sobrescrita vs sobrecarga (Aula 7)
`Atendimento` define `getDuracaoMinutos()` (sem parâmetros, retorna 30). Em `Tosa` alguém escreveu `getDuracaoMinutos(String porte)` retornando 60. Como os parâmetros são diferentes, isso é *overload*: um método novo, que convive com o herdado. Compilou sem erro e nunca era chamado no fluxo real: o controller chama `atendimento.getDuracaoMinutos()` por uma referência `Atendimento`, o que resolve para a versão da superclasse e devolve 30 em vez de 60 para a tosa. Override exige a mesma assinatura. Com `@Override` na versão errada, o compilador teria dito "method does not override or implement a method from a supertype" na hora, e o bug morreria no Eclipse. Na correção usamos exatamente `@Override public int getDuracaoMinutos()`.

### 5. Singleton manual vs bean do Spring (Aula 14)
O `GeradorProtocolo` deve garantir uma única instância na aplicação, com contador global e sequencial. O bug: em `getInstancia()`, o `if (instancia == null)` fazia `return new GeradorProtocolo()` sem atribuir o resultado ao campo estático, então `instancia` continuava `null` para sempre, e cada chamada criava um objeto novo com `contador = 0`; todo atendimento receberia o protocolo 1. Corrigimos guardando a instância; e, num segundo achado, o comentário dizia "thread-safe" mas `getInstancia()` e `proximo()` (`contador++`) não eram, então os tornamos `synchronized`. O `AgendaService`, anotado com `@Service`, não corre esse risco porque nós não escrevemos o padrão: o container do Spring cria um único bean por contexto (escopo *singleton* é o padrão) e injeta a mesma instância em quem precisa, sem `static` nem `new` manual. Além disso o service não guarda estado mutável, só usa o repository.

### 6. Cobertura de testes: onde parar? (Aula 15)
Dos 6 testes novos, 4 ficaram vermelhos (preço do banho, duração da tosa, cancelar concluído, data no passado) e 2 verdes (preço fixo da consulta, horário com atendimento cancelado). Vale manter os verdes: eles documentam uma regra do contrato e protegem contra regressão; o `teste06`, por exemplo, impediria que alguém "simplificasse" o loop do `agendar` e removesse a checagem de `STATUS_AGENDADO`, passando a bloquear horários de atendimentos cancelados. Custam milissegundos. Num projeto real com prazo apertado, priorizaríamos primeiro o caminho feliz de cada fluxo (agendar, concluir, cancelar) e logo em seguida os caminhos de erro das regras de negócio: os 4 bugs escondidos estavam todos em regras que ninguém testava (preço, duração, status, data). Perseguir 100% de cobertura de linhas tem retorno decrescente (getters/setters, por exemplo); cobertura é meio, não fim. O critério é risco: onde um erro custaria dinheiro ou confiança do tutor, o teste vem primeiro.

---

## Parte 5 — Espaço livre (opcional)

```

```
