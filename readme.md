# Mocks

Qual a vantagem de se utilizar mocks ao escrever testes de unidade?
- Substituir dependências externas da classe.


Mocks possuem o objetivo de simular comportamentos das dependências de uma classe, para que os testes de unidade não se tornem testes de integração.

__Quais as maneiras de adicionar o Mockito à uma aplicação?__

- Baixando e adicionando seus jar’s na 
aplicação
     - Podemos baixar os jar’s do Mockito e adicioná-los ao build path da aplicação.

- Declarando-o como dependência da aplicação

    - É possível declarar o Mockito como dependência da aplicação, utilizando o Maven, Gradle ou outra ferramenta de build/gestão de dependências.

### Primeiro teste utilizando o Mockito

```java
package leilao;

import java.util.List;

import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import br.com.alura.leilao.dao.LeilaoDao;
import br.com.alura.leilao.model.Leilao;
import junit.framework.Assert;

public class HelloWorldMockito {
	
	@Test
	void hello() {
		//Cria um mock que vai simular minha classe LeilaoDao
		//Fingir ser minha classe LeilaoDao
		LeilaoDao mock = Mockito.mock(LeilaoDao.class);
		List<Leilao> todos = mock.buscarTodos();
		Assert.assertTrue(todos.isEmpty());
		
		
	}

}

```

__Como podemos pedir ao Mockito para que crie um mock de uma determinada classe?__

> Mockito.mock(Classe.class)

### Escrevendo teste com Mockito

```java
package br.com.alura.leilao.service;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import br.com.alura.leilao.dao.LeilaoDao;
import br.com.alura.leilao.model.Lance;
import br.com.alura.leilao.model.Leilao;
import br.com.alura.leilao.model.Usuario;

class FinalizarLeilaoServiceTest {

	private FinalizarLeilaoService service;

	@Mock // outra forma de criar um mock
	private LeilaoDao leilaoDao;

	@BeforeEach
	public void beforeEach() {
		// Ler as anotações do mockito
		MockitoAnnotations.initMocks(this);
		this.service = new FinalizarLeilaoService(leilaoDao);
	}

	@Test
	void deveriaFinalizarUmLeilao() {
		List<Leilao> leiloes = leiloes();
		
		service.finalizarLeiloesExpirados();
	}

	// Trecho de código omitido

	private List<Leilao> leiloes() {
        List<Leilao> lista = new ArrayList<>();

        Leilao leilao = new Leilao("Celular",
                        new BigDecimal("500"),
                        new Usuario("Fulano"));

        Lance primeiro = new Lance(new Usuario("Beltrano"),
                        new BigDecimal("600"));
        Lance segundo = new Lance(new Usuario("Ciclano"),
                        new BigDecimal("900"));

        leilao.propoe(primeiro);
        leilao.propoe(segundo);

        lista.add(leilao);

        return lista;

    }
}


```

### Anotação @Mock

__Para que serve essa anotação?__

- Para indicar ao Mockito quais atributos são mocks

>Alternativa correta! Essa anotação substitui a chamada ao método Mockito.mock(Classe.class).

### Modificando os comportamento dos mocks

```java
@Test
void deveriaFinalizarUmLeilao() {
		List<Leilao> leiloes = leiloes();
		
		//when --> serve para manipular o mock
		//simular/manipular um comportamenteo de um determinado método mock
		Mockito.when(leilaoDao.buscarLeiloesExpirados())
		.thenReturn(leiloes);
		
		service.finalizarLeiloesExpirados();
		
		Leilao leilao = leiloes.get(0);
		Assert.assertTrue(leilao.isFechado());
		Assert.assertEquals(new BigDecimal("900"),
				leilao.getLanceVencedor().getValor());
		
		Mockito.verify(leilaoDao).salvar(leilao);
	}

```
Aprendemos a utilizar os métodos when e thenReturn para alterar o comportamento de um método do mock.

Se não utilizarmos tais métodos, o que acontece ao chamar algum método do mock?

- Será devolvido um valor padrão

> __Por padrão, os métodos de um mock sempre devolvem um valor padrão, como 0, false, null, etc.__

