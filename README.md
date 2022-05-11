# Desenvolvimento com Spring Batch — Overview

![](https://miro.medium.com/max/1400/1*6UFN_0jjWwjthVDcUKo4uw.jpeg)

Imagine um banco que precisa processar diversas transações financeiras todos os dias. Esse processamento precisa ser rápido e confiável, já que tratam-se de dados críticos. Além disso, ele deve ser feito periodicamente e sem interrupção. Esse tipo de aplicação pode ser implementada como um job (tarefa):

Um job é uma aplicação que processa uma quantidade finita de dados sem interação ou interrupção.

Implementar jobs pode ser algo custoso. Algumas tarefas são simples, outras mais complexas. Algumas são rápidas, outras demoradas. É importante considerar diversos requisitos para garantir uma implementação adequada. Para lidar com essas questões, foi criado o framework [Spring Batch](https://spring.io/projects/spring-batch), que hoje em dia é um dos mais usados na implementação de jobs. Tendo isso em mente, elaborei esse post para descrever como ele funciona e suas principais features.

Esse post é baseado principalmente [neste livro](https://www.amazon.com.br/Definitive-Guide-Spring-Batch-Processing-ebook/dp/B07V57NTT1/ref=asc_df_B07V57NTT1/?tag=googleshopp00-20&linkCode=df0&hvadid=379725685153&hvpos=1o1&hvnetw=g&hvrand=12482602994081499700&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1001655&hvtargid=pla-812268305883&psc=1), que contém as orientações mais atuais em 2019 sobre o uso do Spring Batch.

##Arquitetura Spring Batch

O Spring Batch veio para facilitar o processo de criação de jobs. Um job no Spring Batch é basicamente uma máquina de estados com sequência de etapas (steps) que possuem uma lógica própria.

![](https://miro.medium.com/max/1400/1*bYMHRTsH7cMGv9GMXEXWGQ.png)

Os componentes que compõe essa máquina de estados são exibidos na imagem abaixo:

![](https://miro.medium.com/max/1400/1*IotPHxVJNyYea9t34G0wzg.png)

* Job Repository: Mantém o estado do job (duração da execução, status da execução, erros, escritas, leituras, …), que é compartilhado com os outros componentes da solução.
* Step: Representa uma etapa ou passo na qual uma lógica é executada. Steps são encadeados para obterem o produto final do processamento. Se o step for baseado em [chunk](https://docs.spring.io/spring-batch/docs/current/reference/html/step.html#chunkOrientedProcessing) (pedaços), ele terá etapas de leitura (ItemReader), processamento (ItemProcessor) e escrita de dados (ItemWriter). O step pode ser também uma tarefa simples, e nesse caso seria baseado em [tasklets](https://docs.spring.io/spring-batch/docs/current/reference/html/step.html#taskletStep).
* Job Launcher: Executa o job de fato, considerando fatores como a forma de execução (única thread, distribuído), validação de parâmetros, restart, e outras propriedades da execução.

## Principais features

Para atender a diversos cenários, o Spring Batch conta com uma série de features que permitem a elaboração de uma solução que atenda ao máximo os seus requisitos funcionais e não funcionais. Vamos listar algumas delas:

* Leitura de banco de dados: Esse é uma funcionalidade essencial. Muitos jobs fazem leitura e escrita em banco de dados, por isso o Spring Batch já provê [componentes](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#database) de acesso ao banco de forma diferenciada (paginada, em lote, transacional, …).
* Manipulação de arquivos: Assim como em banco de dados, a leitura e escrita em arquivos é algo muito comum em jobs. Pensando nisso, o Spring Batch disponibiliza diferentes [manipuladores de arquivo](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#flatFiles), que podem ser escolhidos de acordo com a natureza dos dados a serem lidos.
* Tratamento de exceções: É importante que um job se recupere de falhas sem comprometer o processamento. Para isso, existem [mecanismos de retry e manipuladores de exceção](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#faultTolerant) para que o job encerre mantendo o seu estado consistente para posterior reinicialização.
* Restart: Essa capacidade é essencial para batches demorados, que podem ter sua execução interrompida. Seria muito custoso começar o processamento do zero, por isso o Spring Batch possui um [mecanismo de reinicialização](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#restartability), que utiliza os [metadados do job](https://docs.spring.io/spring-batch/docs/current/reference/html/schema-appendix.html#metaDataSchema) salvos na sua última execução para retomá-la.
* Paralelismo: Existem [opções](https://docs.spring.io/spring-batch/docs/current/reference/html/scalability.html#scalability) que permitem escalar a aplicação horizontalmente (chunking e particionamento remoto) e verticalmente (steps paralelos e multithread). Isso é importante para otimizar o tempo de processamento quando a abordagem de única thread não é suficiente.

## Começando

Para começar a desenvolver com o Spring Batch é simples. Primeiramente você precisa escolher o ambiente de desenvolvimento. Existe uma plataforma específica do Spring chamada [Spring Tool Suite](https://spring.io/tools) (STS), que é baseada no [Eclipse](https://medium.com/@giu.drawer/eclipse-para-desenvolvimento-java-b589667f3f7a). Ela é voltada para a utilização Spring, o que facilita o desenvolvimento de aplicações Spring Batch.

Uma vez escolhida a IDE, precisamos criar o projeto. Para isso o Spring também oferece uma facilidade de criação de projetos, o [Spring Initializr](https://start.spring.io/). Nele você pode selecionar as dependências desejadas e baixar o projeto já configurado. No STS, podemos utilizá-lo diretamente através do Wizard no menu File ➤ New ➤ Spring Starter Project:

![](https://miro.medium.com/max/1400/1*FLM9t6muk0_ht76BjVeFOw.png)

Nas próxima tela você poderá selecionar as dependências do projeto. No nosso exemplo iremos usar apenas a Batch Spring Boot starter. Pronto, o seu projeto será criado no STS.

## Hello World

Para entendermos como funciona o Spring Batch, vamos implementar o clássico Hello World. Para isso, utilizaremos o projeto criado anteriormente adicionando as seguintes dependências do Maven:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
```

A primeira dependência é a que permite utilizar o framework Spring Batch, já as duas últimas são necessárias para armazenar os dados de execução do job. Como estamos configurando um projeto para testes, utilizar um banco em memória (h2) é aceitável.

A classe principal é a DemoApplication. Dessa forma, iremos editá-la para adicionar a lógica do nosso batch:

```java
@EnableBatchProcessing
@SpringBootApplication
public class DemoApplication {
	@Autowired
	private JobBuilderFactory jobBuilderFactory;

	@Autowired
	private StepBuilderFactory stepBuilderFactory;

	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}

	@Bean
	public Step step() {
		return stepBuilderFactory.get("step1").tasklet(new Tasklet() {
			@Override
			public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
				System.out.println("Hello, World!");
				return RepeatStatus.FINISHED;
			}
		}).build();
	}

	@Bean
	public Job job() {
		return this.jobBuilderFactory.get("nomeDoJob").start(step()).build();
	}
}
```

A solução é composta pelos seguintes componentes chave:

* @EnableBatchProcessing: Essa anotação permite que o Spring monte toda a estrutura necessária para executar o batch. Todos aqueles componentes que vimos na arquitetura do Spring Batch serão configurados automaticamente, precisaremos apenas alterar o que desejarmos para a nossa aplicação (e.g. Banco de dados).
* jobBuilderFactory e stepBuilderFactory: Injetamos esses componentes para construir de forma fluente o job e seus steps.
* step(): Injeta (@Bean) e configura os steps do job. No nosso exemplo, é criada uma simples tasklet que imprime o Hello World.
* job(): Esse método é injetado com o @Bean para retornar o job que será construído a partir dos steps configurados.

O job está criado! Já é possível executá-lo no STS (atalho F11).

## Conclusão

Esse artigo apresentou o Spring Batch como solução para o desenvolvimento de jobs. Foi discutida a arquitetura dos componentes dessa solução e o seu uso foi ilustrado com uma aplicação de exemplo.

Se você deseja aprender mais sobre Spring Batch, recomendo [esse curso](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/?referralCode=8743E206FA9240686B20), que aborda os principais conceitos do framework de forma prática e divertida. Caso você já domine os conceitos e deseje aprender mais sobre otimização de rotinas batch, tem [esse curso aqui](https://www.udemy.com/course/otimizacao-de-desempenho-para-jobs-spring-batch/?referralCode=EE9FA1CDD3AF90B0198D) no mesmo formato.