### Resumo

- Como passar um mock para uma classe via construtor;
- Como utilizar a anotação @Mock para marcar um atributo como sendo um mock;
- Como utilizar os métodos when e thenReturn para alterar o retorno padrão de um método no mock;
- Como utilizar o método verify para checar se o mock teve um determinado método chamado.

### Verificando comportamentos en casos de exceptions


> Mockito.any() --> qualquer um

**Lançar exception depois que o leilao for salvo**

> Mockito.when(leilaoDao.salvar(Mockito.any())).thenThrow(null)

###  Lidando com exceptions

#### Qual a maneira correta de configurar um mock para lançar uma ```exception```?

> Mockito.when(mock.metodo()).thenThrow(Exception.class)

**OBS!  O método ```thenThrow``` forca o mock a lançar uma ```exception``` quando determinado método for chamado**

```java
@Test
void naoDeveriaEnviarEmailParaVencedorDoLeilaoEmCasoDeErroAoEncerrar() {
		List<Leilao> leiloes = leiloes();
		
		//when --> serve para manipular o mock
		//simular/manipular um comportamenteo de um determinado método mock
		Mockito.when(leilaoDao.buscarLeiloesExpirados())
		.thenReturn(leiloes);
		
		Mockito.when(leilaoDao.salvar(Mockito.any()))
			.thenThrow(RuntimeException.class);
		
		try {
			
			service.finalizarLeiloesExpirados();
//			Nao pode ter interacao com o enviador de emails
			Mockito.verifyNoInteractions(enviadorDeEmails);
			
		} catch (Exception e) {}
		
	}

```

### Resumo: Lidando com exceptions

- A como lidar com exceptions ao utilizar mocks;
- A utilizar o método thenThrow para configurar um mock para lançar uma exception.

## Outras situações

### Capturando objetos com Mockito


>**Para capturar o seguinte objeto é preciso colocar uma annotation chamada ```@Captor``` e criar um atributo com nome da classe ```ArgumentCaptor<Object>``` e dentra de classe a gente o nome de nossa classe o objeto a ser capturado**

**EX:**

```java
	@Captor
	private ArgumentCaptor<Pagamento> captor;
```


**Argument Captor**

> Para capturar um parâmetro passado para um método do mock
> 
> **OBS!  O recurso de ```Argument``` ```Captor``` nos ajuda a capturar um objeto criado internamente na classe sendo testada, quando ele é passado como parâmetro para um método do mock.**

```java
@Test
void deveriaCriarPagamentoParaVencedorDoLeilao() {
		Leilao leilao = leilao();
		
		Lance vencedor = leilao.getLanceVencedor();
		geradorDePagamento.gerarPagamento(vencedor);
//		capturar o meu objeto
		Mockito.verify(pagamentoDao).salvar(captor.capture());
		
//		pegar o valor do meu objeto
		Pagamento pagamento = captor.getValue();
		
		Assert.assertEquals(LocalDate.now().plusDays(1),
							pagamento.getVencimento());
		
		Assert.assertEquals(vencedor.getValor(), pagamento.getValor());
		Assert.assertFalse(pagamento.getPago());
		Assert.assertEquals(vencedor.getUsuario(), pagamento.getUsuario());
		Assert.assertEquals(leilao, pagamento.getLeilao());
	}

```

### Lidando com métodos estáticos

**Mudando a classe GeradoeDEPagamentos**

```Java
package br.com.alura.leilao.service;

import java.time.Clock;
import java.time.DayOfWeek;
import java.time.LocalDate;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import br.com.alura.leilao.dao.PagamentoDao;
import br.com.alura.leilao.model.Lance;
import br.com.alura.leilao.model.Pagamento;

@Service
public class GeradorDePagamento {

	private PagamentoDao pagamentos;
	
	private Clock clock;

	@Autowired
	public GeradorDePagamento(PagamentoDao pagamentos, Clock clock) {
		this.pagamentos = pagamentos;
		this.clock = clock;
	}

	// Trecho de código suprimido

	public void gerarPagamento(Lance lanceVencedor) {
		LocalDate vencimento = LocalDate.now(clock).plusDays(1);
		Pagamento pagamento = new Pagamento(lanceVencedor, proximoDiaUtil(vencimento));
		this.pagamentos.salvar(pagamento);
	}

	private LocalDate proximoDiaUtil(LocalDate dataBase) {
		DayOfWeek diaDaSemana = dataBase.getDayOfWeek();
		if (diaDaSemana == DayOfWeek.SATURDAY) {
			return dataBase.plusDays(2);
		} else if (diaDaSemana == DayOfWeek.SUNDAY) {
			return dataBase.plusDays(1);
		}

		return dataBase;
	}

}

```

***Sobre a classe Clock***

- Um Clock fornece acesso ao instante atual, data e hora usando um fuso horário.
- As instâncias desta classe são usadas para encontrar o instante atual, que pode ser interpretado usando o fuso horário armazenado para encontrar a data e hora atuais. Como tal, um relógio pode ser usado em vez de System.currentTimeMillis() e TimeZone.getDefault().

**Por que não é considerada uma boa prática fazer chamadas a métodos estáticos, do ponto de vista de testes automatizados?**

> Porque métodos estáticos dificultam a utilização de mocks
> 
> **OBS! Métodos estáticos dificultam a utilização de mocks, quando precisamos simular comportamentos.**

## Resumo: Outras situações

- A como utilizar o recurso de Argument Captor do Mockito;
- Os problemas de se fazer chamadas a métodos estáticos quando precisamos escrever testes automatizados com a utilização de mocks;
- A como pensar em abstrações para substituir chamadas a métodos estáticos.

#### Teste GeradorDePagamento

```Java
package br.com.alura.leilao.service;

import java.math.BigDecimal;
import java.time.Clock;
import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;

import org.junit.Assert;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

import br.com.alura.leilao.dao.PagamentoDao;
import br.com.alura.leilao.model.Lance;
import br.com.alura.leilao.model.Leilao;
import br.com.alura.leilao.model.Pagamento;
import br.com.alura.leilao.model.Usuario;

class GeradorDePagamentoTest {
	
	private GeradorDePagamento geradorDePagamento;
	
	@Mock
	private PagamentoDao pagamentoDao;
	
	@Captor
	private ArgumentCaptor<Pagamento> captor;
	
	@Mock
	private Clock clock;

	@BeforeEach
	void test() {
		MockitoAnnotations.initMocks(this);
		this.geradorDePagamento = new GeradorDePagamento(pagamentoDao, clock);
	}
	
	@Test
	void deveriaCriarPagamentoParaVencedorDoLeilao() {
		Leilao leilao = leilao();
		
		Lance vencedor = leilao.getLanceVencedor();
		
		LocalDate data = LocalDate.of(2022, 11 , 6);

		Instant instant = data.atStartOfDay(ZoneId.systemDefault()).toInstant();

		Mockito.when(clock.instant()).thenReturn(instant);
		Mockito.when(clock.getZone()).thenReturn(ZoneId.systemDefault());
		
		geradorDePagamento.gerarPagamento(vencedor);
		
//		capturar o meu objeto
		Mockito.verify(pagamentoDao).salvar(captor.capture());
		
//		pegar o valor do meu objeto
		Pagamento pagamento = captor.getValue();
		
		Assert.assertEquals(LocalDate.now().plusDays(1),
							pagamento.getVencimento());
		
		Assert.assertEquals(vencedor.getValor(), pagamento.getValor());
		Assert.assertFalse(pagamento.getPago());
		Assert.assertEquals(vencedor.getUsuario(), pagamento.getUsuario());
		Assert.assertEquals(leilao, pagamento.getLeilao());
	}
	
	private Leilao leilao() {

        Leilao leilao = new Leilao("Celular",
                        new BigDecimal("500"),
                        new Usuario("Fulano"));

        Lance lance = new Lance(new Usuario("Ciclano"),
                        new BigDecimal("900"));

        leilao.propoe(lance);
        leilao.setLanceVencedor(lance);

        return leilao;

    }

}
```